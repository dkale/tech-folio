# My Questions for Conducting an Interview

## Table of Contents
- [General, Leadership & Interview Approach](#general-leadership--interview-approach)
- [Object-Oriented Programming, C# & .NET](#object-oriented-programming-c--net)
- [Concurrency, Asynchronous & Reactive Programming](#concurrency-asynchronous--reactive-programming)
- [JavaScript](#javascript)
- [Angular & RxJS](#angular--rxjs)
- [Vue.js](#vuejs)
- [Web Components, DOM & UI Concepts](#web-components-dom--ui-concepts)
- [Globalization & Localization](#globalization--localization)

## General, Leadership & Interview Approach

- Have you worked closely on the basics of the technology or simply worked on the business implementation?
- Will you be able to work independently, drive yourself, or lead a team? Do you have experience of leading a team?
- Rate yourself: OOPS, JavaScript, Angular, Vue

[↑ Back to Top](#table-of-contents)

## Object-Oriented Programming, C# & .NET

### What is the “Direct Connection” and “Indirect Connection” w.r.t Entity Framework? (Diaspark Interview)

### What is Eager loading and Lazy loading in Entity Framework?

### What is the difference between static and singleton?

OOPS

What are SOLID Principles?

What is dependency injection?

What is a module? How do you implement lazy loading?

How to extend an existing ASP.NET application to support an Angular application as a part of it.

[↑ Back to Top](#table-of-contents)

## Concurrency, Asynchronous & Reactive Programming

What is Reactive Programming? What is Responsive Programming?

What is multi-threading?

What is asynchronous programming?

Difference between Concurrency and Parallelism?

Concurrency means executing multiple tasks at the same time but not necessarily simultaneously. When you have to perform more than one task but you have a single resource then we go for concurrency. In a single-core environment, concurrency is achieved by context switching.

Parallelism is like performing more than one task simultaneously like you can sing and bath together. Now you are doing the tasks in parallel.

The term asynchronous is related to thread execution. In an asynchronous model, when one task gets executed, you can switch to a different task without waiting for the previous task to get completed.

Asynchronous programming helps us to achieve concurrency. Asynchronous programming in a multi-threaded environment is a way to achieve parallelism.

Is JS Multithread?

How to achieve multi-threading in JS?

JS is not MT bcos JS interpreter in browser is single threaded. You can use setTimeout to allow somesort of scheduling and "fake" concurrecy.

What is required for JS(or any other language) to be multi-threaded?

Shared memory, locks, semaphors and other concurrent programming tools.

What are Web Workers?

A web worker is a JavaScript that runs in the background, independently of other scripts, without affecting the performance of the page. You can continue to do whatever you want: clicking, selecting things, etc., while the web worker runs in the background.

What is Event Queue, Event Loop?

[↑ Back to Top](#table-of-contents)

## JavaScript

Difference between == and === operator in JS?

### What are JavaScript closures? When would you use them?

Two one sentence summaries:

A closure is the local variables for a function - kept alive after the function has returned, or

A closure is a stack-frame which is not deallocated when the function returns.

A closure takes place when a function creates an environment that binds local variables to it in such a way that they are kept alive after the function has returned. A closure is a special kind of object that combines two things: a function, and any local variables that were in-scope at the time that the closure was created.

The following code returns a reference to a function:

```javascript
function sayHello2(name) {
var text = ‘Hello ‘ + name;	// local variable
var sayAlert = function() {
```

alert(text);

```javascript
}
return sayAlert;
}
```

test = sayHello(' World')

test()

Closures reduce the need to pass state around the application. The inner function has access to the variables in the outer function so there is no need to store the information somewhere that the inner function can get it.

This is important when the inner function will be called after the outer function has exited. The most common example of this is when the inner function is being used to handle an event. In this case you get no control over the arguments that are passed to the function so using a closure to keep track of state can be very convenient.

What are promises?

What is async and await in JS?

### How this works in JavaScript : What is the result of the following code ? Explain your answer.

```javascript
var fullname = 'John Doe';
var obj = {
```

fullname: 'Colin Ihrig',

prop: {

fullname: 'Aurelio De Rosa',

getFullname: function() {

```javascript
return this.fullname;
}
}
};
console.log(obj.prop.getFullname());
var test = obj.prop.getFullname;
console.log(test());
```

Answer:

The code prints Aurelio De Rosa and John Doe. The reason is that the context of a function, what is referred with the ‘this’ keyword, in JavaScript depends on how a function is invoked, not how it’s defined.

In the first console.log() call, getFullName() is invoked as a function of the obj.prop object. So, the context refers to the latter and the function returns the fullname property of this object. On the contrary, when getFullName() is assigned to the test variable, the context refers to the global object (window). This happens because test is implicitly set as a property of the global object. For this reason, the function returns the value of a property called fullName of window, which in this case is the one the code set in the first link of the snippet.

### Hoisting : What’s the result of executing this code and why.

```javascript
function test() {
console.log(a);
console.log(foo());
var a = 1;
function foo() {
return 2;
}
}
```

test();

Answer:

The result of this code is undefined and 2.

The reason is that both variables and functions are hoisted (moved at the top of the function) but variables don’t retain any assigned value. So, at the time the variable a is printed, it exists in the function (it’s declared) but it’s still undefined. Stated in other words, the code above is equivalent to the following:

```javascript
function test() {
var a;
function foo() {
return 2;
}
console.log(a);
console.log(foo());
```

a = 1;

```javascript
}
```

test();

### What will the following code output to the console and why:

```javascript
var hero = {
```

_name: 'John Doe',

getSecretIdentity: function (){

```javascript
return this._name;
}
};
var stoleSecretIdentity = hero.getSecretIdentity;
console.log(stoleSecretIdentity());
console.log(hero.getSecretIdentity());
```

What is the issue with this code and how can it be fixed ?

Answer:

The code will output:

undefined

John Doe

The first console.log prints undefined because we are extracting the method from the hero object, so stoleSecretIdentity() is being invoked in the global context (i.e., the window object) where _name property does not exist.

One way to fix the stoleSecretIdentity() function is as follows:

```javascript
var stoleSecretIdentiy = hero.getSecretIdentity.bind(hero);
```

### In what order will the numbers 1-4 be logged to the console when the code below is executed ? Why ?

(function() {

```javascript
console.log(1);
setTimeout(function() {console.log(2)}, 1000);
setTimeout(function() {console.log(3)}, 0);
console.log(4);
})();
```

Answer:

The values will be logged in the following order:

1

4

3

2

Let’s first explain the parts of this that are presumably more obvious:

1 and 4 are displayed first since they are logged by simple calls to console.log() without any delay.

2 is displayed after 3 because 2 is being logged after a delay of 1000 msecs (i.e. 1 second) whereas 3 is being logged after a delay of 0 msecs.

OK, fine. But if 3 is being logged after a delay of 0 msecs, doesn’t that mean it is being logged right away ? And, if so, shouldn’t it be logged before 4, since 4 is being logged by a later line of code ?

The answer has to do with properly understanding JavaScript events and timing.

The browser has an event loop which checks the event queue and processes pending events. For example, if an event happens in the background (e.g., a script onload event while the browser is busy (e.g., processing an onclick), the event gets appended to the queue. When the onclick handler is complete, the queue is checked and the event is then handled (e.g., the onload script is executed).

Similarly, setTimeout() also puts execution of its referenced function into the event queue if the browser is busy.

When a value of zero is passed as the second argument to setTimeout(), it attempts to execute the specified function “as soon as possible”. Specifically, execution of the function is placed on the event queue to occur on the next timer tick. Note, though, that this is not immediate; the function is not executed until the next tick. That’s why in the above example, the call to console.log(4) occurs before the call to console.log(3) (since the call to console.log(3) is invoked via setTimeout, so it is slightly delayed.

[↑ Back to Top](#table-of-contents)

## Angular & RxJS

What are Observable? Hot-Cold? BehaviorSubject? ReplaySubject? EventEmitter?

Difference between ngOnInit() and constructor()? What kind of code you will write where?

What are Angular Lifecycle Hooks and why are they ?

How does angular change detection work? ChangeDetectionStrategy?

What are Decorators?

What is content projection?

What is Tree Shaking in Angular?

Tree shaking is a step in a build process that removes unused code so the application becomes smaller. It can be visualised as shaking a physical tree causing dead branches and leaves to fall off.

[↑ Back to Top](#table-of-contents)

## Vue.js

Explain what the <slot> element is in Vue.js.

With Vue slots, you can structure an HTML interface (like with TypeScript), which you can then use as a guide to build out your components by template injection. It is a very scalable and efficient solution for passing down template code from one component to another.

What is a scoped slot in Vue.js?

A scoped slot in Vue.js is a way to pass data from a parent component to a child component, allowing the child component to render content based on that data. The slot provides a template that the parent can fill in with data, and the child component can use that data to render content in a specific way. The scope of the data is limited to the child component, hence the term "scoped" slot.

What is a single-file component?

[↑ Back to Top](#table-of-contents)

## Web Components, DOM & UI Concepts

What are Web components?

What is Shadow DOM, Virtual DOM, Difference?

What is ViewEncapsulation?

[↑ Back to Top](#table-of-contents)

## Globalization & Localization

What is Globalization/Localization?

Globalization is making software adaptable to different languages and regions, so it’s available around the world.

Localization is adapting software for a particular region or language. It involves not only the language but also cultural elements specific to a region, such as the preferred date and time format, currency and number formatting, and units of measurement.

How could you write a method on an instance of a date which will give you the next day ?

I have to declare a method on the prototype of the Date object. To get access to the current value of the instance of the date, I will use this.

```javascript
Date.prototype.nextDay = function(){
var currentDate = this.getDate();
return new Date(this.setDate(currentDate +1));
}
var date = new Date();
```

date; //Fri May 16 2014 20:47:14 GMT-0500 (Central Daylight Time)

date.nextDay();//Sat May 17 2014 20:47:14 GMT-0500 (Central Daylight Time)

[↑ Back to Top](#table-of-contents)
