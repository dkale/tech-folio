# Angular Senior/Architect Interview — MCQ Question Bank

> **80 questions across 4 phases:** Beginner → Senior → Expert/Architect → Scenario-Based
> Each question includes the correct answer and detailed reasoning.
> Compiled from a live interactive quiz session — scored 78/80 (97.5%).

---

## Table of Contents

| Phase | Level | Questions |
|---|---|---|
| [Phase 1](#phase-1-beginner-level-q1q20) | Beginner | Q1–Q20 |
| [Phase 2](#phase-2-senior-level-s-q1s-q20) | Senior | S-Q1–S-Q20 |
| [Phase 3](#phase-3-expertarchitect-level-e-q1e-q20) | Expert/Architect | E-Q1–E-Q20 |
| [Phase 4](#phase-4-scenario-based-questions-sc-q1sc-q20) | Scenario-Based | SC-Q1–SC-Q20 |

---

## Phase 1: Beginner Level (Q1–Q20)

---

### Q1. What decorator is used to define a component in Angular?

✅ **`@Component`**

**Reasoning:** `@Component` is the metadata decorator that tells Angular a class is a component. It accepts metadata like `selector`, `templateUrl`, and `styleUrls`. `@NgModule` organises modules, `@Injectable` marks a class for dependency injection, and `@Pipe` defines a pipe.

---

### Q2. Which Angular CLI command generates a new component called `hero`?

✅ **`ng generate component hero`**

**Reasoning:** `ng generate component <name>` (or shorthand `ng g c <name>`) is the correct CLI command. `ng add` is for adding third-party packages/schematics. `ng create` and `ng make` are not valid Angular CLI commands.

---

### Q3. What is the correct syntax for two-way data binding in Angular?

✅ **`[(ngModel)]`**

**Reasoning:** The "banana in a box" syntax `[(ngModel)]` combines property binding `[]` (data flows in) and event binding `()` (data flows out), creating true two-way binding. `[ngModel]` is one-way property binding only. `(ngModel)` is event binding only. `{{ngModel}}` is interpolation for display only.

---

### Q4. Which file contains the root module of an Angular application by default?

✅ **`app.module.ts`**

**Reasoning:** `app.module.ts` exports the `AppModule` class decorated with `@NgModule`, which is the root module Angular bootstraps. `main.ts` references `AppModule` but is the entry point. `index.html` is the shell HTML. `app.component.ts` is the root component, not the root module.

---

### Q5. What does the `*ngIf` structural directive do?

✅ **Adds or removes a DOM element based on a boolean expression**

**Reasoning:** `*ngIf` is a structural directive — it physically adds or removes DOM elements (not just hides them with CSS). When false, the element is destroyed and removed from the DOM, releasing memory. `*ngFor` handles iteration. `[ngClass]` applies classes conditionally. `#ref` creates template reference variables.

---

### Q6. What is interpolation in Angular?

✅ **`{{ expression }}` — renders component data in the template**

**Reasoning:** Interpolation uses double curly braces `{{ }}` to embed component property values or expressions directly into the HTML template. Angular evaluates the expression and converts the result to a string. It is automatically XSS-safe because Angular HTML-encodes the output.

---

### Q7. Which Angular module must be imported to enable two-way binding with `ngModel`?

✅ **`FormsModule`**

**Reasoning:** `ngModel` directive is part of `FormsModule` (`@angular/forms`). Without importing it in your `NgModule`, Angular will throw a template error. `ReactiveFormsModule` enables reactive forms (FormGroup/FormControl) but does not provide `ngModel`.

---

### Q8. What is the primary purpose of the `@Input()` decorator?

✅ **Pass data from a parent component into a child component**

**Reasoning:** `@Input()` marks a class property as an input binding, allowing a parent component to pass data down to a child via property binding: `<child [data]="parentValue">`. It is the "data flows down" half of Angular's unidirectional data flow pattern.

---

### Q9. What is `@Output()` used for in Angular?

✅ **Emit custom events from a child component to its parent**

**Reasoning:** `@Output()` marks an `EventEmitter` property so a child can emit events to its parent. The parent listens using event binding: `<child (action)="handler($event)">`. Together with `@Input()`, this forms Angular's parent↔child communication pattern.

---

### Q10. What does the `*ngFor` structural directive do?

✅ **Iterates over a collection and renders a template per item**

**Reasoning:** `*ngFor="let item of items"` is a structural directive that stamps out a template for each item in an iterable. It also exposes local variables like `index`, `first`, `last`, `even`, `odd` for use within the loop.

---

### Q11. What is the primary purpose of the Angular Router?

✅ **Navigate between views/components in a single-page application**

**Reasoning:** The Angular Router (`@angular/router`) enables client-side navigation between component views without full page reloads, preserving the SPA experience. It maps URL paths to components, supports nested routes, lazy loading, and route guards.

---

### Q12. What best describes an Angular service?

✅ **A singleton class for sharing data and business logic across components**

**Reasoning:** Services are plain TypeScript classes decorated with `@Injectable()`. When provided at root level (`providedIn: 'root'`), Angular creates a single instance shared across the entire app. They encapsulate business logic, API calls, and state — keeping components lean.

---

### Q13. What does the `@Injectable()` decorator indicate about a class?

✅ **The class can be provided and injected as a dependency**

**Reasoning:** `@Injectable()` marks a class as participanting in Angular's dependency injection system. The `providedIn` metadata tells Angular where to register the provider (e.g. `'root'` for app-wide singleton). Without this decorator, Angular's DI tree won't register the class.

---

### Q14. What is the purpose of Angular's `HttpClient`?

✅ **Makes HTTP requests to communicate with backend APIs**

**Reasoning:** `HttpClient` (from `@angular/common/http`) provides a simplified, Observable-based API for making HTTP requests (GET, POST, PUT, DELETE, etc.). It includes built-in support for JSON parsing, request/response interception, and typed responses.

---

### Q15. What are Angular pipes primarily used for?

✅ **Transform and format values for display in templates**

**Reasoning:** Pipes like `DatePipe`, `CurrencyPipe`, `UpperCasePipe`, and custom pipes transform raw values into display-friendly formats inside templates using the `|` operator: `{{ date | date:'short' }}`. They are pure functions by default (memoized) for performance.

---

### Q16. What is a template reference variable in Angular?

✅ **A variable declared with `#` in a template to reference a DOM element or directive**

**Reasoning:** Template reference variables (`#myInput`) give you direct access to a DOM element, Angular component, or directive within the template. Example: `<input #emailInput>` then `{{ emailInput.value }}`. They can also be passed to methods: `(click)="submit(emailInput)"`.

---

### Q17. What is lazy loading in the context of Angular routing?

✅ **Loading feature modules only when their associated route is navigated to**

**Reasoning:** Lazy loading splits the app into separate JS bundles per feature module. The bundle is only downloaded when the user navigates to that route, reducing the initial bundle size and improving startup performance. Configured via `loadChildren: () => import(...).then(m => m.FeatureModule)`.

---

### Q18. What is the purpose of the `ngOnDestroy` lifecycle hook?

✅ **Called just before Angular destroys the component — used for cleanup**

**Reasoning:** `ngOnDestroy` is the last lifecycle hook called before Angular destroys a component. It is the correct place to unsubscribe from Observables, detach event listeners, clear timers, and cancel pending HTTP requests to prevent memory leaks.

---

### Q19. What is the key difference between a component's constructor and `ngOnInit`?

✅ **Constructor is for DI wiring; ngOnInit is for initialization logic after @Input values are set**

**Reasoning:** The constructor runs first and is Angular's DI entry point — services are injected here. However, `@Input()` properties are **not yet set** in the constructor. `ngOnInit` runs after the first `ngOnChanges` and is when all bound inputs are available, making it the right place for initialization logic.

---

### Q20. What does `ng build --configuration=production` do?

✅ **Compiles and bundles the app with AOT, minification, and tree-shaking optimizations**

**Reasoning:** A production build enables: **AOT compilation** (templates compiled at build time), **minification** (removes whitespace/comments), **tree-shaking** (removes unused code), **source map exclusion** (smaller files), and **bundle hashing** (cache busting). The output is placed in the `dist/` folder.

---

*Phase 1 Complete — 20/20 questions.*

---

## Phase 2: Senior Level (S-Q1–S-Q20)

### S-Q1. What does `ChangeDetectionStrategy.OnPush` actually do in Angular?

✅ **Triggers CD only when an `@Input` reference changes, an Observable emits via async pipe, or `markForCheck()` is called**

**Reasoning:** `OnPush` opts a component out of Angular's default "check everything" CD cycle. Angular will only re-check the component when: (1) an `@Input` property receives a **new reference** (not mutation), (2) an event originates from the component or its children, (3) an Observable bound via the `async` pipe emits, or (4) `ChangeDetectorRef.markForCheck()` is called explicitly. This significantly reduces CD overhead in large component trees.

### S-Q2. What is the key behavioral difference between RxJS `switchMap` and `mergeMap`?

✅ **`switchMap` cancels the previous inner observable when a new emission arrives; `mergeMap` keeps all concurrent subscriptions**

**Reasoning:** `switchMap` **cancels and unsubscribes** from the previous inner observable whenever the source emits a new value — ideal for type-ahead search (discard stale requests). `mergeMap` subscribes to every inner observable concurrently and merges all their emissions — ideal for independent parallel operations like file uploads. Choosing the wrong one causes race conditions or dropped events.

### S-Q3. What is the difference between `CanActivate` and `CanActivateChild` route guards?

✅ **`CanActivate` guards the route itself; `CanActivateChild` guards every child route beneath it**

**Reasoning:** `CanActivate` is checked when navigating **to** a specific route. `CanActivateChild` is placed on a parent route and is checked on every navigation into any of its **child routes**, without needing to repeat the guard on each child individually. This is useful for protecting an entire feature section (e.g., an admin area) without repeating auth checks on every child route.

### S-Q4. What is the key difference between `@ViewChild` and `@ContentChild`?

✅ **`@ViewChild` queries the component's own template; `@ContentChild` queries content projected via `ng-content`**

**Reasoning:** `@ViewChild` gives access to elements/components defined **inside the component's own template** (`*.component.html`). `@ContentChild` gives access to elements/components that are **projected into** the component via `<ng-content>` by the parent consumer. They are available at different lifecycle points: `ngAfterViewInit` for ViewChild, `ngAfterContentInit` for ContentChild.

### S-Q5. What is the role of `FormBuilder` in Angular Reactive Forms?

✅ **`FormBuilder` is a service providing shorthand to create `FormGroup`, `FormControl`, and `FormArray` instances**

**Reasoning:** `FormBuilder` (injected from `@angular/forms`) reduces boilerplate when building complex reactive forms. Instead of `new FormGroup({ name: new FormControl('') })`, you write `this.fb.group({ name: [''] })`. It is purely a convenience wrapper — the resulting `FormGroup` structure is identical. It belongs to `ReactiveFormsModule`, not `FormsModule`.

### S-Q6. What are Angular HTTP Interceptors and what are they used for?

✅ **Intercept HTTP requests/responses globally to add headers, handle errors, or transform data**

**Reasoning:** HTTP Interceptors implement the `HttpInterceptor` interface and are registered as multi-providers. They form a middleware chain around every `HttpClient` call, making them ideal for: injecting auth tokens into every request, logging, caching, retrying failed requests, transforming response shapes, and centralising error handling — all without modifying individual service calls.

### S-Q7. Why should you use `trackBy` with `*ngFor`, and what problem does it solve?

✅ **`trackBy` provides a unique identity per item so Angular reuses existing DOM nodes instead of destroying and recreating them**

**Reasoning:** Without `trackBy`, when the array reference changes Angular destroys and recreates **all** DOM nodes in the list — even if only one item changed. With `trackBy: trackById`, Angular compares identities and only destroys/recreates nodes for items that genuinely changed. This dramatically reduces DOM churn, preserves focus/scroll state, and improves rendering performance for large or frequently updated lists.

### S-Q8. What does `ViewEncapsulation.None` mean for a component's styles?

✅ **No encapsulation — component styles become global and leak to the entire document**

**Reasoning:** Angular offers three encapsulation modes: `Emulated` (default — adds unique attributes like `_ngcontent-xxx` to scope styles), `ShadowDom` (uses native browser Shadow DOM), and `None` (styles are injected globally with no scoping). `None` is useful for theming/reset styles but risks unintentional style overrides across unrelated components.

### S-Q9. What does Angular's `async` pipe do, and what problem does it solve?

✅ **Subscribes to an Observable/Promise, returns the latest emitted value, and auto-unsubscribes on component destruction**

**Reasoning:** The `async` pipe handles the full Observable lifecycle in the template: it subscribes on creation, triggers change detection on each emission, and **automatically unsubscribes** when the component is destroyed — eliminating memory leaks without manual `ngOnDestroy` cleanup. It pairs perfectly with `OnPush` change detection because it calls `markForCheck()` internally.

### S-Q10. What is the difference between AOT and JIT compilation in Angular?

✅ **AOT compiles templates at build time for faster startup and early error detection; JIT compiles in the browser at runtime**

**Reasoning:** **JIT (Just-In-Time):** The Angular compiler is shipped to the browser and compiles templates at runtime. Slower startup, larger bundle, used in development. **AOT (Ahead-Of-Time):** Templates are compiled to TypeScript/JavaScript at build time. Benefits: faster rendering (no compiler in browser), smaller bundles (compiler excluded), earlier error detection (template errors caught at build time), better security (no `eval`-based template compilation). AOT has been the default since Angular 9 (Ivy).

### S-Q11. What is the key difference between RxJS `Subject`, `BehaviorSubject`, and `ReplaySubject`?

✅ **`Subject` emits to current subscribers only; `BehaviorSubject` holds a current value for new subscribers; `ReplaySubject` replays the last N emissions**

**Reasoning:**
✅ **`Subject`**: No memory — late subscribers miss previous emissions. Use for fire-and-forget events.
✅ **`BehaviorSubject`**: Requires an initial value; new subscribers immediately receive the **current value**. Use for state that always has a value (e.g., auth state, current user).
✅ **`ReplaySubject(n)`**: Buffers the last `n` emissions; new subscribers receive the buffered history. Use for caching or when late subscribers need context.

### S-Q12. What is the `CanDeactivate` route guard used for?

✅ **Prevents navigation away from a route — useful for unsaved change warnings**

**Reasoning:** `CanDeactivate` runs when a user attempts to **leave** a route. It is typically used to display a confirmation dialog ("You have unsaved changes. Leave anyway?"). The guard receives the current component instance, allowing you to check its internal state (e.g., `form.dirty`) and conditionally block or allow navigation. It returns `boolean | Observable<boolean> | Promise<boolean>`.

### S-Q13. What is the purpose of a Route Resolver in Angular?

✅ **Pre-fetches data before route activation so the component has data immediately on `ngOnInit`**

**Reasoning:** A Resolver implements the `Resolve<T>` interface and runs before the route is activated. Angular waits for the resolved Observable/Promise to complete before navigating. The resolved data is available via `this.route.snapshot.data['key']`. This eliminates loading spinners inside the component and ensures data is ready on component init — at the cost of delaying navigation until data loads.

### S-Q14. What is the purpose of `ng-content` (content projection) in Angular?

✅ **Allows a component to accept and render externally-provided HTML between its tags**

**Reasoning:** Content projection (`ng-content`) is Angular's slot mechanism, similar to `<slot>` in Web Components. A reusable component like `<card>` can define `<ng-content>` placeholder(s) in its template, and consumers fill them with arbitrary HTML: `<card><h1>Title</h1></card>`. Multi-slot projection uses `select` attribute: `<ng-content select="[header]">`.

### S-Q15. What is the difference between structural and attribute directives in Angular?

✅ **Structural directives modify DOM layout by adding/removing elements; attribute directives change appearance or behavior of existing elements**

**Reasoning:** **Structural directives** (prefixed with `*`) manipulate the DOM structure — they add, remove, or replace elements by working with `<ng-template>`. Examples: `*ngIf`, `*ngFor`, `*ngSwitch`. **Attribute directives** sit on an existing element and modify its properties, appearance, or behavior without changing DOM structure. Examples: `[ngClass]`, `[ngStyle]`, custom `[highlight]`. Both can be custom-built.

### S-Q16. What role does Zone.js play in an Angular application?

✅ **Zone.js patches async APIs (`setTimeout`, Promises, XHR) so Angular knows when to trigger change detection**

**Reasoning:** Zone.js monkey-patches all asynchronous browser APIs (timers, XHR, Promises, DOM events). Angular runs inside `NgZone` — a special zone that notifies Angular after every async task completes, triggering change detection automatically. This is why updating a property inside a `setTimeout` updates the view without manual intervention. Angular 17+ introduces the option to run Zoneless with Signals.

### S-Q17. What is the behavioral difference between RxJS `debounceTime` and `throttleTime`?

✅ **`debounceTime` waits until silence for X ms then emits; `throttleTime` emits the first value then suppresses for X ms**

**Reasoning:**
✅ **`debounceTime(300)`**: Waits for a pause of 300ms before emitting the latest value. Best for search input — waits until the user stops typing. Discards intermediate values.
✅ **`throttleTime(300)`**: Emits the first value immediately, then ignores subsequent values for 300ms. Best for rate-limiting button clicks or scroll events while still responding instantly to the first interaction.

### S-Q18. What is the purpose of the `APP_INITIALIZER` injection token?

✅ **Runs initialization logic (e.g. load remote config) before the Angular app fully starts**

**Reasoning:** `APP_INITIALIZER` is a multi-provider token that accepts factory functions returning `Promise` or `Observable`. Angular waits for all initializers to complete before bootstrapping the root component. Common use cases: fetching remote configuration, loading feature flags, initialising logging/monitoring SDKs, or pre-warming caches before any component renders.

### S-Q19. What does calling `ChangeDetectorRef.detach()` on a component do?

✅ **Removes the component from the CD tree — no automatic change detection until reattached**

**Reasoning:** `detach()` disconnects the component's view from Angular's change detection tree. Angular will completely skip it during CD cycles, giving maximum performance control. You can manually trigger CD by calling `detectChanges()` when needed, and reattach with `reattach()`. This pattern is used in high-frequency update scenarios (e.g., real-time charts) where you want to batch or manually control when the view refreshes.

### S-Q20. What is the core architectural difference between Reactive and Template-Driven Forms?

✅ **Reactive Forms are model-driven (defined in the class), more testable and scalable; Template-Driven are directive-driven and defined in the template**

**Reasoning:**
| Aspect | Reactive Forms | Template-Driven Forms |
|---|---|---|
| **Definition** | TypeScript class (model-driven) | HTML template (directive-driven) |
| **Module** | `ReactiveFormsModule` | `FormsModule` |
| **Validation** | Synchronous & async validators | HTML attributes + directives |
| **Testability** | High — form model is plain TS | Lower — requires DOM rendering |
| **Scalability** | Better for complex, dynamic forms | Better for simple forms |
| **Source of truth** | Component class | Template |

Both support async validators. The misconception in option B is that Template-Driven forms do not support async validators — they do, via the `AsyncValidator` interface applied to a directive.

*Phase 2 Complete — 20 Senior-Level questions.*


---

## Phase 3: Expert/Architect Level (E-Q1–E-Q20)

---

### E-Q1. What architectural advantage does a monorepo (e.g. Nx workspace) provide for multi-app Angular organizations?

✅ **A monorepo with Nx enables shared libraries, consistent versioning, and optimized builds via affected-only task runs**

**Reasoning:** Nx monorepos provide: **shared libraries** (component libraries, utilities, data-access layers shared across apps without npm publishing), **consistent versioning** (all apps and libs on the same Angular version), **affected-only builds/tests** (`nx affected:test` only runs tests for code changed since the last commit — dramatically faster CI), and **enforced module boundaries** via lint rules. Other tools (Turborepo, Lerna) also support monorepos but Nx has the deepest Angular integration.

---

### E-Q2. What insights does bundle analysis provide in production optimization?

✅ **Bundle analysis identifies unused imports, quantifies library overhead, and detects duplicate dependencies to enable targeted optimization**

**Reasoning:** Tools like `webpack-bundle-analyzer` or `source-map-explorer` visualize the treemap of your production bundle. Key findings include: unexpectedly large dependencies (e.g., `moment.js` vs `date-fns`), duplicate packages bundled at multiple versions, entire libraries imported instead of specific functions, and dead code that tree-shaking missed. Acting on these insights can reduce bundle size by 30–70% in real-world apps.

---

### E-Q3. What is the core concept of component-based micro frontends at runtime?

✅ **Multiple independently-built Angular applications deployed as static JS files, composed at runtime via Module Federation**

**Reasoning:** True micro frontends are independently **built, deployed, and versioned** applications that are composed at runtime — not at build time. Webpack 5 Module Federation enables a shell app to load remote entry points from separately deployed apps at runtime. Each team owns their own build pipeline, release cycle, and tech choices. This is fundamentally different from shared component libraries, which are build-time dependencies.

---

### E-Q4. How do Angular dependency injection scopes (root, component, module) work?

✅ **Services injected in a component are destroyed when the component is destroyed; root-level services persist for the app lifetime**

**Reasoning:** Angular's DI hierarchy mirrors the component tree. `providedIn: 'root'` creates a **lazily instantiated singleton** for the entire app. Providing a service in a **component's `providers` array** creates a new instance scoped to that component subtree — destroyed when the component is destroyed. **Lazy-loaded modules** get their own injector child, so services provided there are scoped to that module. Understanding scope prevents subtle bugs like shared state between instances or premature service destruction.

---

### E-Q5. What are Angular Signals and how do they enable fine-grained reactivity?

✅ **Signals are a fine-grained reactivity primitive (Angular 16+) that auto-track dependencies and enable skipping change detection on unrelated branches**

**Reasoning:** Signals (`signal()`, `computed()`, `effect()`) introduce fine-grained reactivity to Angular. Unlike Zone.js-triggered CD (which checks the entire component tree), Signals track **exactly which components depend on which values**. When a signal changes, only components that read it are re-rendered. `computed()` memoizes derived values and only recalculates when dependencies change. Signals and RxJS are complementary — `toObservable()` and `toSignal()` bridge both worlds. Angular 17+ enables Zoneless mode using Signals.

---

### E-Q6. What are LCP and Core Web Vitals (CWV) and their relevance to Angular apps?

✅ **LCP and CWV are SEO/UX signals; LCP must be <2.5s and Cumulative Layout Shift <0.1 for a "good" rating**

**Reasoning:** Google's Core Web Vitals are:
✅ **LCP (Largest Contentful Paint)**: Time until the largest visible element loads. Target: **<2.5s**. Affected by SSR, lazy loading, image optimization.
✅ **INP (Interaction to Next Paint)**: Responsiveness of user interactions. Target: **<200ms**. Affected by long JS tasks, heavy change detection.
✅ **CLS (Cumulative Layout Shift)**: Visual stability. Target: **<0.1**. Affected by async-loaded images without dimensions, late-injected content.
CWV directly impacts Google Search ranking. Angular-specific fixes: SSR/prerendering, `@defer` blocks, image optimization directive (`NgOptimizedImage`).

---

### E-Q7. What are the most common causes of memory leaks in Angular SPAs?

✅ **Typical causes: retaining DOM nodes after destruction, uncanceled subscriptions, and long-lived listeners holding references to destroyed components**

**Reasoning:** The most common Angular memory leak patterns:
1. **Unsubscribed Observables** — a long-lived Observable (e.g., `interval()`, `WebSocket`) keeps a destroyed component in memory.
2. **Event listeners** — `renderer.listen()` or native `addEventListener` not removed in `ngOnDestroy`.
3. **Detached DOM nodes** — component destroyed but parent holds a reference.
4. **Closures in `setInterval`/`setTimeout`** — not cleared, capturing component references.
5. **RxJS `share()`/`shareReplay()`** without `refCount: true` — keeps subscription alive.
Angular does **not** automatically unsubscribe — `ngOnDestroy` must be implemented explicitly or `takeUntilDestroyed()` / `async` pipe used.

---

### E-Q8. How does tree-shaking work in Angular builds and what enables it?

✅ **Tree-shaking removes unused exports at build time when code is ES module-based and `sideEffects` metadata is correctly configured**

**Reasoning:** Tree-shaking is performed by the bundler (Webpack/esbuild) during production builds. It works by: (1) **static ES module analysis** — `import`/`export` are statically analyzable, unlike CommonJS `require()`, (2) **`sideEffects: false`** in `package.json` tells the bundler the package has no side effects and unused exports can be safely removed, (3) **`/*#__PURE__*/` annotations** mark function calls as pure (no side effects). CommonJS packages defeat tree-shaking — Angular's build warns about CommonJS imports for this reason.

---

### E-Q9. What architectural improvements does Ivy introduce over View Engine?

✅ **A) Ivy is View Engine's successor; it delivers smaller bundles, faster compilation, and better IDE integration via locality principle**

**Reasoning:** Ivy's key architectural improvements:
✅ **Locality principle**: Each component compiles independently without needing global knowledge of the app — enabling better incremental compilation.
✅ **Smaller bundles**: The runtime is tree-shakeable; unused framework features are excluded.
✅ **Faster compilation**: Incremental rebuilds are faster because only changed components recompile.
✅ **Better debugging**: `ng.getComponent()`, `ng.applyChanges()` available in DevTools console.
✅ **Improved type checking**: Stricter template type checking catches errors at build time.
✅ **Enables Signals/Zoneless**: Ivy's rendering architecture is the foundation for Signal-based fine-grained updates.

---

### E-Q10. What is the difference between lazy loading and code splitting?

✅ **Lazy loading (routing-based) loads a module on route navigation; code splitting (bundler-level) splits code into chunks defined by build config or dynamic imports**

**Reasoning:** **Lazy loading** is an Angular routing concept — `loadChildren` defers downloading a feature module's JS bundle until the user navigates to that route. **Code splitting** is a bundler concept — Webpack/esbuild splits the output into multiple `.js` chunks. Angular's lazy loading **triggers** code splitting automatically, but code splitting can also be configured independently via dynamic `import()` calls anywhere in code (e.g., dynamically importing a heavy charting library only when a chart is rendered). Both reduce the initial bundle (main.js) size.

---

### E-Q11. What is the ideal test strategy for scaling an Angular application?

✅ **A well-balanced pyramid: many unit tests (fast, isolated), fewer integration tests, even fewer e2e tests (full user flows)**

**Reasoning:** The **testing pyramid** for Angular:
✅ **Unit tests (base — most)**: Test components, services, pipes, and guards in isolation using `TestBed` or plain TS instantiation. Fast, deterministic. Tools: Jasmine/Jest.
✅ **Integration tests (middle)**: Test a component with its real child components and services (not mocked). Tools: Angular Testing Library, `TestBed` with `NO_ERRORS_SCHEMA` avoided.
✅ **E2E tests (top — fewest)**: Test full user flows in a real browser. Slow and brittle. Tools: Cypress, Playwright.
Avoid the "ice cream cone" anti-pattern (mostly E2E) — it leads to slow, flaky pipelines.

---

### E-Q12. What is the architectural relationship between `OnPush` change detection and unidirectional data flow?

✅ **`OnPush` enforces unidirectional data flow: data flows down via `@Input`, events flow up via `@Output`; parent controls all mutations**

**Reasoning:** `OnPush` and unidirectional data flow are architecturally coupled. Because `OnPush` only re-renders when `@Input` **references** change (not mutations), it forces immutable data patterns: instead of mutating an object, you replace it with a new reference. This creates a predictable, top-down data flow that is easier to debug (state changes are traceable) and enables memoization. It is the Angular equivalent of React's `React.memo` + immutable state philosophy.

---

### E-Q13. What is the Smart/Container vs Presentational component architecture pattern?

✅ **Smart components handle business logic and data fetching; presentational components receive data via `@Input` and emit actions via `@Output`**

**Reasoning:** This pattern (also called Container/Dumb components) separates concerns:
✅ **Smart (Container)**: Knows about services, state stores (NgRx), routing. Fetches data, handles business logic, passes data down. Should use `Default` CD.
✅ **Presentational (Dumb)**: Receives data exclusively via `@Input()`, communicates only via `@Output()`. No service dependencies. Ideal candidate for `OnPush` CD. Highly reusable and testable in isolation.
The pattern is not about stateful vs stateless — a presentational component can have local UI state (e.g., open/closed accordion) without being "smart."

---

### E-Q14. How should errors be handled within RxJS pipelines in Angular?

✅ **The `catchError` operator recovers from stream errors by returning a fallback Observable or rethrowing**

**Reasoning:** In RxJS, an unhandled error **terminates the stream** — no more emissions. `catchError` intercepts the error, allows logging/reporting, and returns either: a fallback `Observable` (e.g., `of([])` to emit empty data), `EMPTY` (complete without error), or `throwError(() => err)` to rethrow. For retrying transient failures, `retry(n)` or `retryWhen()` (or `retry({ count, delay })` in RxJS 7+) are used before `catchError`. Proper error handling keeps HTTP streams alive and prevents silent failures.

---

### E-Q15. What are best practices for safely unsubscribing from Observables in Angular components?

✅ **`takeUntil(destroy$)` pattern or `takeUntilDestroyed()` (Angular 16+) enables automatic cleanup without manual subscription tracking**

**Reasoning:** Multiple valid unsubscription strategies exist:
✅ **`async` pipe**: Best — subscribe and unsubscribe handled entirely by Angular. No code needed.
✅ **`takeUntilDestroyed()`** (Angular 16+): Most ergonomic for imperative subscriptions — automatically completes when the component/service is destroyed.
✅ **`takeUntil(this.destroy$)`**: Classic pattern — emit on `ngOnDestroy` to complete all piped streams.
✅ **`SubSink` library**: Collects subscriptions and disposes all at once.
✅ **Manual `unsubscribe()`**: Fine for a small number of subscriptions but doesn't scale.
Option D is partially correct but incomplete — `async` pipe is not the only way, and imperative subscriptions (e.g., inside `ngOnInit`) require explicit handling.

---

### E-Q16. What is hydration in the context of Angular Server-Side Rendering (SSR)?

✅ **Hydration is the process of attaching Angular event listeners and state to pre-rendered HTML from the server**

**Reasoning:** SSR renders the Angular app on the server and sends HTML to the browser for fast initial paint (good LCP). Without hydration, Angular then **destroys that HTML and re-renders** the entire app on the client — causing a flash and duplicate work. **Hydration** (Angular 16+ stable) instead reuses the server-rendered DOM nodes, attaches event listeners, and reconciles state without re-rendering, combining SSR's fast paint with Angular's full client-side interactivity. Angular 17 introduced **incremental hydration** via `@defer` blocks.

---

### E-Q17. How do CORS preflight requests work in Angular HTTP calls?

✅ **A CORS preflight (OPTIONS) request is sent automatically by the browser when certain conditions are met; the server must allow the origin in `Access-Control` headers**

**Reasoning:** CORS is enforced by the **browser**. A preflight `OPTIONS` request is triggered automatically when: the request uses methods other than GET/POST, has custom headers (e.g., `Authorization`), or uses a non-simple `Content-Type`. The server must respond with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`. Angular's `HttpClient` has no CORS control — it's a browser security mechanism. Common dev fix: use a proxy (`angular.json` `proxyConfig`) to avoid CORS during local development.

---

### E-Q18. What is `ViewContainerRef` used for in dynamic component loading?

✅ **`ViewContainerRef` is used to programmatically insert components into the DOM at runtime**

**Reasoning:** `ViewContainerRef` represents a container in the DOM where views (components or templates) can be dynamically inserted. Use cases: modals/dialogs rendered outside the component tree, dynamic dashboards where component types are determined at runtime, portal patterns. API: `vcr.createComponent(ComponentClass)` (Angular 13+ simplified API — no need for `ComponentFactoryResolver`). The inserted component participates fully in Angular's DI, CD, and lifecycle.

---

### E-Q19. What is Module Federation and its role in distributed Angular architecture?

✅ **Module Federation (webpack 5) enables independently-built Angular apps to share code and dependencies at runtime without version conflicts**

**Reasoning:** Webpack 5 Module Federation allows a **host (shell)** app to dynamically load **remote** entry points from separately deployed applications at runtime. Key capabilities: share common dependencies (Angular core, RxJS) across remotes to avoid duplicated bundles, independently deploy each remote without rebuilding the shell, and compose micro frontends at runtime. The `@angular-architects/module-federation` package provides Angular-specific schematics. It works with any build tool that supports it — not exclusively Nx.

---

### E-Q20. What does `fixture.detectChanges()` do in Angular component unit tests?

✅ **`fixture.detectChanges()` manually triggers change detection in `TestBed` component tests**

**Reasoning:** In `TestBed`, Angular does **not** automatically run change detection after every action — you must call `fixture.detectChanges()` explicitly. This gives tests precise control: set up state, then trigger CD to see the rendered result. First call also triggers `ngOnInit`. For async operations, use `fakeAsync` + `tick()` or `async` + `fixture.whenStable()`. `ComponentFixture.autoDetectChanges(true)` enables automatic detection but reduces test control. Understanding this is essential for writing reliable Angular component tests.

---

*Phase 3 Complete — 20 Expert/Architect-Level questions.*


---

## Phase 4: Scenario-Based Questions (SC-Q1–SC-Q20)

---

### SC-Q1. A user reports your Angular app becomes laggy after navigating through 5+ routes. How do you diagnose the root cause?

✅ **Profile the app using Chrome DevTools performance recorder; identify long tasks, frequent CD cycles, and memory leaks via the heap profiler**

**Reasoning:** Systematic diagnosis steps:
1. **Chrome DevTools → Performance tab**: Record a navigation session. Look for long tasks (>50ms), excessive "Recalculate Style", and layout thrashing.
2. **Angular DevTools**: Visualize component tree, identify which components trigger CD excessively — look for components without `OnPush`.
3. **Memory tab**: Take heap snapshots before and after navigation. Growing detached DOM nodes or retained component instances = memory leak from unsubscribed Observables.
4. **Network tab**: Check for redundant API calls caused by missing `shareReplay()` or duplicate subscriptions.
5. **Hypothesis**: Lag after 5+ routes suggests memory leak (components not destroyed) rather than pure rendering cost.

---

### SC-Q2. You have three independently-built Angular micro-apps on the same domain. They need to share authentication state. What is the recommended approach?

✅ **Store JWT in a shared auth library via Module Federation; apps subscribe to a global auth state stream and share the token via a common service or BroadcastChannel**

**Reasoning:** The recommended pattern:
1. **Single Sign-On (SSO)**: Centralize auth in the shell/host app or a dedicated auth micro-frontend.
2. **Shared auth library**: Published via Module Federation's `shared` config so all remotes use the same singleton `AuthService`.
3. **Token storage**: `localStorage` (accessible across same-origin tabs) or `sessionStorage` (per-tab). Avoid storing tokens in `localStorage` if XSS is a risk — use `httpOnly` cookies for sensitive apps.
4. **Cross-tab sync**: Use `BroadcastChannel` API or `storage` events to sync logout/login events across all open tabs.
5. Never share raw tokens via `window` globals — XSS exposure risk.

---

### SC-Q3. Your shared Angular component library has grown to 2.5 MB. Consumer apps are experiencing bundle bloat. How do you architect the library for optimal tree-shaking?

✅ **Split into feature-based secondary entry points and configure `sideEffects: false` in `package.json`**

**Reasoning:** Library optimization strategies:
1. **Secondary entry points**: Split `@mylib/buttons`, `@mylib/forms`, `@mylib/charts` so consumers only import what they use. Angular Package Format (APF) supports this via multiple `ng-package.json` files.
2. **`sideEffects: false`**: Tells bundlers the library has no side effects — unused exports can be safely dropped.
3. **Avoid barrel files** (`index.ts` re-exporting everything): They import entire modules into the consumer's bundle even if only one symbol is used.
4. **Peer dependencies**: Declare Angular, RxJS as `peerDependencies` (not `dependencies`) so they are not bundled into the library itself.
5. **ES2015+ output**: Ensure the library is published as ES modules (not CommonJS) to enable tree-shaking in consumer builds.

---

### SC-Q4. Your form requires async validation (check if a username already exists). You need to avoid excessive server calls and provide real-time feedback. What is your strategy?

✅ **Implement a custom async validator with `debounceTime` to throttle requests; show per-field error messages in real time**

**Reasoning:** Implementation pattern:
```typescript
// Custom async validator
usernameAvailable(): AsyncValidatorFn {
  return (control: AbstractControl) =>
    control.valueChanges.pipe(
      debounceTime(400),           // wait 400ms after last keystroke
      distinctUntilChanged(),      // skip if value hasn't changed
      switchMap(v => this.api.checkUsername(v)),  // cancel prior request
      map(taken => taken ? { usernameTaken: true } : null),
      first()                      // complete after one emission
    );
}
```
Key points: `debounceTime` reduces API calls, `switchMap` cancels in-flight requests, `distinctUntilChanged` skips duplicate values, `first()` completes the Observable so the form control settles. Display errors using `form.get('username').errors?.['usernameTaken']` in the template.

---

### SC-Q5. Your app has 10+ feature modules with complex, interconnected state. Simple service sharing leads to prop drilling chaos. How do you architect state management?

✅ **Use NgRx with normalized state shape, memoized selectors, and Effects for side effects**

**Reasoning:** NgRx state management architecture:
✅ **Store**: Single immutable state tree — all feature states combined via `StoreModule.forFeature()`.
✅ **Normalized shape**: Entities stored as `{ ids: [], entities: {} }` (like a DB table) using `@ngrx/entity` — O(1) lookups, no array scanning.
✅ **Selectors**: `createSelector()` memoizes derived data — only recomputes when input selectors change. Prevents unnecessary re-renders.
✅ **Effects**: Handle async operations (HTTP, WebSocket) outside components — components dispatch actions, Effects handle the side effects and dispatch result actions.
✅ **DevTools**: Time-travel debugging via Redux DevTools.
Alternative for lighter use cases: **NgRx ComponentStore** (local state) or **Signals-based stores** (Angular 17+).

---

### SC-Q6. A user types in a live search box. Each keystroke triggers an API call. Old responses can arrive after newer ones. How do you prevent stale data?

✅ **Use `switchMap` to cancel in-flight requests when a new keystroke arrives**

**Reasoning:** Implementation:
```typescript
this.searchControl.valueChanges.pipe(
  debounceTime(300),          // wait for typing pause
  distinctUntilChanged(),     // skip unchanged values
  filter(term => term.length >= 2),
  switchMap(term =>           // cancels previous HTTP call
    this.api.search(term).pipe(
      catchError(() => of([]))  // handle errors gracefully
    )
  )
).subscribe(results => this.results = results);
```
`switchMap` unsubscribes from (and the browser cancels via `AbortController`) the previous HTTP call when a new value arrives. `mergeMap` would allow all concurrent requests, meaning a slow early request could overwrite fast later results — the classic race condition. JavaScript being single-threaded doesn't prevent HTTP responses from arriving out of order.

---

### SC-Q7. Module A imports `ServiceX` from Module B, and Module B imports `ServiceY` from Module A. What is the architectural pattern to resolve this cycle?

✅ **Create a shared `CoreModule` or facade service as a dependency-free intermediary that both modules import**

**Reasoning:** The **facade pattern** breaks circular dependencies:
1. Create a `SharedModule` or `CoreModule` that neither Module A nor Module B depends on.
2. Move `ServiceX` and `ServiceY` into the shared module, or extract the shared logic into a new `SharedService`.
3. Both Module A and Module B now import only from `SharedModule` — no cycle.
Alternative: **Event-driven decoupling** — Module A and Module B communicate via a shared `EventBusService` (using `Subject`) in `CoreModule`, eliminating direct service dependencies between modules. This is the correct architectural fix; simply merging modules trades one problem for a monolith.

---

### SC-Q8. Two users update the same field concurrently. How do you handle client-server state reconciliation in your Angular app?

✅ **Implement optimistic updates: reflect the action immediately in UI, then sync; on conflict rollback and display an error**

**Reasoning:** Optimistic update pattern with NgRx:
1. **Dispatch action** → **Reducer** applies the change to store immediately (user sees instant feedback).
2. **Effect** sends the HTTP request to the server.
3. **On success**: Dispatch a success action to confirm state.
4. **On conflict (409)**: Dispatch a failure action → Reducer **rolls back** to previous state → display conflict error with the server's current value.
5. For real-time sync: **WebSocket** or **Server-Sent Events** push server state changes to all connected clients. Use `last-write-wins` or **operational transformation** (like Google Docs) for complex collaborative editing.

---

### SC-Q9. You have multiple incomplete feature branches that must not reach production users but must stay in the codebase. How do you manage this?

✅ **Wrap incomplete features in a `FeatureFlagService`; disable flags in production environment config**

**Reasoning:** Feature flag architecture:
```typescript
@Injectable({ providedIn: 'root' })
export class FeatureFlagService {
  private flags = inject(APP_CONFIG).featureFlags;
  isEnabled(flag: string): boolean {
    return this.flags[flag] ?? false;
  }
}
```
In templates: `*ngIf="featureFlags.isEnabled('new-checkout')"`. Flags are loaded from environment config or a remote service (LaunchDarkly, ConfigCat). Benefits: code stays in `main` (no long-lived branches → no merge conflicts), features can be toggled per environment or per user segment, incomplete code is never deleted (reduces risk). For complete isolation, combine with lazy-loaded modules — the module bundle isn't even downloaded if the flag is off.

---

### SC-Q10. You need to break backward compatibility in a shared library used by 15+ apps. What is the rollout strategy?

✅ **Maintain both API versions simultaneously; deprecate the old over 2–3 releases; provide migration guides and automated codemods**

**Reasoning:** Semantic versioning + deprecation lifecycle:
1. **Version bump**: Release as a new **major version** (`v3.0.0`) per semver — signals breaking change.
2. **Dual API period**: In `v2.x`, mark old APIs as `@deprecated` (JSDoc + `console.warn`). New API available alongside.
3. **Automated migration**: Provide an `ng update` schematic (`ng-update` in `package.json`) that automatically rewrites consumer code — similar to Angular's own migration schematics.
4. **Migration guide**: Document what changed, why, and how to migrate manually for edge cases.
5. **Sunset timeline**: Announce removal in `v3.0.0` release notes. Give teams at least 2 sprint cycles.
6. **Communicate via changelog**, Slack announcement, and a recorded demo for larger teams.

---

### SC-Q11. Your app must support 20 languages and regional variants with fast load times. How do you architect i18n?

✅ **Use `@angular/localize` with i18n extraction, locale-specific builds, and lazy-loaded translation files to avoid loading all locales upfront**

**Reasoning:** Scalable i18n architecture:
1. **`@angular/localize`**: Mark strings with `$localize` or `i18n` attributes. Run `ng extract-i18n` to generate XLIFF/XMB translation files.
2. **Build-time i18n**: `ng build --localize` generates one optimized bundle per locale — strings are inlined at build time (no runtime translation overhead). Deploy each locale to its own path (`/en/`, `/fr/`, `/de/`).
3. **Runtime i18n** (for 20+ locales where build-time is slow): Use a library like `@ngx-translate/core` with lazy-loaded JSON translation files — only the user's locale is downloaded.
4. **CDN routing**: Use a CDN or edge worker to route users to their locale bundle based on `Accept-Language` header.
5. **Pluralization & date/currency**: Use Angular's built-in `DatePipe`, `CurrencyPipe`, `DecimalPipe` — they are locale-aware automatically.

---

### SC-Q12. You need to render user-submitted rich HTML content. How do you prevent XSS while safely displaying formatted text?

✅ **Use `[innerHTML]` with `DomSanitizer.sanitize()` or prefer property binding and interpolation which are auto-escaped by default**

**Reasoning:** Angular's XSS protection layers:
1. **Interpolation `{{ }}`** and **property binding `[prop]`**: Angular auto-escapes HTML — safe by default. Use these wherever possible.
2. **`[innerHTML]`**: Bypasses Angular's escaping. Angular applies **automatic sanitization** that strips dangerous tags (`<script>`, event handlers). Still risks if `bypassSecurityTrustHtml()` is misused.
3. **`DomSanitizer.sanitize(SecurityContext.HTML, value)`**: Explicitly sanitizes before binding — preferred over `bypassSecurityTrustHtml()`.
4. **Content Security Policy (CSP)**: Server-level defense — use `Content-Security-Policy` headers to block inline scripts even if XSS injection occurs.
5. **Never use `bypassSecurityTrustHtml()`** on unsanitized user input — it explicitly disables Angular's protection.

---

### SC-Q13. A production bug affects only 2% of users and cannot be reproduced locally. How do you debug it?

✅ **Use source maps + an error tracking service (Sentry/Datadog); analyse stack traces, user sessions, and browser/device metadata to isolate the condition**

**Reasoning:** Production debugging playbook:
1. **Error tracking**: Integrate Sentry, Datadog, or Bugsnag — captures stack traces, user context, browser version, and custom breadcrumbs automatically.
2. **Source maps**: Upload source maps to the error tracking service (not publicly served). Stack traces show original TypeScript line numbers, not minified output.
3. **Session replay**: Tools like LogRocket or Sentry Session Replay record exact user interactions leading to the error.
4. **Feature flags**: If the bug correlates with a new feature, disable it for affected users instantly.
5. **Canary analysis**: Check if the 2% correlates with a specific browser version, OS, screen size, or locale.
6. **Angular DevTools** in production: `enableProdMode()` is set but source maps let you inspect component state if DevTools is enabled.

---

### SC-Q14. Your team targets both modern and legacy browsers (old Safari, some IE11 scenarios). How do you ensure compatibility?

✅ **Use BrowserStack/Sauce Labs for automated cross-browser tests; maintain `.browserslistrc` to define targets and drive differential builds**

**Reasoning:** Cross-browser compatibility strategy:
1. **`.browserslistrc`**: Defines target browsers — shared by Angular CLI (for polyfills), Autoprefixer (for CSS), and Babel (for JS transpilation). Example: `last 2 Chrome versions, last 2 Firefox versions, Safari >= 14`.
2. **Differential loading**: Angular CLI generates ES5 bundles (for legacy) and ES2015+ bundles (for modern) — modern browsers download the smaller ES2015 bundle.
3. **Polyfills**: `zone.js` and targeted polyfills added via `polyfills.ts` based on `.browserslistrc`.
4. **Automated cross-browser testing**: BrowserStack Automate or Sauce Labs runs your Cypress/Playwright suite across real browsers/devices in CI.
5. **Lighthouse CI**: Run accessibility and compatibility checks on every PR.

---

### SC-Q15. Your CEO asks: "Can our frontend handle 100k concurrent users?" How do you answer with data?

✅ **Define performance budgets; use Lighthouse CI + WebPageTest; simulate real-user load; measure TTFB, LCP, and INP under realistic conditions**

**Reasoning:** Frontend capacity planning framework:
1. **Clarify the question**: "100k concurrent" for a frontend SPA is primarily a CDN/static asset delivery problem, not compute. The real question is: does the app stay performant under load?
2. **Performance budgets**: Set via `angular.json` `budgets` — fail the build if bundle size exceeds thresholds.
3. **Synthetic monitoring**: Lighthouse CI runs on every deploy; WebPageTest measures real network conditions.
4. **Real User Monitoring (RUM)**: Tools like SpeedCurve or Datadog RUM measure actual user experience (LCP, INP, CLS) in production.
5. **CDN load testing**: Use `k6`, `Artillery`, or AWS CloudFront's built-in analytics to test asset delivery at scale.
6. **Backend bottleneck**: If 100k users trigger 100k API calls simultaneously, the Angular app is fine but the backend will be the bottleneck — measure API response times under load with `k6` or Gatling.

---

### SC-Q16. A data grid must render 50,000 rows smoothly with server-side filtering and sorting. How do you architect this?

✅ **Implement CDK Virtual Scroll with server-side pagination; apply `OnPush` to row components; only render visible rows**

**Reasoning:** High-performance data grid architecture:
1. **CDK Virtual Scroll (`cdk-virtual-scroll-viewport`)**: Renders only the DOM nodes visible in the viewport — typically 20–40 rows regardless of total dataset size.
2. **Server-side pagination**: Fetch data in pages (e.g., 100 rows per request) as the user scrolls — never load all 50,000 rows at once.
3. **`OnPush` on row components**: Rows only re-render when their `@Input` data reference changes — critical for scroll performance.
4. **`trackBy`**: Essential — prevents row DOM nodes from being recreated on each page fetch.
5. **Debounce filter/sort**: `debounceTime(300)` on filter inputs to avoid triggering server requests on every keystroke.
6. **Web Workers** (for client-side sorting of large datasets): Offload heavy computation off the main thread via `@angular/web-workers`.

---

### SC-Q17. Eight product teams share your Angular design system. Divergence is growing rapidly. How do you govern and scale it?

✅ **Publish a versioned library with Storybook for documentation, strict semver, and a dedicated design system team as owners**

**Reasoning:** Design system governance at scale:
1. **Dedicated ownership**: A design system team (designers + engineers) owns and maintains the library — not spread across product teams.
2. **Storybook**: Living documentation — every component has interactive examples, prop documentation, and accessibility checks. Teams adopt confidently because they can see behavior before installing.
3. **Semver + CHANGELOG**: Breaking changes are major versions with migration guides. Minor/patch versions are safe to auto-update.
4. **Contribution model**: Product teams submit PRs; design system team reviews for consistency. Prevents ad-hoc "just make it work" additions.
5. **Automated visual regression**: Chromatic (Storybook's CI integration) or Percy detects unintended visual changes in every PR.
6. **Design tokens**: Shared CSS custom properties or Sass variables ensure colors, spacing, and typography stay consistent across all consuming apps.

---

### SC-Q18. You need to release a risky new checkout flow to 5% of users first without separate builds. How do you implement a canary release?

✅ **Use server-driven feature flags (LaunchDarkly/ConfigCat) consumed by `FeatureFlagService`; route the canary % via infrastructure (CDN/load balancer) without separate app builds**

**Reasoning:** Canary release architecture:
1. **Server-driven flags**: Flags are evaluated server-side and served to the client via an API call on app init (`APP_INITIALIZER`). The server controls which users (by user ID hash, geography, account type) see the new feature.
2. **`FeatureFlagService`**: Consumes the flag API; components check `flagService.isEnabled('new-checkout')`. No rebuild required — the code ships but is gated.
3. **Infrastructure-level canary**: Route 5% of traffic to a new deployment via load balancer (AWS ALB weighted routing, Kubernetes canary deployments) — completely independent of feature flags.
4. **Monitoring**: Track conversion rate, error rate, and performance metrics separately for canary vs control cohort using analytics segmentation.
5. **`environment.ts` flags** are build-time only — they require a rebuild per flag change and cannot be toggled in production without redeployment. Not suitable for canary releases.

---

### SC-Q19. You're building a trading dashboard that must update 50 charts per second with live market data. How do you architect this without freezing the UI?

✅ **Use WebSockets for push; pipe through RxJS; apply `OnPush` + `async` pipe per chart; debounce burst updates and run heavy computation in Web Workers**

**Reasoning:** Real-time high-frequency update architecture:
1. **WebSocket**: Single persistent connection pushes updates — far more efficient than polling (no HTTP overhead per update).
2. **RxJS stream per symbol**: `webSocket$` → `filter(msg => msg.symbol === this.symbol)` → `share()` — each chart component subscribes to its own filtered stream.
3. **`OnPush` + `async` pipe**: Chart components only re-render when their specific symbol's Observable emits. Unrelated chart updates don't trigger CD.
4. **`auditTime(16)` or `throttleTime(16)`**: Limit updates to ~60fps (one frame = 16ms). Prevents rendering more frames than the display can show.
5. **Web Workers**: Heavy calculations (e.g., moving averages, candlestick aggregation) run off the main thread — prevents UI jank.
6. **`NgZone.runOutsideAngular()`**: Process high-frequency WebSocket messages outside Angular's zone; only re-enter the zone when updating the component view.

---

### SC-Q20. Concurrent API calls return 401 simultaneously after a JWT expires. How do you handle token refresh without triggering multiple refresh calls?

✅ **Intercept 401 responses; queue concurrent requests; refresh the token once using a shared Observable; replay all queued requests after refresh**

**Reasoning:** Token refresh interceptor pattern:
```typescript
// In HttpInterceptor
let isRefreshing = false;
let refreshToken$: Observable<string> | null = null;

intercept(req, next) {
  return next.handle(req).pipe(
    catchError(err => {
      if (err.status !== 401) return throwError(() => err);

      if (!isRefreshing) {
        isRefreshing = true;
        refreshToken$ = this.auth.refresh().pipe(
          shareReplay(1),  // all concurrent callers share ONE refresh call
          tap(() => isRefreshing = false),
          catchError(e => { isRefreshing = false; return throwError(() => e); })
        );
      }

      return refreshToken$.pipe(
        switchMap(token => next.handle(addToken(req, token)))
      );
    })
  );
}
```
Key: `shareReplay(1)` ensures only **one** refresh HTTP call is made even if 10 requests hit 401 simultaneously. All 10 requests share the same refresh Observable and replay the new token. After receiving the new token, all queued requests are retried with `switchMap`. If refresh fails (refresh token expired), redirect to login.

**Simple Explanation:**

**The Problem:** Your app makes 10 API calls at the same time. Your JWT token has just expired. All 10 calls come back with a `401 Unauthorized` error simultaneously. The naive fix would trigger 10 separate token refresh calls — wasteful and potentially broken.

**The Solution: Refresh Once, Share With Everyone**

1. **First 401 arrives** → set `isRefreshing = true`, start the refresh HTTP call, store it in a shared `refreshToken$` Observable
2. **401s #2–10 arrive** → `isRefreshing` is already `true`, so they skip starting a new refresh and just subscribe to the same `refreshToken$`
3. **Refresh completes** → `shareReplay(1)` replays the new token to all 10 waiting subscribers at once
4. **All 10 original requests retry** with the new token via `switchMap`
5. **If refresh fails** → everyone gets the error, redirect to login

**Why `shareReplay(1)` is the Hero:**

```
Without shareReplay:  10 requests × 1 refresh each = 10 refresh HTTP calls ❌
With shareReplay(1):  10 requests × 1 shared refresh = 1 refresh HTTP call ✅
```

`shareReplay(1)` makes the Observable "hot" and caches the last emission — so any late subscriber (request #8, #9, #10) still gets the token even if they subscribe after the refresh already completed.

**The `isRefreshing` Flag** acts as a simple lock: `false` means you're the first 401, start the refresh; `true` means a refresh is already in progress, just wait for `refreshToken$`. This prevents the race condition where two requests both see `isRefreshing = false` and both try to start a refresh.

---

*Phase 4 Complete — 20 Scenario-Based questions.*


