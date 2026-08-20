# Database Layer Management Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. What "Database Layer Management" Covers

The database layer is the code between your application logic and the actual data store — it decides how you model data, run queries, evolve schemas, and keep things fast and safe under load.

> **Note on Angular and React:** these are client-side (browser) frameworks — they never talk to a database directly. Sections 3 and 4 cover the database layer of the **Node.js backend** that typically accompanies an Angular or React frontend (e.g., NestJS or Express/Next.js API routes), since that's where "database layer management" actually happens in those stacks.

| Concern | What it solves |
|---|---|
| ORM / query builder | Maps objects/code to rows and tables (or skips that, for raw SQL control) |
| Connection management | Pooling, reuse, and lifecycle of DB connections |
| Migrations | Versioned, repeatable schema changes |
| Transactions | Atomic multi-step operations |
| Repository pattern | Abstracting data access behind an interface |
| N+1 / query optimization | Avoiding excessive round-trips |
| Caching | Reducing repeated reads from the database |
| Gen AI equivalent | Storing/retrieving embeddings in a vector database for RAG and memory |

---

## 2. Blazor (C# / .NET)

.NET's data layer is built around **Entity Framework Core** (full ORM) or **Dapper** (lightweight micro-ORM) — both work identically whether the API is called from Blazor Server or a Blazor WASM client via a Web API backend.

### 2.1 Defining the model and DbContext (EF Core)
```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>().HasIndex(p => p.Name);
    }
}
```

### 2.2 Registering the DbContext with connection pooling
```csharp
// Program.cs
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

### 2.3 Basic queries
```csharp
public class ProductService(AppDbContext db)
{
    public Task<List<Product>> GetAllAsync() => db.Products.AsNoTracking().ToListAsync();

    public Task<Product?> GetByIdAsync(int id) => db.Products.FindAsync(id).AsTask();

    public async Task<Product> CreateAsync(Product product)
    {
        db.Products.Add(product);
        await db.SaveChangesAsync();
        return product;
    }
}
```

### 2.4 Transactions
```csharp
using var transaction = await db.Database.BeginTransactionAsync();
try
{
    db.Products.Add(new Product { Name = "Widget", Price = 9.99m });
    await db.SaveChangesAsync();

    db.Orders.Add(new Order { ProductId = 1, Quantity = 5 });
    await db.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### 2.5 Migrations
```bash
dotnet ef migrations add AddProductsTable
dotnet ef database update
```

### 2.6 Avoiding N+1 — eager loading
```csharp
var orders = await db.Orders
    .Include(o => o.Product)   // JOIN instead of per-row queries
    .ThenInclude(p => p.Category)
    .ToListAsync();
```

### 2.7 Repository pattern (abstraction layer)
```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<List<Product>> GetAllAsync();
    Task AddAsync(Product product);
}

public class ProductRepository(AppDbContext db) : IProductRepository
{
    public Task<Product?> GetByIdAsync(int id) => db.Products.FindAsync(id).AsTask();
    public Task<List<Product>> GetAllAsync() => db.Products.ToListAsync();
    public async Task AddAsync(Product product) { db.Products.Add(product); await db.SaveChangesAsync(); }
}
```

### 2.8 Lightweight alternative — Dapper (raw SQL, high performance)
```csharp
using var connection = new SqlConnection(connectionString);
var products = await connection.QueryAsync<Product>(
    "SELECT Id, Name, Price FROM Products WHERE Price > @MinPrice",
    new { MinPrice = 10 });
```

### 2.9 Recommendation summary
| Need | Recommended approach |
|---|---|
| Full ORM, change tracking, migrations | Entity Framework Core |
| Maximum raw-SQL performance | Dapper |
| Connection reuse under load | `AddDbContextPool<T>` |
| Schema evolution | EF Core Migrations (`dotnet ef migrations`) |
| Abstracting data access | Repository interfaces over `DbContext` |
| Avoiding N+1 queries | `.Include()` / `.ThenInclude()` (eager loading) |

---

## 3. Angular's Backend (Node.js / NestJS)

Angular apps typically call a Node.js backend (often NestJS) for data access — TypeORM and Prisma are the two dominant choices there.

### 3.1 Defining an entity (TypeORM)
```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column("decimal")
  price: number;
}
```

### 3.2 NestJS module wiring with connection pooling
```typescript
TypeOrmModule.forRoot({
  type: "postgres",
  url: process.env.DATABASE_URL,
  entities: [Product],
  extra: { max: 20 }, // connection pool size
});
```

### 3.3 Repository-based queries (NestJS service)
```typescript
@Injectable()
export class ProductsService {
  constructor(@InjectRepository(Product) private repo: Repository<Product>) {}

  findAll() { return this.repo.find(); }
  findOne(id: number) { return this.repo.findOneBy({ id }); }
  create(data: Partial<Product>) { return this.repo.save(this.repo.create(data)); }
}
```

### 3.4 Transactions (TypeORM)
```typescript
await this.dataSource.transaction(async (manager) => {
  const product = await manager.save(Product, { name: "Widget", price: 9.99 });
  await manager.save(Order, { productId: product.id, quantity: 5 });
});
```

### 3.5 Migrations
```bash
npx typeorm migration:generate src/migrations/AddProductsTable -d src/data-source.ts
npx typeorm migration:run -d src/data-source.ts
```

### 3.6 Alternative — Prisma (type-safe query builder)
```prisma
// schema.prisma
model Product {
  id    Int    @id @default(autoincrement())
  name  String
  price Decimal
}
```
```typescript
const products = await prisma.product.findMany({ where: { price: { gt: 10 } } });
```

### 3.7 Avoiding N+1 — relations in one query
```typescript
// TypeORM
const orders = await this.orderRepo.find({ relations: ["product", "product.category"] });

// Prisma
const orders = await prisma.order.findMany({ include: { product: { include: { category: true } } } });
```

### 3.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Decorator-based ORM, NestJS-native | TypeORM |
| Best-in-class type safety, migrations UX | Prisma |
| Connection pooling | Configure pool size in the driver/ORM connection options |
| Schema evolution | `typeorm migration:generate` / `prisma migrate dev` |
| Avoiding N+1 | `relations` (TypeORM) / `include` (Prisma) |

---

## 4. React's Backend (Node.js / Express / Next.js API Routes)

React apps similarly rely on a Node.js backend for data access. **Prisma** and the newer, lighter **Drizzle ORM** are the most common choices in this ecosystem today.

### 4.1 Prisma schema + client
```prisma
// schema.prisma
model Product {
  id    Int     @id @default(autoincrement())
  name  String
  price Decimal
}
```
```typescript
// lib/prisma.ts — singleton to avoid exhausting connections in dev/serverless
import { PrismaClient } from "@prisma/client";
export const prisma = globalThis.prisma ?? new PrismaClient();
if (process.env.NODE_ENV !== "production") globalThis.prisma = prisma;
```

### 4.2 Basic queries (Next.js API route / server action)
```typescript
export async function GET() {
  const products = await prisma.product.findMany();
  return Response.json(products);
}

export async function POST(request: Request) {
  const data = await request.json();
  const product = await prisma.product.create({ data });
  return Response.json(product, { status: 201 });
}
```

### 4.3 Transactions
```typescript
await prisma.$transaction(async (tx) => {
  const product = await tx.product.create({ data: { name: "Widget", price: 9.99 } });
  await tx.order.create({ data: { productId: product.id, quantity: 5 } });
});
```

### 4.4 Alternative — Drizzle ORM (lightweight, SQL-like)
```typescript
export const products = pgTable("products", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  price: numeric("price").notNull(),
});
```
```typescript
const db = drizzle(pool);
const results = await db.select().from(products).where(gt(products.price, 10));
```

### 4.5 Migrations
```bash
# Prisma
npx prisma migrate dev --name add_products_table

# Drizzle
npx drizzle-kit generate
npx drizzle-kit migrate
```

### 4.6 Avoiding N+1
```typescript
// Prisma
const orders = await prisma.order.findMany({ include: { product: true } });

// Drizzle
const results = await db.select().from(orders).leftJoin(products, eq(orders.productId, products.id));
```

### 4.7 Recommendation summary
| Need | Recommended approach |
|---|---|
| Rich type safety, mature ecosystem | Prisma |
| Minimal, SQL-close, no codegen step | Drizzle ORM |
| Serverless/edge connection reuse | Singleton client instance + connection pooler (e.g., PgBouncer, Neon/Supabase pooling) |
| Schema evolution | `prisma migrate` / `drizzle-kit` |
| Avoiding N+1 | `include` (Prisma) / explicit joins (Drizzle) |

---

## 5. Python Web Apps

Python has two dominant data-access layers: **SQLAlchemy** (framework-agnostic, used directly or via FastAPI) and the **Django ORM** (built into Django).

### 5.1 Defining models — SQLAlchemy 2.x (typed declarative style)
```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class Product(Base):
    __tablename__ = "products"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    price: Mapped[float]
```

### 5.2 Engine and connection pooling
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine(
    "postgresql+psycopg://user:pass@localhost/db",
    pool_size=10,
    max_overflow=20,
)
SessionLocal = sessionmaker(bind=engine)
```

### 5.3 Basic queries (sync) / FastAPI dependency (async)
```python
from sqlalchemy import select

def get_products(session):
    return session.scalars(select(Product).where(Product.price > 10)).all()
```
```python
# FastAPI — async session as a dependency
async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

@app.get("/products")
async def list_products(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Product))
    return result.scalars().all()
```

### 5.4 Transactions
```python
with SessionLocal() as session:
    with session.begin():
        product = Product(name="Widget", price=9.99)
        session.add(product)
        session.flush()  # get product.id before commit
        session.add(Order(product_id=product.id, quantity=5))
    # commits automatically on successful exit, rolls back on exception
```

### 5.5 Migrations — Alembic
```bash
alembic revision --autogenerate -m "add products table"
alembic upgrade head
```

### 5.6 Avoiding N+1 — eager loading
```python
from sqlalchemy.orm import selectinload

orders = session.scalars(
    select(Order).options(selectinload(Order.product).selectinload(Product.category))
).all()
```

### 5.7 Django ORM — models, queries, migrations
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
```
```python
Product.objects.filter(price__gt=10)
Product.objects.select_related("category")       # avoids N+1 for FK
Order.objects.prefetch_related("items")           # avoids N+1 for reverse FK/M2M
```
```bash
python manage.py makemigrations
python manage.py migrate
```

### 5.8 Django transactions
```python
from django.db import transaction

with transaction.atomic():
    product = Product.objects.create(name="Widget", price=9.99)
    Order.objects.create(product=product, quantity=5)
```

### 5.9 Repository pattern (framework-agnostic layer)
```python
class ProductRepository:
    def __init__(self, session):
        self.session = session

    def get_by_id(self, product_id: int) -> Product | None:
        return self.session.get(Product, product_id)

    def add(self, product: Product) -> None:
        self.session.add(product)
        self.session.commit()
```

### 5.10 Recommendation summary
| Need | Recommended approach |
|---|---|
| Framework-agnostic ORM (FastAPI, Flask) | SQLAlchemy 2.x (typed declarative models) |
| Built-in ORM for a Django app | Django ORM |
| Connection pooling | `pool_size`/`max_overflow` (SQLAlchemy) or a pooler like PgBouncer |
| Schema evolution | Alembic (SQLAlchemy) / `makemigrations` (Django) |
| Avoiding N+1 | `selectinload`/`joinedload` (SQLAlchemy) / `select_related`/`prefetch_related` (Django) |
| Abstracting data access | Repository classes wrapping the session/queryset |

---

## 6. Database Layer in Gen AI Applications

Gen AI apps need a data layer for two distinct needs: **relational/document storage** for conventional app data (users, conversations, billing), and **vector storage** for embeddings used in RAG and long-term memory.

### 6.1 Storing conversation/session data (conventional DB)
Chat history, user accounts, and usage metering use the exact same relational/document patterns as Sections 2–5 — nothing Gen AI–specific here.

```python
class Conversation(Base):
    __tablename__ = "conversations"
    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int]
    messages: Mapped[list["Message"]] = relationship(back_populates="conversation")
```

### 6.2 Embedding text for storage
```python
def embed_text(text: str) -> list[float]:
    response = client.embeddings.create(model="text-embedding-3-small", input=text)
    return response.data[0].embedding
```

### 6.3 Vector database — dedicated service (Pinecone)
```python
from pinecone import Pinecone

pc = Pinecone(api_key=os.environ["PINECONE_API_KEY"])
index = pc.Index("documents")

index.upsert(vectors=[
    {"id": "doc-1", "values": embed_text("..."), "metadata": {"user_id": "u123"}}
])

results = index.query(vector=embed_text("search query"), top_k=5, filter={"user_id": "u123"})
```

### 6.4 Vector database — self-hosted/embedded (Chroma)
```python
import chromadb

chroma_client = chromadb.PersistentClient(path="./chroma_data")
collection = chroma_client.get_or_create_collection("documents")

collection.add(ids=["doc-1"], embeddings=[embed_text("...")], metadatas=[{"user_id": "u123"}])
results = collection.query(query_embeddings=[embed_text("search query")], n_results=5)
```

### 6.5 Vector search inside your existing relational database — `pgvector`
Keeps embeddings alongside relational data instead of standing up a separate vector store — often the simplest choice if you already run PostgreSQL.

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE document_chunks (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536),
    user_id INTEGER
);

CREATE INDEX ON document_chunks USING ivfflat (embedding vector_cosine_ops);
```
```python
# SQLAlchemy + pgvector
results = session.scalars(
    select(DocumentChunk)
    .where(DocumentChunk.user_id == user_id)
    .order_by(DocumentChunk.embedding.cosine_distance(query_embedding))
    .limit(5)
)
```

### 6.6 Combining relational + vector data in a RAG query
```python
def retrieve_context(query: str, user_id: int) -> list[str]:
    query_embedding = embed_text(query)
    chunks = session.scalars(
        select(DocumentChunk)
        .where(DocumentChunk.user_id == user_id)   # authorization scoping — see Auth guide §6.3
        .order_by(DocumentChunk.embedding.cosine_distance(query_embedding))
        .limit(5)
    ).all()
    return [c.content for c in chunks]
```

### 6.7 Migrations for vector schemas
Vector columns/indexes are just another column type in most relational tools — Alembic and Django migrations handle `pgvector` columns the same way as any other column.

```python
# Alembic migration
def upgrade():
    op.add_column("document_chunks", sa.Column("embedding", Vector(1536)))
    op.execute("CREATE INDEX ON document_chunks USING ivfflat (embedding vector_cosine_ops)")
```

### 6.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| App data (users, conversations, billing) | Standard relational/document DB layer (Sections 2–5) |
| Large-scale, managed vector search | Pinecone (or Qdrant, Weaviate) |
| Self-hosted/embedded vector store, simple setup | Chroma |
| Keep vectors alongside relational data | PostgreSQL + `pgvector` |
| Scoping retrieval to the right user | Metadata filters at query time (never rely on the model to self-restrict) |
| Schema evolution for vector columns | Same migration tool as the rest of the schema (Alembic/Django/etc.) |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular (backend) | React (backend) | Python | Gen AI |
|---|---|---|---|---|---|
| Primary ORM | Entity Framework Core | TypeORM | Prisma | SQLAlchemy / Django ORM | pgvector / dedicated vector DB |
| Lightweight alternative | Dapper | Prisma | Drizzle ORM | raw SQL / `asyncpg` | Chroma (embedded) |
| Connection pooling | `AddDbContextPool` | driver pool options (`max`) | pooler (PgBouncer/Neon) + singleton client | `pool_size`/`max_overflow` | provider-managed (Pinecone) or PG pool |
| Migrations | `dotnet ef migrations` | `typeorm migration:generate` | `prisma migrate` / `drizzle-kit` | Alembic / `makemigrations` | same tool, vector column included |
| Transactions | `BeginTransactionAsync()` | `dataSource.transaction()` | `prisma.$transaction()` | `session.begin()` / `transaction.atomic()` | standard relational transaction (vectors are eventually-consistent in most vector DBs) |
| Avoiding N+1 | `.Include()` | `relations` / `include` | `include` / explicit joins | `selectinload` / `select_related` | pre-filter by metadata before vector search |
| Abstraction layer | Repository interfaces | NestJS repository services | data-access module/service | Repository classes | Retrieval service wrapping embed + query |

---

## 8. Library & Package Reference

Versions verified against npm/NuGet/PyPI in **August 2026**.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `Microsoft.EntityFrameworkCore` | Core ORM | 10.0.0 | `dotnet add package Microsoft.EntityFrameworkCore` |
| `Microsoft.EntityFrameworkCore.SqlServer` | SQL Server provider | tracks EF Core version | `dotnet add package Microsoft.EntityFrameworkCore.SqlServer` |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | PostgreSQL provider | check NuGet | `dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL` |
| `Dapper` | Lightweight micro-ORM | check NuGet | `dotnet add package Dapper` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| EF Core | `AddDbContextPool<T>()` | Registers a `DbContext` with connection/instance pooling. |
| EF Core | `DbSet<T>.AsNoTracking()` | Reads data without change tracking (faster for read-only queries). |
| EF Core | `SaveChangesAsync()` | Persists all pending changes in the current context. |
| EF Core | `Include()` / `ThenInclude()` | Eager-loads related entities to avoid N+1 queries. |
| EF Core | `Database.BeginTransactionAsync()` | Starts an explicit transaction spanning multiple `SaveChanges` calls. |
| Dapper | `connection.QueryAsync<T>(sql, params)` | Executes a SQL query and maps rows to typed objects. |
| Dapper | `connection.ExecuteAsync(sql, params)` | Executes a non-query SQL statement (insert/update/delete). |

### 8.2 Angular's Backend — Node.js/NestJS (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `typeorm` | Decorator-based ORM, NestJS-native | check npm | `npm i typeorm` |
| `@nestjs/typeorm` | NestJS integration module for TypeORM | check npm | `npm i @nestjs/typeorm` |
| `prisma` / `@prisma/client` | Type-safe ORM and query builder | 7.9.1 | `npm i prisma @prisma/client` |
| `pg` | PostgreSQL driver (used under the hood by most ORMs) | check npm | `npm i pg` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| TypeORM | `Repository.find(options)` | Queries rows with filters, relations, pagination. |
| TypeORM | `Repository.save(entity)` | Inserts or updates an entity. |
| TypeORM | `DataSource.transaction(fn)` | Runs a function inside a database transaction. |
| TypeORM | `migration:generate` (CLI) | Diffs entities against the DB and generates a migration file. |
| Prisma | `prisma.model.findMany(query)` | Queries records with type-safe filters/includes. |
| Prisma | `prisma.$transaction(fn)` | Runs multiple operations atomically. |
| Prisma | `prisma migrate dev` (CLI) | Generates and applies a migration in development. |

Check latest: `npm view typeorm version` / `npm view prisma version`

### 8.3 React's Backend — Node.js/Next.js (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `prisma` / `@prisma/client` | Type-safe ORM and query builder | 7.9.1 | `npm i prisma @prisma/client` |
| `drizzle-orm` | Lightweight, SQL-like TypeScript ORM | 0.45.2 | `npm i drizzle-orm` |
| `drizzle-kit` | Migration generator/CLI for Drizzle | tracks `drizzle-orm` releases | `npm i -D drizzle-kit` |
| `pg` | PostgreSQL driver | check npm | `npm i pg` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Prisma | `prisma.model.create({data})` | Inserts a new record. |
| Prisma | `prisma.model.findMany({include})` | Queries with related records eagerly loaded (avoids N+1). |
| Drizzle | `db.select().from(table).where(condition)` | Builds a type-safe, SQL-like SELECT query. |
| Drizzle | `db.insert(table).values(data)` | Inserts a row. |
| Drizzle | `db.transaction(async (tx) => {...})` | Runs multiple statements atomically. |
| Drizzle Kit | `drizzle-kit generate` (CLI) | Generates a SQL migration from schema changes. |

Check latest: `npm view drizzle-orm version`

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `sqlalchemy` | Framework-agnostic ORM/Core toolkit | 2.1.0b1 (beta) / 2.0.x (stable) | `pip install sqlalchemy` |
| `alembic` | Migrations for SQLAlchemy | check PyPI | `pip install alembic` |
| `asyncpg` | High-performance async PostgreSQL driver | check PyPI | `pip install asyncpg` |
| `psycopg` | PostgreSQL driver (sync/async) | check PyPI | `pip install "psycopg[binary]"` |
| `django` (built-in ORM) | Full-stack framework with its own ORM | check PyPI | ships with `django` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| SQLAlchemy | `select(Model).where(condition)` | Builds a typed SELECT query (2.x style). |
| SQLAlchemy | `session.add(obj)` / `session.commit()` | Stages and persists a new/updated object. |
| SQLAlchemy | `selectinload()` / `joinedload()` | Eager-loads relationships to avoid N+1 queries. |
| SQLAlchemy | `session.begin()` | Context manager for an explicit transaction. |
| Alembic | `alembic revision --autogenerate` (CLI) | Generates a migration from model changes. |
| Alembic | `alembic upgrade head` (CLI) | Applies pending migrations. |
| Django ORM | `Model.objects.filter(**kwargs)` | Queries rows matching given conditions. |
| Django ORM | `select_related()` / `prefetch_related()` | Eager-loads FK/M2M relations to avoid N+1 queries. |
| Django ORM | `transaction.atomic()` | Context manager/decorator for an atomic transaction block. |

Check latest: `pip index versions sqlalchemy` / `pip index versions alembic`

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `pgvector` | Vector similarity search extension/types for PostgreSQL | check PyPI | `pip install pgvector` |
| `chromadb` | Embedded/self-hosted vector database | check PyPI | `pip install chromadb` |
| `pinecone` (client) | Managed vector database client | check PyPI | `pip install pinecone` |
| `qdrant-client` | Client for the Qdrant vector database | check PyPI | `pip install qdrant-client` |
| `anthropic` | Model SDK (for generating embeddings-adjacent responses/tool calls) | check PyPI | `pip install anthropic` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `pgvector` | `Vector(dimensions)` column type | Declares an embedding column on a SQLAlchemy/Django model. |
| `pgvector` | `.cosine_distance(vector)` | Orders/filters rows by similarity to a query embedding. |
| `chromadb` | `collection.add(ids, embeddings, metadatas)` | Inserts embeddings with associated metadata. |
| `chromadb` | `collection.query(query_embeddings, n_results)` | Retrieves the most similar stored items. |
| `pinecone` | `index.upsert(vectors)` | Inserts or updates vectors in a managed index. |
| `pinecone` | `index.query(vector, top_k, filter)` | Retrieves nearest vectors, optionally filtered by metadata. |
| `qdrant-client` | `client.upsert(collection_name, points)` | Inserts vector points into a Qdrant collection. |
| `qdrant-client` | `client.search(collection_name, query_vector)` | Performs a similarity search. |

Check latest: `pip index versions chromadb` / `pip index versions pgvector`

---

*This guide covers the mainstream database layer patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI/vector database space) change frequently — check current docs/registries before pinning versions in production. Always scope queries (relational or vector) by the authenticated user's permissions at the data layer — never rely on application logic or model prompting alone to enforce access control.*
