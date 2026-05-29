# 📘 Naming Conventions Reference

| Category          | **Python (PEP 8)**      | **C# (.NET Style)**       | **React JS**                  | **Next.js**                  |
|-------------------|--------------------------|----------------------------|-------------------------------|-------------------------------|
| **Modules/Files** | `snake_case` → `data_utils.py` | `PascalCase` or `camelCase` → `DataUtils.cs` | `PascalCase` for components → `UserCard.jsx` | `PascalCase` for components, `camelCase` for helpers |
| **Classes**       | `PascalCase` → `UserProfile` | `PascalCase` → `UserProfile` | `PascalCase` → `UserProfile` | Same as React (PascalCase) |
| **Functions/Methods** | `snake_case` → `process_data()` | `PascalCase` → `ProcessData()` | `camelCase` → `handleClick()` | `camelCase` → `getServerSideProps()` |
| **Variables**     | `snake_case` → `user_name` | `camelCase` → `userName` | `camelCase` → `userName` | `camelCase` → `userName` |
| **Constants**     | `UPPER_SNAKE_CASE` → `MAX_SIZE` | `PascalCase` → `MaxSize` or `ALL_CAPS` for static readonly | `UPPER_SNAKE_CASE` → `API_URL` | `UPPER_SNAKE_CASE` → `BASE_URL` |
| **Private/Internal** | `_prefix` → `_cache` | `_camelCase` → `_cache` | Convention: prefix with `_` if internal | Same as React |
| **Special Methods** | `__dunder__` → `__init__`, `__next__` | Not applicable (use interfaces/overrides) | Lifecycle methods → `componentDidMount` | Framework methods → `getStaticProps`, `getServerSideProps` |

