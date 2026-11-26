# React Notes - Udemy Course

<a id="top"></a>

## Table of Contents

- [JavaScript Foundations](#javascript-foundations)
- [React Application Structure and Components](#react-application-structure)
- [Props and Component Composition](#`props`-and-component-composition)
- [Events, State and Rendering](#events-state-and-rendering)
- [Styling, Refs and DOM Access](#styling-refs-and-dom)
- [Sharing State and Context](#sharing-state-and-context)
- [Portals and Form Handling](#portals-and-form-handling)
- [Context Providers and Reducer State](#context-providers-and-reducer-state)

---

- Type “react.new” in browser url bar to create a brand new react workspace in codesandbox

- You’ll need NodeJs for local development.

- You can use the Vite tool for local React project creation:

```bash
npm create vite@latest react-project
```

Running the above command will prompt you with a few questions. Follow this with `npm install`, and then a command to start a local development server.

<a id="javascript-foundations"></a>
## JavaScript Foundations

### JavaScript Refresher

### let & const

Read more about [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let).

Read more about [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const).

`let` and `const` basically replace `var`. You use `let` instead of `var` and `const` instead of `var` if you plan on never reassigning this variable (effectively turning it into a constant).


### ES6 Arrow Functions

Read more: [MDN - Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).

Arrow functions are a different way of creating functions in JavaScript. Besides a shorter syntax, they offer advantages when it comes to keeping the scope of the `this` keyword.

Arrow function syntax may look strange but it's actually simple.

```jsx
function callMe(name) {
console.log(name);
}
```

which you could also write as:

```jsx
const callMe = function(name) {
console.log(name);
}
```

becomes:

```jsx
const callMe = (name) => {
console.log(name);
}
```

**Important:**

When having **no arguments**, you have to use empty parentheses in the function declaration:

```jsx
const callMe = () => {
console.log('Max!');
}
```

When having **exactly one argument**, you may omit the parentheses:

```jsx
const callMe = name => {
console.log(name);
}
```

When **just returning a value**, you can use the following shortcut:

```jsx
const returnMe = name => name
```

That's equal to:

```jsx
const returnMe = name => {
return name;
}
```

### Exports & Imports

In React projects (and actually in all modern JavaScript projects), you split your code across multiple JavaScript files - so-called modules. You do this, to keep each file/ module focused and manageable.

To still access functionality in another file, you need `export` (to make it available) and `import` (to get access) statements.

You got two different types of exports: **default** (unnamed) and **named** exports:

```jsx
default => export default ...;
named => export const someData = ...;
```

You can import **default exports** like this:

```jsx
import someNameOfYourChoice from './path/to/file.js';
```

Surprisingly, `someNameOfYourChoice` is totally up to you.

**Named exports** have to be imported by their name:

```jsx
import { someData } from './path/to/file.js';
```

A file can only contain one default and an unlimited amount of named exports. You can also mix the one default with any amount of named exports in one and the same file.

When importing **named exports**, you can also import all named exports at once with the following syntax:

```jsx
import * as upToYou from './path/to/file.js';
```

`upToYou` is - well - up to you and simply bundles all exported variables/functions in one JavaScript object. For example, if you export `const someData = ...` from `/path/to/file.js`, you can access it on `upToYou` like this: `upToYou.someData`.

### Classes

Classes are a feature which basically replace constructor functions and prototypes. You can define blueprints for JavaScript objects with them.

Like this:

```jsx
class Person {
constructor () {
this.name = 'Max';
}
}
const person = new Person();
console.log(person.name); // prints 'Max'
```

In the above example, not only the class but also a property of that class (=> name ) is defined. The syntax you see there, is the "old" syntax for defining properties. In modern JavaScript projects (as the one used in this course), you can use the following, more convenient way of defining class properties:

```jsx
class Person {
name = 'Max';
}
const person = new Person();
console.log(person.name); // prints 'Max'
```

You can also define methods. Either like this:

```jsx
class Person {
name = 'Max';
printMyName () {
console.log(this.name); // this is required to refer to the class!
}
}
const person = new Person();
person.printMyName();
```

Or like this:

```jsx
class Person {
name = 'Max';
printMyName = () => {
console.log(this.name);
}
}
const person = new Person();
person.printMyName();
```

The second approach has the same advantage as all arrow functions have: the `this` keyword retains its lexical reference.

You can also use **inheritance** when using classes:

```jsx
class Human {
species = 'human';
}
class Person extends Human {
name = 'Max';
printMyName = () => {
console.log(this.name);
}
}
const person = new Person();
person.printMyName();
console.log(person.species); // prints 'human'
```

### Spread & Rest Operator

The spread and rest operators actually use the same syntax: ...

Yes, that is the operator - just three dots. It's usage determines whether you're using it as the spread or rest operator.

#### Using the Spread Operator

The spread operator allows you to pull elements out of an array (=> split the array into a list of its elements) or pull the properties out of an object. Here are two examples:

```jsx
const oldArray = [1, 2, 3];
const newArray = [...oldArray, 4, 5]; // This now is [1, 2, 3, 4, 5];
```

Here's the spread operator used on an object:

```jsx
const oldObject = {
name: 'Max'
};
const newObject = {
...oldObject,
age: 28
};
```

`newObject` would then be

```jsx
{
name: 'Max',
age: 28
}
```

The spread operator is extremely useful for cloning arrays and objects. Since both are , copying them safely (i.e. preventing future mutation of the copied original) can be tricky. With the spread operator you have an easy way of creating a (shallow!) clone of the object or array.

### Destructuring

Destructuring allows you to easily access the values of arrays or objects and assign them to variables.

Here's an example for an array:

```jsx
const array = [1, 2, 3];
const [a, b] = array;
console.log(a); // prints 1
console.log(b); // prints 2
console.log(array); // prints [1, 2, 3]
```

And here for an object:

```jsx
const myObj = {
name: 'Max',
age: 28
}
const {name} = myObj;
console.log(name); // prints 'Max'
console.log(age); // prints undefined
console.log(myObj); // prints {name: 'Max', age: 28}
```

Destructuring is very useful when working with function arguments. Consider this example:

```jsx
const printName = (personObj) => {
console.log(personObj.name);
}
printName({name: 'Max', age: 28}); // prints 'Max'
```

Here, we only want to print the name in the function but we pass a complete person object to the function. Of course this is no issue but it forces us to call personObj.name inside of our function. We can condense this code with destructuring:

```jsx
const printName = ({name}) => {
console.log(name);
}
printName({name: 'Max', age: 28}); // prints 'Max')
```

We get the same result as above but we save some code. By destructuring, we simply pull out the `name` property and store it in a variable/argument named `name`  which we then can use in the function body.

<a id="react-application-structure"></a>
## React Application Structure and Components

- `index.html` is the main file loaded. It primarily contains the following code:

```html
<div id="root"></div>
<script type="module" src="/src/index.jsx"></script>
```

- The `index.jsx` file primarily serves as the bootstrap for the React app:

```jsx
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';

const entryPoint = document.getElementById('root');
ReactDOM.createRoot(entryPoint).render(<App />);
```

- React components are just JavaScript functions.

- React Component functions must follow two important rule:
The function name must start with an uppercase character.
The function must return a value that can be rendered (“displayed on screen”) by React.
Example:
// header.jsx
export default function Header() {
	return ( // should return a tree with only 1 root element
		// alternative is to use fragment - surround this template with empty <> and </> brackets.
		// fragment helps us avoid adding redundant root wrapper element to the template.
		<div>
			<h1>Title</h1>
		</div>
	);

```jsx
}
```

- The React components can then be used are HTML elements in template:
<Header></Header>

- Bindings can be done using single curly braces, ex: {viewCount}

- React component files can have functions to segregate business logic. The functions can then be referred inside JSX template.

- Images should be loaded using an import statement:
import reactImg from “./assets/react-dom/client”;
And then in the template:
<img src={reactImg} alt=”Stylized atom” />

- Props enable components to be made reusable.
export default function Header(`props`)
React itself explicitly sets the individual properties on the `props` object.
Usage: <h1>{`props`.title}</h1>

<a id="`props`-and-component-composition"></a>
## Props and Component Composition

### More Prop Syntaxes

Beyond the various ways of setting and extracting `props` about which you learned in the previous lecture, there are **even more ways of dealing** with `props`.

But no worries, you'll see all these different features & syntaxes in action throughout the course!

#### Passing a Single Prop Object

If you got data that's already organized as a JavaScript object, you can pass that object as a single prop value instead of splitting it across multiple `props`.

I.e., instead of

```jsx
<CoreConcept
title={CORE_CONCEPTS[0].title}
description={CORE_CONCEPTS[0].description}
image={CORE_CONCEPTS[0].image} />
```

or

```jsx
<CoreConcept
```

- {...CORE_CONCEPTS[0]} />

you could also pass a single concept (or any name of your choice) prop to the CoreConcept component:

```jsx
<CoreConcept
concept={CORE_CONCEPTS[0]} />
```

In the CoreConcept component, you would then get that one single prop:

```jsx
export default function CoreConcept({ concept }) {
// Use concept.title, concept.description etc.
// Or destructure the concept object: const { title, description, image } = concept;
}
```

It is entirely up to you which syntax & approach you prefer.

#### Grouping Received Props Into a Single Object

You can also pass multiple `props` to a component and then, in the component function, group them into a single object via JavaScript's  syntax.

I.e., if a component is used like this:

```jsx
<CoreConcept
title={CORE_CONCEPTS[0].title}
description={CORE_CONCEPTS[0].description}
image={CORE_CONCEPTS[0].image} />
```

You could group the received `props` into a single object like this:

```jsx
export default function CoreConcept({ ...concept }) {
// ...concept groups multiple values into a single object
// Use concept.title, concept.description etc.
// Or destructure the concept object: const { title, description, image } = concept;
}
```

If that syntax is a bit confusing - worry not! You'll also see concrete examples for this syntax (and for why you might want to use it in certain situations) throughout the course!

#### Default Prop Values

Sometimes, you'll build components that may receive an optional prop. For example, a custom Button component may receive a type prop.

So the Button component should be usable either with a type being set:

```jsx
<Button type="submit" caption="My Button" />
```

Or without it:

```jsx
<Button caption="My Button" />
```

To make this component work, you might want to set a default value for the type prop - in case it's not passed.

This can easily be achieved since JavaScript supports default values when using object destructuring:

```jsx
export default function Button({ caption, type = "submit" }) {
// caption has no default value, type has a default value of "submit"
}
```

- **Component Composition:** `props.children` contains whatever content you have between your component tags.

- React provides many ‘on’ `props` to handle events. Example:
<button `onClick`={handleClick} />
function handleClick() {
	console.log(‘Clicked!’);
}
Important: we should not use it as `onClick`={handleClick()}, otherwise it will get executed immediately. We should only pass the handler function as a value, which will then be called/executed whenever the button is clicked.

- **Passing Functions as Values to Props:**
export default function TabButton({ children, onSelect }) {
	return (
		<button `onClick`={onSelect}>{children}</button>
	);
}

- **Passing Custom Arguments to Event Functions:**
We can pass custom values to event handler functions using arrow functions.
Example:
<TabButton onSelect={() => handleSelect()}>Components</TabButton>
And what this code will do is it will now make sure that this anonymous arrow function is now actually the value that's passed as a value to onSelect, and therefore in the end also to `onClick`. And hence, we can now write code inside of this arrow function body. So on the right side of this arrow that will be executed when this anonymous function will be executed. And therefore, here, we can and should now execute handleSelect, because this code will now not run immediately when this line of code gets parsed. Instead, when that line of code gets parsed it's just this arrow function that will be defined not the code inside of the arrow function that will be executed yet. And this arrow function is then passed as a value to the tap button on this onSelect prop. And therefore, when this function is executed because the button was clicked, it's then that this code inside of the arrow function will get executed. And therefore, now here we can execute handleSelect manually and we can control how it will be executed. And for example, pass this component's string identifier to it.

- React will only execute a component function ONCE.

- **Rules of Hooks:**
React Hooks must not be called outside of React component functions.
React Hooks must not be called in nested code statements (e.g. inside of if-statements)

- `useState()` Hook
useState() yields an array with exactly two elements
const [selectedTopic, setSelectedTopic] = useState(‘Please click a button’);

- The second element returned by useState will always be a function provided by React, that can be executed to update the state i.e., the selectedTopic variable.
So calling the setSelectedTopic function will also tell React to execute the component function again.

- The HTML content template in JSX code need not only be written in the return statement. We can extract out certain sections as variables in the component function, which can then eventually be used in the returned statement.

- CSS classes cannot be set on html elements simply using the class attribute.
We need to use the React `className` attribute instead.
We can also use a ternary operator to set the css class conditionally.
<button `className`={isSelected ? ‘active’ : undefined} />

#### Outputting List Data Dynamically
{CORE_CONCEPTS.map((conceptItem) => (
	<CoreConcept `key`={conceptItem.title} {...conceptItem} />
))}
Here, CORE_CONCEPTS is an array of concept objects.
CoreConcept is a custom React component.
We are using object destructuring on the conceptItem object, so that its properties are passed into the `props` of the CoreConcept component.
While iterating arrays in JSX to render html, React mandates us to have a `key` prop. The value for this `key` prop should be unique for each rendered instance of the iterated component.

#### Forwarding Props to Wrapped Elements

**Prop forwarding** means passing `props` received by a component down to a child component or HTML element.

It is mostly used when building reusable components (like custom buttons or inputs) so they accept standard HTML attributes (`disabled`, `onClick`, `aria-*`, `data-testid`) without listing every single prop manually.

### Simple Example

Instead of defining every property one by one, collect remaining `props` with `...rest` and spread them onto the target element:

```jsx
import React from 'react';

// Custom Button component that forwards extra props using ...rest
function CustomButton({ label, variant = 'primary', ...rest }) {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {label}
    </button>
  );
}

// Usage
export default function App() {
  return (
    <CustomButton
      label="Click Me"
      variant="secondary"
      // These extra props get forwarded to the inner <button>:
      disabled={false}
      onClick={() => alert('Clicked!')}
      data-testid="submit-btn"
    />
  );
}

```

##### Key Takeaways

1. **`...rest`** collects all extra `props` passed to the parent component.
2. **`{...rest}`** spreads those `props` directly onto the internal JSX element.
3. It makes components flexible, accessible, and easy to reuse.

#### JSX Slots

**JSX slots** is a pattern where you pass JSX elements into a component as `props` (such as `children` or custom `props`) so the component can place them in specific layout "slots."

Instead of hardcoding content inside a layout component, slots let the parent decide what goes where.

---

### Simple Example

```jsx
import React from 'react';

// Card component defines two slots: "header" and "children" (body)
function Card({ header, children }) {
  return (
    <div className="card">
      <div className="card-header">{header}</div>   {/* Header Slot */}
      <div className="card-body">{children}</div>    {/* Default Slot */}
    </div>
  );
}

// Usage
export default function App() {
  return (
    <Card header={<h2>User Profile</h2>}>
      {/* Anything inside the tags fills the default "children" slot */}
      <p>Name: Alex</p>
      <p>Role: Developer</p>
    </Card>
  );
}

```

---

##### Key Takeaways

1. **Default Slot (`children`):** Represents whatever you wrap inside opening and closing tags (`<Card>...</Card>`).
2. **Named Slots (Custom Props):** Pass JSX via named `props` (like `header={<Header/>}` or `footer={<Footer/>}`) to fill specific layout areas.
3. **Benefit:** Keeps wrapper components flexible, highly reusable, and free from hardcoded UI logic.

#### Setting Component Types Dynamically

This pattern is called **Setting Component Types Dynamically** (or passing polymorphic component slots).

It allows a parent component to decide **which HTML tag or custom component** should wrap its content.

---

### Key Rules

1. **Identifier Name:** To render a prop as a JSX tag, its variable name **must start with a capital letter** (e.g., `ButtonsContainer`, not `buttonsContainer`).
2. **Built-in HTML Tags:** Pass as a string (e.g., `"menu"`, `"ul"`, `"div"`).
3. **Custom Components:** Pass as a direct function reference inside curly braces without angle brackets (e.g., `{Section}`).

---

### Code Example

#### 1. The Dynamic Wrapper Component (`Tabs.jsx`)

You can accept the prop directly with a capital letter (e.g., `ButtonsContainer`), or remap a lowercase prop (`buttonsContainer`) to a capital variable inside the component.

```jsx
import React from 'react';

// Option A: Destructure directly as a capitalized variable
export default function Tabs({ buttons, children, ButtonsContainer = 'menu' }) {
  return (
    <>
      <ButtonsContainer>{buttons}</ButtonsContainer>
      {children}
    </>
  );
}

/*
// Option B: Remap lowercase prop to an uppercase local variable
export default function Tabs({ buttons, children, buttonsContainer = 'menu' }) {
  const ButtonsContainer = buttonsContainer;
  return (
    <>
      <ButtonsContainer>{buttons}</ButtonsContainer>
      {children}
    </>
  );
}
*/

```

#### 2. Usage (`Examples.jsx`)

```jsx
import React from 'react';
import Tabs from './Tabs.jsx';
import Section from './Section.jsx'; // Custom component

export default function Examples() {
  return (
    <>
      {/* 1. HTML tag as a String */}
      <Tabs
        ButtonsContainer="menu"
        buttons={<button>Tab 1</button>}
      >
        <p>Tab Content</p>
      </Tabs>

      {/* 2. Custom Component as a Function Reference */}
      <Tabs
        ButtonsContainer={Section}
        buttons={<button>Tab 2</button>}
      >
        <p>Tab Content</p>
      </Tabs>
    </>
  );
}

```

---

### Summary

* **Capital Letter:** React treats `<buttonsContainer>` as an unknown HTML element. Capitalizing it (`<ButtonsContainer>`) tells React to render the evaluated value (either a string tag or a component function).
* **Flexibility:** Keeps wrapper structure flexible across different parts of an application while maintaining component logic.

- Props can have default values - similar to C# method parameters.

- Use the public/ folder for any images that should not be handled by the build process and that should be generally available. Good candidates are images used directly in the `index.html` file or favicons.

On the other hand, images that are used inside of components should typically be stored in the src/ folder (e.g., in `src/`assets/).
Any files (of any format) stored in src (or subfolders like `src/`assets/) are not made available to the public. They can't be accessed by website visitors. If you try loading localhost:5173/`src/`assets/some-image.jpg, you'll get an error.

<a id="events-state-and-rendering"></a>
## Events, State and Rendering

### Updating State Based on Old State
Wrong: setIsEditing(!isEditing);
If your new state depends on your previous state value, you should not update the state like this.
Correct: setIsEditing(isEditing => !isEditing);
Instead, pass a function to your state updating function. The function will automatically be called by React and will receive the guaranteed latest state value.
Reason: React is schedule state updates. State updates are not performed instantly but at some time in the future (when React has time for it). In most cases, those state updates of course still are almost instantly.

### Two-Way Binding

**Two-way data binding** in React connects a component's state to an input element so that:

1. **State updates the UI:** When the state variable changes, the input's visible value updates automatically.
2. **UI updates the state:** When the user types or interacts with the input, an `onChange` handler updates the state.

React does not have built-in two-way binding like Angular or Vue; you construct it manually by combining a **`value` prop** and an **`onChange` event**.

---

### Code Example

```jsx
import React from 'react';

export default function InputExample() {
  const [name, setName] = React.useState('');

  return (
    <div>
      {/* 1. Binding State -> UI (value={name}) */}
      {/* 2. Binding UI -> State (onChange updates 'name') */}
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />

      <p>Your name is: {name}</p>
    </div>
  );
}

```

---

### How It Works

* **Data to UI (`value={name}`):** The `<input>` gets its value directly from the `name` state variable.
* **UI to Data (`onChange`):** Every keypress fires the event listener, reading `e.target.value` and calling `setName()` to update the state, which triggers a re-render.

### Updating Object State Immutably

This topic covers two critical React state management rules: **updating state based on previous state** and **updating objects/arrays immutably**.

---

### Key Concepts

1. **Functional State Updates:** When your new state depends on the previous state, pass a function to the state setter (`setBoard(prevBoard => ...)`). React guarantees `prevBoard` is the latest snapshot.
2. **Immutable State Updates:** Never mutate state directly (e.g., `prevBoard[row][col] = 'X'`). Objects and arrays are reference values in JavaScript. Mutating them directly edits memory before React processes the update, causing bugs. Instead, **make a deep copy**, modify the copy, and return it.
3. **Passing Arguments to Event Handlers:** Use an anonymous function inside `onClick={() => handleSelectSquare(rowIndex, colIndex)}` to pass parameters to event functions.

---

### Code Example

```jsx
import React from 'react';

const initialGameBoard = [
  [null, null, null],
  [null, null, null],
  [null, null, null]
];

export default function GameBoard() {
  const [gameBoard, setGameBoard] = React.useState(initialGameBoard);

  function handleSelectSquare(rowIndex, colIndex) {
    setGameBoard((prevGameBoard) => {
      // 1. Deep copy the nested array immutably
      const updatedBoard = prevGameBoard.map(row => [...row]);

      // 2. Modify the copy, not the original state
      updatedBoard[rowIndex][colIndex] = 'X';

      // 3. Return the new updated array
      return updatedBoard;
    });
  }

  return (
    <ol id="game-board">
      {gameBoard.map((row, rowIndex) => (
        <li key={rowIndex}>
          <ol>
            {row.map((playerSymbol, colIndex) => (
              <li key={colIndex}>
                {/* Anonymous function wrapper passes parameters */}
                <button onClick={() => handleSelectSquare(rowIndex, colIndex)}>
                  {playerSymbol}
                </button>
              </li>
            ))}
          </ol>
        </li>
      ))}
    </ol>
  );
}

```

---

### Summary

* **Do:** `const updated = prev.map(row => [...row]);`
* **Don't:** `prev[row][col] = 'X'; return prev;`
* **Why:** Immutable updates ensure predictable component renders and prevent subtle state synchronization bugs.

<a id="styling-refs-and-dom"></a>
## Styling, Refs and DOM Access

- How is CSS Style scoping achieved in React?

React doesn't scope CSS by default—standard CSS imports apply to the entire page.

The two simplest ways to achieve style scoping are:

### 1. CSS Modules (Most Popular)

Rename your file to `.module.css`. Build tools automatically rename your classes into unique hashes so they don't leak.

```css
/* Button.module.css */
.btn {
  background-color: blue;
  color: white;
}

```

```jsx
// Button.jsx
import styles from './Button.module.css';

export function Button() {
  // Generates class="_btn_1a2b3_5" under the hood
  return <button className={styles.btn}>Click Me</button>;
}

```

---

### 2. Inline Styles (JavaScript Objects)

Pass styles directly as JavaScript objects using the `style` prop. Since the styles apply directly to that HTML element, they cannot leak.

```jsx
export function Button() {
  const btnStyle = {
    backgroundColor: 'blue',
    color: 'white',
  };

  return <button style={btnStyle}>Click Me</button>;
}

```

---

### Quick Choice:

* Use **CSS Modules** for real-world projects (supports hover states, media queries, and pseudoclass selectors).
* Use **Inline Styles** for small, simple dynamic tweaks.

### `useRef` React Hook

`useRef` is a React Hook that lets you **persist values across renders** without triggering a re-render when that value changes.

Think of it like a secret pocket in your component: you can store data in it or grab hold of a DOM element, and changing what's inside won't force React to redraw the screen.

---

### Key Characteristics

* **Returns an Object:** It gives you a plain object with a single property: `{ current: initialValue }`.
* **No Re-renders:** Modifying `ref.current` does **not** cause your component to re-render (unlike `useState`).
* **Persistent:** The stored value stays intact across all component re-renders.

---

### The 2 Main Use Cases

#### 1. Accessing DOM Elements directly

The most common use case is interacting directly with HTML elements (focusing inputs, measuring size, scrolling).

```jsx
import { useRef } from 'react';

function TextInputWithFocusButton() {
  // 1. Create the ref
  const inputEl = useRef(null);

  const onButtonClick = () => {
    // 3. Access the DOM element directly via .current
    inputEl.current.focus();
  };

  return (
    <>
      {/* 2. Attach the ref to the HTML element */}
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}

```

#### 2. Storing Mutable Data (Like a Timer ID)

Use `useRef` when you need to keep track of a variable that shouldn't cause the UI to update when changed.

```jsx
import { useRef, useState } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  // Store the timer ID without triggering re-renders when it changes
  const timerId = useRef(null);

  const startTimer = () => {
    if (timerId.current !== null) return;

    timerId.current = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerId.current);
    timerId.current = null; // Reset ref
  };

  return (
    <div>
      <h1>Time: {seconds}s</h1>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}

```

---

### `useState` vs `useRef`

| Feature | `useState` | `useRef` |
| --- | --- | --- |
| **Updating triggers re-render?** | Yes | No |
| **Used for...** | Data displayed on the screen | Internal values, timers, DOM nodes |
| **Access syntax** | `stateValue` | `refValue.current` |

- Whenever state changes, the React component function is re-executed. And that, of course, also means that all the variables and constants local to the component function are recreated.

- Use refs when you need to store data that:

must survive re-renders,

is specific to a component instance,

does not directly affect the UI.

<a id="sharing-state-and-context"></a>
## Sharing State and Context

### Sharing State Across Components in React

Sharing state across components in React depends on how far apart those components are in your tree structure.

---

### 1. Lifting State Up (Parent & Siblings)

When two sibling components need access to the same state, move that state up to their closest common parent and pass it down via `props`.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <Display count={count} />
      <Button setCount={setCount} />
    </>
  );
}

function Display({ count }) {
  return <p>Count: {count}</p>;
}

function Button({ setCount }) {
  return <button onClick={() => setCount(c => c + 1)}>Increment</button>;
}

```

---

### 2. React Context API (Global or Deeply Nested State)

To avoid passing `props` through many intermediate components ("prop drilling"), wrap a section of your app in a `Context.Provider`.

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create Context
const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 2. Consume Context anywhere below
function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      Current Theme: {theme}
    </button>
  );
}

```

---

### 3. Component Composition (Passing JSX as Props)

Instead of drilling state down to a deep child, render that child directly at the top level and pass it as `children` or a named prop.

```jsx
function Layout({ userProfile }) {
  // Pass JSX directly instead of passing the user object down through Layout
  return <Sidebar>{userProfile}</Sidebar>;
}

```

---

### 4. External State Management Libraries

For complex applications where state updates frequently across unrelated components, external libraries offer global stores optimized for selective re-renders:

* **Zustand / Redux Toolkit:** Best for scalable application-wide state management.
* **Jotai / Recoil:** Atomic state models ideal for fine-grained dependency tracking.
* **TanStack Query (React Query) / SWR:** Specifically designed to manage **server/API state** (caching, fetching, updating).

---

### Summary Checklist

| Method | Best Used For |
| --- | --- |
| **Lifting State Up** | Closely related components (parent-child or direct siblings). |
| **Component Composition** | Avoiding mild prop drilling by restructuring markup. |
| **React Context** | Low-frequency updates shared globally (e.g., Theme, Auth, User Settings). |
| **External Stores** | Large apps, frequent updates, or server data caching. |

<a id="portals-and-form-handling"></a>
## Portals and Form Handling

### Portals in React

**React Portals** let you render a component's HTML outside of its parent component's DOM hierarchy, while still keeping it connected to the React component tree.

Think of it as a portal gun: your code lives in one place in the React tree, but its rendered visual output shoots out into a completely different HTML element on the page.

---

### Why Use Portals?

By default, child components are nested directly inside their parent's HTML element. This causes layout issues when a child needs to float over the whole page (like modals, tooltips, or popovers) and gets clipped or hidden by a parent with CSS styles like `overflow: hidden`, `z-index`, or `position: relative`.

Portals bypass these CSS constraints by mounting the element directly under `document.body` or a top-level container.

---

### Syntax

```jsx
import { createPortal } from 'react-dom';

createPortal(children, container);

```

* **`children`**: Any renderable React node (JSX, string, component, etc.).
* **`container`**: The DOM element where you want to render the portal (e.g., `document.body`).

---

### Basic Example: Modal

**1. Set up a target container in your HTML (`index.html`)**

```html
<div id="root"></div>
<!-- Target container for portals -->
<div id="modal-root"></div>

```

**2. Create the Modal component with `createPortal**`

```jsx
import { createPortal } from 'react-dom';

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  // Render the modal directly into #modal-root instead of its parent div
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.getElementById('modal-root')
  );
}

```

**3. Use it inside any normal parent component**

```jsx
import { useState } from 'react';

function ProfileCard() {
  const [showModal, setShowModal] = useState(false);

  return (
    // Even if this container has `overflow: hidden`, the Modal won't get cut off!
    <div style={{ overflow: 'hidden', height: '100px' }}>
      <h3>User Profile</h3>
      <button onClick={() => setShowModal(true)}>Open Modal</button>

      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>Modal Title</h2>
        <p>This renders outside the regular DOM hierarchy!</p>
      </Modal>
    </div>
  );
}

```

---

### Key Things to Remember

* **Event Bubbling Works Normally:** Even though the DOM element is rendered elsewhere, React events (like `onClick`) will still bubble up through the *React parent tree*, not the physical DOM tree.
* **Common Use Cases:** Modals, dialog boxes, tooltips, dropdown menus, and toast notifications.

### 1. Ref-Based Form Handling vs. Controlled State

```jsx
**Controlled Components (State per Keystroke):** Listening to onChange events to update React state on every single keystroke. While useful for instant validation or reactive UI updates, it causes the component to re-render on every character typed.
**Uncontrolled Components (useRef for Input Extraction):** Using React refs to attach directly to underlying DOM elements (inputs/textareas) to pull their .value property only when an action occurs (e.g., clicking a "Save" button). This avoids unnecessary re-renders during typing when immediate reactivity isn't needed.
```

### 2. React Version Differences in Ref Forwarding (`forwardRef`)

```jsx
**React 18 and Earlier:** ref is a reserved keyword and is **not** passed as a regular prop to custom components. To pass a ref down to a custom component's underlying DOM element, you must wrap the child component function with forwardRef((props, ref) => ...) to explicitly receive and forward the ref parameter.
**React 19+:** The ref prop behaves like any normal prop. It can be passed directly to custom functional components and accessed from props.ref without needing to wrap the component in forwardRef.
```

### 3. Imperative Component APIs (useImperativeHandle)

```jsx
**Exposing Imperative Methods:** React follows a declarative paradigm, but occasionally you need an imperative handle (e.g., opening or closing a modal from a parent component).
**`useImperativeHandle` Hook:** Used alongside useRef and forwardRef to customize the object instance exposed to a parent component when using a ref. Instead of exposing the entire inner DOM node, it selectively exposes specific methods (such as an .open() method).
```

### 4. DOM Portals (`createPortal`)

```jsx
**React Portals:** Render JSX children into a DOM node that exists **outside** the parent component's DOM hierarchy (such as a <div id="modal-root"> directly under <body>).
**Why Use It:** Helps avoid layout bugs (e.g., z-index stacking context issues or overflow: hidden parent containers clipping popups) while maintaining standard React event bubbling behavior.
```

### 5. Native HTML <dialog> Element

```jsx
**Built-in Dialog Features:** Utilizing the native HTML5 <dialog> element alongside its JavaScript API methods like .showModal().
**Method `method="dialog"` in Forms:** Placing a <form method="dialog"> inside a <dialog> element automatically closes the modal when any submit button inside that form is clicked, without needing explicit custom JavaScript event handlers.
```

### 6. React State Lifting & Immutability

```jsx
**Lifting State Up:** Moving state to the closest common ancestor (App.jsx) when two sibling components (NewProject and Sidebar) need access to the same dynamic data (generating new project data vs. listing projects).
**Functional State Updates:** Updating state using the callback signature—setProjectsState(prevState => ...)—to ensure updates safely rely on the most recent state snapshot without race conditions.
**Immutable State Operations:** Using the JavaScript spread operator (...) to create shallow copies of objects and arrays before adding new items, ensuring React state is never mutated directly.
```

### 7. React Development Behaviors (`StrictMode`)

```jsx
**Double Execution in StrictMode:** <React.StrictMode> intentionally mounts and renders components twice in development environments to help developers spot side effects, memory leaks, and unintended state mutations early.
```

### 8. Dynamic List Rendering & Key Props

```jsx
**Array `.map()`:** Transforming arrays of JavaScript objects into arrays of dynamic JSX elements.
**key Prop:** Providing a unique string/number identifier (key={project.id}) to root list items so React can efficiently track, re-order, add, or remove elements in the Virtual DOM without unmounting unnecessary UI elements.
```

<a id="context-providers-and-reducer-state"></a>
## Context Providers and Reducer State

```jsx
**useContext**
```

Here is a concise breakdown of the concepts explained in the transcript.

---

### Core Concept: Eliminating Prop Drilling with React Context

**Prop drilling** occurs when data is passed through intermediate components that do not actually need it, just to reach a deeply nested component. **React Context** solves this by providing a global state that any component in the tree can consume directly.

```
BEFORE (Prop Drilling):
App -> Header -> CartModal -> Cart (Props passed through intermediate layers)

AFTER (Context API):
CartContext (Store) <--- consumed directly by ---> Header & Cart

```

---

### Step-by-Step Refactoring Process

#### 1. Decouple Unnecessary Prop Pass-Throughs

Remove `props` from wrapper components that only exist to forward data.

* **Component (`CartModal.jsx`):** Serves solely as an UI container. Instead of accepting `cartItems` or `onUpdateCartItemQuantity` to forward down to ``, remove all cart-related `props` and Context imports from it entirely.

#### 2. Consume Read-Only State Directly

Use ``useContext`` directly inside the component that requires display data.

* **Component (`Header.jsx`):** Needs the current cart item count.

```jsx
import { useContext } from 'react';
import { CartContext } from '../store/shopping-cart-context.jsx';

export default function Header() {
  const { items } = useContext(CartContext);
  const totalQuantity = items.length;

  return

```

Cart Items: {totalQuantity}

;
}

```

#### 3. Expose State Manipulation Functions via Context
Expose updater functions alongside state data inside the Context provider object.
* **Store (`shopping-cart-context.jsx`):** Add function references to both the default context shape (for auto-completion) and the provider's actual value object.
```
```jsx
export const CartContext = createContext({
  items: [],
  updateItemQuantity: () => {} // Added for auto-completion
});

// Inside Context Provider component:
const contextValue = {
  items: cartState.items,
  updateItemQuantity: handleUpdateCartItemQuantity
};

```

#### 4. Consume Actions at the Action Point

Access state updater functions directly inside the child component where the action occurs.

* **Component (`Cart.jsx`):** Directly trigger actions via the consumed context without parent props.

```jsx
import { useContext } from 'react';
import { CartContext } from '../store/shopping-cart-context.jsx';

export default function Cart() {
  const { items, updateItemQuantity } = useContext(CartContext);

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          <button onClick={() => updateItemQuantity(item.id, -1)}>-</button>
          {item.quantity}
          <button onClick={() => updateItemQuantity(item.id, 1)}>+</button>
        </li>
      ))}
    </ul>
  );
}
```

---

##### Key Takeaway

By converting state management to **React Context**:

* Intermediate components (`App`, `Header`, `CartModal`, `Product`) become cleaner and prop-free.
* Components consume only the state or functions they actually require (`Header` consumes `items`; `Cart` consumes `items` and `updateItemQuantity`).

### React Context Provider Pattern

### React Context Provider Pattern (Short Summary)

#### 1. Problem: Bloated `App` Component
When you manage context state directly in `App`, it can become cluttered with:
- `useState`
- State update functions
- Context value creation
- Provider setup

**Example:**
```jsx
function App() {
const [cart, setCart] = useState([]);

const contextValue = {
cart,
addItem,
removeItem
};

return (
<CartContext.Provider value={contextValue}>
<Header />
<Shop />
</CartContext.Provider>
);
}
```

---

#### 2. Solution: Create a Custom Context Provider
Move all context-related logic into a dedicated component.

**Example:**
```jsx
export function CartContextProvider({ children }) {
const [cart, setCart] = useState([]);

const contextValue = {
cart,
addItem,
removeItem
};

return (
<CartContext.Provider value={contextValue}>
{children}
</CartContext.Provider>
);
}
```

---

#### 3. Simplify the App Component
Now `App` only wraps components with the provider.

**Example:**
```jsx
function App() {
return (
<CartContextProvider>
<Header />
<Shop />
</CartContextProvider>
);
}
```

---

#### 4. Benefits
✅ Cleaner `App` component

✅ Better separation of concerns

✅ Easier maintenance

✅ Scales well with multiple contexts

✅ Context logic is centralized

---

#### Real-World Analogy

Think of `App` as a **house owner**.

❌ **Without Provider:**
- Owner manages electricity, water, security, and internet directly.

✅ **With Provider:**
- Separate service providers handle each responsibility.
- Owner simply connects to them.

```text
App
├─ CartContextProvider
├─ UserContextProvider
└─ ThemeContextProvider
```

The result is a **lean App component and modular state management**.

---

#### Key Takeaway

Instead of managing context state inside `App`, create a dedicated **Context Provider Component** that:

1. Manages the state.
2. Creates the context value.
3. Wraps child components with the provider.

This keeps your application **clean, scalable, and easier to maintain**.

### `useReducer` Hook

useReducer is a React Hook designed for managing **complex state logic**. While `useState` works well for independent, single values, useReducer is better when state involves multiple sub-values, complex nested updates, or depends heavily on the previous state.

It follows the same state-management pattern used by **Redux**.

---

### The 4 Core Parts of `useReducer`

1. **State:** The current data store (e.g., `{ count: 0 }`).
2. **Action:** An object describing *what event occurred* (e.g., `{ type: 'INCREMENT' }`).
3. **Reducer:** A pure function that takes the `(currentState, action)` and returns the **brand-new state**.
4. **Dispatch:** A function used to send actions to the reducer to trigger state updates.

---

### Basic Syntax

```jsx
const [state, dispatch] = useReducer(reducerFunction, initialState);

```

---

### Step-by-Step Example: Counter App

#### 1. Define the Reducer Function

The reducer takes the current state and an action, evaluating what to return using a `switch` statement:

```jsx
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      return state; // Return existing state if action is unrecognized
  }
}

```

#### 2. Use the Hook in Your Component

```jsx
import { useReducer } from 'react';

export default function Counter() {
  // Initialize useReducer with the reducer function and starting state
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <>
      <h1>Count: {state.count}</h1>
      {/* Dispatch actions when buttons are clicked */}
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </>
  );
}
```

---

### Passing Data via Payloads

Actions can also carry additional data (called a **payload**) to update state dynamically.

```jsx
// Reducer handling payloads
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return { items: [...state.items, action.payload] };
    default:
      return state;
  }
}

// Inside Component:
function handleAdd() {
  dispatch({
    type: 'ADD_ITEM',
    payload: { id: 101, name: 'T-Shirt', price: 20 }
  });
}

```

---

### `useState` vs. `useReducer`

| Feature | `useState` | useReducer |
| --- | --- | --- |
| **State Complexity** | Simple primitives (strings, booleans, numbers) | Complex objects, deeply nested state, arrays |
| **Update Logic** | Inline inside event handlers | Centralized inside a single reducer function |
| **Next State Logic** | Independent or simple updates | Depends on previous state / complex conditionals |
| **Testing** | Harder to test logic in isolation | Easy—reducer is a pure function exportable for testing |

---

[Back to top](#top)
