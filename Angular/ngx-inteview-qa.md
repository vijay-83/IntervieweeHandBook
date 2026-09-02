# Angular Interview Guide — Fundamentals to Advanced (Angular 18+)

A complete interview prep guide covering Angular from basics to advanced concepts, including the latest Angular (v17/18) features like Signals, standalone components, and the new control-flow syntax.

---

## Table of Contents

1. [Fundamental Level](#fundamental-level)
2. [Intermediate Level](#intermediate-level)
3. [Advanced Level](#advanced-level)
4. [Latest Angular Features (Signals, Standalone, Control Flow)](#latest-angular-features)
5. [Debugging & Error-Handling Level](#debugging--error-handling-level)
6. [Granular / Deep-Dive Advanced Level](#granular--deep-dive-advanced-level)
7. [Architecture Diagrams](#architecture-diagrams)
8. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Fundamental Level

### Q1. What is Angular, and how is it different from AngularJS?

Angular is a TypeScript-based, component-driven front-end framework for building single-page applications (SPAs), maintained by Google. AngularJS (v1.x) was JavaScript-based and used `$scope`/controllers; Angular (v2+) is a full rewrite using components, TypeScript, and a hierarchical dependency injection system.

| Aspect | AngularJS | Angular (2+) |
|---|---|---|
| Language | JavaScript | TypeScript |
| Architecture | MVC | Component-based |
| Mobile support | Limited | Built-in |
| Performance | Slower (digest cycle) | Faster (change detection strategies) |

### Q2. What are the building blocks of an Angular application?

- **Modules (NgModules)** — group related code (legacy; standalone components replace this in newer Angular)
- **Components** — UI + logic
- **Templates** — HTML with Angular syntax
- **Services** — reusable business logic
- **Directives** — DOM behavior extensions
- **Dependency Injection (DI)** — how services are provided/consumed
- **Routing** — navigation between views

```
Diagram: Angular App Building Blocks

        ┌────────────────────────┐
        │     AppComponent        │
        │  (Root Component)       │
        └───────────┬─────────────┘
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
  ┌───────────┐ ┌───────────┐ ┌───────────┐
  │ Component │ │ Component │ │ Component │
  │  (Header) │ │  (Body)   │ │ (Footer)  │
  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘
        │             │             │
        ▼             ▼             ▼
   Services      Directives      Pipes
   (DI'd in)     (structural/    (transform
                  attribute)      data)
```

### Q3. What is a Component in Angular? Give an example.

A component controls a patch of screen (a "view") through a TypeScript class, an HTML template, and optional CSS.

```typescript
// hello.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  standalone: true,
  template: `<h1>Hello, {{ name }}!</h1>`,
  styles: [`h1 { color: teal; }`]
})
export class HelloComponent {
  name = 'Angular';
}
```

### Q4. Explain data binding types in Angular with examples.

| Type | Syntax | Direction |
|---|---|---|
| Interpolation | `{{ value }}` | Component → View |
| Property binding | `[property]="value"` | Component → View |
| Event binding | `(event)="handler()"` | View → Component |
| Two-way binding | `[(ngModel)]="value"` | Both |

```html
<!-- Interpolation -->
<p>{{ username }}</p>

<!-- Property binding -->
<img [src]="imageUrl">

<!-- Event binding -->
<button (click)="onSave()">Save</button>

<!-- Two-way binding -->
<input [(ngModel)]="username">
```

### Q5. What are Directives? Name the three types.

Directives are classes that add behavior to elements.

1. **Component directives** — directives with a template (every `@Component` is technically a directive).
2. **Structural directives** — change DOM layout by adding/removing elements: `*ngIf`, `*ngFor`, `*ngSwitch` (or the new `@if`/`@for` control flow).
3. **Attribute directives** — change appearance/behavior of an element: `ngClass`, `ngStyle`, custom directives.

```typescript
// Custom attribute directive
import { Directive, ElementRef } from '@angular/core';

@Directive({ selector: '[appHighlight]', standalone: true })
export class HighlightDirective {
  constructor(el: ElementRef) {
    el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

### Q6. What is a Service, and how does Dependency Injection work?

A service is a class with a focused purpose (e.g., data fetching) that can be injected into components. Angular's DI container resolves and provides instances automatically.

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  getUsers() { return ['Alice', 'Bob']; }
}

@Component({ selector: 'app-users', standalone: true, template: `...` })
export class UsersComponent {
  constructor(private userService: UserService) {}
  // or, in modern Angular:
  // userService = inject(UserService);
}
```

### Q7. What are Pipes? Give a built-in and a custom example.

Pipes transform data for display.

```html
<p>{{ birthday | date:'longDate' }}</p>
<p>{{ price | currency:'USD' }}</p>
```

```typescript
@Pipe({ name: 'reverse', standalone: true })
export class ReversePipe implements PipeTransform {
  transform(value: string): string {
    return value.split('').reverse().join('');
  }
}
```

### Q8. What is the Angular CLI, and what are common commands?

The CLI scaffolds, builds, tests, and serves Angular apps.

```bash
ng new my-app --standalone
ng generate component my-feature
ng serve
ng build --configuration production
ng test
ng add @angular/material
```

### Q9. What is the difference between `ngOnInit` and a constructor?

The **constructor** is a TypeScript/JS class feature used for dependency injection and simple field initialization. **`ngOnInit`** is an Angular lifecycle hook called once after Angular has initialized all data-bound input properties — the right place for initialization logic that depends on `@Input()` values or services.

```typescript
export class ProfileComponent implements OnInit {
  @Input() userId!: string;

  constructor(private userService: UserService) {} // DI only

  ngOnInit(): void {
    this.userService.load(this.userId); // safe: @Input() is set by now
  }
}
```

---

## Intermediate Level

### Q10. Explain the Angular Component Lifecycle Hooks.

```
Diagram: Component Lifecycle Order

  constructor()
       │
       ▼
  ngOnChanges()  ◄──── (fires again on every @Input change)
       │
       ▼
  ngOnInit()
       │
       ▼
  ngDoCheck()
       │
       ▼
  ngAfterContentInit()
       │
       ▼
  ngAfterContentChecked()
       │
       ▼
  ngAfterViewInit()
       │
       ▼
  ngAfterViewChecked()
       │
       │   (change detection loop continues:
       │    ngDoCheck → ngAfterContentChecked → ngAfterViewChecked)
       ▼
  ngOnDestroy()   ← cleanup (unsubscribe, clear timers)
```

| Hook | Purpose |
|---|---|
| `ngOnChanges` | React to `@Input()` changes |
| `ngOnInit` | One-time init logic |
| `ngDoCheck` | Custom change detection |
| `ngAfterContentInit/Checked` | After content projection (`<ng-content>`) |
| `ngAfterViewInit/Checked` | After view (and child views) render |
| `ngOnDestroy` | Cleanup — unsubscribe observables, clear intervals |

### Q11. What is Angular Routing? Show a basic setup.

Routing maps URL paths to components, enabling SPA navigation without full page reloads.

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'admin', loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES) },
  { path: '**', component: NotFoundComponent }
];
```

```typescript
// main.ts (standalone bootstrap)
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)]
});
```

### Q12. Difference between Template-Driven and Reactive Forms.

| Aspect | Template-Driven | Reactive |
|---|---|---|
| Setup | Directives in template (`ngModel`) | FormGroup/FormControl in class |
| Scalability | Better for simple forms | Better for complex/dynamic forms |
| Testability | Harder | Easier (form logic in TS) |
| Validation | Template-based validators | Explicit `Validators` in code |

```typescript
// Reactive form
form = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', [Validators.required, Validators.email]),
});
```

```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <input formControlName="name">
  <input formControlName="email">
  <button [disabled]="form.invalid">Submit</button>
</form>
```

### Q13. What is RxJS, and how does Angular use Observables?

RxJS is a reactive-programming library for composing asynchronous, event-based code with Observables. Angular uses it for `HttpClient` responses, reactive forms value changes, router events, and the `EventEmitter` behind `@Output()`.

```typescript
this.http.get<User[]>('/api/users')
  .pipe(
    filter(users => users.length > 0),
    map(users => users.map(u => u.name)),
    catchError(err => of([]))
  )
  .subscribe(names => this.names = names);
```

### Q14. Explain `@Input()` and `@Output()` with a parent-child example.

```typescript
// child.component.ts
@Component({ selector: 'app-child', standalone: true, template: `
  <button (click)="notify.emit('clicked!')">Send</button>
`})
export class ChildComponent {
  @Input() label = '';
  @Output() notify = new EventEmitter<string>();
}
```

```html
<!-- parent.component.html -->
<app-child [label]="'Hi'" (notify)="onNotify($event)"></app-child>
```

```
Diagram: Parent ↔ Child Communication

   ParentComponent
       │  [label]="'Hi'"          (Input: data flows down)
       ▼
   ChildComponent
       │  (notify)="onNotify($event)"   (Output: event flows up)
       ▲
   ParentComponent.onNotify()
```

### Q15. What are Angular Modules (NgModules), and are they still required?

An `NgModule` groups components, directives, pipes, and services, declaring what belongs together and what's exported. Since Angular 14+, **standalone components** let you skip NgModules for most apps — Angular 17+ generates standalone components by default. NgModules are still supported for legacy code and some library patterns.

```typescript
// Legacy NgModule style
@NgModule({
  declarations: [AppComponent, HeaderComponent],
  imports: [BrowserModule, FormsModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### Q16. What is Content Projection (`<ng-content>`)?

It lets a parent component pass template content into a child component's view — similar to "slots."

```typescript
// card.component.ts
@Component({ selector: 'app-card', standalone: true, template: `
  <div class="card"><ng-content></ng-content></div>
`})
export class CardComponent {}
```

```html
<app-card>
  <p>This content is projected inside the card.</p>
</app-card>
```

### Q17. What is `HttpClient`, and how do you handle errors?

```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);

  getData(): Observable<Data[]> {
    return this.http.get<Data[]>('/api/data').pipe(
      retry(2),
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse) {
    console.error(error.message);
    return throwError(() => new Error('Something went wrong'));
  }
}
```

---

## Advanced Level

### Q18. Explain Angular's Change Detection mechanism.

Angular's default change detection walks the component tree ("Zone.js patches" async APIs like `setTimeout`, `Promise`, DOM events, and triggers a check) and re-evaluates bindings top-down.

```
Diagram: Change Detection Tree Walk (Default Strategy)

          AppComponent
          /     |      \
     Comp A   Comp B   Comp C
     /   \              |
   A1     A2            C1

Event fires (click, HTTP response, timer)
        │
        ▼
Zone.js detects async task completion
        │
        ▼
Angular triggers change detection from ROOT
        │
        ▼
Every component in the tree is checked top-to-bottom
(unless OnPush skips a subtree)
```

**`ChangeDetectionStrategy.OnPush`** limits checks: a component only re-checks when
- an `@Input()` reference changes,
- an event originates from within the component/its template, or
- a bound `Observable`/`Signal` emits (via `async` pipe or signal read), or
- `ChangeDetectorRef.markForCheck()` is called manually.

```typescript
@Component({
  selector: 'app-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<li *ngFor="let item of items">{{ item }}</li>`
})
export class ListComponent {
  @Input() items: string[] = [];
}
```

### Q19. What is Zone.js, and can Angular run without it?

Zone.js monkey-patches async browser APIs so Angular knows *when* to check for changes. As of Angular 18, **zoneless change detection** is available as a developer preview, relying on Signals to know exactly what changed instead of blanket tree walks — improving performance and reducing bundle size.

```typescript
// main.ts - experimental zoneless bootstrap (Angular 18+)
bootstrapApplication(AppComponent, {
  providers: [provideExperimentalZonelessChangeDetection()]
});
```

### Q20. Explain Lazy Loading and how it improves performance.

Lazy loading splits the app into chunks loaded on demand (e.g., per route), reducing the initial bundle size.

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes').then(m => m.REPORTS_ROUTES)
  }
];
```

```
Diagram: Lazy Loading Bundle Split

Initial Bundle (eagerly loaded)
 ┌─────────────────────────┐
 │ AppComponent             │
 │ HomeComponent             │
 │ Core services              │
 └─────────────┬─────────────┘
               │ user navigates to /admin
               ▼
     Network fetches chunk
 ┌─────────────────────────┐
 │ admin.component.js (lazy) │  ← downloaded only when needed
 └─────────────────────────┘
```

### Q21. What is Dependency Injection hierarchy (Injector Tree)?

Angular has a hierarchical injector system: `root` (app-wide singleton), module-level, and component-level injectors. A component-level provider creates a new instance scoped to that component subtree, shadowing the parent's.

```
Diagram: Injector Hierarchy

     Platform Injector
             │
        Root Injector  ── providedIn: 'root' services live here
             │
     ┌───────┴────────┐
     ▼                ▼
 ModuleInjector   ModuleInjector (lazy-loaded module gets its own)
     │
     ▼
 Component Injector  ── `providers: [...]` on @Component creates
     │                  a new instance for this subtree
     ▼
 Child Component Injector (inherits unless overridden)
```

```typescript
@Component({
  selector: 'app-cart',
  standalone: true,
  providers: [CartService] // new instance scoped to this component tree
})
export class CartComponent {}
```

### Q22. How do you optimize performance in a large Angular app?

- Use `OnPush` change detection + immutable data patterns
- `trackBy` in `*ngFor` / built-in tracking in `@for`
- Lazy load feature modules/routes
- Use `async` pipe (auto-unsubscribes) instead of manual subscriptions
- Virtual scrolling (`cdk-virtual-scroll-viewport`) for large lists
- Use Signals to avoid unnecessary Zone-triggered checks
- Defer non-critical UI with `@defer` (Angular 17+)
- Tree-shakable providers, standalone components, and route-level code splitting
- Avoid function calls in templates (they re-run every CD cycle) — use pipes or computed signals instead

```html
<!-- trackBy example -->
<li *ngFor="let user of users; trackBy: trackByUserId">{{ user.name }}</li>
```
```typescript
trackByUserId(index: number, user: User) { return user.id; }
```

### Q23. Explain NgRx / state management at a high level.

NgRx implements the Redux pattern (unidirectional data flow) for Angular using RxJS.

```
Diagram: NgRx Unidirectional Data Flow

  Component
     │  dispatch(action)
     ▼
   Store  ────────────────► Reducer (pure fn: (state, action) => newState)
     │                            │
     │   Effects (side effects,   │
     │   e.g. API calls) ─────────┘
     │
     ▼
 Selectors (derive slices of state)
     │
     ▼
  Component (subscribes via async pipe / Signals)
```

```typescript
// action
export const loadUsers = createAction('[User] Load Users');

// reducer
export const userReducer = createReducer(initialState,
  on(loadUsersSuccess, (state, { users }) => ({ ...state, users }))
);

// selector
export const selectUsers = createSelector(selectUserState, s => s.users);

// component
users = this.store.selectSignal(selectUsers);
```

### Q24. How does Angular handle Server-Side Rendering (SSR) and Hydration?

Angular Universal enables SSR — rendering the app to HTML on the server for faster First Contentful Paint and SEO. Since Angular 16/17, **non-destructive full hydration** reuses server-rendered DOM on the client instead of re-rendering, improving performance.

```bash
ng add @angular/ssr
```

```typescript
// app.config.server.ts
export const serverConfig: ApplicationConfig = {
  providers: [provideClientHydration()]
};
```

### Q25. What is Angular's Ahead-of-Time (AOT) vs Just-in-Time (JIT) compilation?

| | AOT | JIT |
|---|---|---|
| When | Build time | Runtime (browser) |
| Bundle size | Smaller (no compiler shipped) | Larger |
| Startup speed | Faster | Slower |
| Error detection | At build time | At runtime |
| Default | Production builds | (JIT largely phased out; AOT is default everywhere now) |

### Q26. How do you write unit tests for a component? (Jasmine/Karma or Jest)

```typescript
describe('CounterComponent', () => {
  let fixture: ComponentFixture<CounterComponent>;
  let component: CounterComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent] // standalone component
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
  });

  it('should increment count', () => {
    component.increment();
    fixture.detectChanges();
    const el: HTMLElement = fixture.nativeElement;
    expect(el.querySelector('span')?.textContent).toContain('1');
  });
});
```

---

## Latest Angular Features

*(Angular 17 / 18 — for interviews on the "latest version")*

### Q27. What are Standalone Components, and why did Angular move to them?

Standalone components/directives/pipes don't need an `NgModule` — they declare their own imports directly, simplifying the mental model and improving tree-shaking. Since Angular 17, `ng new` scaffolds standalone apps by default.

```typescript
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, CommonModule],
  template: `<router-outlet></router-outlet>`
})
export class AppComponent {}
```

### Q28. What are Angular Signals? How do they differ from RxJS Observables?

Signals are a reactive primitive introduced (developer preview in v16, stable in v17) for fine-grained, synchronous reactivity — Angular tracks exactly which components read a signal and updates only those, without needing Zone.js.

| | Signals | Observables (RxJS) |
|---|---|---|
| Nature | Synchronous, pull+push value holder | Async stream of events over time |
| Use case | Component/UI state | Async operations, streams (HTTP, websockets) |
| Unsubscribe needed? | No | Yes (or use `async` pipe) |
| Composition | `computed()`, `effect()` | Operators (`map`, `switchMap`, etc.) |

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ double() }}</p>
    <button (click)="increment()">+</button>
  `
})
export class CounterComponent {
  count = signal(0);
  double = computed(() => this.count() * 2);

  constructor() {
    effect(() => console.log('Count changed to', this.count()));
  }

  increment() {
    this.count.update(c => c + 1);
  }
}
```

```
Diagram: Signal Reactivity Graph

   signal: count
        │
        ▼
  computed: double  ──► automatically recomputed when count() changes
        │
        ▼
  Template binding {{ double() }}
        │
        ▼
  Only THIS component/subtree re-renders
  (no full Zone.js tree walk needed)
```

### Q29. What is the new Control Flow syntax (`@if`, `@for`, `@switch`)?

Introduced in Angular 17, replacing `*ngIf`/`*ngFor`/`*ngSwitch` with built-in, more performant block syntax (no `CommonModule` import needed, better type-narrowing, ~90% faster rendering for `@for`).

```html
@if (user(); as u) {
  <p>Welcome, {{ u.name }}</p>
} @else if (loading()) {
  <p>Loading...</p>
} @else {
  <p>Please log in.</p>
}

@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items found.</li>
}

@switch (status()) {
  @case ('active') { <span>Active</span> }
  @case ('inactive') { <span>Inactive</span> }
  @default { <span>Unknown</span> }
}
```

### Q30. What is `@defer` (Deferrable Views)?

`@defer` lazily loads and renders a template block only when a trigger condition is met (viewport visibility, idle time, interaction, timer), splitting it into a separate JS chunk automatically — great for below-the-fold or heavy components.

```html
@defer (on viewport) {
  <app-heavy-chart [data]="chartData()" />
} @placeholder {
  <div class="skeleton"></div>
} @loading (minimum 500ms) {
  <app-spinner />
} @error {
  <p>Failed to load chart.</p>
}
```

### Q31. What is the `inject()` function, and how does it compare to constructor injection?

`inject()` lets you obtain dependencies without a constructor — usable in field initializers, functional guards/resolvers/interceptors, and Signals-based composition functions.

```typescript
// Functional route guard (Angular 15+)
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true : router.parseUrl('/login');
};

// Functional interceptor (Angular 15+)
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token;
  return next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
};
```

### Q32. How does bootstrapping work in a modern (standalone) Angular app?

```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations(),
    provideClientHydration(),
  ]
});
```

```
Diagram: Standalone Bootstrap Flow

  main.ts
     │
     ▼
 bootstrapApplication(AppComponent, { providers })
     │
     ▼
 Root Injector created with provided services
     │
     ▼
 AppComponent instantiated (imports its own deps directly)
     │
     ▼
 RouterOutlet renders matched route's standalone component
```

### Q33. What are Signal-based Inputs, Outputs, and Model (Angular 17.1+ / 17.2+)?

Newer APIs replace decorator-based `@Input`/`@Output` with signal-based equivalents for full reactivity integration.

```typescript
@Component({ selector: 'app-child', standalone: true, template: `...` })
export class ChildComponent {
  // Signal input (Angular 17.1+)
  name = input<string>('default');
  requiredId = input.required<number>();

  // Signal output (Angular 17.3+)
  saved = output<void>();

  // Two-way bindable signal (model, Angular 17.2+)
  value = model<string>('');
}
```

```html
<app-child [requiredId]="5" [(value)]="parentValue" (saved)="onSave()" />
```

### Q34. What is the difference between Angular's Ivy renderer and older View Engine?

Ivy (default since Angular 9) compiles components to more efficient, tree-shakable instructions, enabling smaller bundles, faster compilation, better debugging, and features like locality (each component compiles independently). View Engine is fully removed in modern Angular — Ivy is the only renderer today.

---

## Debugging & Error-Handling Level

*(Real-world bugs, common runtime errors, and how to diagnose/fix them)*

### Q35. What is `ExpressionChangedAfterItHasBeenCheckedError` (NG0100), and how do you fix it?

Angular's dev-mode runs a **second change-detection pass** after the first to verify no binding value changed during the check ("unidirectional data flow" guard). This error fires when a value read in a template changes *between* those two passes — commonly caused by mutating state inside `ngAfterViewInit`/`ngAfterContentInit`, or a child updating a parent-bound value during CD.

```typescript
// ❌ Causes NG0100
ngAfterViewInit() {
  this.message = 'Loaded'; // mutates a bound value after view was checked
}

// ✅ Fix 1: defer to next tick
ngAfterViewInit() {
  Promise.resolve().then(() => this.message = 'Loaded');
}

// ✅ Fix 2: use setTimeout
ngAfterViewInit() {
  setTimeout(() => this.message = 'Loaded');
}

// ✅ Fix 3 (best in modern Angular): use a signal — signals are designed
// to be updated safely and propagate without this class of error.
message = signal('Loading...');
ngAfterViewInit() { this.message.set('Loaded'); }
```

```
Diagram: Why NG0100 Happens

  Pass 1 (change detection): reads message = "Loading..."
        │
        ▼
  ngAfterViewInit() runs → sets message = "Loaded"
        │
        ▼
  Pass 2 (dev-mode verification): reads message = "Loaded"
        │
        ▼
  Mismatch! "Loading..." !== "Loaded" → NG0100 thrown
```

### Q36. What causes `NG0200: Circular dependency in DI detected`, and how do you resolve it?

Two (or more) injectables depend on each other directly or via a chain — Angular can't resolve which to construct first.

```typescript
// ServiceA depends on ServiceB, ServiceB depends on ServiceA → NG0200

@Injectable({ providedIn: 'root' })
export class ServiceA {
  constructor(private b: ServiceB) {}
}

@Injectable({ providedIn: 'root' })
export class ServiceB {
  constructor(private a: ServiceA) {}
}
```

**Fixes:**
- Break the cycle by extracting shared logic into a third service both depend on.
- Use `forwardRef(() => ServiceA)` only as a last resort for legitimate circular class references (rare, mostly relevant to decorators, not typical service design).
- Re-architect: circular DI is almost always a design smell — prefer an event bus/shared state service over direct cross-references.

### Q37. How do you implement global (uncaught) error handling in Angular?

Implement Angular's `ErrorHandler` to catch all otherwise-uncaught synchronous and template errors app-wide (e.g., to log to a monitoring service like Sentry).

```typescript
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(error: unknown): void {
    console.error('Global error caught:', error);
    // send to logging service (Sentry, Application Insights, etc.)
  }
}

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [{ provide: ErrorHandler, useClass: GlobalErrorHandler }]
};
```

Note: `ErrorHandler` does **not** catch HTTP errors (those resolve as Observable errors) — handle those with an `HttpInterceptor` (see Q38).

### Q38. How do you centralize HTTP error handling with an interceptor?

```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((err: HttpErrorResponse) => {
      if (err.status === 401) {
        inject(Router).navigate(['/login']);
      } else if (err.status === 0) {
        console.error('Network error / CORS issue');
      } else {
        console.error(`Server returned ${err.status}: ${err.message}`);
      }
      return throwError(() => err);
    })
  );
};
```

### Q39. What causes memory leaks in Angular, and how do you find/fix them?

**Common causes:**
- Subscribing to an `Observable`/`Subject` in a component without unsubscribing in `ngOnDestroy`.
- Attaching DOM event listeners manually (`addEventListener`) without removing them.
- Holding references to detached DOM nodes or components in a service (singleton) that outlives the component.
- `setInterval`/`setTimeout` not cleared on destroy.

```typescript
// ❌ Leak: subscription never cleaned up
ngOnInit() {
  this.dataService.stream$.subscribe(v => this.value = v);
}

// ✅ Fix 1: takeUntil pattern
private destroy$ = new Subject<void>();
ngOnInit() {
  this.dataService.stream$
    .pipe(takeUntil(this.destroy$))
    .subscribe(v => this.value = v);
}
ngOnDestroy() { this.destroy$.next(); this.destroy$.complete(); }

// ✅ Fix 2: async pipe auto-unsubscribes (preferred when possible)
// template: {{ dataService.stream$ | async }}

// ✅ Fix 3 (Angular 16+): takeUntilDestroyed()
private value = toSignal(
  this.dataService.stream$.pipe(takeUntilDestroyed())
);
```

**Diagnosing:** use Chrome DevTools → Memory tab → take heap snapshots before/after navigating away from a component repeatedly; look for growing detached DOM trees or retained component instances. Angular DevTools' Profiler can also flag components that keep re-rendering unexpectedly.

### Q40. How do you debug unwanted or excessive change detection cycles?

- Install **Angular DevTools** (Chrome extension) → Profiler tab → record a session → look for components checked far more often than expected.
- Temporarily add `console.count('MyComponent CD')` inside a template-bound getter to see how often it's called.
- Common culprits: calling a function directly in a template (`{{ getTotal() }}` re-runs every CD cycle), missing `trackBy`/`track` in loops causing full DOM re-renders, or a deeply nested `OnPush`-less tree.

```html
<!-- ❌ Re-executes on every CD cycle -->
<p>{{ calculateTotal() }}</p>

<!-- ✅ Use a pipe (pure, memoized) or a computed signal instead -->
<p>{{ total() }}</p>
```
```typescript
total = computed(() => this.items().reduce((sum, i) => sum + i.price, 0));
```

### Q41. What does "`NG0300: Selector not found`" or "`Component X is not part of any NgModule`" usually mean, and how do you fix it?

This typically appears after migrating to standalone components (or mixing standalone + NgModule-based code) when a component/directive/pipe used in a template isn't imported anywhere Angular can see.

```typescript
// ❌ Forgot to import CommonModule/RouterOutlet/etc. in a standalone component
@Component({
  selector: 'app-shell',
  standalone: true,
  template: `<router-outlet></router-outlet>`, // fails: RouterOutlet not imported
})
export class ShellComponent {}

// ✅ Fix
@Component({
  selector: 'app-shell',
  standalone: true,
  imports: [RouterOutlet],
  template: `<router-outlet></router-outlet>`,
})
export class ShellComponent {}
```

### Q42. How do you handle errors inside an RxJS stream without killing the subscription?

An unhandled error in an RxJS `Observable` **completes the stream permanently** — the source stops emitting even future valid values, which silently breaks features (e.g., a search-as-you-type box that stops working after one failed request).

```typescript
// ❌ One failed request kills the whole stream forever
searchTerm$.pipe(
  switchMap(term => this.api.search(term)) // if this errors, stream dies
).subscribe(results => this.results = results);

// ✅ Catch the error INSIDE the inner observable, not outside
searchTerm$.pipe(
  switchMap(term =>
    this.api.search(term).pipe(
      catchError(err => { console.error(err); return of([]); })
    )
  )
).subscribe(results => this.results = results);
```

### Q43. How do you test asynchronous code and catch timing-related bugs?

```typescript
it('should load data after debounce', fakeAsync(() => {
  component.search('angular');
  tick(300); // simulate the passage of time (e.g., debounceTime(300))
  fixture.detectChanges();
  expect(component.results.length).toBeGreaterThan(0);
}));

it('should resolve async observable', waitForAsync(() => {
  service.getData().subscribe(data => {
    expect(data).toBeTruthy();
  });
}));
```

A frequent bug source: forgetting `tick()`/`flush()` in `fakeAsync` tests, or not calling `fixture.detectChanges()` after async state changes — leading to false-positive/false-negative test results that don't reflect real UI behavior.

### Q44. What are common Angular production bugs and their typical root cause?

| Symptom | Likely Root Cause |
|---|---|
| Works in `ng serve`, breaks in prod build | Relying on non-AOT-safe code, missing `@Input`/template type strictness issues caught only by AOT |
| Blank white screen in prod, no console error | Uncaught error in `ErrorHandler`-less setup, or a lazy chunk 404 (stale deployment / cache) |
| Data flashes then disappears | Race condition — a later API response overwrites a slower earlier one (`switchMap` needed instead of `mergeMap`) |
| Form value not updating UI | Missing `ChangeDetectorRef.markForCheck()` under `OnPush`, or mutating an array/object in place instead of creating a new reference |
| "Cannot read properties of undefined" in template | Accessing nested async data before it's loaded — fix with `?.` safe navigation or `@if (data(); as d)` |
| Duplicate HTTP calls | Cold observable subscribed to multiple times (e.g., in template without `async as`) instead of sharing via `shareReplay(1)` |

```html
<!-- ❌ Subscribes twice (two HTTP calls) -->
<p>{{ (user$ | async)?.name }}</p>
<p>{{ (user$ | async)?.email }}</p>

<!-- ✅ Subscribe once, reuse -->
@if (user$ | async; as user) {
  <p>{{ user.name }}</p>
  <p>{{ user.email }}</p>
}
```

---

## Granular / Deep-Dive Advanced Level

*(Fine-grained internals often reserved for senior/staff-level interviews)*

### Q45. Explain `ViewChild`/`ContentChild` and the `static` option.

```typescript
@Component({ selector: 'app-parent', standalone: true, template: `
  <input #box>
  <app-child></app-child>
`})
export class ParentComponent implements AfterViewInit {
  @ViewChild('box', { static: false }) inputRef!: ElementRef<HTMLInputElement>;
  @ViewChild(ChildComponent) childCmp!: ChildComponent;

  ngAfterViewInit() {
    this.inputRef.nativeElement.focus(); // only safe here, not in ngOnInit
  }
}
```

`static: true` resolves the reference before change detection (only valid for elements/directives not inside an `*ngIf`/`@if`); `static: false` (default) resolves it after view initialization — hence why `ViewChild` refs are only guaranteed inside `ngAfterViewInit`, not `ngOnInit`.

### Q46. When and how do you use `NgZone.runOutsideAngular()`?

Used to run code (e.g., a high-frequency event listener, `requestAnimationFrame` loop, or third-party library like a map/canvas) **outside** Angular's Zone so it doesn't trigger a full change-detection cycle on every tick — a common performance fix for laggy UIs.

```typescript
constructor(private ngZone: NgZone) {}

ngAfterViewInit() {
  this.ngZone.runOutsideAngular(() => {
    window.addEventListener('scroll', this.onScroll);
  });
}

onScroll = () => {
  // heavy computation here doesn't trigger CD
  // re-enter the zone only if you need to update bound UI:
  if (shouldUpdateUI) {
    this.ngZone.run(() => this.scrollPosition = window.scrollY);
  }
};
```

### Q47. How do injection tokens and multi-providers work?

`InjectionToken` lets you inject plain values/configuration (not just classes) through DI; `multi: true` collects multiple providers into an array (used internally for `HTTP_INTERCEPTORS`, validators, etc.).

```typescript
export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

// provide it
providers: [{ provide: API_BASE_URL, useValue: 'https://api.example.com' }]

// consume it
private baseUrl = inject(API_BASE_URL);

// multi-provider example
providers: [
  { provide: NG_VALIDATORS, useValue: myCustomValidator, multi: true }
]
```

### Q48. How do you write a custom RxJS operator?

```typescript
function logTiming<T>(label: string) {
  return (source: Observable<T>): Observable<T> => {
    return new Observable(subscriber => {
      const start = performance.now();
      return source.subscribe({
        next: v => subscriber.next(v),
        error: e => subscriber.error(e),
        complete: () => {
          console.log(`${label} took ${performance.now() - start}ms`);
          subscriber.complete();
        }
      });
    });
  };
}

// usage
this.api.getData().pipe(logTiming('getData')).subscribe();
```

### Q49. What is `Renderer2`, and why prefer it over direct DOM access?

`Renderer2` abstracts DOM manipulation so components remain platform-agnostic (works with server-side rendering and Web Workers, where `document`/`window` aren't directly available) and safer against XSS than raw `nativeElement` mutation.

```typescript
constructor(private renderer: Renderer2, private el: ElementRef) {}

ngOnInit() {
  this.renderer.setStyle(this.el.nativeElement, 'color', 'red');
  this.renderer.addClass(this.el.nativeElement, 'highlighted');
}
```

### Q50. How does Angular protect against XSS, and when do you need `DomSanitizer`?

Angular auto-sanitizes values bound via interpolation and property binding (e.g., `[innerHTML]`) by default, stripping dangerous content. `DomSanitizer` is used to explicitly mark trusted content (e.g., a known-safe URL or SVG) when you're certain it's safe — bypassing sanitization should be rare and deliberate.

```typescript
constructor(private sanitizer: DomSanitizer) {}

trustedUrl = this.sanitizer.bypassSecurityTrustResourceUrl(externalVideoUrl);
```

```html
<iframe [src]="trustedUrl"></iframe>
```

⚠️ Never call `bypassSecurityTrust*` on unvalidated user input — that reintroduces the exact XSS vector Angular protects against by default.

### Q51. What are Angular Elements, and when would you use them?

Angular Elements packages a component as a native **custom element** (Web Component) that can be embedded in non-Angular apps (legacy jQuery pages, React/Vue hosts, CMS platforms).

```typescript
const el = createCustomElement(WidgetComponent, { injector });
customElements.define('app-widget', el);
```
```html
<!-- usable in ANY web page, no Angular required -->
<app-widget></app-widget>
```

### Q52. What's the difference between `Subject`, `BehaviorSubject`, `ReplaySubject`, and `AsyncSubject`?

| Type | Emits to new subscribers | Use case |
|---|---|---|
| `Subject` | Nothing (only future emissions) | Simple event bus |
| `BehaviorSubject` | Last emitted value immediately (requires initial value) | Current app state (e.g., logged-in user) |
| `ReplaySubject(n)` | Last `n` emitted values | Replay history/cache |
| `AsyncSubject` | Only the final value, only on complete | Result of a one-shot operation |

### Q53. How would you debug a "component not updating" bug under `OnPush`?

```
Diagram: OnPush Debug Checklist

  Is the bound value's REFERENCE changing?
        │
   No ──┴── Yes
   │         │
   ▼         ▼
 Mutating   Should update automatically —
 in place?  check for a missed markForCheck()
   │         after an async callback outside
   ▼         Angular's normal triggers
 Replace with
 new object/array
 (spread/immutable
  update) OR call
 cdRef.markForCheck()
 manually after mutation
```

```typescript
// ❌ Mutation — OnPush won't see a "new" reference
this.items.push(newItem);

// ✅ Immutable update — new reference triggers OnPush check
this.items = [...this.items, newItem];
```

### Q54. What is Module Federation / Micro-Frontends with Angular?

Module Federation (via `@angular-architects/module-federation` or native `esbuild`/Vite plugins) lets independently-built and independently-deployed Angular apps ("micro-frontends") load each other's code at runtime — a "shell" app dynamically loads remote bundles instead of everything being one monolithic build.

```
Diagram: Micro-Frontend Shell Architecture

        ┌───────────────┐
        │  Shell App     │  (routing, layout, auth)
        └───────┬────────┘
       ┌─────────┼─────────────┐
       ▼         ▼             ▼
  Remote:      Remote:      Remote:
  Catalog MFE  Checkout MFE  Account MFE
  (own repo,   (own repo,    (own repo,
   own deploy)  own deploy)   own deploy)
```

### Q55. How do you handle "flickering"/race-condition bugs from overlapping HTTP requests (e.g., in a search box)?

Use `switchMap` (not `mergeMap`/`concatMap`) so a new request automatically cancels the previous in-flight one — the classic fix for "stale response overwrites the newer one" bugs.

```typescript
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.api.search(term).pipe(catchError(() => of([]))))
).subscribe(results => this.results = results);
```

| Operator | Behavior | Use when |
|---|---|---|
| `switchMap` | Cancels previous inner observable | Search-as-you-type, latest-wins |
| `mergeMap` | Runs all in parallel, no cancel | Independent parallel requests |
| `concatMap` | Queues, runs one at a time in order | Order-sensitive sequential writes |
| `exhaustMap` | Ignores new triggers until current completes | Prevent duplicate submit on double-click |

---

## Architecture Diagrams

### Overall Angular App Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                                │
│                                                                  │
│   User Interaction                                              │
│         │                                                       │
│         ▼                                                       │
│   Component (event handler)                                     │
│         │                                                       │
│         ▼                                                       │
│   Service (business logic) ──────► HttpClient ──► Backend API   │
│         │                               ▲                       │
│         ▼                               │                       │
│   State (Signal / Store / Observable) ◄─┘                       │
│         │                                                       │
│         ▼                                                       │
│   Template re-render (via bindings)                             │
│         │                                                       │
│         ▼                                                       │
│   DOM Update                                                    │
└─────────────────────────────────────────────────────────────┘
```

### Module vs Standalone Comparison

```
  Legacy (NgModule-based)          Modern (Standalone, v17+ default)

  AppModule                         AppComponent
   ├─ declarations: [Comp,...]       ├─ imports: [RouterOutlet, ...]
   ├─ imports: [BrowserModule,...]    (self-contained, no NgModule)
   └─ bootstrap: [AppComponent]

  FeatureModule                     FeatureComponent (standalone)
   ├─ declarations: [...]             ├─ imports: [CommonModule, ...]
   └─ imports: [SharedModule]         (lazy-loaded via loadComponent)
```

---

## Quick Reference Cheat Sheet

| Topic | Key Command / Syntax |
|---|---|
| New app | `ng new app-name --standalone` |
| New component | `ng generate component name` |
| Serve | `ng serve` |
| Build (prod) | `ng build --configuration production` |
| Signal | `const s = signal(0); s.set(1); s.update(v => v+1);` |
| Computed | `const c = computed(() => s() * 2);` |
| Effect | `effect(() => console.log(s()));` |
| Control flow | `@if / @for (track) / @switch` |
| Defer | `@defer (on viewport) {...} @placeholder {...}` |
| Inject | `private svc = inject(MyService);` |
| Route guard | `CanActivateFn` functional guard |
| Change detection | `ChangeDetectionStrategy.OnPush` |
| Lazy route | `loadComponent: () => import(...).then(m => m.X)` |

---

### Suggested Interview Flow

1. Start with fundamentals (Q1–Q9) to gauge core understanding.
2. Move to intermediate (Q10–Q17) for practical app-building skills.
3. Probe advanced (Q18–Q26) for performance, architecture, and testing depth.
4. Finish with latest-version questions (Q27–Q34) to check if the candidate is current with Signals, standalone APIs, and the new control flow — a strong signal of active, up-to-date Angular experience.
