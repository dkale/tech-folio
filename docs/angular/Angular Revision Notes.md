# Angular Revision Notes
A consolidated, topic-organized reference covering Angular fundamentals, architecture, and common interview questions.

## Table of Contents

1. [Change Detection & Rendering Performance](#change-detection-rendering-performance)
   - [Angular Change Detection](#angular-change-detection)
   - [How does change detection work in Angular?](#how-does-change-detection-work-in-angular)
   - [What strategies can be used to optimize change detection performance?](#what-strategies-can-be-used-to-optimize-change-detection-performance)
   - [What is trackBy in Angular?](#what-is-trackby-in-angular)
   - [OnPush Change Detection: Mutation Scenario](#onpush-change-detection-mutation-scenario)
   - [How does a browser render a page?](#how-does-a-browser-render-a-page)
2. [Components, Directives & Architecture](#components-directives-architecture)
   - [Directives](#directives)
   - [Decorators](#decorators)
   - [View Encapsulation](#view-encapsulation)
   - [ShadowDOM vs Virtual DOM](#shadowdom-vs-virtual-dom)
   - [Web Components](#web-components)
   - [Content Projection](#content-projection)
   - [Explain the ‘Smart-Dumb’ / ‘Container-Presenter’ component architecture pattern in Angular](#explain-the-smart-dumb-container-presenter-component-architecture-pattern-in-angular)
   - [How does Angular implement a component-based architecture, and what are the benefits?](#how-does-angular-implement-a-component-based-architecture-and-what-are-the-benefits)
   - [Describe the purpose and use of Angular modules](#describe-the-purpose-and-use-of-angular-modules)
   - [How do you organize modules in a large Angular application to ensure maintainability and scalability?](#how-do-you-organize-modules-in-a-large-angular-application-to-ensure-maintainability-and-scalability)
   - [Component Inheritance in Angular](#component-inheritance-in-angular)
   - [Dynamic Component Loading](#dynamic-component-loading)
3. [Async Programming: Promises, Observables & RxJS](#async-programming-promises-observables-rxjs)
   - [Promises, Observables, Subject, BehaviorSubject, EventEmitter](#promises-observables-subject-behaviorsubject-eventemitter)
   - [How to decide when to use a Promise or an Observable? When do we need to use Promise in an Angular application?](#how-to-decide-when-to-use-a-promise-or-an-observable-when-do-we-need-to-use-promise-in-an-angular-application)
   - [RxJS Operators](#rxjs-operators)
   - [Does Cancelling an Observable Cancel the Server-Side API Request?](#does-cancelling-an-observable-cancel-the-server-side-api-request)
   - [Can an Observable Have Multiple Subscribers?](#can-an-observable-have-multiple-subscribers)
   - [How Long Does shareReplay Cache?](#how-long-does-sharereplay-cache)
   - [Unsubscribing from Multiple Subscriptions](#unsubscribing-from-multiple-subscriptions)
   - [RxJS: Higher-Order Mapping Operators](#rxjs-higher-order-mapping-operators)
   - [RxJS: Error Handling Operators](#rxjs-error-handling-operators)
   - [RxJS: Combining Observables](#rxjs-combining-observables)
4. [Dependency Injection & Services](#dependency-injection-services)
   - [Dependency Injection, Angular Injectors, and Providers](#dependency-injection-angular-injectors-and-providers)
   - [Dependency Provider & Dependency Injector](#dependency-provider-dependency-injector)
   - [What are the advantages of using dependency injection in Angular?](#what-are-the-advantages-of-using-dependency-injection-in-angular)
   - [What are Angular services, and why are they used?](#what-are-angular-services-and-why-are-they-used)
   - [How would you create a singleton service in Angular, and what are the potential pitfalls?](#how-would-you-create-a-singleton-service-in-angular-and-what-are-the-potential-pitfalls)
   - [DI Scope: Root vs Component-Level Providers](#di-scope-root-vs-component-level-providers)
5. [Forms](#forms)
   - [Angular Forms](#angular-forms)
   - [Template Driven Forms vs. Reactive Forms](#template-driven-forms-vs-reactive-forms)
   - [How would you handle form validation and error handling in Angular?](#how-would-you-handle-form-validation-and-error-handling-in-angular)
6. [Routing & Navigation](#routing-navigation)
   - [Angular Route Guards](#angular-route-guards)
   - [Angular Routing](#angular-routing)
   - [Discuss lazy loading and its benefits in the context of Angular routing](#discuss-lazy-loading-and-its-benefits-in-the-context-of-angular-routing)
7. [Component Communication & Lifecycle](#component-communication-lifecycle)
   - [Sharing data between Angular Components: Four Methods](#sharing-data-between-angular-components-four-methods)
   - [Difference between ngOnInit() and constructor()](#difference-between-ngoninit-and-constructor)
   - [Angular Lifecycle Hooks](#angular-lifecycle-hooks)
   - [DOM Manipulation with ngAfterViewInit](#dom-manipulation-with-ngafterviewinit)
   - [Preventing Component Initialization with CanActivate](#preventing-component-initialization-with-canactivate)
8. [Pipes](#pipes)
   - [Angular Pipes](#angular-pipes)
   - [What are Pipes? Pure vs Impure Pipes](#what-are-pipes-pure-vs-impure-pipes)
   - [Custom Pipe Example: Smart Number Formatter](#custom-pipe-example-smart-number-formatter)
9. [HTTP, Security & Authentication](#http-security-authentication)
   - [Angular Interceptors](#angular-interceptors)
   - [JSON Web Token (JWT) Authentication](#json-web-token-jwt-authentication)
   - [Server-side & Client-side Communication (CORS)](#server-side-client-side-communication-cors)
   - [What security measures should be taken into account when developing an Angular application?](#what-security-measures-should-be-taken-into-account-when-developing-an-angular-application)
10. [State Management](#state-management)
   - [Discuss the role of state management in Angular applications](#discuss-the-role-of-state-management-in-angular-applications)
   - [Explain the concepts of store, actions, reducers, and effects in context of NgRx state management library](#explain-the-concepts-of-store-actions-reducers-and-effects-in-context-of-ngrx-state-management-library)
11. [Build & Performance Optimization](#build-performance-optimization)
   - [AOT (Ahead of Time) vs JIT (Just in Time) Compilation](#aot-ahead-of-time-vs-jit-just-in-time-compilation)
   - [Discuss the benefits of Ahead-Of-Time (AOT) compilation](#discuss-the-benefits-of-ahead-of-time-aot-compilation)
   - [What strategies can be employed to optimize the performance of an Angular application?](#what-strategies-can-be-employed-to-optimize-the-performance-of-an-angular-application)
12. [Testing](#testing)
   - [How do you mock service dependencies in an Angular service while implementing unit tests?](#how-do-you-mock-service-dependencies-in-an-angular-service-while-implementing-unit-tests)
   - [How do you write unit tests for Angular components and services?](#how-do-you-write-unit-tests-for-angular-components-and-services)
   - [Explain the use of testing utilities such as TestBed](#explain-the-use-of-testing-utilities-such-as-testbed)
   - [Testing Component Lifecycle Hooks](#testing-component-lifecycle-hooks)
13. [Internationalization & SEO](#internationalization-seo)
   - [How can you implement internationalization in an Angular application?](#how-can-you-implement-internationalization-in-an-angular-application)
   - [How can you optimize an Angular application for search engine optimization (SEO)?](#how-can-you-optimize-an-angular-application-for-search-engine-optimization-seo)
14. [Modern Angular: Signals](#modern-angular-signals)
   - [What are Angular Signals?](#what-are-angular-signals)
15. [Interview Prep Reference & Resources](#interview-prep-reference-resources)
   - [Angular Scenario-Based Interview Questions](#angular-scenario-based-interview-questions)
   - [Angular System Design Interview Questions — Topic Map](#angular-system-design-interview-questions-topic-map)
   - [Things You Need To Learn To Become An Effective Angular Developer](#things-you-need-to-learn-to-become-an-effective-angular-developer)

---

## Change Detection & Rendering Performance

### Angular Change Detection

**Change detection** is the mechanism Angular uses to synchronize the component data model with the DOM — determining what needs to re-render and when.

---

**How Angular Detects Changes: Zone.js**

Angular uses **Zone.js**, a library that monkey-patches all asynchronous browser APIs (`setTimeout`, `setInterval`, `addEventListener`, HTTP requests, etc.). This allows Angular to intercept async operations and schedule a change detection cycle when they complete.

There are two zones:

| Zone | Also called | Triggers CD? |
| :--- | :--- | :--- |
| **Outer Zone** | Parent zone | No |
| **Inner Zone** | Angular zone (`NgZone`) | Yes — on every subscribed event |

Any async operation running inside the Angular zone (`setTimeout`, `setInterval`, `addEventListener`) will automatically trigger change detection.

---

**The Two Change Detection Strategies**

**1. Default (CheckAlways)**

Angular traverses the entire component tree top-to-bottom and calls `refreshView()` on every component on every CD cycle, regardless of whether their inputs changed. Simple but expensive for large trees.

```typescript
@Component({
  selector: 'app-example',
  // changeDetection: ChangeDetectionStrategy.Default is implicit
})
export class ExampleComponent {}
```

**2. OnPush (CheckOnce until dirty)**

Angular skips a component and all its children *unless the component is marked as dirty*. This dramatically reduces the number of checks in large applications.

```typescript
@Component({
  selector: 'app-example',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ExampleComponent {}
```

---

**What Marks a View as Dirty? (OnPush triggers)**

With `OnPush`, a component only re-checks when its view is marked dirty via one of these four mechanisms:

| Trigger | What happens |
| :--- | :--- |
| **1. Event binding** `(click)="handler()"` | Marks the component and all ancestors dirty; CD runs top-to-bottom to that component |
| **2. Async pipe** `data$ \| async` | When the observable emits, `markForCheck()` is called internally, marking the view and ancestors dirty |
| **3. `cdRef.markForCheck()`** | Explicitly marks the view and all its ancestors dirty; use when data changes outside Angular's awareness |
| **4. New `@Input()` reference** | Passing a new object reference (immutable update) marks the view dirty — mutating the same object does NOT |

> **Key rule with OnPush:** Mutating an existing object or array in-place will NOT trigger re-rendering. You must produce a new reference (e.g. spread operator, `Object.assign`, or immutable libraries).

```typescript
// Will NOT trigger OnPush change detection:
this.items.push(newItem);          // mutates existing array

// WILL trigger OnPush change detection:
this.items = [...this.items, newItem]; // new array reference
```
### How does change detection work in Angular?

Change detection is a critical aspect of Angular’s architecture that ensures the synchronization between the application state and the DOM (Document Object Model). It is the mechanism by which Angular determines which parts of the view need to be updated when there are changes in the application state.

Here is an overview of how change detection works in Angular:

1. **Initialisation:**  
   * When an Angular application is bootstrapped, the framework creates a tree of components that represent the application structure.  
   * Each component is associated with a view, and Angular initializes the view and sets up the initial bindings.  
2. **Component Lifecycle Hooks:**  
   * Angular components go through a series of lifecycle hooks during their creation, rendering, and destruction phases.  
   * The ‘**ngOnInit**’ lifecycle hook is commonly used to perform initialization logic.  
3. **Data Binding:**  
   * Angular applications typically use two-way data binding, where the model (component properties) is bound to the view.  
   * Any change in the model triggers the change detection mechanism.  
4. **Zones:**  
   * Angular leverages Zones, a concept from the **Zone.js** library, to intercept asynchronous operations and trigger change detection.  
   * Zones allow Angular to capture and handle asynchronous events, such as HTTP requests or timers, by executing a callback within a zone.  
5. **Triggers for Change Detection:**  
   * Change detection is triggered by various events, such as user interaction, timers, HTTP requests, or other asynchronous tasks.  
   * Angular listens for these events and schedules change detection cycles.  
6. **Change Detection Strategies:**  
   * Angular supports two change detection strategies: **Default** and **OnPush**.  
   * In the default strategy, change detection is only performed when input properties of a component change or when an event is emitted from that component.  
   * With OnPush, a component will only be checked for changes if its input properties reference a new object. This promotes immutability and can optimize performance.

```typescript
// Example of using OnPush change detection strategy
@Component({
  selector: 'app-example',
  templateUrl: 'example.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ExampleComponent {}
```

     

7. **Checking Components and their children:**  
   * During a change detection cycle, Angular checks each component and its children to determine if any values have changed.  
   * This process is recursive, going through the component tree until no more changes are detected.  
8. **Marking Components for Check:**  
   * Angular uses a mechanism to mark components as needing check, and during the change detection cycle, it checks only those components marked for check.  
   * The markForCheck() method can be used to explicitly mark a component and its ancestors for check.  
9. **Immutable Data and Pure Pipes:**  
   * The use of immutable data structures and pure pipes can improve change detection performance.  
   * Immutable data ensures that changes result in new object references, and pure pipes cache the result for a given set of inputs.

```typescript
// Example of using a pure pipe
@Pipe({
  name: 'myPurePipe',
  pure: true,
})
export class MyPurePipe implements PipeTransform {
  transform(value: any): any {
    // Pipe logic
  }
}
```

10. **NgZone Service:**  
    * The NgZone service allows developers to explicitly run code outside or inside Angular’s zone.  
    * This is useful for managing asynchronous tasks and explicitly triggering or suppressing change detection.

```typescript
constructor(private ngZone: NgZone) {}
runOutsideAngular() {
  this.ngZone.runOutsideAngular(() => {
    // Code to run outside Angular's zone
  });
}
```

      

In summary, change detection in Angular is a dynamic process that ensures the synchronization of the application state with the DOM. It leverages Zones, follows a strategy for detecting changes, and checks components recursively, Developers can optimize change detection performance by using strategies like OnPush, marking components for check, and employing best practices such as immutable data and pure pipes.

### What strategies can be used to optimize change detection performance?

Optimizing change detection performance is crucial for ensuring that Angular applications run smoothly, especially as the complexity of the application grows. Here are several strategies to optimize change detection performance in Angular:

1. **Change Detection Strategy:**  
   * Choose the appropriate change detection strategy for your components. Angular provides two main strategies: Default and OnPush.  
   * Default strategy checks all components on every asynchronous event.  
   * OnPush strategy checks a component only if its input properties change or if an event is emitted from that component.

```typescript
// Example of using OnPush change detection strategy
@Component({
  selector: 'app-example',
  templateUrl: 'example.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ExampleComponent {}
```

2. **Immutable Data:**  
   * Use immutable data structures to ensure that changes result in new object references. This helps Angular efficiently determine if changes have occurred.  
   * Libraries like Immutable.js can assist in working with immutable data structures.

3. **Pure Pipes:**  
   * Utilize pure pipes for data transformations. Pure pipes only recalculate their output when their input values change.  
   * Be cautious when using impure pipes, as they can lead to more frequent and unnecessary recalculations.

```typescript
// Example of using a pure pipe
@Pipe({
  name: 'myPurePipe',
  pure: true,
})
export class MyPurePipe implements PipeTransform {
  transform(value: any): any {
    // Pipe logic
  }
}
```

   

4. **OnPush and Inputs:**  
   * Combine the OnPush change detection strategy with the use of @Input() properties. This ensures that the component is only checked when its input properties change.

```typescript
@Component({
  selector: 'app-onpush-component',
  templateUrl: 'onpush-component.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OnPushComponent {
  @Input() data: any;
}
```

5. **Detaching Change Detector:**  
   * Manually detach and reattach the change detector when working with components that don't need frequent updates. This prevents unnecessary checks.

```typescript
constructor(private cdr: ChangeDetectorRef) {}
ngOnInit() {
  this.cdr.detach();
}
updateData() {
  // Perform update logic
  this.cdr.reattach();
}
```

     

6. **AsyncPipe for Observables:**  
   When working with Observables, use the **AsyncPipe** in the template. The **AsyncPipe** automatically subscribes and unsubscribes, reducing the need for manual management.

```html
<div>{{ observableData$ | async }}</div>
```

   

7. **NgZone Service:**  
   * Use the NgZone service to explicitly run code outside or inside Angular's zone.  
   * Running certain tasks outside the zone can prevent unnecessary change detection.

```typescript
constructor(private ngZone: NgZone) {}
runOutsideAngular() {
  this.ngZone.runOutsideAngular(() => {
    // Code to run outside Angular's zone
  });
}
```

8. **Avoid Heavy Computations in Templates:**  
   * Minimize heavy computations in Angular templates. Complex expressions can lead to increased computation during change detection.  
   * Move complex computations to the component class or use helper functions.

   

9. **Lazy Loading:**  
   * Consider lazy loading modules to load components only when needed. This helps reduce the initial payload and improves application startup time.

   

10. **Profile and Optimize:**  
    * Profile your application using tools like Chrome DevTools to identify performance bottlenecks.  
    * Optimize specific areas based on profiling results, such as reducing the number of bindings or improving the efficiency of change detection cycles.

By applying these strategies judiciously, developers can significantly optimize change detection performance in Angular applications, resulting in faster and more responsive user interfaces.

### What is trackBy in Angular?

In Angular, the ‘**trackBy**’ function is used in conjunction with ‘ngFor’ to improve the efficiency of rendering lists or collections of items in the Angular template. The ‘ngFor’ directive is commonly used to iterate over a list and generate a set of elements based on the data provided. When dealing with dynamic lists, using ‘trackBy’ can optimize the rendering process.

Here is how it works:

1. **Basic Usage of ngFor without ‘trackBy’:**

```html
<ul>
  <li *ngFor="let item of items">{{ item.name }}</li>
</ul>
```

   In this example, Angular uses the default tracking mechanism based on the item’s index to determine which elements in the list have changed.

2. **Usage of ‘trackBy’ with a Function:**

```html
<ul>
  <li *ngFor="let item of items; trackBy: trackByFunction">{{ item.name }}</li>
</ul>
```

     
   In the component class, define the **trackByFunction** method:

```typescript
trackByFunction(index: number, item: any): number {
  return item.id; // Use a unique identifier (e.g., item's id) for more efficient tracking
}
```

   

   By providing a ‘**trackBy**’ function, Angular uses the specified unique identifier to track changes in the list more efficiently. It allows Angular to update only the elements that have changed, rather than re-rendering the entire list.

   

Benefits of using ‘**trackBy**’:  
**Improved Performance:**Angular can identify which items have changed, added, or removed more efficiently, resulting in improved rendering performance.  
**Preventing Unnecessary DOM Manipulation:** Without ‘**trackBy**’, Angular might re-render every element in the list, even if only a few items have changed. With ‘trackBy’, it can selectively update only the relevant elements, reducing unnecessary DOM manipulations.

It’s important to note that the unique identifier provided by the ‘trackBy’ function should be stable and unique for each item in the list. Commonly, an item’s unique identifier, such as an ID property is used for tracking.

### OnPush Change Detection: Mutation Scenario

**Scenario:** A parent passes an object as `@Input()` to a child with `ChangeDetectionStrategy.OnPush`. The parent mutates a property on that object.

```typescript
@Component({
  selector: 'app-child',
  template: `{{ data?.name }}`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChildComponent {
  @Input() data!: { name: string };
}

@Component({
  selector: 'app-parent',
  template: `<app-child [data]="person"></app-child>`,
})
export class ParentComponent {
  person = { name: 'Alice' };

  update() {
    this.person.name = 'Bob'; // mutates the same object
  }
}
```

**Will the child reflect the change after `update()`?**

**No.** `OnPush` compares input references using `===`. Because `this.person` still points to the same object, Angular sees no reference change and skips the child.

**Fix 1 — New object reference (recommended)**

```typescript
update() {
  this.person = { ...this.person, name: 'Bob' }; // new reference
}
```

**Fix 2 — Manually trigger change detection**

```typescript
constructor(private cd: ChangeDetectorRef) {}

update() {
  this.person.name = 'Bob';
  this.cd.markForCheck(); // marks component for re-check
}
```

**Fix 3 — Remove OnPush** (not recommended — defeats the performance benefit)

> **Rule:** With `OnPush`, always use immutable updates (spread operator, `Object.assign`, or libraries like Immer) so Angular detects the reference change automatically.

### How does a browser render a page?

\- When a page is requested from the server, the browser gets a simple utf8 text file.  
\- Browser then traverses your page and parses it.  
\- Each HTML tag has its own corresponding JavaScript classes.  
\- For every tag an object of its JS class is created and added to the browser's window.document object.  
\- Window.document is one of the many objects of the browsers window object. Others are window.console, window.navigator, etc. 

## Components, Directives & Architecture

### Directives

* Directives are the building blocks of Angular applications. Angular components are simply a directive with a template.  
* Directive is basically a class with a @Directive decorator. A component is also a directive-with-a-template.  
* Directives are technically components, but they operate on the non-UI part of the DOM.  
* Their work is to add extra functionality to the DOM.

**Types of Angular directives**

* **Component Directive** They are the building blocks of an angular application. They are typically associated with a custom HTML element and encapsulates a piece of the user interface along with its behavior. Components have a template, styles and can have their own logic.  
* **Structural Directive** (Responsible for HTML layout. They share or reshape the DOM’s structure, typically by adding, removing or manipulating elements) ex. **ngIf**, **ngFor**, **ngSwitch**  
* **Attribute Directive** (change behavior of elements) ex. **ngStyle**, **ngClass**

### Decorators

Decorator marks a class as an Angular component and provides configuration metadata about how the component should be processed, instantiated, and used at runtime.  
Decorators are functions that are invoked with a **prefixed @ symbol**, and immediately followed by a class, parameter, method or property.  
Ex. **@Component**, **@Input**, **@Output**, etc

### View Encapsulation

A component’s styles can be encapsulated within the component’s host element so that they don’t affect the rest of the application.  
Angular was built in such a way that the styles defined inside the component stylesheet are scoped to only that component alone no matter the class name. This is a lot like local and global variable definition and how they are scoped. This scoping mechanism is known as encapsulation.  
**Shadow DOM**: Shadow DOM allows for the creation of encapsulated components with isolated styling and DOW structures.  
Three view encapsulation types:  
**ViewEncapsulation.None** \- No Shadow DOM at all. Therefore, also no style encapsulation.  
**ViewEncapsulation.Emulated (default)** \- No Shadow DOM but styles are emulated to provide a level of encapsulation by adding unique attributes to the component’s elements.  
**ViewEncapsulation.Native** \- Uses the browser’s native Shadow DOM with all its goodness.

### ShadowDOM vs Virtual DOM

**ShadowDOM** allows us to hide DOM logic behind other elements. Additionally, it enables us to apply **scoped style** to elements without them bleeding out to the outer world.  
Whenever we create a component, Angular puts its template into a **shadowRoot**, which is the Shadow DOM of that particular component. Doing that, we get DOM tree and style encapsulation for free,right? But what if we don’t have Shadow DOM in the browser? Answer is, **Angular doesn’t use native Shadow DOM by default, it uses an emulation.**

Shadow DOM allows hidden DOM trees to be attached to elements in the regular DOM tree \- this shadow DOM tree starts with a shadow root, underneath which you can attach an element, in the same way as the normal DOM.

**Shadow DOM** is a browser technology whose main objective is to provide encapsulation when creating elements. On the other hand, the virtual DOM is managed by JavaScript libraries e.g. ReactJS and it’s mostly a strategy to optimize performance.

**What is a real DOM?**  
DOM stands for Document Object Model. It’s a representation of a web page structure in the form of a tree, in which each element of a page (roughly speaking, each HTML tag) is a node.  
Keep in mind, though, that the DOM isn’t identical to the HTML code. For instance, browsers are typically forgiving of errors in an HTML document and will do what they can to “guess” what the correct intention was and fix the issue.  
The neat thing about the DOM is that it acts as an API for the web document. It allows you to programmatically interact with the elements of a page. That’s basically what you’ll do in client-side applications, of course: DOM manipulation.

**What is the Shadow DOM, and how is it different from the real DOM?**  
The shadow DOM is a relatively recent API whose goal is to provide encapsulation when implementing [web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components). The idea is to allow you to implement an element without worrying it will clash in some way with the rest of the webpage. A classic example of the kind of problem that the shadow DOM solves is CSS issues due to auto-generated classes. The embedded style of the component didn’t affect the global style either.

**What is the difference between DOM and Virtual DOM?**  
First of all, code that manipulates the DOM is typically very imperative/procedural. For instance, the following code excerpt retrieves the first list item from the previous example and changes its content:

var item \= document.getElementsByTagName('li')\[0\];  
item.innerText \= 'A different text';

It doesn’t look that bad for this silly example of ours. But when we’re talking about a large application, with rich interactions and complex logic, code can become quite messy very quickly.

However, the main concern when it comes to DOM manipulation is performance. DOM rewrites are costly, so you want to do them sparingly. That’s where the concept of virtual DOM can come in handy.

The virtual DOM is a “virtual,” in-memory representation of the DOM. Since it lives in memory, manipulating it is less costly than handling the real DOM.

Libraries such as React use the concept of virtual DOM to abstract the real DOM away from the developer. When you use React, for instance, you define the desired state of each component in a declarative way, with JSX code or calls to React functions. React then does what it needs to reconcile its virtual DOM with the real DOM, only changing what’s necessary, so you don’t have to bear the cost of real DOM rewrites.

|  | Shadow DOM	 | Virtual DOM |
| :---- | ----- | ----- |
| **What It Is**	 | An API allowing developers to attach a “hidden” DOM to an element for encapsulation purposes	 | An API allowing developers to attach a “hidden” DOM to an element for encapsulation purposes	 |
| **Purpose** | Encapsulate logic and presentation inside an element, protecting it from effects from the rest of the page	 | Abstract the real DOM away, allowing for better performance and a more declarative style of coding |
| **Who Implements It**	 | Browsers | JavaScript libraries, such as React and Vue |

**What is the difference between “Emulated DOM” in Angular, and “Virtual DOM” in React?**

The "Emulated DOM" in Angular and the "Virtual DOM" in React are both techniques used to optimize the rendering performance of web applications, but they operate in different ways.

**1\. Emulated DOM in Angular:**  
Angular uses a technique called "Emulated Encapsulation" for its rendering strategy.  
Emulated Encapsulation involves adding unique attributes to the elements of each component during runtime to scope the styles.  
The styles of one component do not leak into other components, preventing unintended side effects due to global styling.  
The browser's native rendering engine is used, and Angular manipulates the DOM to achieve encapsulation.  
While Angular provides style encapsulation, it may involve some performance overhead as it interacts with the actual DOM.

**2\. Virtual DOM in React:**  
React uses a Virtual DOM to optimize rendering by reducing the number of direct manipulations to the actual DOM.  
The Virtual DOM is a lightweight copy of the real DOM, maintained in memory by React.  
When the state of a component changes, React first updates the Virtual DOM rather than the actual DOM.  
React then performs a "reconciliation" process, comparing the changes in the Virtual DOM with the previous state to determine the minimal set of changes needed to update the actual DOM.  
This minimizes the number of direct manipulations to the real DOM, resulting in improved performance.

**Key Differences:**  
Angular's Emulated DOM primarily focuses on style encapsulation to prevent global styling issues within components.  
React's Virtual DOM focuses on optimizing the efficiency of updates to the actual DOM, minimizing the need for direct DOM manipulations.  
Angular interacts with the real DOM, adding unique attributes to achieve encapsulation, while React operates on a separate Virtual DOM before updating the real DOM.  
Angular's approach to encapsulation is more explicit, relying on the browser's native rendering engine, whereas React's Virtual DOM is a conceptual layer added by the framework for efficient updates.  
In summary, while both Angular's Emulated DOM and React's Virtual DOM aim to improve performance, they have different focuses and implementation strategies. Angular's Emulated DOM is more concerned with style encapsulation, while React's Virtual DOM is primarily about optimizing the process of updating the actual DOM for improved rendering performance.

### Web Components

**Web Components** is a suite of different technologies allowing you to create **reusable custom elements** \- with their **functionality encapsulated away from the rest of your code** \- and utilize them in your web apps.

**Angular elements** are **angular components** packaged as custom elements (also called **Web Components**), a web standard for defining new HTML elements in a **framework-agnostic way**.

**Custom elements** are a **Web Platform feature** currently supported by a number of browsers. A **custom element extends HTML** by allowing you to **define a tag whose content is created and controlled by JavaScript code**. The browser maintains a **CustomElementRegistry** of defined custom elements, which **maps an instantiable JavaScript class to an HTML tag**.

The **@angular/elements** package exports a **createCustomElement() API** that provides a **bridge from Angular’s component interface and change detection functionality to the built-in DOM API**.

Custom elements bootstrap themselves \- they start automatically when they are added to the DOM, and are automatically destroyed when removed from the DOM. Once a custom element is added to the DOM for any page, it looks and behaves like any other HTML element, and does not require any special knowledge of Angular terms or usage conventions.

### Content Projection

Allows HTML to be injected into a component. We use \<ng-content\> inside the component which is a placeholder for the injected HTML.

### Explain the ‘Smart-Dumb’ / ‘Container-Presenter’ component architecture pattern in Angular

Smart and Dumb components, also known as container and presentational components, are concepts in Angular that help organize and structure components within an application. These concepts are part of a broader architectural pattern known as the ‘Container-Presenter’ or ‘Smart-Dumb’ pattern. Understanding this pattern can lead to more maintainable and module Angular applications.

**Smart(Container) Components:**

1. **Responsibility:**  
   Smart components are responsible for managing state, interacting with services, and handling complex logic. They are often associated with a specific route or a high-level application feature.  
2. **State Management:**  
   Smart components are aware of the application state and may use services or state management libraries (like NgRx) to manage and update the state.  
3. **Communication:**  
   Smart components handle communication with services, fetch data, and may dispatch actions to update the state. They act as a bridge between the application’s state and the presentational components.  
4. **No UI Markup:**  
   Smart components typically have minimal or no UI markup of their own. They delegate the rendering of UI to the presentational components.  
5. **Example:**  
   A smart component might be responsible for fetching a list of items from a server, managing the selected item, and handling user interactions. It would pass this information down to a dumb component for rendering.

**Dumb(Presentational) Components:**

1. **Responsibility:**  
   Dumb components are primarily responsible for presenting data and capturing user interactions. THey receive data as inputs and emit events to notify the parent (smart) components about user actions.  
2. **No State Management:**  
   Dumb components should not manage the application state. They are focused on rendering the UI based on the input they receive.  
3. **Reusability:**  
   Dumb components are highly reusable since they are decoupled from the application’s state and logic. They can be used in different parts of the application without modification.  
4. **Pure UI:**  
   Dumb components are often referred to as ‘pure’ UI components because they don’t contain business logic. They are concerned only with how the UI looks and behaves.  
5. **Example:**  
   A dumb component might receive a list of items as input and display them in a simple list. It would emit an event when an item is selected, but it would not handle the logic associated with fetching or managing the items.

**Benefits:**

1. Separation of Concerns:  
2. Reusability:  
3. Testability:  
4. Readability:

By employing the smart/dumb component pattern, developers can create more scalable, maintainable, and understandable Angular applications.

### How does Angular implement a component-based architecture, and what are the benefits?

Angular implements a component-based architecture, which means that the user interface is built using self-contained, reusable components. Here's an explanation of how Angular achieves this and the associated benefits:

1. **Components:**  
   Definition: Components are the fundamental building blocks of an Angular application. They encapsulate the template, styles, and behavior, allowing for the creation of reusable and modular UI elements.  
   Structure: Each component consists of a TypeScript class that defines the component's behavior and properties, an HTML template that defines its structure, and styles for its presentation.  
     
2. **Hierarchical Structure:**  
   Component Hierarchy: Components in Angular can be arranged in a hierarchical structure. The application itself is often structured as a tree of components, where each component is responsible for a specific part of the UI.

3. **Data Binding:**  
   Two-way Data Binding: Angular provides two-way data binding, enabling automatic synchronization of the data between the component and the view. Changes in the component update the view, and vice versa, without the need for manual intervention.

4. **Reusability:**  
   Component Reusability: Components can be reused across different parts of the application or even in different projects. This promotes code reuse, maintainability, and consistency in the user interface.

5. **Encapsulation:**  
   Encapsulation of Logic and Styles: Each component encapsulates its own logic and styles. This encapsulation helps in avoiding naming conflicts and unintended interference between different parts of the application.

6. **Dependency Injection:**  
   Dependency Injection (DI): Angular uses dependency injection to provide components with the services they need. This promotes loose coupling and makes components more modular and easier to test.

7. **Event-Driven Architecture:**  
   Event Emitters: Components in Angular can communicate with each other using events. Parent components can communicate with child components and vice versa, enabling a flexible and event-driven architecture.

8. **Separation of Concerns:**  
   Separation of Template, Styles, and Logic: Angular enforces a clear separation of concerns by separating the template (HTML), styles (CSS), and logic (TypeScript) of a component. This separation enhances code readability and maintainability.

9. **Change Detection:**  
   Efficient Change Detection: Angular's change detection mechanism efficiently tracks changes in the component's state and updates the view accordingly. This ensures optimal performance by only updating what is necessary.

In summary, Angular's component-based architecture provides a structured and scalable way to build web applications by promoting reusability, encapsulation, and maintainability. The clear separation of concerns and the use of dependency injection contribute to a modular and flexible application design.

### Describe the purpose and use of Angular modules

Angular modules play a crucial role in structuring and organizing an Angular application. Here’s an overview of their purpose and use:

1. **Purpose:**  
   * **Organization and Modularisation:** Angular modules, also known as NgModules, help in organizing an application’s code into cohesive, reusable, and manageable units. Each module encapsulates a set of related components, services, directives, and pipes.  
   * **Dependency Management:** Modules define the boundaries within which dependencies are managed. Components, services and other	artifacts within a module can easily reference and depend on each other.  
   * **Encapsulation:** Modules provide a level of encapsulation, helping to keep different parts of the application separate. This separation is essential for maintaining a clean and modular codebase.  
   * **Configuration:** Modules serve as a place for configuring the Angular application by declaring the components, directives, pipes, and services that the module will use.  
2. **Use:**  
   * **Creating a Module:**  
     To create a module, use the @NgModule decorator in a TypeScript class. This decorator takes a metadata object that specifies the components, directives, pipes, and services that belong to the module.  
   * **Declaration:**  
     The declarations property in the @NgModule decorator lists the components, directives, and pipes that belong to the module.  
   * **Imports:**  
     The imports property is used to import other modules that this module depends on. This allows you to compose your application from smaller, self-contained modules.  
   * **Providers:**  
     The provider's property specifies the services that the module contributes to the global collection of services. These services are available for injection throughout the application.  
   * **Bootstrap:**  
     The bootstrap property is used in the root module (AppModule) to specify the main component that should be bootstrapped when the application starts.

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

In this example, **AppModule** is the root module of the Angular application. It declares the AppComponent and imports the BrowserModule. The bootstrap property specifies that the AppComponent should be the starting point of the application.

In summary, Angular modules serve as a foundational concept for structuring Angular applications, promoting code organization, maintainability, and reusability. They provide a way to modularize the application, manage dependencies, and configure various aspects of the application.

### How do you organize modules in a large Angular application to ensure maintainability and scalability?

Organizing modules in a large Angular application is crucial for ensuring **maintainability**, **scalability**, and overall **code structure**. Here are some best practices and strategies for organizing modules in a large Angular application:

1. **Feature Modules:**  
   * Divide your application into **feature modules based on functionality**. Each feature module should encapsulate a specific feature or a set of related features.  
   * Feature modules should be self-contained and independent, making it easier to manage, test, and scale the application.

```typescript
// Example of a Feature Module
@NgModule({
  declarations: [FeatureComponent, FeatureDirective, FeaturePipe],
  imports: [CommonModule, SharedModule],
  providers: [FeatureService],
})
export class FeatureModule {}
```

   

2. **Shared Module:**  
   * Create a shared module to house commonly used components, directives, and pipes. This module can be imported into other modules, preventing code duplication and promoting consistency. Facilitates the organization of commonly used UI elements and promotes code reusability.  
   * Key Characteristics: Reusable UI Components, Common Directives and Pipes, Third-party Module Imports, No Services(services should be placed in Core Module), Focussed on UI and Presentation.

```typescript
// Shared Module
@NgModule({
  declarations: [SharedComponent, SharedDirective, SharedPipe],
  exports: [SharedComponent, SharedDirective, SharedPipe],
})
export class SharedModule {}
```

   

3. **Core Module:**  
   * Create a core module for services, singleton objects, and other application-wide components. The core module should be imported only once in the root module to ensure a singleton pattern for shared services.  
   * Key Characteristics: Singleton Services, Application-wide Components, Initialization Logic, App Shell Components, Single Responsibility.

```typescript
// Core Module
@NgModule({
  providers: [AuthService, LoggingService, ApiService],
})
export class CoreModule {}
```

4. **Lazy Loading:**  
   * Use lazy loading to load feature modules on-demand. This can significantly improve initial loading times and isolate specific functionality to separate bundles.

```typescript
// App Routing Module
const routes: Routes = [
  { path: 'dashboard', loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule) },
];
```

5. **Routing Modules:**  
   * Consider creating separate routing modules for each feature module. This helps keep the routing configuration organized and manageable.

```typescript
// Feature Routing Module
const routes: Routes = [
  { path: 'feature', component: FeatureComponent },
];
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class FeatureRoutingModule {}
```

6. **Index Barrel:**  
   * Use an index barrel file (index.ts) in each directory to simplify imports and exports.

```typescript
// Inside a Feature Module Directory
// feature.module.ts
export * from './feature.component';
export * from './feature.service';
```

     

7. **Avoid Circular Dependencies:**  
   * Be mindful of circular dependencies, as they can lead to difficult-to-diagnose issues. Use the Angular dependency injection system to manage dependencies effectively.

8. **Naming Conventions:**  
   * Establish clear and consistent naming conventions for modules, components, services, and directories. This makes it easier for developers to understand the structure of the application.

By following these best practices, you can create a modular and scalable Angular application that is easier to maintain and extend as your project grows. Keep in mind that the specific structure may vary based on your application’s requirements and team preferences.

### Component Inheritance in Angular

Component inheritance uses TypeScript class inheritance to share logic, properties, and lifecycle hooks across components.

```typescript
// base.component.ts
import { Component, Input } from '@angular/core';

@Component({ template: '' })
export class BaseComponent {
  @Input() title: string = '';

  logTitle(): void {
    console.log(this.title);
  }
}
```

```typescript
// child.component.ts
import { Component } from '@angular/core';
import { BaseComponent } from './base.component';

@Component({
  selector: 'app-child',
  template: '<h2>{{ title }}</h2><p>{{ extra }}</p>',
})
export class ChildComponent extends BaseComponent {
  extra = 'Child-specific content';
}
```

```html
<app-child [title]="'My Title'"></app-child>
```

**Key points:**
- The child inherits `@Input()` properties and methods from the base
- Each class must have its own `@Component` decorator
- Lifecycle hooks are inherited — call `super.ngOnInit()` if the base implements them
- Prefer composition (services, mixins) over deep inheritance hierarchies

### Dynamic Component Loading

Load components at runtime into a `ViewContainerRef` using `ComponentFactoryResolver` (Angular < 13) or `createComponent()` (Angular 13+).

```typescript
// dynamic-a.component.ts
@Component({ selector: 'app-dynamic-a', template: '<p>Dynamic Component A</p>' })
export class DynamicComponentA {}

// dynamic-b.component.ts
@Component({ selector: 'app-dynamic-b', template: '<p>Dynamic Component B</p>' })
export class DynamicComponentB {}
```

```typescript
// host.component.ts
import { Component, ViewChild, ViewContainerRef } from '@angular/core';
import { DynamicComponentA } from './dynamic-a.component';
import { DynamicComponentB } from './dynamic-b.component';

@Component({
  selector: 'app-host',
  template: `
    <button (click)="load(true)">Load A</button>
    <button (click)="load(false)">Load B</button>
    <ng-template #container></ng-template>
  `,
})
export class HostComponent {
  @ViewChild('container', { read: ViewContainerRef }) container!: ViewContainerRef;

  load(loadA: boolean): void {
    this.container.clear();
    this.container.createComponent(loadA ? DynamicComponentA : DynamicComponentB);
  }
}
```

> **Note:** In Angular 13+, `createComponent()` is called directly on `ViewContainerRef` without `ComponentFactoryResolver`. For older versions, use `ComponentFactoryResolver.resolveComponentFactory(ComponentClass)` first.

## Async Programming: Promises, Observables & RxJS

### Promises, Observables, Subject, BehaviorSubject, EventEmitter

**PROMISE:**

* **Promise** is used to handle asynchronous operations.  
* **Promise** emits a single value, whereas Observables execute multiple values over a period of time.  
* A **Promise** is just an object with properties in JavaScript.  
* **Promise** is a returned object to which you attach callbacks, instead of passing callbacks into a function.  
* **Promise** overcomes the problem of callback hell associated with multiple callback functions.  
* **Promise** supports promise chaining \- a common need to support two asynchronous operations back-to-back.  
* **Promise** executes once and then they are done. On the contrary **Observables** continue to be observed even after the event occurs.  
* **Promise** provides better error handling than callbacks and events.  
* **Promises** cannot be canceled. **Observables** can be canceled.  
* **Promises** are not lazy, they are eager. Meaning they start the execution of the asynchronous operation as soon as the Promise constructor is invoked. The asynchronous operation within the Promise begins immediately, regardless of whether there is a consumer( a ‘then()’ or ‘catch()’ callback) waiting for the result.  
* No functional operators are available on **Promises**.  
* **Disadvantages of Promises**:  
  * A **Promise** is defined where the data is created, not where it is being consumed.  
  * As your application gets bigger, **Promises** become hard to manage.  
  * What if I want to retry a failed call? Now we are back to callback hell again.

**OBSERVABLES**(*reactive programming*)**:**

* **Definition: An Observable is a data stream that emits multiple values over time and can be subscribed to for reactive programming in JavaScript.**  
* **Reference Article:** [https://www.tektutorialshub.com/angular/angular-observable-tutorial-using-rxjs/\#what-is-an-observable-in-angularnbsp](https://www.tektutorialshub.com/angular/angular-observable-tutorial-using-rxjs/#what-is-an-observable-in-angularnbsp)  
* In **Reactive Programming** we deal in programming with **(*asynchronous*)data streams**, and stream is data which arrives over a period of time.  
* Stream of data can be anything: variables, user inputs, properties, caches, data structures, etc.  
* Stream not only emits values, but also error and complete notifications.  
* RxJS is a reactive programming library that allows us to work with asynchronous data streams. Angular uses RxJS library heavily in its framework to implement Reactive Programming.  
* RxJS has two main players: **Observable** & **Observers(Subscribers)**  
* **Observable** stream or simply Observable emits the value from the stream asynchronously. It emits the complete signal when the stream completes or an error signal if the stream errors out.  
* Angular uses the Observer pattern which simply means \- Observable objects are registered, and other objects observe (in Angular using the subscribe method) them and take action when the observable object is acted on in some way.  
* **Observers**: The Observable on its own is useless unless someone consumes the value emitted by the observable. We call them observers or subscribers.  
* The observer must subscribe with the observable to receive the value from the observable. While subscribing it optionally passes the three callbacks. next(), error() & complete(). The observable starts emitting the value as soon as the observer or consumer subscribes to it.  
* The observable invokes the next() callback whenever the value arrives in the stream. It passes the values as the argument to the next callback. If the error occurs, then the error() callback is invoked. It invokes the complete() callback when the stream completes.  
* **Observables** are a sequence of events over time.  
* **Observables** are **lazy by default**.  
* **Observables** emit multiple values over a period of time.  
* **Observable** has two participants: Publisher/Creator and Subscriber/Observer.  
* **Observables** are just things you wish to observe and take action on.  
* **Observables** continue to be observed even after the event occurs. Whereas **Promise** executes once and then they are done.  
* **Observables** can be canceled using Unsubscribe(). using **Promises** cannot be canceled.  
* Difference between **Unicast & Multicast Analogy**:  
  **Unicast**: People watching a movie on OTT platforms.  
  **Multicast**: People watching a movie in theaters.  
* **Observables are Unicast.** Observable on subscription creates a separate instance of the producer. Hence each gets different data.  
* **Observable** provides an operator called Pipeable operator, which allows you to “do this” nesting of operations in a more efficient way.  
* **Observables** provide support for passing messages between publishers and subscribers in your application.  
* **Observables** provide operators like map, forEach, filter, reduce, retry, retryWhen, etc.  
* **Observables** can do everything a Promise can do, and more.  
* **Observables** offer significant benefits over other techniques for event handling, asynchronous programming, and handling multiple values.  
* **Observables** are declarative \- that is, you define a function for publishing values, but it is not executed until a consumer subscribes to it. The subscribed consumer then receives notifications until the function completes, or until the unsubscribe.  
* As a publisher, you create an Observable instance that defines a subscriber function. This is the function that is executed when a consumer calls the subscribe() method. The subscriber function defines how to obtain or generate values or messages to be published.  
* Angular makes use of **observables** as an interface to handle a variety of common asynchronous operations. For example:  
  * You can define custom events that send observable output data from a child to a parent component.  
  * The HTTP module uses observables to handle AJAX requests and responses.  
  * The Router and Forms modules use observables to listen for and respond to user-input events.  
* Observables are classified into two groups: **Cold** & **Hot**  
  * **Cold Observable**  
    A cold observable starts producing values only when it is subscribed to, and each subscriber gets its own independent stream of values. It does not share its subscription with other observers.  
  * **Hot Observable**  
    A hot observable, on the other hand, emits values regardless of whether there are subscribers. Subscribers that join later miss some or all the emitted values, depending on the specific hot observable implementation.  
* **Why use Observables over Promises?**  
  The Angular team has chosen to leverage RxJS Observables. This means a lot of Angular features are built around Observables. Also RxJS Observables offer a super-set of the capabilities available with promises. A promise can deliver only the first value emitted by an asynchronous call. An Observable can emit multiple values. Also, the large number of RxJS operators available to deal with Observables allow us to build tailored streams with a few lines of code. This is not possible with promises.

**SUBJECT:**

* **Reference Article**: [https://www.tektutorialshub.com/angular/subjects-in-angular/\#what-are-subjects](https://www.tektutorialshub.com/angular/subjects-in-angular/#what-are-subjects)  
* **Subject** is a **special type of Observable** that **acts as both Observable and Observer**.  
* A **Subject is a special type of Observable** that allows values to be multicast to many Observers.  
* The **Subjects are also Observers** because they can subscribe to another observable and get value from it, which it will multicast to all of its subscribers.  
* **Subject** implements the **next()**, **error()** and **complete()** methods. Hence it can subscribe to another observable and receive values from it. It also maintains a collection of observers\[\].  
* An Observable can subscribe to the Subject and receive value from it. Subject adds them to its collection observers. Whenever there is a value in the stream it notifies all of its Observers.  
* **Subject** is **HOT OBSERVABLE**.  
* **Subjects are MULTICAST.**  
  * Another important distinction between Observable and Subject is that Subjects are multicast.  
  * More than one subscriber can subscribe to a subject. They will share the same instance of the observable. This means that all of them receive the same event when the Subject emits it.  
  * Multiple Observers of an Observable, on the other hand, will receive a separate instance of the observable.  
* It’ll emit a new value using next() method. All the subscribers who subscribe to the subject when emitted will receive the same value.  
* It is recommended to use Subject for transferring data from one component to another as EventEmitter extends Subject, adding a method emit().  
* Use EventEmitter when transferring data from child component to parent component.  
* A subscriber will only get published values that were emitted after the subscription. Ask yourself, is that what you want? Does the subscriber need to know anything about previous values? If not, then you can use this, otherwise choose one of the others. For example, with component-to-component communication. Say you have a component that publishes events for other components on a button click. You can use a service with a subject to communicate.  
* **Subscriber maintains a list of subscribers**  
  Whenever a subscriber subscribes to a subject, it will add it to an array of subscribers. This way Subject keeps track of its subscribers and emits the event to all of them.

**BehaviorSubject:**

* **Reference Article:** [https://www.tektutorialshub.com/angular/replaysubject-behaviorsubject-asyncsubject-in-angular/](https://www.tektutorialshub.com/angular/replaysubject-behaviorsubject-asyncsubject-in-angular/)  
* BehaviorSubject requires an initial value and stores the current value and emits it to the new subscribers.  
* The last value is cached. A subscriber will get the latest value upon initial subscription. The semantics for this subject is to represent a value that changes over time. For example a logged in user. The initial user might be an anonymous user. But once a user logs in, then the new value is the authenticated user state.  
* The **BehaviorSubject** is initialized with an initial value. This is sometimes important to coding preference. Say for instance you initialize it with null. Then in your subscription, you need to do a null check. Maybe OK, or may be annoying.

**ReplaySubject:**

* It can cache up to a specified number of emissions. Any subscribers will get all the cached values upon subscription.  
  If you initialize a ReplaySubject with a buffer size of 1, then it actually *behaves* just like a BehaviorSubject. The last value is always cached, so it acts like a value changing over time. With this, there is no need for a null check like in the case of the BehaviorSubject initialized with a null. In this instance, no value is ever emitted to the subscriber until the first publishing.  
* What you want to *avoid* is using the vanilla Subject when what you really need is some caching behavior. Take for example you are writing a routing guard or a resolve. You fetch some data in that guard and set it in a service Subject. Then in the routed component you subscribe to the service subject to try to get that value that was emitted in the guard. OOPs. Where's the value? It was already emitted, DUH. Use a "caching" subject\!

**EventEmitter:**

* It is used with @Output directive to emit custom events synchronously or asynchronously using emit() method. Using EventEmitter we can transfer data from the child component to the parent component. It is true, we can transfer data from one component to another using EventEmitter, but it is not recommended.

**AsyncSubject:**

* An AsyncSubject is a type of Subject that emits only the last value of the observable sequence, only after it completes.  
* Example:

```typescript
// RxJS v6+
import { AsyncSubject } from 'rxjs';
const sub = new AsyncSubject();
sub.subscribe(console.log);
sub.next(123); //nothing logged
sub.subscribe(console.log);
sub.next(456); //nothing logged
sub.complete(); //456, 456 logged by both subscribers
```

---

### How to decide when to use a Promise or an Observable? When do we need to use Promise in an Angular application?

The choice between using a Promise or an Observable in an Angular application depends on the nature of the asynchronous operation and the requirements of your application. Here are some considerations to help you decide when to use a Promise and when to use an Observable:

**Use Promises When:**

1. **Single-Value Operations:**  
   Promises are suitable for handling single asynchronous operations that result in a single value, such as fetching data from a server or loading a resource.  
2. **No need for multiple values:**  
   If the asynchronous operation is expected to produce a single result and there’s no need for continuous updates or a stream of values, a Promise is a simpler choice.  
3. **Simple Syntax:**  
   Promises have a simpler syntax, making them more straightforward for handling basic asynchronous operations.

```typescript
// Example using a Promise
getData(): Promise<Data> {
  return this.http.get<Data>(url).toPromise();
}
```

4. **Compatibility:**  
   If you are working with APIs or libraries that return Promises, using Promise can help maintain compatibility.

**Use Observables When**

1. **Continuous Data Streams:**  
   Observables are designed for handling continuous streams of data over time. If the asynchronous operation involves multiple values, such as real-time updates or user interactions, Observables are a better fit.  
2. **Event Handling:**  
   Observables are well-suited for scenarios where you want to handle events or listen for changes over time, such as user input, DOM events, or data changes.

```typescript
// Example using an Observable
getData(): Observable<Data> {
  return this.http.get<Data>(url);
}
```

3. **Advanced Features:**  
   Observables provide mechanisms for cancellation and better error handling. You can unsubscribe from an Observable to stop receiving updates, and Observables handle errors more explicitly.  
4. **Lazy Execution:**  
   Observables are lazy by nature, meaning they only execute when someone subscribes to them. This can be advantageous in scenarios where you want to delay the execution until it’s necessary.

```typescript
// Lazy execution with Observables
const observable = new Observable(observer => {
  // Async operation
  observer.next(data);
});
// No execution until someone subscribes
const subscription = observable.subscribe(result => {
  console.log(result);
});
```

   

**Considerations:**

1. **Combining both:**  
   Angular allows you to convert Observables to Promises and vice versa using **toPromise()** and **from()** operators. You can choose the best tool for a particular situation and convert between them if needed.

```typescript
// Convert Observable to Promise
const promise = this.http.get<Data>(url).toPromise();
// Convert Promise to Observable
const observable = from(this.http.get<Data>(url));
```

   

2. **Library and API Support:**  
   Consider the libraries or APIs you are working with. Some libraries and Angular services may return Promises or Observables, and your choice might be influenced by compatibility.

3. **Consistency:**  
   Maintain consistency in your codebase. If your application primarily uses Observables for handling asynchronous operations, it might be beneficial to stick with Observables for consistency.  
   

**Good To Know:**

- **AngularCLI** behind the scene uses Webpack  
- **Webpack** is a build tool that runs on top of NodeJS.  
- **Grunt**: It is a task-scheduler/automation tool.  
- **Unit Testing**: Jasmine(like xUnit), Karma.  
- **Angular** is a Framework.**React** is a library.  
- **main.ts** is the entry point of your application.  
- Once all packages are installed, remove the ^ prefixed to package version. This will ensure that the next install will not update any of the packages which might break your application behavior.

### RxJS Operators

Example:

```html
this.results = this.searchForm.controls.search.valueChanges.pipe(
    debounceTime(500),
    filter(value => value.length > 3),
    distinctUntilChanged(),
    switchMap(searchTerm => this.http.get<any>
               (`${API_URL}?q=${searchTerm}&key=${API_KEY}&part=snippet`)),
    map(response => response.items)
);
```

* switchMap  
  switchMap will cancel out the prior event and just use the value in the latest event. With switchMap, every time a new Observable is returned, it unsubscribes from the previous one and subscribes to the latest one. Basically, it’s switching from one Observable to the latest Observable.  
  * debounceTime  
    It will discard emitted values that take less than the specified time between outputs.  
  * filter  
    Filter only emits those items from an Observable that pass a predicate test  
  * distinctUntilChanged

  Only emits when the current value is different from the last.


### Does Cancelling an Observable Cancel the Server-Side API Request?

**Not automatically — it depends on the implementation.**

When you unsubscribe from an `HttpClient` observable in Angular, the **browser cancels the in-flight HTTP request** (via `XMLHttpRequest.abort()` or the Fetch API's `AbortController`). This is one of the key advantages over Promises.

```typescript
const sub = this.http.get('/api/data').subscribe(...);

// Later:
sub.unsubscribe(); // Cancels the actual HTTP request in the browser
```

**What gets cancelled at each layer?**

| Layer | What Happens |
| :--- | :--- |
| **Browser** | Request is aborted — no response is processed |
| **Network** | TCP connection may or may not be torn down depending on how far the packet got |
| **Server** | **Usually keeps processing** — the server has no reliable way to know the client abandoned the request unless it explicitly checks for a disconnected socket |

The server typically **completes its work** (DB queries, business logic, etc.) even if the client unsubscribed. Only the browser waiting for and processing the response is cancelled.

**Practical example**

```typescript
// On component destroy, the browser aborts the request — but the server still runs the query
ngOnInit() {
  this.http.get('/api/heavy-computation')
    .pipe(takeUntilDestroyed(this.destroyRef))
    .subscribe(...);
}
```

**When the server *can* detect cancellation**

Server-side cancellation requires explicit support:

- **ASP.NET Core** — use `CancellationToken` in controller actions; the framework signals it when the client disconnects
- **Node.js** — listen to `req.on('close', ...)`

```csharp
// ASP.NET Core — cancelled if client disconnects
[HttpGet]
public async Task<IActionResult> HeavyQuery(CancellationToken ct)
{
    var result = await _db.RunExpensiveQuery(ct);
    return Ok(result);
}
```

> **Summary:** Angular unsubscribe cancels the browser-side request, but the server continues unless it is explicitly coded to handle client disconnection via cancellation tokens.

---

### Can an Observable Have Multiple Subscribers?

**Yes — and the behavior depends on whether it is Cold or Hot.**

**Cold Observables (default)**

Each subscriber gets its **own independent execution**. The producer restarts for every subscription.

```typescript
const obs$ = this.http.get('/api/data');

obs$.subscribe(v => console.log('Sub 1:', v)); // makes HTTP request #1
obs$.subscribe(v => console.log('Sub 2:', v)); // makes HTTP request #2
```

`HttpClient` observables are **cold** — two subscribers means two separate HTTP calls.

**Hot Observables**

All subscribers share the **same execution**. Late subscribers may miss earlier emissions.

```typescript
const subject = new Subject<number>();

subject.subscribe(v => console.log('Sub 1:', v));
subject.subscribe(v => console.log('Sub 2:', v));

subject.next(42); // both subscribers receive 42
```

`Subject`, `BehaviorSubject`, `ReplaySubject`, DOM events, and WebSockets are **hot**.

**Making a Cold Observable Hot — `share()` / `shareReplay()`**

This is the common pattern to avoid duplicate HTTP calls:

```typescript
const data$ = this.http.get('/api/data').pipe(
  shareReplay(1) // multicasts to all subscribers, caches last value
);

data$.subscribe(v => console.log('Sub 1:', v)); // triggers 1 HTTP request
data$.subscribe(v => console.log('Sub 2:', v)); // reuses the same request + gets cached value
```

| Type | Subscribers | Execution | Example |
| :--- | :--- | :--- | :--- |
| **Cold** | Multiple | One per subscriber | `HttpClient`, `of()`, `interval()` |
| **Hot** | Multiple | Shared | `Subject`, DOM events, WebSockets |
| **Cold + `shareReplay`** | Multiple | Shared + cached | HTTP with multicasting |

> **Rule of thumb:** Use `shareReplay(1)` when you need multiple components to share a single HTTP request result.

---

### How Long Does shareReplay Cache?

By default (`shareReplay(1)`), the cache lifetime depends on the `refCount` option:

| Config | Cache Lifetime |
| :--- | :--- |
| `shareReplay(1)` *(legacy default)* | **Forever** — even after all subscribers unsubscribe |
| `shareReplay({ bufferSize: 1, refCount: true })` | **Until all subscribers unsubscribe**, then clears |
| `shareReplay({ bufferSize: 1, refCount: false })` | **Forever** — same as legacy |

```typescript
// Cache persists even after all unsubscribes (potential memory leak)
shareReplay(1)

// Preferred — cache clears when no subscribers remain
shareReplay({ bufferSize: 1, refCount: true })
```

**Do N subscribers get the same value?**

Yes — with a timing nuance:

```typescript
const data$ = this.http.get('/api/data').pipe(
  shareReplay({ bufferSize: 1, refCount: true })
);

data$.subscribe(v => console.log('Sub 1:', v)); // triggers the HTTP request
data$.subscribe(v => console.log('Sub 2:', v)); // joins the same request — both get the same response
data$.subscribe(v => console.log('Sub 3:', v)); // subscribes after response — gets cached value immediately
```

The `bufferSize: 1` means it replays the **last 1 emitted value** to any late subscriber.

**Practical recommendation for HTTP**

```typescript
// In a service
readonly data$ = this.http.get<MyData>('/api/data').pipe(
  shareReplay({ bufferSize: 1, refCount: true })
);
```

- First subscriber triggers **one** HTTP call
- All concurrent subscribers **share** it
- Late subscribers get the cached response **immediately**
- After **all** unsubscribe, cache is cleared — the next subscription triggers a **new** HTTP call

### Unsubscribing from Multiple Subscriptions

**Option 1: Subscription array**

Store all subscriptions in an array and iterate to unsubscribe.

```typescript
export class MyComponent implements OnInit, OnDestroy {
  private subscriptions: Subscription[] = [];

  ngOnInit() {
    this.subscriptions.push(
      this.service.getData().subscribe(data => console.log(data)),
      this.service.getOtherData().subscribe(data => console.log(data))
    );
  }

  ngOnDestroy() {
    this.subscriptions.forEach(sub => sub.unsubscribe());
  }
}
```

**Option 2: `takeUntil` with a destroy Subject (recommended)**

Avoids tracking subscriptions manually. All piped observables auto-complete when `destroy$` emits.

```typescript
export class MyComponent implements OnInit, OnDestroy {
  private readonly destroy$ = new Subject<void>();

  ngOnInit() {
    this.service.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => console.log(data));

    this.service.getOtherData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => console.log(data));
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Option 3: `takeUntilDestroyed` (Angular 16+)**

The cleanest modern approach — no `ngOnDestroy` needed.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class MyComponent {
  constructor(private service: MyService) {
    this.service.getData()
      .pipe(takeUntilDestroyed())
      .subscribe(data => console.log(data));
  }
}
```

| Approach | Scalable | Clean | Angular version |
| :--- | :--- | :--- | :--- |
| Manual `unsubscribe()` | Poor | Poor | Any |
| Subscription array | OK | OK | Any |
| `takeUntil` + `destroy$` | Good | Good | Any |
| `takeUntilDestroyed()` | Good | Best | 16+ |

### RxJS: Higher-Order Mapping Operators

Higher-order mapping operators transform each emission from a source observable into a new inner observable, then decide how to subscribe to and merge those inner observables.

| Operator | Inner observable behavior | Use when |
| :--- | :--- | :--- |
| `switchMap` | Cancels the previous inner observable when a new source value arrives | HTTP search, autocomplete — only latest result matters |
| `mergeMap` | All inner observables run concurrently | Parallel independent requests |
| `concatMap` | Inner observables run one at a time, in order | Sequential operations where order matters |
| `exhaustMap` | Ignores new source values while an inner observable is active | Prevent duplicate form submits |

**switchMap — cancel and switch to latest**

```typescript
import { fromEvent } from 'rxjs';
import { switchMap, debounceTime } from 'rxjs/operators';

// Only the latest HTTP request survives — previous ones are cancelled
fromEvent(searchInput, 'input').pipe(
  debounceTime(300),
  switchMap(event => this.http.get(`/api/search?q=${event.target.value}`))
).subscribe(results => this.results = results);
```

**mergeMap — concurrent inner observables**

```typescript
import { from } from 'rxjs';
import { mergeMap, delay } from 'rxjs/operators';

const fetchAdditionalData = (userId: number) =>
  of(`Data for ${userId}`).pipe(delay(1000));

// All requests run in parallel — order of results not guaranteed
from([1, 2, 3]).pipe(
  mergeMap(id => fetchAdditionalData(id))
).subscribe(console.log);
```

**concatMap — sequential, ordered**

```typescript
import { from, of } from 'rxjs';
import { concatMap, delay } from 'rxjs/operators';

const fetchUserDetails = (userId: number) =>
  of(`User details for ${userId}`).pipe(delay(1000));

// Requests run one after another — order is preserved
from([1, 2, 3]).pipe(
  concatMap(id => fetchUserDetails(id))
).subscribe(console.log);
// Logs: "User details for 1", then "User details for 2", then "User details for 3"
```

### RxJS: Error Handling Operators

**catchError — handle errors and recover**

```typescript
import { catchError } from 'rxjs/operators';
import { of, throwError } from 'rxjs';

this.http.get('/api/data').pipe(
  catchError(error => {
    console.error('Request failed:', error);
    return of([]); // return a fallback value so the stream continues
    // OR: return throwError(() => error); // re-throw to propagate
  })
).subscribe(data => this.data = data);
```

**retry — automatically resubscribe on error**

```typescript
import { retry, catchError } from 'rxjs/operators';

this.http.get('/api/data').pipe(
  retry(3),           // retry up to 3 times before failing
  catchError(error => of([]))
).subscribe(data => this.data = data);
```

**finalize — cleanup regardless of success or error**

```typescript
import { finalize } from 'rxjs/operators';

this.http.get('/api/data').pipe(
  finalize(() => this.isLoading = false) // always runs — success or error
).subscribe(data => this.data = data);
```

**Combining operators in practice**

```typescript
this.http.get('/api/data').pipe(
  retry(2),
  catchError(err => {
    this.showError(err.message);
    return of([]);
  }),
  finalize(() => this.spinner = false)
).subscribe(data => this.data = data);
```

### RxJS: Combining Observables

| Operator | Behavior | Completes when |
| :--- | :--- | :--- |
| `forkJoin` | Waits for ALL observables to complete, emits last values as array | All complete |
| `combineLatest` | Emits whenever ANY observable emits, using latest value from each | Any source completes |
| `zip` | Pairs emissions by index — waits for one from each source | Any source completes |

**forkJoin — parallel HTTP requests, wait for all**

```typescript
import { forkJoin } from 'rxjs';

// Kick off both requests simultaneously, wait for both to finish
forkJoin({
  user: this.http.get('/api/user/1'),
  orders: this.http.get('/api/orders?userId=1')
}).subscribe(({ user, orders }) => {
  this.user = user;
  this.orders = orders;
});
```

**combineLatest — react to latest value from any source**

```typescript
import { combineLatest } from 'rxjs';

// Re-emits whenever either filter$ or sort$ changes
combineLatest([this.filter$, this.sort$]).pipe(
  switchMap(([filter, sort]) => this.http.get(`/api/items?filter=${filter}&sort=${sort}`))
).subscribe(items => this.items = items);
```

**zip — pair emissions by index**

```typescript
import { zip } from 'rxjs';

zip(names$, ages$).subscribe(([name, age]) => {
  console.log(`${name} is ${age}`); // pairs: name[0] with age[0], name[1] with age[1], ...
});
```

**Key differences: forkJoin vs combineLatest**

```typescript
// forkJoin: emits ONCE after all complete
forkJoin([obs1$, obs2$]).subscribe(([v1, v2]) => console.log('Both done:', v1, v2));

// combineLatest: emits EVERY TIME any source emits (after all have emitted at least once)
combineLatest([obs1$, obs2$]).subscribe(([v1, v2]) => console.log('Latest:', v1, v2));
```

## Dependency Injection & Services

### Dependency Injection, Angular Injectors, and Providers

* **Injector:**  
  * An **Injector** is responsible for creating instances of services or other components and injecting them into component classes that request them.  
  * Angular creates injectors for you as it executes the app, starting with the root injector that it creates during the bootstrap process.  
* **Providers:**  
  * A **Provider** tells an injector how to create the service. You must configure an injector with a provider before that injector can create a service instance.  
  * Dependencies are defined by providers. A provider is a recipe for creating a dependency and tells the injector how to obtain or create an instance of a particular service or value.  
  * Providers are specified in the **@Injectable()** decorator for services and in the **providers** array of the **@NgModule()** decorator for modules.

```typescript
// Example of a service with @Injectable decorator
@Injectable({
  providedIn: 'root', // providedIn: 'root' registers the service in the root injector
})
export class MyService {}
```


* **Injection Token:**  
  * Dependencies are identified using injection tokens, which are special objects used as keys to look up services in the injector.  
  * Angular provides a built-in **‘Injectable’** decorator to create injection tokens.

```typescript
// Example of using an injection token
import { InjectionToken } from '@angular/core';
export const MY_TOKEN = new InjectionToken<string>('MyToken');
```

* **Dependency Injection in Components:**  
  * Components, services or other Angular artifacts can declare their dependencies in their constructors.  
  * Angular automatically injects the dependencies when creating instances of these components.

```typescript
// Example of a component with injected service
import { Component } from '@angular/core';
import { MyService } from './my.service';
@Component({
  selector: 'app-my-component',
  template: '<p>{{ message }}</p>',
})
export class MyComponent {
  constructor(private myService: MyService) {
    this.message = myService.getMessage();
  }
}
```

    

* **Hierarchical Injection:**  
  * Angular’s dependency injection system is hierarchical. Each component or service has its own injector, and there is a hierarchy of injectors based on the component tree.  
  * Injectors are inherited, which means that if a given injector cannot resolve a dependency, it asks the parent injector to resolve it.


* **providedIn: ‘root’ and Singleton Pattern:**  
  * When a service is provided at the root level(providedIn: ‘root’), Angular creates a single instance of that service for the entire application. This follows the singleton pattern, ensuring that the same instance is reused whenever the service is injected.

```typescript
// Example of a service provided at the root level
@Injectable({
  providedIn: 'root',
})
export class MySingletonService {}
```

    

Basically Angular associates an Injector corresponding to each Module/Component. Each Injector is responsible for looking up for services registered with it, instantiating that service, and then injecting the service into the component constructor method.  
A service can be registered in the providers array, either at the component-level or at the module-level.  
A component can get services from its own injector, from the injectors of its component ancestors, from the injectors of its parent NgModule, or from the root injector.

### Dependency Provider & Dependency Injector

* A dependency provider configures an injector with a DI token, which that injector uses to provide the concrete, runtime version of a dependency value.  
* The injector relies on the provider configuration to create instances of the dependencies that it injects into components, directives, pipes, and other services.  
* You must configure an injector with a provider, or it won’t know how to create the dependency.

### What are the advantages of using dependency injection in Angular?

**Dependency Injection (DI)** is a core design principle in Angular, and its use brings several advantages to the development of applications. Here are the key advantages of using dependency injection in Angular:

**Modularity and Reusability:**  
Dependency injection promotes the creation of modular and reusable components. Components, services, and other Angular artifacts can be developed independently and then injected wherever needed. This modularity enhances code organization and maintainability.

**Loose Coupling:**  
Components and services in Angular are loosely coupled through dependency injection. This means that components are not responsible for creating their dependencies but instead receive them through their constructors. This reduces the interdependence between components and makes it easier to replace or modify dependencies without affecting the entire application.

**Unit Testing:**  
Dependency injection simplifies unit testing. By injecting mock or test versions of services into components during testing, developers can isolate and test individual components without relying on the actual implementation of their dependencies. This makes it easier to write robust and isolated unit tests.

**Configuration and Consistency:**  
Dependencies are configured at the module level in Angular. This central configuration helps maintain consistency across the application. If a service is provided at the root level, it becomes a singleton instance shared throughout the application, ensuring a consistent state.

**Scalability and Maintainability:**  
As an application grows, dependency injection facilitates scalability and maintainability. The modular structure allows for the addition of new features and the modification of existing ones without disrupting the entire application. Developers can focus on building small, self-contained modules that interact seamlessly.

**Easier Refactoring:**  
DI makes refactoring easier by enabling developers to make changes to services or components without having to modify all the places where they are used. Since dependencies are injected rather than hardcoded, changes to the internal implementation of a service have minimal impact on the rest of the application.

**Promotes Best Practices:**  
Dependency injection encourages the use of best practices, such as writing services for business logic, using interfaces for abstraction, and keeping components focused on presentation. This adherence to best practices enhances code quality and makes the application more maintainable.

**Centralised Control:**  
Angular's DI system provides centralized control over the instantiation and management of dependencies. This centralization allows Angular to efficiently handle the creation and disposal of instances, contributing to the overall performance of the application.

In summary, dependency injection in Angular brings advantages such as modularity, loose coupling, unit testing support, configuration consistency, scalability, maintainability, easier refactoring, and the promotion of best practices. These benefits collectively contribute to the development of robust, maintainable, and scalable Angular applications.

### What are Angular services, and why are they used?

Angular services are a crucial part of the Angular framework, providing a way to organize and share code across different parts of an application. Angular services are singleton objects that are instantiated only once during the lifetime of an Angular application. They play a central role in promoting modularity, reusability, and maintaining a separation of concerns in Angular applications.

Key characteristics for using Angular services:

1. **Singleton Instances:**  
   * Angular services are designed as singleton instances. This means that there is only one instance of a service in the entire Angular application.  
   * The singleton pattern ensures that services maintain a single, shared state across the application, promoting consistency and avoiding unnecessary duplication.  
2. **Code Organization:**  
   * Services help organize code by encapsulating specific functionalities or business logic. They provide a way to modularize an application by separating concerns and keeping the codebase more maintainable.  
3. **Reuse and Sharing:**  
   * Angular services are intended for reuse across different components, directives, or other services within the application.  
   * By encapsulating functionality in services, developers can share common functionalities among different parts of the application, avoiding code duplication and promoting a DRY (Don’t Repeat Yourself) coding philosophy.  
4. **Dependency Injection:**  
   * Angular services are injectable, and they leverage the Angular Dependency Injection(DI) system.  
   * Dependency Injection allows components, directives, and other services to declare their dependencies in the constructor, and Angular’s injector automatically provides the required services when creating instances of these components or services.  
5. **Separation of Concerns:**  
   * Services contribute to the separation of concerns by isolating specific functionalities into independent units. This separation makes it easier to understand, maintain, and test different parts of the application.  
6. **Async Operations and HTTP Requests**:  
   * Services are commonly used to encapsulate asynchronous operations, such as HTTP requests. This allows for a centralized location to handle data fetching and manipulation, providing a clean abstraction for components.  
7. **State Management:**  
   * Services can be used for managing state within an application. They provide a central place to store and manipulate data that needs to be shared among multiple components.  
   * While Angular services can manage state, for more advanced state management, developers often use additional libraries like NgRx for a more reactive and predictable state management approach.  
8. **Lifecycle Hooks:**  
   * Services can implement Angular lifecycle hooks, such as OnInit or OnDestroy, allowing them to perform initialization or cleanup tasks at specific points in their lifecycle.  
9. **Testing:**  
   * Angular services are easily testable. Since they are independent and can be injected into components, services can be unit-tested in isolation.  
   * Angular’s dependency injection system allows for easy mocking of services during testing.

In summary, Angular services are essential for structuring, organizing, and maintaining large-scale Angular applications. They enable the sharing of functionality, encapsulation, of business logic, and the creation of reusable, testable, and modular code. By following the principles of dependency injection and separation of concerns, Angular services contribute to the overall architecture and maintainability of Angular applications.

### How would you create a singleton service in Angular, and what are the potential pitfalls?

In Angular, creating a singleton service is a straightforward process. By default, when you provide a service at the root level using the providedIn: 'root' property in the @Injectable decorator, Angular creates a single instance of that service and shares it across the entire application. Here's an example:

```typescript
// Example of a singleton service
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'root',
})
export class MySingletonService {
  // Service logic...
}
```

In this example, the **providedIn: 'root'** property ensures that Angular creates a single instance of MySingletonService for the entire application. Any component, directive, or other service that injects MySingletonService will receive the same instance.

**Potential Pitfalls:**  
While creating singleton services in Angular is generally straightforward, there are some pitfalls to be aware of:

**Module Level Providing:**  
While **providedIn: 'root'** is a common and convenient way to create singleton services, it's also possible to provide a service at the module level. If a service is provided at the module level, it becomes a singleton within that module's context, but not globally across the entire application.

```typescript
// Example of providing a service at the module level
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'my-module',
})
export class MyModuleService {
  // Service logic...
}
```

Be cautious when providing a service at the module level if a global singleton is intended.

**Manual Instantiation:**  
Avoid manually instantiating singleton services using the new keyword. Always rely on Angular's dependency injection system to provide instances.

```typescript
// Avoid manually instantiating a service
const myService = new MySingletonService(); // Avoid doing this
```

Manually creating instances can lead to unexpected behavior, and it doesn't take advantage of Angular's dependency injection.

**Provider Redundancy:**  
Be mindful of redundant providers. If the same service is provided in multiple modules, Angular will create separate instances for each module, breaking the global singleton pattern.

```typescript
// Redundant providers can lead to multiple instances
@NgModule({
  providers: [MySingletonService],
})
export class ModuleA {}
@NgModule({
  providers: [MySingletonService],
})
export class ModuleB {}
```

To ensure a global singleton, only provide the service in the root module or use providedIn: 'root'.

**Lazy Loaded Modules:**  
When using lazy-loaded modules, be aware that services provided in a lazy-loaded module create a separate instance for that module. If a singleton is required, use providedIn: 'root' or provide the service in a non-lazy-loaded module.

```typescript
// Example of providing a service in a non-lazy-loaded module
@NgModule({
  providers: [MySingletonService],
})
export class NonLazyLoadedModule {}
```

Alternatively, you can provide the service in the root module to ensure a global singleton.

By understanding these potential pitfalls and following best practices, you can effectively create and use singleton services in Angular while ensuring consistent behavior across your application.


### DI Scope: Root vs Component-Level Providers

When a service is declared with `providedIn: 'root'` AND also listed in a component's `providers: []`, Angular creates **two separate instances**.

| Configuration | Instances | Scope |
| :--- | :--- | :--- |
| `providedIn: 'root'` | 1 (singleton) | Entire application |
| `providedIn: SomeModule` | 1 per module (new instance for lazy-loaded modules) | Module |
| Component `providers: [MyService]` | 1 per component declaration | Component + its children |
| Both root + component `providers` | 2 | Root singleton + component-scoped copy |

**Practical implication:**

```typescript
// service.ts
@Injectable({ providedIn: 'root' })
export class MyService { state = 0; }

// component-a.component.ts — uses the ROOT singleton
@Component({ selector: 'app-a', template: '' })
export class ComponentA {
  constructor(public svc: MyService) {} // gets root instance
}

// component-b.component.ts — creates its OWN instance
@Component({
  selector: 'app-b',
  template: '',
  providers: [MyService]  // scoped instance, isolated from root
})
export class ComponentB {
  constructor(public svc: MyService) {} // gets a NEW instance
}
```

`ComponentA.svc` and `ComponentB.svc` are different objects — state changes in one do not affect the other.

> **Rule:** Only add a service to `providers: []` when you intentionally need an isolated instance per component (e.g., a stateful form service). Otherwise, rely on `providedIn: 'root'` to share state.

## Forms

### Angular Forms

RxJS is about bringing the principles of functional programming to JavaScript.  
[https://dzone.com/articles/angular-observables-and-promises-how-to-use-them](https://dzone.com/articles/angular-observables-and-promises-how-to-use-them)

### Template Driven Forms vs. Reactive Forms

- Template Driven Forms need the FormsModule, while Reactive forms need the ReactiveFormsModule.  
- Template Driven Forms are based only on template directives, while Reactive forms are defined programmatically at the level of the component class.  
- Reactive Forms are a better default choice for new applications, as they are more powerful and easier to use.  
- Reactive approach removes validation logic from the template, keeping templates cleaner.  
- Reactive forms are easier to use in general and support better more advanced use cases via its Observable-based API.

### How would you handle form validation and error handling in Angular?

Handling form validation and error handling is an essential aspect of building robust and user-friendly Angular applications. Angular provides a powerful and flexible set of tools for form validation and error management. Here's a guide on how to handle form validation and errors in Angular:

**Form Validation:**

1. **Template-Driven Forms:**  
   For template-driven forms, Angular provides various directives for validation, such as **ngModel**, **ngModelGroup**, and **ngForm**.  
   Use attributes like required, min, max, pattern, etc., to define validation rules directly in the template.

```html
<form #myForm="ngForm">
  <input type="text" name="username" ngModel required>
  <div *ngIf="myForm.controls.username.errors?.required">Username is required</div>
</form>
```

2. **Reactive Forms:**  
   Reactive forms provide a more programmatic approach to form validation.  
   Create a form model using the **FormGroup** and **FormControl** classes, and apply validation rules using validators from **@angular/forms**.

```typescript
import { FormGroup, FormControl, Validators } from '@angular/forms';
const myForm = new FormGroup({
  username: new FormControl('', [Validators.required]),
});
```

3. **Dynamic Validation:**  
   You can apply dynamic validation based on user input or other conditions. Use the setValidators method to dynamically update validation rules.

```typescript
// Dynamic validation example
myForm.get('username').setValidators([Validators.required, Validators.minLength(3)]);
```

4. **Cross-Field Validation:**  
   Implement cross-field validation for scenarios where the value of one field depends on the value of another.  
   Use the **FormGroup** level validation.

```typescript
const myForm = new FormGroup({
  password: new FormControl(''),
  confirmPassword: new FormControl(''),
}, { validators: passwordMatchValidator });
```

```typescript
function passwordMatchValidator(form: FormGroup) {
  const password = form.get('password').value;
  const confirmPassword = form.get('confirmPassword').value;
  return password === confirmPassword ? null : { mismatch: true };
}
```

**Error Handling:**

1. **Displaying Form Errors:**  
   Use Angular's error handling features to display form errors to users.  
   Leverage the errors property of form controls to access validation errors.

```html
<div *ngIf="myForm.controls.username.errors?.required">Username is required</div>
```

2. **Styling Invalid Controls:**  
   Apply styles to visually highlight invalid form controls using Angular's ng-invalid, ng-valid, ng-dirty, and ng-pristine classes.

```html
<input type="text" name="username" ngModel required [ngClass]="{'is-invalid': myForm.controls.username.invalid && myForm.controls.username.touched}">
```

3. **Accessing Errors in Code:**  
   Programmatically access form errors within your component class to perform custom error handling logic.

```typescript
if (this.myForm.hasError('mismatch')) {
  // Handle mismatch error
}
```

4. **Async Validation:**  
   Implement asynchronous validation for scenarios where validation requires making asynchronous calls (e.g., checking the availability of a username).

```typescript
const myForm = new FormGroup({
  username: new FormControl('', [Validators.required], [asyncUsernameValidator]),
});
```

```typescript
function asyncUsernameValidator(control: AbstractControl): Observable<ValidationErrors | null> {
  // Perform async validation, return an observable with validation result
}
```

5. **Global Error Handling:**  
   * **Global HTTP Error Handling with HttpInterceptor:**  
     Create an ‘**HttpInterceptor**’ to intercept HTTP requests and responses. The interceptor can be used to handle errors globally.

```typescript
// http-error.interceptor.ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
@Injectable()
export class HttpErrorInterceptor implements HttpInterceptor {
  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(request).pipe(
      catchError((error: HttpErrorResponse) => {
        // Handle HTTP errors globally
        console.error('HTTP error occurred:', error);
        return throwError(error);
      })
    );
  }
}
```

     

     Register the interceptor in your ‘**app.module.ts**’:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { HttpErrorInterceptor } from './http-error.interceptor';
@NgModule({
  imports: [HttpClientModule],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: HttpErrorInterceptor,
      multi: true,
    },
  ],
})
export class AppModule {}
```

     

     

   * **Global Error Handling for Uncaught Exceptions:**  
     Create a global error handler service that implements Angular’s ‘ErrorHandler’ interface.

```typescript
// global-error-handler.service.ts
import { ErrorHandler, Injectable } from '@angular/core';
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(error: any): void {
    // Handle uncaught exceptions globally
    console.error('An uncaught error occurred:', error);
    // Optionally, you can log the error to a logging service.
  }
}
```

   Register the global error handler in your ‘app.module.ts’:

```typescript
// app.module.ts
import { NgModule, ErrorHandler } from '@angular/core';
import { GlobalErrorHandler } from './global-error-handler.service';
@NgModule({
  providers: [
    {
      provide: ErrorHandler,
      useClass: GlobalErrorHandler,
    },
  ],
})
export class AppModule {}
```

**Additional Considerations:**

- You can extend the ‘**HttpErrorInterceptor**’ to handle specific HTTP error status codes or customize error messages based on your application’s requirements.  
- The ‘**GlobalErrorHandler**’ can be extended to include additional logging, notification, or error reporting features.

## Routing & Navigation

### Angular Route Guards

Angular Guards are interfaces that tell the router if it allows or not the user to go to a given URL. To make the decision, Angular relies on a class implementing the Guard interface and outputs true or false.  
There are **five kinds of guards**:  
**CanActivate**: It is called for each request, and allows or forbids the access to the user.  
**CanActivateChild**: Same as previous one, but for child routes.  
**CanDeactivate**: This Guard is called each time a user wishes to leave the current route (for saving a form data when a user leaves early).  
**CanLoad**: This Guard checks if a user can access a lazily loaded module route.  
**Resolve**: This Guard allows you to fetch some data before going to a given route.

### Angular Routing

[20 Angular Routing Interview Questions and Answers](https://climbtheladder.com/angular-routing-interview-questions/)

1. What is Angular Routing?  
   Angular routing is a way to manage the different states or views of an Angular application. It lets you define different URLs for different parts of your app, and map those URLs to the corresponding Angular components. Angular routing also lets you specify which components should be displayed when a user navigates to a specific URL.  
2. Can you list the main features of Angular Router?  
   The Angular Router is a powerful routing library that allows developers to create single-page applications with rich routing capabilities. The router is easy to use and can be configured to work with a variety of different backends. The router also supports lazy loading, which can improve the performance of large applications.  
3. How do you define a route in an Angular application?  
4. Can you explain how to pass parameters via routes in Angular?  
   In Angular, you can pass parameters to routes in order to specify which data you would like to access or manipulate. To do this, you will need to use the **routerLink directive**. The routerLink directive takes an array of parameters, where each parameter is a key-value pair. The key is the name of the parameter, and the value is the data you wish to pass.  
5. What are auxiliary routes? When would you use them?  
   Angular supports the concept of auxiliary routes, which allow you to set up and navigate multiple independent routes in a single app. Auxiliary routes allow the user to access or toggle portions of the page, such as a side-bar or dialog, using the URL.  
   Auxiliary routes are used to add additional functionality to an Angular application beyond the normal routing. They can be used to create child routes, lazy-loaded routes, or to provide additional data to a component.  
6. What’s the difference between routerLink and href? Which one should be preferred when defining routes?  
   The main difference between routerLink and href is that **routerLink** is used to define routes within your Angular application, while **href** is used to define routes to external resources. When defining routes within your Angular application, you should always use routerLink to ensure that the routing logic is handled correctly.  
7. How can we make use of wildcard routes with Angular?  
   Wildcard routes are useful for situations where we want to redirect users to a specific page if they enter an incorrect URL. For example, if we have a website with the URL www.example.com/products, and a user tries to go to www.example.com/product, we can redirect them to the correct page using a wildcard route.  
8. In what ways can we redirect requests using Angular routers?  
   There are a few ways to redirect requests using Angular routers. One way is to use the routerLink directive. This directive can be used on anchor tags and will automatically generate the href attribute based on the route you provide. Another way to redirect requests is to use the **router.navigate method**. This method takes an array of commands and a navigation extras object. The navigation extras object can be used to specify things like query parameters and fragment identifiers.  
9. Can you show me an example of implementing Lazy Loading in an Angular app?  
   Lazy loading is a technique used in Angular to load modules only when they are needed, instead of loading them all upfront. This can improve the performance of an Angular app, especially if there are a lot of modules. To implement lazy loading, you can use the loadChildren() method in the route configuration. This method takes a function that returns a promise for the module that should be loaded. The module will then be loaded when the route is accessed.  
10. What does lazy loading mean?  
    Lazy loading is a technique used in Angular to load modules only when they are needed, instead of loading all modules upfront. This can improve the performance of an Angular application, since the browser will not need to load unnecessary code. Lazy loading is typically used for large applications with many modules, where not all modules need to be loaded immediately.  
11. What’s the difference between **loadChildren** and **children** in Angular?  
    The difference between loadChildren and children in Angular is that **loadChildren** is used to lazy-load components, while **children** is used to eagerly load components. With loadChildren, the component is not loaded until the user navigates to the route. With children, the component is loaded immediately.  
12. How can I get the URL of the current page in my Angular application?  
    You can use the Router service to get the URL of the current page in your Angular application. The Router service has a method called routerState that returns an observable of the current router state. You can subscribe to this observable to get the current URL.  
13. Can you describe how navigation works in Angular?  
    Angular uses a component-based router to handle navigation. This means that there is a designated component for each route, and the router will automatically render the correct component for the current route. The router also takes care of automatically updating the URL to match the current route, and it can also handle things like passing parameters to components.  
14. Is it possible to render specific components based on the route in Angular? If yes, then how?  
    Yes, it is possible to render specific components based on the route in Angular. This can be done using the router-outlet directive. The router-outlet directive tells the router where to place the components that are rendered based on the current route.  
    

### Discuss lazy loading and its benefits in the context of Angular routing

Lazy loading is a technique used in Angular to load modules or components on demand, rather than loading everything when the application initially starts. In the context of Angular routing, lazy loading allows you to load parts of your application only when the user navigates to specific routes, which can significantly improve the application's performance and reduce the initial loading time.

Here are some key aspects and benefits of lazy loading in the context of Angular routing:

1. **Reduced Initial Load Time:**  
   One of the primary benefits of lazy loading is a reduced initial load time for your application. Instead of downloading the entire application at once, only the essential modules are loaded initially. This is especially beneficial for large applications with numerous features and modules.  
     
2. **Faster Page Loads:**  
   Lazy loading helps in achieving faster page loads, particularly for applications with a complex routing structure. Users only download the code required for the current view, making the initial page load quicker.

   

3. **Improved User Experience:**  
   Users experience faster load times when navigating between different sections or features of the application. This can lead to a smoother and more responsive user experience.  
     
4. **Optimized Bandwidth Usage:**  
   Lazy loading optimizes bandwidth usage by fetching only the necessary resources when navigating to a specific route. This is advantageous for users on slower or limited network connections.  
     
5. **Code Splitting:**  
   Lazy loading is often achieved through code splitting, a technique where the application code is divided into smaller chunks. Each chunk corresponds to a different module or feature.  
   Code splitting allows for better management of the application's size, and it helps in optimizing the loading process.

   

6. **Smaller Initial Bundle Size:**  
   Lazy loading results in a smaller initial bundle size because the application doesn't need to include the code for all possible routes upfront. Only the code for the initial route is loaded initially.

   

7. **Scalability:**  
   As your application grows, lazy loading becomes essential for managing its size and ensuring that users only download what they need. This scalability is crucial for maintaining a performant application as new features are added.

   

8. **Dynamic Loading:**  
   Lazy loading enables dynamic loading of modules based on user interactions. For example, additional modules can be loaded when a user clicks on a specific tab or navigates to a particular section of the application.

   

9. **Granular Control:**  
   Lazy loading provides granular control over the loading process. Developers can decide which routes or modules should be lazily loaded based on the application's requirements and the user's navigation patterns.

   

10. **Angular Router Integration**:  
    Angular's router seamlessly integrates with lazy loading. You can configure lazy-loaded routes using the loadChildren property in the route configuration.

```typescript
const routes: Routes = [
  { path: 'lazy', loadChildren: () => import('./lazy/lazy.module').then(m => m.LazyModule) },
  // Other routes...
];
```

In summary, lazy loading in Angular routing is a powerful optimization technique that improves the overall performance and user experience of your application. It allows you to load modules on demand, reducing the initial bundle size, improving page load times, and providing better scalability as your application grows. Lazy loading is a recommended practice for Angular applications, particularly those with complex routing structures or a significant amount of functionality.

## Component Communication & Lifecycle

### Sharing data between Angular Components: Four Methods

* Sharing data via Input  
  Most common and straight-forward method of sharing data.  
  * Sharing data via ViewChild  
    ViewChild allows a component to be injected into another, giving the parent access to its attributes and functions. One caveat, however, is that the child won’t be available until after the view has been initialized. This means we need to implement the AfterViewInit lifecycle hook to receive the data from the child.  
  * Sharing data via Output and EventEmitter

  Another way to share data is to emit data from the child, which can be listened to by the parent. The parent can subscribe to the event that’s outputted by the child component.

  * Sharing data with a Service (*Subject / BehaviorSubject*)

  When implementing component communication among sibling components in Angular, both ‘Subject’ and ‘BehaviorSubject’ from the RxJS library can be used. The choice between them depends on the specific requirements of your application and how you want to handle the initial value.

- If you don’t need an initial value or are handling it differently, a regular Subject is sufficient.  
- If you need to provide an initial value to subscribers or want subscribers to receive the most recent value immediately upon subscription, then use ‘BehaviorSubject’.

### Difference between ngOnInit() and constructor()

\- Constructor and OnInit seem the same, but they are called at different phases. Angular has two phases: the bootstrapping phase and the change detection phase.  
\- Angular does not have any control over when the constructor gets called or initiated on page load.  
\- We use constructor() for all the initialization/declaration.  
\- It’s better to avoid writing actual work in the constructor.  
\- The constructor() should only be used to initialize class members but shouldn’t do actual “work”.  
\- We should use constructor() to set up Dependency Injection, Initialisation of class fields, etc.  
\- ngOnInit() is a better place to write “actual work code” that we need to execute as soon as the class is instantiated.  
\- Like loading data from a database \- such code should be written in ngOnInit().  
\- Angular does not guarantee that @Input properties have been properly initialized with their values if accessed inside a constructor.  
\- Anything that is NOT Input OR Output should be initialized in the constructor.  
\- Only Input & Output should be initialized in the ngOnInit()

### Angular Lifecycle Hooks

Angular **lifecycle hooks are methods** that allow developers to **tap into key moments in the lifecycle of a component or directive**, enabling them to **perform actions at different stages**, such as **initialization**, **change detection**, and **destruction**.  
Angular manages components and directives for us when it creates them, updates them, or destroys them.

1. **ngOnChanges:**  
   **When it’s called:** Called whenever one or more data-bound input properties change.  
   **Use case:** Useful for responding to changes in input properties and performing actions based on these changes.  
2. **ngOnInit:**  
   **When it’s called:** Called once, after the component is initialized.  
   **Use case:** Ideal for initialization logic, such as fetching data from a server or setting up subscriptions.  
3. **ngDoCheck:**  
   **When it’s called:** Called during every change detection cycle. It is called after the default change detector has checked the component’s data-bound properties and the component’s view.  
   **Use case:** This hook provides an opportunity for developers to implement custom change detection logic. Developers can perform additional checks or actions based on changes that are not automatically detected by Angular’s default change detection mechanism.  
   **Typical Scenarios for use:** Checking for changes in complex data structures or objects that Angular’s default change detection might not automatically recognize. Or implementing custom change detection strategies for optimization in specific scenarios.  
4. **ngAfterContentInit:**  
   **When it’s called:** Called once, after the content (projected content \~ ng-content) has been initialized.  
   **Use case:** Suitable for initialisation logic that depends on content projected into the component.  
5. **ngAfterContentChecked:**  
   **When it’s called:** Called after every check of the content (projected content).  
   **Use case:** Useful for performing additional checks or operations after the content has been checked.  
6. **ngAfterViewInit:**  
   **When it’s called:** Called once, after the component’s view (and child views) have been initialized.  
   **Use case:** Useful for interacting with the DOM or performing operations that require the view to be fully rendered.  
7. **ngAfterViewChecked:**  
   **When it’s called:** Called after every check of the component’s view (and child views).  
   **Use case:** Useful for performing additional checks or operations after the view has been checked.  
8. **OnDestroy:**  
   **When it’s called:** Called just before the component is destroyed.  
   **Use case:** Useful for cleaning up resources, such as unsubscribing from observables or detaching event listeners, to prevent memory leaks.

**Difference between ngOnInit and ngAfterViewInit**  
**ngOnInit()** is called right after the directive's data-bound properties have been checked for the first time, and before any of its children have been checked. It is invoked only once when the directive is instantiated. **ngAfterViewInit()** is called once after **ngAfterContentChecked()**.

**How would you handle data initialization in a component with ngOnInit and constructor?**  
The constructor is used for basic initialization, while ngOnInit is recommended for more complex initialization tasks that depend on input properties being set up. It's common to perform a one-time setup in ngOnInit.

**What is the significance of ngAfterContentInit and ngAfterContentChecked?**  
ngAfterContentInit is called after content projection (ng-content) has been initialized, and ngAfterContentChecked is called after each check of the content. They are useful for tasks related to projected content.

**Explain ngDoCheck lifecycle hook.**  
Unlike other lifecycle hooks, ngDoCheck is not triggered automatically by Angular; rather, it is called during every change detection cycle. This hook provides developers with an opportunity to perform manual checks for changes and optimize the performance of their Angular applications.  
**Why Use ngDoCheck:**  
Custom Change Detection:  
Angular's default change detection mechanism is powerful, but there are scenarios where a more fine-grained and optimized approach is necessary. ngDoCheck allows developers to implement custom logic to check for changes in specific properties or conditions.

Performance Optimization:  
By implementing custom change detection in ngDoCheck, developers can skip unnecessary work when changes are not detected. This can lead to performance improvements, especially in scenarios where frequent and intensive change detection may impact the application's responsiveness.

Avoiding Unnecessary Operations:  
In some cases, certain operations or calculations may be expensive and do not need to be performed on every change detection cycle. With ngDoCheck, developers can selectively decide when to execute these operations based on their own conditions.

**Can you implement a custom change detection strategy? If so, how?**  
Answer: **Yes**, you can implement a custom change detection strategy by creating a class that implements the **ChangeDetectorRef** interface and overriding its methods. You can then use this custom change detector in the @Component decorator. This approach is more advanced and is typically used in specific optimization scenarios.

**Explain the role of ChangeDetectorRef in custom change detection.**  
Answer:  
The **ChangeDetectorRef** provides methods for manually interacting with the change detection mechanism. Developers can use methods like **markForCheck()** to mark a component and its ancestors for check, ensuring that change detection is triggered. This is particularly useful in scenarios where changes occur outside the Angular zone.

**How does Angular handle dynamic components and their lifecycle hooks?**  
Answer:  
Dynamic components created using **ComponentFactoryResolver** have their lifecycle hooks executed similarly to statically defined components. The key is to manually attach them to the change detection tree using **ApplicationRef** or **ChangeDetectorRef**. Developers need to ensure that the component is properly destroyed to avoid memory leaks.

**What is Component ChangeDetectionStrategy**?  
[https://blogs.halodoc.io/understanding-angular-change-detection-strategy/](https://blogs.halodoc.io/understanding-angular-change-detection-strategy/)  
**Change Detection**: Change Detection means updating the view(DOM) when the data has changed). The basic mechanism is to perform checks against two states, one is the current state, the other is the new state. If one of these states is different from the other, then something has changed, meaning we need to update (or re-render) the view.  
During the **Change Detection Cycle**, Angular searches for all bindings, re-executes all expressions, compares them to the previous values and propagates the change to the DOM elements if a change is identified.  
An Angular application is basically made up of what is known as a component tree. Once a change is detected in a component, Angular CD (change detection) goes through every component in the component tree from top to bottom.

Angular Change Detection Strategies:  
**Default**: With the default change detection strategy, Angular will have no assumption on the component’s dependency and will check every component from the component tree top to bottom every time an event triggers change detection from on browser events, timers, XHRs and promises. This is not a performant strategy for a large application that has many components.  
**OnPush**: With the OnPush strategy, we can skip checks for components that use OnPush strategy and all of its child components. With OnPush strategy, Angular will only update the component if one or more of these conditions happens: Basically, with OnPush strategy, you get more performance because Angular doesn’t need to check or guess on every change detection cycle. It’s also making you embrace immutability with input reference detection.  
\- The input reference changed:  
	By default, angular will run the change detector every time @Input() data is changed or modified. But with OnPush strategy, the change detector is only triggered if the data passed on @Input() has a new reference. This is why using immutable objects is preferred, because immutable objects can be modified only by creating a new object reference. With immutable objects plus OnPush change detection strategy, we can guarantee that OnPush change detection is triggered for each change.  
\- An event from the component or one of its children:  
	With OnPush change detection strategy, every time a component or one of its child triggers an event handler, change detection will be triggered for all components in the component tree.  
\- Async pipe linked to the template emits a new value:  
	Angular built-in AsyncPipe internally calls markForCheck each time a new value is emitted.  
\- Manually triggered the change detection  
We can trigger change detection manually by using **detectChanges()**, **ApplicationRef.tick()**, and **markForCheck()** that we mentioned earlier on AsyncPipe


### DOM Manipulation with ngAfterViewInit

Use `ngAfterViewInit` when you need to interact with the component's rendered DOM. It fires once after Angular fully initializes the component's view and all child views.

```typescript
import { Component, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-example',
  template: `<div class="box">Resize me</div>`,
  styles: ['.box { width: 200px; height: 200px; background: yellow; }']
})
export class ExampleComponent implements AfterViewInit {
  constructor(private el: ElementRef) {}

  ngAfterViewInit() {
    const box = this.el.nativeElement.querySelector('.box') as HTMLElement;
    box.style.backgroundColor = 'steelblue'; // safe — DOM is fully rendered
  }
}
```

> **Avoid** direct DOM manipulation in `ngOnInit` — child views may not be rendered yet. Use `AfterViewInit` for `@ViewChild` access and DOM queries.

### Preventing Component Initialization with CanActivate

To conditionally block a component from loading (e.g., based on a route parameter), use a route guard rather than a lifecycle hook.

```typescript
// my-route.guard.ts
@Injectable({ providedIn: 'root' })
export class MyRouteGuard implements CanActivate {
  constructor(private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const param = route.paramMap.get('paramName');
    if (param === 'blocked') {
      this.router.navigate(['/error']);
      return false;
    }
    return true;
  }
}
```

```typescript
// routing module
const routes: Routes = [
  {
    path: 'my-component/:paramName',
    component: MyComponent,
    canActivate: [MyRouteGuard]
  }
];
```

If `canActivate` returns `false`, Angular cancels navigation and the component's constructor and lifecycle hooks never execute.

## Pipes

### Angular Pipes

A Pipe is a feature that allows you to transform data in a template expression. Pipes are used to format, filter, and manipulate data before displaying it in the user interface.  
**Key Characteristics:**

1. **Syntax**:  
   {{ expression | pipeName:arg1:arg2:... }}  
   E.g., {{ greeting | uppercase }}  
2. **Chaining**  
   {{ date | date:'medium' | uppercase }}  
3. **Built-in Pipes**:  
   ‘date’, ‘uppercase’, ‘lowercase’, ‘number’, ‘currency’, ‘JSON’  
4. **Custom Pipes**:  
   You can create custom pipes by implementing the ‘PipeTransform’ interface and define the ‘transform’ method. The ‘transform’ method contains the logic for transforming the input data. Additionally, you need to decorate the class with the ‘@Pipe’ decorator, specifying the pipe name.

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'custom',
})
export class CustomPipe implements PipeTransform {
  transform(value: any, arg1: any, arg2: any): any {
    // Custom logic here
    return transformedValue;
  }
}
```

A pipes library for Angular.  
[https://github.com/fknop/angular-pipes](https://github.com/fknop/angular-pipes)

### What are Pipes? Pure vs Impure Pipes

**Pipes are functions for Markup**. Pipes are simple functions designed to accept input values, process, and return as an output, a transformed value in a more technical understanding.  
Key features: Pipes are defined using the pipe “|” symbols, Pipes can be chained with other pipes, Pipes can be provided with arguments by using the colon (:) sign.

**What are Pure Pipes?**  
These pipes are pipes that use functions. As a result of this, a pure pipe doesn’t use any internal state, and the output remains the same as long as the parameters passed stay the same. Angular calls the pipe only when it detects a change in the parameters being passed. A single instance of the pure pipe is used throughout all components.

**What are Impure Pipes?**  
For every change detection cycle in Angular, an impure pipe is called regardless of the change in the input fields. Multiple pipe instances are created for these pipes. Inputs passed to these pipes can be mutable. By default, all pipes are pure. However, you can specify impure pipes using the pure property.


### Custom Pipe Example: Smart Number Formatter

A pipe that abbreviates large numbers with K/M/B/T suffixes.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'smartNumber' })
export class SmartNumberPipe implements PipeTransform {
  transform(value: number | string, decimals: number = 1): string {
    if (isNaN(Number(value))) return '-';
    const num = Number(value);
    const thresholds = [1e12, 1e9, 1e6, 1e3];
    const suffixes  = ['T',  'B', 'M', 'K'];

    for (let i = 0; i < thresholds.length; i++) {
      if (num >= thresholds[i]) {
        return (num / thresholds[i]).toFixed(decimals) + suffixes[i];
      }
    }
    return num.toString();
  }
}
```

```html
<p>{{ 1200 | smartNumber }}</p>          <!-- 1.2K -->
<p>{{ 2500000 | smartNumber:2 }}</p>     <!-- 2.50M -->
<p>{{ 9876543210 | smartNumber }}</p>    <!-- 9.9B -->
<p>{{ 'hello' | smartNumber }}</p>       <!-- - -->
```

**Alternative using `Intl.NumberFormat` (built-in, i18n-aware):**

```typescript
transform(value: number | string): string {
  if (isNaN(Number(value))) return '-';
  return new Intl.NumberFormat('en-US', { notation: 'compact' }).format(Number(value));
}
```

## HTTP, Security & Authentication

### Angular Interceptors

- Angular interceptors are a way to intercept HTTP requests and responses globally, allowing you to perform actions such as modifying headers, logging, or handling errors. They provide a mechanism for executing code before a request is sent to the server or after a response is received.  
- Interceptors in Angular serve a purpose similar to middleware components in .NET Core, as both allow you to inject custom behavior into the request/response processing pipeline.  
- Most interceptors transform the outgoing request before passing it to the next interceptor in the chain, by calling next.handle(transformedRequest). An interceptor may transform the response event stream as well, by applying additional RxJS operators on the stream returned by next.handle().  
- We also have to provide it to the base NgModule providers array, as below:

```typescript
@NgModule({
   providers: [
       {
           provide: HTTP_INTERCEPTORS,
		 useClass: AnHttpInterceptor,
 		 multi: true
       }
   ]
})
```


- **Chaining Multiple Angular Interceptors**  
  You can chain multiple interceptors in Angular. Interceptors are executed in the order they are provided in the Angular application configuration. Each interceptor can modify the request or response and pass control to the next interceptor in the chain.

Here's an example of how to create and chain multiple interceptors in Angular:

1. **Create Interceptor Classes:**

```typescript
// logging-interceptor.ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpHandler, HttpRequest, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';
@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Logging Interceptor - Before Request');
    return next.handle(req).pipe(
      tap((event) => {
        if (event instanceof HttpResponse) {
          console.log('Logging Interceptor - After Response');
        }
      })
    );
  }
}
```

```typescript
// auth-interceptor.ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpHandler, HttpRequest, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Auth Interceptor - Adding Authorization Header');
    const authReq = req.clone({
      setHeaders: { Authorization: 'Bearer your_token_here' },
    });
    return next.handle(authReq);
  }
}
```

2. **Register Interceptors in AppModule:**

```typescript
// app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { AppComponent } from './app.component';
import { LoggingInterceptor } from './logging-interceptor';
import { AuthInterceptor } from './auth-interceptor';
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoggingInterceptor,
      multi: true,
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true,
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

   

   In this example, we've created two interceptors (**LoggingInterceptor** and **AuthInterceptor**). They are registered in the AppModule using the HTTP\_INTERCEPTORS multi-provider. The order in which interceptors are provided in the providers array determines the order of execution.

   

   When an HTTP request is made, it goes through the interceptors in the specified order. Each interceptor can modify the request or response and pass control to the next interceptor. The **multi: true** configuration allows for chaining multiple interceptors.

- [https://blog.bitsrc.io/7-ways-to-leverage-httpinterceptors-in-angular-59436611844d](https://blog.bitsrc.io/7-ways-to-leverage-httpinterceptors-in-angular-59436611844d)  
- [https://www.knowledgehut.com/blog/web-development/using-interceptor-in-angular\#10-ways-to-use-interceptors-in-angular](https://www.knowledgehut.com/blog/web-development/using-interceptor-in-angular#10-ways-to-use-interceptors-in-angular)

**What is a Bearer token and how does it work?**  
[https://www.devopsschool.com/blog/what-is-bearer-token-and-how-it-works/](https://www.devopsschool.com/blog/what-is-bearer-token-and-how-it-works/)

### JSON Web Token (JWT) Authentication

JSON Web Tokens (JWT) authentication is commonly used in Angular applications to secure communication between the client and the server.

**1. Set up a Backend Server**

Ensure your backend server supports JWT authentication. You'll need an endpoint for user authentication (login) that returns a JWT upon successful authentication.

**2. Install Required Packages**

```bash
npm install jsonwebtoken @auth0/angular-jwt
```

**3. Create a JWT Service**

Create a service to handle JWT-related functionality — token storage, retrieval, and validation.

```typescript
// jwt.service.ts
import { Injectable } from '@angular/core';
import { JwtHelperService } from '@auth0/angular-jwt';

@Injectable({
  providedIn: 'root',
})
export class JwtService {
  private tokenKey = 'auth_token';

  constructor(private jwtHelper: JwtHelperService) {}

  getToken(): string | null {
    return localStorage.getItem(this.tokenKey);
  }

  saveToken(token: string): void {
    localStorage.setItem(this.tokenKey, token);
  }

  removeToken(): void {
    localStorage.removeItem(this.tokenKey);
  }

  isTokenExpired(): boolean {
    const token = this.getToken();
    return token ? this.jwtHelper.isTokenExpired(token) : true;
  }
}
```

**4. Create an Authentication Service**

Build an authentication service that communicates with your backend for user login/logout.

```typescript
// auth.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { JwtService } from './jwt.service';

@Injectable({
  providedIn: 'root',
})
export class AuthService {
  private apiUrl = 'your-backend-api-url';

  constructor(private http: HttpClient, private jwtService: JwtService) {}

  login(credentials: { username: string; password: string }): Observable<{ token: string }> {
    return this.http.post<{ token: string }>(`${this.apiUrl}/login`, credentials);
  }

  logout(): void {
    this.jwtService.removeToken();
  }

  isAuthenticated(): boolean {
    return !this.jwtService.isTokenExpired();
  }
}
```

**5. Implement Login Component**

```typescript
// login.component.ts
import { Component } from '@angular/core';
import { AuthService } from './auth.service';

@Component({
  selector: 'app-login',
  template: `
    <div>
      <label>Username: <input [(ngModel)]="username" /></label>
      <label>Password: <input [(ngModel)]="password" type="password" /></label>
      <button (click)="login()">Login</button>
    </div>
  `,
})
export class LoginComponent {
  username = '';
  password = '';

  constructor(private authService: AuthService) {}

  login(): void {
    this.authService.login({ username: this.username, password: this.password }).subscribe(
      (response) => {
        this.authService.saveToken(response.token);
        console.log('Login successful');
      },
      (error) => {
        console.error('Login failed', error);
      }
    );
  }
}
```

**6. Protect Routes with AuthGuard**

Implement an `AuthGuard` to protect routes that require authentication.

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    if (this.authService.isAuthenticated()) {
      return true;
    } else {
      this.router.navigate(['/login']);
      return false;
    }
  }
}
```

**7. Use AuthGuard in Route Configuration**

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { LoginComponent } from './login.component';
import { AuthGuard } from './auth.guard';

const routes: Routes = [
  { path: 'login', component: LoginComponent },
  {
    path: 'protected',
    canActivate: [AuthGuard],
    loadChildren: () => import('./protected/protected.module').then((m) => m.ProtectedModule),
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

When a user logs in, the JWT is obtained from the backend and stored in local storage. The `AuthGuard` ensures that protected routes are only accessible when the user is authenticated. The `AuthService` handles authentication operations, and the `JwtService` is responsible for JWT storage and validation.

### Server-side & Client-side Communication (CORS)

The server-side project runs on localhost:5001 and the client side runs on localhost:4200, so in order to establish communication between those two, we need to enable CORS. Let’s open the Program class and modify it.

```typescript
builder.Services.AddCors(options =>
{
    options.AddPolicy("CorsPolicy", builder => builder
        .WithOrigins("http://localhost:4200")
        .AllowAnyMethod()
        .AllowAnyHeader()
        .AllowCredentials());
});
builder.Services.AddControllers();
var app = builder.Build();
// Configure the HTTP request pipeline.
app.UseHttpsRedirection();
app.UseCors("CorsPolicy");
app.UseAuthorization();
```

Please note that we are not using AllowAnyOrigin() method to enable CORS from any origin, but we explicitly say which origin to allow WithOrigins(“[http://localhost:4200](http://localhost:4200)”). We are doing this because from .NET Core 3.0 the combination of AllowAnyOrigin and AllowCredentials is considered as an insecure CORS configuration. One more important thing is to call the UseCors method before the UseAuthorization method.

### What security measures should be taken into account when developing an Angular application?

When developing an Angular application, it's crucial to consider various security measures to protect against common web vulnerabilities and ensure the safety of your application and its users. Here are some important security measures to take into account:

* **Cross-Site Scripting (XSS) Prevention:**  
  * Use Angular's built-in security mechanisms to prevent XSS attacks. Angular automatically sanitizes user input by default, but you should also be cautious when using innerHTML, and use bindings and the DomSanitizer service when necessary.  
* **Cross-Site Request Forgery (CSRF) Protection**:  
  * Implement anti-CSRF tokens to protect against CSRF attacks. Ensure that your server-side code includes CSRF tokens in forms, and verify these tokens on the server side.  
* **HTTP Security Headers:**  
  * Set proper HTTP security headers, including Content Security Policy (CSP), Strict-Transport-Security (HSTS), and X-Content-Type-Options. These headers help prevent various attacks, such as clickjacking and MIME sniffing.  
* **Secure Communication (HTTPS):**  
  * Always use HTTPS to encrypt data transmitted between the client and server. This prevents data interception and man-in-the-middle attacks.  
* **Input Validation and Sanitization:**  
  * Validate and sanitize all user inputs on the client and server sides. Use input validation on the server to ensure that only expected and valid data is accepted.  
* **Authentication and Authorization:**  
  * Implement secure authentication mechanisms, such as JSON Web Tokens (JWT), and ensure that sensitive information, such as passwords, is stored securely. Implement proper authorization checks on the server side to control access to resources.  
* **Content Security Policy (CSP):**  
  * Configure a Content Security Policy to restrict the types of content that can be loaded by your application. This helps mitigate risks associated with malicious scripts and unauthorized content.  
* **Error Handling:**  
  * Implement proper error handling to avoid exposing sensitive information in error messages. Provide generic error messages to users and log detailed error information for developers.  
* **Dependency Scanning:**  
  * Regularly scan and update third-party dependencies and libraries to ensure that your application is not vulnerable to known security issues. Use tools like npm audit to identify and fix vulnerabilities in your project dependencies.  
* **Secure File Uploads:**  
  * If your application involves file uploads, implement proper validation and checks to ensure that only safe and allowed file types are accepted. Store uploaded files in a secure location, and consider using unique file names to prevent directory traversal attacks.  
* **Session Management:**  
  * Implement secure session management practices, including secure cookie settings, session timeout, and token rotation. Avoid storing sensitive information in client-side storage.  
* **Security Reviews and Testing:**  
  * Conduct regular security reviews and testing, including code reviews, penetration testing, and security audits. Identify and fix security vulnerabilities early in the development process.  
* **Security Headers:**  
  * Use security headers, such as X-Frame-Options, X-XSS-Protection, and X-Content-Type-Options, to enhance browser security and protect against various attacks.  
* **Rate Limiting:**  
  * Implement rate limiting to protect against brute force attacks and ensure that your application can handle a reasonable number of requests from a single user or IP address within a specific time frame.

By incorporating these security measures into your Angular application development process, you can significantly reduce the risk of security vulnerabilities and create a more robust and secure web application. Regularly staying informed about the latest security best practices and updates is also crucial for maintaining a secure application over time.

## State Management

### Discuss the role of state management in Angular applications

State management plays a crucial role in Angular applications, helping to manage and share data across different parts of the application, maintain a consistent state, and facilitate communication between components. Here are key aspects and the role of state management in Angular applications:

1. **Single Source of Truth:**  
   * State management provides a single source of truth for the application’s data. It ensures that the state is centralized, making it easier to manage, update, and track changes.  
2. **Component Communication:**  
   * State management facilitates communication between components. Instead of relying solely on input and output bindings, components can share and update state through a centralized store.  
3. **Global Accessibility:**  
   * State management allows data to be accessed globally throughout the application. Components at different levels of the component tree can access and modify shared state.  
4. **Complex UI Scenarios:**  
   * In applications with complex user interfaces or multiple interacting components, managing state becomes critical. State management helps coordinate and synchronize the behavior of various components.  
5. **Improved Scalability:**  
   * As an Angular application grows in complexity, the need for a scalable state management becomes apparent. Well-managed state ensures that the application remains maintainable and scalable.  
6. **Predictable Data Flow:**  
   * State management frameworks often enforce a unidirectional data flow, making it easier to predict and debug how data changes propagate through the application.  
7. **Undo/Redo Functionality:**  
   * State management systems can enable features like undo/redo functionality by maintaining a history of state changes.  
8. **Middleware and Side Effects:**  
   * State management solutions often include middleware and side effect handling, allowing developers to manage asynchronous actions and side effects in a controlled manner.  
9. **Caching and Memoization:**  
   * State management can incorporate caching and memoization techniques to optimize performance by avoiding redundant calculations or fetching data unnecessarily.  
10. **Support for Time-Travel Debugging:**  
    * Some state management libraries offer time-travel debugging capabilities, allowing developers to rewind and replay state changes for debugging purposes.  
11. **Consistent Application State Across Routes:**  
    * Angular applications with multiple routes can benefit from state management to maintain a consistent application state across different views.  
12. **Integration with External Services:**  
    * State management can integrate with external services such as APIs or WebSocket connections, to synchronize and update the application state based on external events.

Popular state management solutions in Angular include:  
**NgRx:** NgRx is a reactive state management library for Angular inspired by Redux. It follows the principles of a unidirectional data flow and provides a store, actions, reducers, and effects.  
**RxJS Observables:** While not a dedicated state management library. Angular applications often use RxJS Observables to manage and share state between components.

### Explain the concepts of store, actions, reducers, and effects in context of NgRx state management library

In NgRx, a state management library for Angular applications inspired by Redux, key concepts include Store, Actions, Reducers, and Effects. These elements work together to manage and maintain the state of an application in a predictable and scalable way.

1. **Store:**  
   The Store represents the application state and provides methods to interact with it. It is a single, immutable data store that holds the entire state of the application. Components dispatch actions to the store to trigger state changes, and they can also subscribe to changes in the state.

```typescript
// app.state.ts
export interface AppState {
  counter: number;
}
```

```typescript
// app.module.ts
import { StoreModule } from '@ngrx/store';
import { counterReducer } from './counter.reducer';
import { AppState } from './app.state';
@NgModule({
  imports: [
    StoreModule.forRoot({ counter: counterReducer } as AppState),
    // other modules and configurations
  ],
})
export class AppModule {}
```

In this approach, the ‘**AppState**’ interface is defined to provide strong typing for the entire application state. The ‘**as AppState**’ assertion is used to inform TypeScript about the expected type of the state structure.

2. **Actions:**  
   Actions are plain JavaScript objects that represent events or intentions to change the state. They have a ‘**type**’ property that describes the action and may carry additional data payload.  
   Example:

```typescript
// counter.actions.ts
import { createAction, props } from '@ngrx/store';
export const increment = createAction('[Counter] Increment');
export const decrement = createAction('[Counter] Decrement');
export const reset = createAction('[Counter] Reset');
export const set = createAction('[Counter] Set', props<{ value: number }>());
```

3. **Reducers**  
   Reducers specify how the application’s state changes in response to actions. They are pure functions that take the current state and an action and return a new state. Reducers should not modify the state directly but rather create a new copy.  
   Example:

```typescript
// counter.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { increment, decrement, reset, set } from './counter.actions';
export const initialState = 0;
export const counterReducer = createReducer(
  initialState,
  on(increment, (state) => state + 1),
  on(decrement, (state) => state - 1),
  on(reset, () => initialState),
  on(set, (state, { value }) => value)
);
```

4. **Effects:**  
   Effects are used for managing side effects in the application, such as asynchronous operations (e.g., HTTP Requests). They listen for dispatched actions, perform operations, and then dispatch new actions to the store.  
   Example:

```typescript
// counter.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { EMPTY } from 'rxjs';
import { map, mergeMap, catchError } from 'rxjs/operators';
import { CounterService } from './counter.service';
import * as CounterActions from './counter.actions';
@Injectable()
export class CounterEffects {
  loadCounters$ = createEffect(() =>
    this.actions$.pipe(
      ofType(CounterActions.increment),
      mergeMap(() =>
        this.counterService.increment().pipe(
          map((data) => CounterActions.set({ value: data })),
          catchError(() => EMPTY)
        )
      )
    )
  );
  constructor(private actions$: Actions, private counterService: CounterService) {}
}
```

**Connecting everything:**  
Finally, components dispatch actions to the store, and the store triggers reducers to update the state. Effects listen for specific actions and can perform asynchronous operations. The overall flow ensures a unidirectional data flow and helps maintain a predictable state.

```typescript
// counter.component.ts
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import * as CounterActions from './counter.actions';
@Component({
  selector: 'app-counter',
  template: `
    <div>
      <p>Current Count: {{ count$ | async }}</p>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
      <button (click)="reset()">Reset</button>
    </div>
  `,
})
export class CounterComponent {
  count$ = this.store.select('counter');
  constructor(private store: Store) {}
  increment() {
    this.store.dispatch(CounterActions.increment());
  }
  decrement() {
    this.store.dispatch(CounterActions.decrement());
  }
  reset() {
    this.store.dispatch(CounterActions.reset());
  }
}
```

## Build & Performance Optimization

### AOT (Ahead of Time) vs JIT (Just in Time) Compilation

AOT (Ahead Of Time) Compilation

- Compiler converts your code during the build time. This is the default compiling option in Angular 9\.  
- Compilation happens only once, while you build your project.  
- Minimizes the size of your application.  
- Browser does not need to compile the code in run time, it can render the application immediately, without waiting to compile the app first, so it provides quicker rendering.  
- AOT detects template error earlier.  
- Suitable for production mode.  
- Smaller optimized build size.

JIT (Just In Time) Compilation

- JIT provides the execution of the program at a run time before execution. In other words, code gets compiled when it’s needed, not at the build time.  
- JIT compiler compiles code at runtime which means instead of interpreting bytecode at build time, it will compile bytecode when that component is called.  
- Suitable for development mode.  
- Bigger build size.

### Discuss the benefits of Ahead-Of-Time (AOT) compilation

Ahead-Of-Time (AOT) compilation is a build process in Angular where the application’s templates and components are compiled during the build phase rather than at runtime. This process offers several benefits, contributing to improved performance, smaller bundle size, and enhanced security in Angular applications. Here are the key benefits of Ahead-of-Time (AOT) compilation:

**Faster Rendering and Startup:**  
AOT-compiled code is pre-processed during the build phase, resulting in optimized and pre-compiled templates. This leads to faster rendering and reduced startup times because the browser doesn't need to compile templates at runtime.

**Smaller Bundle Sizes:**  
AOT compilation eliminates the need to ship the Angular compiler to the client, resulting in smaller bundle sizes. Templates are pre-compiled into efficient JavaScript code, reducing the overall payload sent to the browser.

**Reduced Runtime Overhead:**  
AOT compilation moves much of the work from the client's browser to the build process. This reduces the runtime overhead and the need for the Angular compiler to be included in the final application.

**Template Error Detection:**  
AOT compilation performs thorough template checking during the build process. It detects errors in templates, such as missing components or incorrect bindings, at build time rather than runtime. This leads to more robust applications with fewer runtime errors.

**Early Identification of Issues:**  
AOT compilation catches issues like misspelled component names, incorrect property bindings, or missing dependencies early in the development process. This helps developers address potential problems before deploying the application.

**Better Tree Shaking:**  
AOT compilation improves the effectiveness of tree shaking. Unused parts of the Angular framework can be more effectively removed during the build process, resulting in smaller bundle sizes.

**Enhanced Security:**  
AOT compilation helps improve security by preventing certain types of injection attacks that could occur with Just-In-Time (JIT) compilation. AOT compilation makes it more difficult for attackers to exploit vulnerabilities related to dynamic template execution.

**Improved Angular Dependency Injection:**  
AOT compilation allows for better optimization of Angular's dependency injection system. This leads to more efficient injection of dependencies and improved overall application performance.

**Compatibility with Some Features:**  
Certain Angular features, such as template inlining and internationalization (i18n), are optimized and work better with AOT compilation. This makes it easier to implement these features in Angular applications.

**Better Tooling Integration:**  
AOT compilation integrates well with build tools and development workflows. The Angular CLI, for example, supports AOT compilation out of the box, making it seamless for developers to leverage this optimization.

### What strategies can be employed to optimize the performance of an Angular application?

Here are the various strategies and best practices to employ for optimizing the performance of your Angular application:

1. **Build and Compilation Optimisation:**  
   * **Ahead of TIme (AOT) Compilation**  
     Use AOT compilation to pre-compile Angular templates during the build process. AOT compilation results in smaller bundle sizes and faster rendering in the browser.  
   * **Minification and Uglification**  
     Minify and uglify your code during the build process. Minification reduces the size of your JavaScript files, and uglification obfuscates the code, making it harder to reverse engineer.  
   * **Tree Shaking**  
     Leverage tree shaking to eliminate dead code from your bundles. This process removes unused code, reducing the overall bundle size.  
   * **Bundle Size Analysis**  
     Analyze your application's bundle sizes using tools like Webpack Bundle Analyzer. Identify and optimize large dependencies or unnecessary imports.  
2. **Module and Dependency Management**  
   * **Lazy Loading Strategies**  
     Implement lazy loading for feature modules. This allows you to load modules on demand, reducing the initial payload and improving application startup time.  
   * **Optimize Angular Modules**  
     Carefully organize and optimize Angular modules. Use lazy loading, separate feature modules, and consider breaking down large modules into smaller ones.  
3. **Asset Optimization**  
   * **Optimized Images**  
     Optimize and compress images before including them in your application. Tools like ImageOptim or TinyPNG can help reduce image file sizes without sacrificing quality.  
   * **Caching**  
     Utilize browser caching by configuring appropriate cache headers for your assets. This reduces the need to download resources on subsequent visits.  
   * **Content Delivery Networks (CDN)**  
     Use a CDN to distribute static assets closer to users. This reduces latency and improves the delivery speed of your application.  
4. **Application Structure and Loading Strategies**  
   * **Smart Loading**  
     Load only the necessary resources for the initial view. Consider deferring non-essential script loading and using the async and defer attributes appropriately.  
   * **Server-side Rendering**  
     Consider server-side rendering for Angular Universal applications. SSR improves initial load times by rendering pages on the server and sending pre-rendered content to the client.  
5. **Code and Runtime Optimisation**  
   * **Change Detection Optimization**  
     Use **OnPush** change detection strategy in components to reduce unnecessary checks. Manually trigger change detection only when necessary.  
   * **RxJs Optimisation**  
     Optimize the usage of RxJS observables. Avoid unnecessary subscriptions, use operators like \`**pipe**\` to chain transformations efficiently, and unsubscribe from observables when no longer needed.  
   * **Profile and Benchmark**  
     Use browser developer tools to profile and benchmark your application's performance. Identify bottlenecks, memory leaks, or inefficient code.  
   * **Web Performance Best Practices**  
     Implement general web performance best practices, such as minimizing HTTP requests, leveraging browser caching, and optimizing critical rendering paths.  
6. **Progressive Web App (PWA) Features**  
   * **Service Workers for Progressive Web App (PWA)**  
     Implement service workers to enable Progressive Web App features. Service workers can cache assets, allowing your application to work offline and improving loading times on subsequent visits.  
7. **Browser Compatibility and Maintenance**  
   * **Browser Compatibility**  
     Ensure that your application is optimized for the specific browsers your users are using. Regularly test and address performance issues on different browsers.  
   * **Upgrade Angular Version**  
     Stay up-to-date with the latest Angular version. New releases often include performance improvements and optimizations.

## Testing

### How do you mock service dependencies in an Angular service while implementing unit tests?

In Angular unit testing, you can mock service dependencies using **Jasmine spies** and the Angular testing utilities. This allows you to isolate the service you're testing and control the behavior of its dependencies. Here's a step-by-step guide on how to mock service dependencies in an Angular service unit test:

Let's assume you have a service called **MyService** that depends on another service called **DependencyService**. Here's how you can mock DependencyService in the unit test for MyService:

1. **Create the Service to Test (MyService):**

```typescript
// my.service.ts
import { Injectable } from '@angular/core';
import { DependencyService } from './dependency.service';
@Injectable({
  providedIn: 'root',
})
export class MyService {
  constructor(private dependencyService: DependencyService) {}
  getData(): string {
    return this.dependencyService.getData();
  }
}
```

2. **Create the Dependency Service (DependencyService):**

```typescript
// dependency.service.ts
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'root',
})
export class DependencyService {
  getData(): string {
    return 'Real data';
  }
}
```

3. Write the Unit Test for MyService with Mocked Dependency:

```typescript
// my.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { MyService } from './my.service';
import { DependencyService } from './dependency.service';
describe('MyService', () => {
  let myService: MyService;
  let dependencyServiceSpy: jasmine.SpyObj<DependencyService>;
  beforeEach(() => {
    const spy = jasmine.createSpyObj('DependencyService', ['getData']);
    TestBed.configureTestingModule({
      providers: [
        MyService,
        { provide: DependencyService, useValue: spy },
      ],
    });
    myService = TestBed.inject(MyService);
    dependencyServiceSpy = TestBed.inject(DependencyService) as jasmine.SpyObj<DependencyService>;
  });
  it('should return mocked data from DependencyService', () => {
    // Arrange
    dependencyServiceSpy.getData.and.returnValue('Mocked data');
    // Act
    const result = myService.getData();
    // Assert
    expect(result).toBe('Mocked data');
    expect(dependencyServiceSpy.getData).toHaveBeenCalled();
  });
});
```

   

In this example:

* We use **jasmine.createSpyObj** to create a spy object for the **DependencyService**. This spy object has the same methods as the real **DependencyService** but with spy functionality.  
* In the **TestBed.configureTestingModule**, we provide the spy object instead of the real **DependencyService**. This allows us to control the behavior of **getData** in our test.  
* In the test case, we set up the spy to return a specific value ('**Mocked data**'), and then we call the method on **MyService**. The test asserts that the correct data is returned and that the **getData** method of the dependency service has been called.

This approach allows you to isolate the service being tested and control the behavior of its dependencies, ensuring that your unit tests focus on the specific logic of the service under test.

### How do you write unit tests for Angular components and services?

Writing unit tests is an integral part of Angular development to ensure that components and services behave as expected. Angular provides a testing framework built on top of Jasmine, and the Angular Testing utilities to facilitate writing and running unit tests. Here's a guide on how to write unit tests for Angular components and services:

**Unit Testing Angular Components:**

1. **Setup:**  
   * Import the necessary testing modules from @angular/core/testing and Angular testing utilities.  
   * Create a TestBed configuration by configuring the testing module.

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { MyComponent } from './my.component';
describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [MyComponent],
    });
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
  });
});
```

     

2. **Testing Component Logic:**  
   Write tests to verify component properties, methods, and interactions.

```typescript
it('should have a default value for a property', () => {
  expect(component.myProperty).toBe('default');
});
it('should call a method on button click', () => {
  spyOn(component, 'myMethod');
  const button = fixture.nativeElement.querySelector('button');
  button.click();
  expect(component.myMethod).toHaveBeenCalled();
});
```

   

3. **Testing Template and DOM Interactions:**  
   Test interactions with the component's template and the DOM.

```typescript
it('should display a value in the template', () => {
  component.myProperty = 'test value';
  fixture.detectChanges();
  const span = fixture.nativeElement.querySelector('span');
  expect(span.textContent).toBe('test value');
});
```

   

4. **Asynchronous Testing:**  
   Handle asynchronous operations using async and fakeAsync.

```typescript
it('should handle asynchronous data fetching', async () => {
  spyOn(component.dataService, 'getData').and.returnValue(Promise.resolve('async data'));
  await component.loadData();
  expect(component.data).toBe('async data');
});
```

   

5. **Angular Forms and Inputs:**  
   Test interactions with Angular forms and input properties.

```typescript
it('should update form control on input change', () => {
  const input = fixture.nativeElement.querySelector('input');
  input.value = 'new value';
  input.dispatchEvent(new Event('input'));
  expect(component.myForm.get('myControl').value).toBe('new value');
});
```

   

   

**Unit Testing Angular Services:**

1. **Setup:**  
   * Import the necessary testing modules from @angular/core/testing and Angular testing utilities.  
   * Use TestBed to configure the testing module and create an instance of the service.

```typescript
import { TestBed } from '@angular/core/testing';
import { MyService } from './my.service';
describe('MyService', () => {
  let service: MyService;
  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(MyService);
  });
});
```

     

2. **Testing Service Methods:**  
   Write tests to verify the behavior of service methods.

```typescript
it('should return data from a service method', () => {
  const result = service.getData();
  expect(result).toBeDefined();
});
```

3. **Mocking Dependencies:**  
   Use spies and provide mock implementations for service dependencies.

```typescript
it('should call a method on a dependency', () => {
  const dependencySpy = spyOn(service.dependencyService, 'dependentMethod');
  service.performAction();
  expect(dependencySpy).toHaveBeenCalled();
});
```

4. **Asynchronous Testing:**  
   Handle asynchronous operations within service methods.

```typescript
it('should handle asynchronous data fetching', async () => {
  spyOn(service.httpClient, 'get').and.returnValue(Promise.resolve('async data'));
  const result = await service.fetchData();
  expect(result).toBe('async data');
});
```

   

5. **Testing HTTP Requests:**  
   Use HttpClientTestingModule to mock HTTP requests and responses.

```typescript
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
beforeEach(() => {
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
    providers: [MyService],
  });
});
it('should make an HTTP request', () => {
  const httpMock = TestBed.inject(HttpTestingController);
  service.getData().subscribe(response => {
    expect(response).toBeDefined();
  });
  const req = httpMock.expectOne('api/data');
  expect(req.request.method).toBe('GET');
  req.flush('mocked data');
});
```

   

These examples provide a starting point for unit testing Angular components and services. Remember to adjust and expand based on your application's specific requirements and dependencies. Unit testing is crucial for maintaining the reliability and quality of your Angular application.

### Explain the use of testing utilities such as TestBed

Testing utilities, such as TestBed, are an integral part of Angular's testing framework. They provide a set of tools and services that simplify the process of setting up and interacting with the testing environment, allowing developers to write effective unit tests and integration tests for Angular applications.

**Key Testing Utilities:**

* **TestBed**:  
  * TestBed is the main testing utility in Angular. It is a testing module builder that allows you to configure and create Angular testing modules. A testing module defines a context in which your tests run, including the components, services, and other dependencies required for testing.

```typescript
import { TestBed } from '@angular/core/testing';
// Configuring a testing module
TestBed.configureTestingModule({
 declarations: [MyComponent],
 providers: [MyService],
});
// Creating a component instance within the testing module
const fixture = TestBed.createComponent(MyComponent);
const componentInstance = fixture.componentInstance;
```

*   
* **inject**:  
  * The inject function is used to retrieve instances of services or dependencies within the testing module. It provides a clean way to access and use services in your tests.

```typescript
beforeEach(inject([MyService], (myService: MyService) => {
 // Use myService in your tests
}));
```

*   
* **ComponentFixture**:  
  * ComponentFixture is a wrapper around a component that provides access to the component instance, its DOM element, and other utilities for testing. It is created when you call TestBed.createComponent.

```typescript
const fixture = TestBed.createComponent(MyComponent);
const componentInstance = fixture.componentInstance;
```

*   
* **async and fakeAsync:**  
  * Angular supports both asynchronous and synchronous testing. The async and fakeAsync utilities help you handle asynchronous code in your tests. They enable you to work with promises, observables, and timers in a controlled manner.

```typescript
it('should handle asynchronous code', async(() => {
 // Your asynchronous test code here
}));
it('should handle asynchronous code in a synchronous manner', fakeAsync(() => {
 // Your synchronous test code here
}));
```

*   
* **jasmine.createSpy and jasmine.createSpyObj:**  
  * Jasmine provides functions to create spies, which are mock implementations of functions or methods. These are useful for creating test doubles, mocking dependencies, and asserting interactions.

```typescript
const mySpy = jasmine.createSpy('mySpy');
const myObjSpy = jasmine.createSpyObj('MyObject', ['method1', 'method2']);
```

* 

**Use Cases of TestBed and Testing Utilities:**

* **Setting Up Testing Modules:**  
  * TestBed.configureTestingModule is used to configure Angular testing modules by declaring components, providing services, and configuring other dependencies.  
* **Creating Component Instances:**  
  * TestBed.createComponent creates an instance of a component within the configured testing module. The returned ComponentFixture provides access to the component instance, its DOM element, and other testing utilities.  
* **Injecting Services:**  
  * inject is used to inject instances of services or dependencies into your tests, making it easy to work with the dependencies required for the test scenario.  
* **Handling Asynchronous Code:**  
  * async and fakeAsync utilities help in handling asynchronous operations, such as promises and observables, in a way that ensures the test waits for asynchronous tasks to complete before making assertions.  
* **Creating Spies:**  
  * jasmine.createSpy and jasmine.createSpyObj are used to create spies for functions and objects, allowing you to monitor function calls, control return values, and assert interactions in your tests.

Overall, testing utilities like TestBed provide a structured and efficient way to set up and manage the testing environment in Angular. They play a crucial role in making the testing process more straightforward, allowing developers to write robust and maintainable tests for their Angular applications.


### Testing Component Lifecycle Hooks

Use `fixture.detectChanges()` to trigger `ngOnInit`. Access the component instance directly for other hooks.

```typescript
@Component({
  selector: 'app-lifecycle',
  template: '<p>{{ initialized }}</p>',
})
export class LifecycleComponent implements OnInit {
  initialized = false;

  ngOnInit(): void {
    this.initialized = true;
  }
}
```

```typescript
describe('LifecycleComponent', () => {
  let component: LifecycleComponent;
  let fixture: ComponentFixture<LifecycleComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [LifecycleComponent],
    }).compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(LifecycleComponent);
    component = fixture.componentInstance;
    // detectChanges() triggers ngOnInit
    fixture.detectChanges();
  });

  it('should set initialized to true after ngOnInit', () => {
    expect(component.initialized).toBeTrue();
  });

  it('ngOnDestroy should clean up', () => {
    spyOn(component, 'ngOnDestroy').and.callThrough();
    fixture.destroy(); // triggers ngOnDestroy
    expect(component.ngOnDestroy).toHaveBeenCalled();
  });
});
```

**Key points:**
- `fixture.detectChanges()` is the primary trigger for `ngOnInit`
- `fixture.destroy()` triggers `ngOnDestroy`
- For `ngOnChanges`, set `@Input` values before calling `detectChanges()`
- `ngAfterViewInit` fires after `detectChanges()` completes

## Internationalization & SEO

### How can you implement internationalization in an Angular application?

Internationalization (i18n) in Angular involves making your application capable of supporting multiple languages and locales.  
***Language** and **locale** are related concepts but refer to different aspects when it comes to internationalization and localization.*  
***Language:** In the context of software development and internationalization, providing support for different languages means offering translations of the application’s text and content in multiple languages.*

***Locale:** A local is a set of parameters that define the user’s language, region, and cultural preferences. It includes information such as **language**, **country**, **currency**, **date and time formats**, and other **regional settings**. A local is often represented as a combination of a language code and a country code, such as “en\_US” for English in the United States or “fr\_FR” for French in France. The locale is used to customize the presentation of information in the user interface to match the conventions of the user’s region.*

*In summary, the key difference is that language pertains to the communication system itself, while locale encompasses additional regional and cultural settings associated with a user’s preferences. When building internationalized applications, it’s common to consider both language and locale to provide a comprehensive and culturally appropriate user experience.*

Angular provides a built-in **i18n** module that allows you to easily internationalize your application. Here are the steps to implement internationalization in an Angular application:

1. **Set-up Angular i18n:**  
   1. **Enable i18n in Angular CLI:**  
      1. When creating a new Angular using the Angular CLI, you can enable i18n by using the ‘**\--i18n**’ flag:  
         **ng new your-app \--i18n**  
      2. This sets up the necessary configurations and adds the initial localization files.  
   2. **Generate initial language files:**  
      1. Run the following command to generate initial language files:  
         **ng xi18n**  
      2. This command creates a message.xlf file in the **src/locale** directory.  
2. **Translate Application Context:**  
   1. **Extract Messages:**  
      1. Use the Angular CLI to extract messages from your application:  
         **ng xi18n \--output-path src/locale**  
      2. This generates a ‘.xlf’ file for each language you want to support.  
   2. **Translate Messages:**  
      1. Provide translations for each extracted message in the **.xlf** files. You can use tools like Transifex or manage translations manually.  
   3. **Import Translations**  
      1. Import translations back into your application:  
         ng xi18n \--input-format xlf \--locale fr \--output-path src/locale  
      2. This imports translations for the specified locale (e.g., French)  
3. **Configure Angular i18n:**  
   1. **App Module Configuration:**  
      1. Update your ‘**app.module.ts**’ to include the ‘**LOCALE\_ID**’ provider and import the ‘**registerLocalData**’ function.

```typescript
import { LOCALE_ID } from '@angular/core';
import { registerLocaleData } from '@angular/common';
import localeFr from '@angular/common/locales/fr';
// Register locale data for each supported language
registerLocaleData(localeFr);
@NgModule({
  providers: [
    { provide: LOCALE_ID, useValue: 'fr' }, // Set the default locale
  ],
})
export class AppModule {}
```

      

   2. **App Component Configuration:**  
      1. In your root component (**app.component.ts**), set the locale for the application:

```typescript
import { Component, OnInit } from '@angular/core';
import { TranslateService } from '@ngx-translate/core';
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent implements OnInit {
  constructor(private translateService: TranslateService) {}
  ngOnInit() {
    this.translateService.setDefaultLang('fr'); // Set the default language
    this.translateService.use('fr'); // Use the specified language
  }
}
```

         

         

4. **Use ngx-translate for Translation:**  
   1. Install ngx-translate/core and ngx-translate/http-loader:  
      1. Install the ngx-translate packages:  
         **npm install @ngx-translate/core @ngx-translate/http-loader \--save**  
   2. Configure ngx-translate:  
      1. Set up the translation loader and configure it in your ‘ápp-module.ts’:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule, HttpClient } from '@angular/common/http';
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
export function HttpLoaderFactory(http: HttpClient) {
  return new TranslateHttpLoader(http);
}
@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient],
      },
    }),
  ],
})
export class AppModule {}
```

   3. Use Translation Pipe:  
      1. Use the ‘translate’ pipe in your templates to display translated content.

```html
<h1>{{ 'HELLO' | translate }}</h1>
```

         

5. **Testing:**  
   1. Updates Tests:  
      Update your unit tests to account for i18n. You may need to adjust your test to consider translated content and locales.  
   2. Testing with Different Locales:  
      Use the ‘--configuration’ flag to run your application with different configurations. For example, ‘ng serve \--configuration=fr’ to run with the French configuration.  
      

### How can you optimize an Angular application for search engine optimization (SEO)?

Optimizing an Angular application for search engine optimization (SEO) is important to ensure that **search engines can properly index and rank your content**. Angular applications are often single-page applications (SPAs) that dynamically generate content on the client side. To improve SEO for Angular applications, consider the following strategies:

1. **Use Angular Universal:**  
   **Angular Universal** is a technology that allows you to perform **server-side rendering (SSR)** of Angular applications. This means that the initial rendering of pages can happen on the server side before being sent to the client. SSR helps search engines index your content more effectively because they receive fully rendered HTML content from the server.

   To implement Angular Universal:  
   1. Install Angular Universal by running:  
      **ng add @nguniversal/express-engine**  
   2. Create server-side scripts and update your main.ts file to support server-side rendering.  
   3. Build your application with SSR support using:  
      ng run your-app:server  
   4. Deploy both the client and server applications to your server.

   

2. **Use the Meta tags appropriately:**  
   Search engines rely on meta tags to understand the content and structure of your pages. Ensure that your Angular application uses meta tags appropriately, especially for essential elements like title, description, and keywords. Use Angular's Title service to dynamically update the title tag, and manually include meta tags in your HTML templates.

```typescript
import { Title, Meta } from '@angular/platform-browser';
constructor(private titleService: Title, private metaService: Meta) {}
ngOnInit() {
  this.titleService.setTitle('Your Page Title');
  this.metaService.updateTag({ name: 'description', content: 'Your page description' });
  // Add other meta tags as needed
}
```

   

   

3. **Provide a Sitemap and Robots.txt:**  
   Create a sitemap.xml file that lists all the pages on your site and submit it to search engines like Google. Additionally, create a robots.txt file to specify which pages should or should not be indexed by search engines.

4. **Implement Angular Routes Properly**  
   Use Angular’s routing mechanism to create clean, user-friendly URLs. Ensure that your route makes sense semantically, and include keywords relevant to the content of each page. Avoid using URLs with hashes (\#) unless necessary for certain scenarios.

5. **Pre-render static pages:**  
   For pages with static content or limited interactivity, consider pre-rendering them during the build process. Tools like prerender.io can be integrated into your build process to create static HTML files for specific routes.

6. **Optimize Performance:**  
   Ensure that your Angular application is optimized for performance. Faster load times and improved user experience contribute to better SEO. Optimize images, reduce HTTP requests, and leverage Angular features like lazy loading for efficient resource loading.

7. **Handle 404 Errors:**  
   Implement a custom 404 error page to provide a better user experience. Search engines interpret 404 errors as a negative signal, so handling them gracefully can help maintain a positive SEO status.

8. **Enable Social Sharing:**  
   Implement Open Graph Protocol (OGP) and Twitter Card meta tags to control how your content appears when shared on social media platforms. This can improve the visibility of your content on social networks and indirectly contribute to SEO.

By following these strategies, you can enhance the SEO of your Angular application and improve its visibility in search engine results. It's essential to stay informed about SEO best practices, as search engine algorithms and guidelines can evolve over time.

## Modern Angular: Signals

### What are Angular Signals?

**Angular Signals** is a fine-grained reactivity system (introduced in Angular 16) that tracks exactly where state is read and when it changes, allowing the framework to update only the parts of the UI that depend on that state, without running change detection across the entire component tree.

**The Problem Signals Solve**

Traditional Angular change detection (via Zone.js) works by checking the whole component tree whenever any async event fires. This is simple but inefficient at scale. Signals let Angular know *precisely* which state changed and *exactly* which template expressions depend on it, enabling targeted, zone-free updates.

---

**The Three Reactive Primitives**

**1. `signal` — writable state**

A `signal` is a reactive variable. Reading it inside a template or `computed` automatically registers a dependency. Writing to it notifies all dependents.

**2. `computed` — derived state**

A `computed` signal derives its value from other signals. It is **lazy** (only recalculates when read) and **memoized** (skips recalculation if its dependencies have not changed).

**3. `effect` — reactive side effects**

An `effect` runs a function whenever any signal it reads changes. Useful for logging, syncing to external APIs, or triggering non-template side effects.

---

**Signals in Code**

```typescript
import { signal, computed, effect } from '@angular/core';

// signal: writable reactive value
const count = signal(0);
count();                      // read: 0
count.set(5);                 // write
count.update(n => n + 1);     // update based on current value

// computed: derived, lazy, memoized
const price = signal(100);
const quantity = signal(3);
const total = computed(() => price() * quantity()); // 300

// effect: runs whenever dependencies change
const username = signal('Alice');
effect(() => {
  console.log('User:', username());
});
username.set('Bob'); // logs: "User: Bob"
```

**Using Signals in a Component**

Signals integrate directly into templates — no `async` pipe or manual subscription needed.

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ double() }}</p>
    <button (click)="increment()">Increment</button>
  `,
})
export class CounterComponent {
  count = signal(0);
  double = computed(() => this.count() * 2);

  increment() {
    this.count.update(n => n + 1);
  }
}
```

---

**Signals vs Zone.js Change Detection**

| | Zone.js (Default) | Signals |
| :--- | :--- | :--- |
| **Update trigger** | Any async event checks full component tree | Only components reading changed signals re-render |
| **Granularity** | Component-level | Expression-level |
| **Zone.js required** | Yes | No (enables zoneless Angular) |
| **Optimisation boilerplate** | Requires `OnPush` strategy | Fine-grained by default |

**Key Benefits**
- **Performance** — no unnecessary component checks; only affected views re-render
- **Simplicity** — easier than RxJS for local component state
- **Zoneless** — enables Angular apps to run without Zone.js entirely
- **Interoperability** — `toSignal()` and `toObservable()` bridge Signals and RxJS
## Interview Prep Reference & Resources


### Angular Scenario-Based Interview Questions

Common scenario questions and what they test:

| # | Question | Concepts Tested |
| :--- | :--- | :--- |
| 1 | Why might `this.title` be undefined in a component method, and how do you fix it? | `this` binding, arrow functions, async execution |
| 2 | What are the risks of using `ngOnChanges()` with `OnPush`? | Change detection, immutability, lifecycle |
| 3 | How do you manually trigger change detection, and when is it necessary? | `ChangeDetectorRef`, `markForCheck()`, `detectChanges()`, Zone.js |
| 4 | How do you ensure a service is a singleton across lazy-loaded modules? | DI scopes, lazy loading, `providedIn: 'root'` |
| 5 | What issues can arise when unsubscribing in `ngOnDestroy()`? | RxJS memory leaks, component lifecycle, execution order |
| 6 | How can you bind dynamic event listeners without memory leaks? | `Renderer2`, `HostListener`, `NgZone`, manual event binding |
| 7 | Why does `[(ngModel)]` not work on a custom component by default? | `ControlValueAccessor`, forms, directives |
| 8 | How would you optimize a slow app with many rendering components? | `trackBy`, `OnPush`, memoization, lazy loading, virtual scrolling |
| 9 | What are the differences between `async` pipe and manually subscribing in `ngOnInit()`? | RxJS, change detection, memory leaks |
| 10 | How would you implement debounced search without `FormControl`? | `fromEvent()`, `debounceTime()`, `distinctUntilChanged()` |

### Angular System Design Interview Questions — Topic Map

1. **Component-based Architecture:**  
   * [How does Angular implement a component-based architecture, and what are the benefits?](#bookmark=id.axj6gsi6lyut)  
   * [Explain the lifecycle hooks in Angular and when they are used.](#bookmark=id.keh0g2cqjj76)  
2. **Modules:**  
   * [Describe the purpose and use of Angular modules.](#bookmark=id.cvczq636tbhs)  
   * [How do you organise modules in a large Angular application to ensure maintainability and scalability?](#bookmark=id.d5txwkfvxlok)  
3. **Dependency Injection:**  
   * [Explain how Angular implements dependency injection.](#bookmark=id.yp0fiib2tzbh)  
   * [What are the advantages of using dependency injection in Angular?](#bookmark=id.zai4ws4pj24l)  
4. **Change Detection:**  
   * [How does change detection work in Angular?](#bookmark=id.chnonci1t79y)  
   * [**What strategies can be used to optimise change detection performance?**](#bookmark=id.9je9mnuzin9r)  
5. **Routing and Navigation:**  
   * Explain Angular routing and how it facilitates single-page applications.  
   * [Discuss lazy loading and its benefits in the context of Angular routing.](#bookmark=id.tcwkimvtyeyx)  
6. **Services:**  
   * [What are Angular services, and why are they used?](#bookmark=id.tcwkimvtyeyx)  
   * [How would you create a singleton service in Angular, and what are the potential pitfalls?](#bookmark=id.cn5ax8kx6a0)  
7. **Forms:**  
   * Describe the different types of forms in Angular (Template-driven vs. Reactive).  
   * [How would you handle form validation and error handling?](#bookmark=id.pdlq0d6fiet7)  
8. **State Management:**  
   * [Discuss the role of state management in Angular applications.](#bookmark=id.717k3y89nqsn)  
   * Compare and contrast different state management solutions in Angular (e.g., NgRx, Akita).  
9. **Performance Optimization:**  
   * [What strategies can be employed to optimise the performance of an Angular application?](#bookmark=id.fhmziwrhsy53)  
   * [Discuss the benefits of Ahead-Of-Time (AOT) compilation.](#bookmark=id.xz4ea6qwsvdf)  
10. **Testing:**  
    * [How do you write unit tests for Angular components and services?](#bookmark=id.coyi70e7v7e8)  
    * [Explain the use of testing utilities such as TestBed.](#bookmark=id.et5mjy68jc0i)  
11. **Security:**  
    * [What security measures should be taken into account when developing an Angular application?](#bookmark=id.s03sqstx69rm)  
    * How does Angular handle cross-site scripting (XSS) and cross-site request forgery (CSRF)?  
12. **Integration with Backend:**  
    * How does Angular interact with a backend server?  
    * Discuss the use of HTTP services and observables in Angular for data retrieval.  
13. **Internationalization (18n):**  
    * [How can you implement internationalization in an Angular application?](#bookmark=id.pi8b3962iaw)  
    * Discuss the use of ngx-translate or Angular’s built-in i18n support.  
14. **Optimization for SEO:**  
    * [How can you optimize an Angular application for search engine optimization (SEO)?](#bookmark=id.ke4hwbarrpnp)  
    * Discuss the use of server-side rendering (SSR) in Angular.

### Things You Need To Learn To Become An Effective Angular Developer

Reference: [medium.matcha.fyi article](https://medium.matcha.fyi/19-things-you-need-to-learn-to-become-an-effective-angular-developer-c0ccfa51222a)

* Modular Angular Architecture  
* One-way Dataflow and Immutability  
  *In Angular, "One-way Dataflow" refers to the practice of data moving in a single direction—from parent components to child components. This simplifies data management and enhances predictability.*  
    
  *"Immutability" involves treating data as unchangeable once created. Instead of modifying existing data, new instances are created. This practice enhances stability, prevents unintended side effects, and aligns with Angular's change detection system.*  
    
  *Together, "One-way Dataflow and Immutability" contribute to a more manageable and predictable application state in Angular, making it easier to understand, debug, and maintain. These principles align with the reactive programming paradigm, which Angular embraces with its use of observables and the change detection system.*  
    
* Attribute Directives and Structural Directives  
* Component Lifecycle Hooks  
* Http and Observable Services  
* Smart/Dumb Component Architecture  
* Application Structure and Best Practices  
* Template Binding Syntax  
* Feature Modules and Routing  
* Forms and Validation (Reactive Forms and Validators)  
* Content Projection  
* onPush Change Detection  
* Route Guards, Pre-loads, Lazy-loading  
* Custom Pipes  
* @viewChild and @ContentChild Decorators  
* [Dynamic Components and ng-template](https://malcoded.com/posts/angular-dynamic-components/#:~:text=components%20actually%20are.-,What%20dynamic%20components%20are,in%20the%20application%20at%20runtime.)  
* @Host @Hostbinding and exportAs  
* State Management with RxJS  
* Dependency Injection and Zones
