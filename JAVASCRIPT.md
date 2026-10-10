## 2. JavaScript core

**What & why:** JavaScript (JS) is the programming language of the web. It runs in every browser (making pages interactive) and on servers via Node.js. In support you will read customer code, explain console errors, and write small scripts to reproduce bugs — so focus on **async code, errors, and the DOM**.

### Important functions
---
```js
//to run the call back function only once after specified delay
const id = setTimeout(callback, delay, arg1, arg2, ...);
// ar1 & ar2 is optional parameter passed to callback

//Runs function repeatedly in specified time delay
const id = setInterval(callback,delay,arg1,arg2)
cleanrInterval(id) //clear interval
```

### 2.1 Variables: var, let, const
---

|   |   |   |   |
|---|---|---|---|
|Keyword|Scope|Re-assign?|Hoisting|
|`var`|Function|Yes|Hoisted, starts as `undefined`|
|`let`|Block `{ }`|Yes|Hoisted but unusable before the line ("temporal dead zone")|
|`const`|Block `{ }`|No (but objects inside can still change)|Same as let|

```js
const user = { name: "JK" };
user.name = "Jairam";   // allowed — the object changed, not the binding
// user = {};            // TypeError: Assignment to constant variable
```

### 2.2 Data types and comparisons
---

| Keyword | Scope    | Can reassign? | Can redeclare in the same scope? |
| ------- | -------- | ------------- | -------------------------------- |
| `var`   | Function | Yes           | Yes                              |
| `let`   | Block    | Yes           | No                               |
| `const` | Block    | No            | No                               |
|         |          |               |                                  |
- `string` — `"hello"`
    
- `number` — `42`, `3.14`, and even `NaN`
    
- `boolean` — `true`, `false`
    
- `undefined` — a value has not been assigned
    
- `null` — intentional absence of a value
    
- `bigint` — large integers, such as `123n`

- `symbol` — unique identifiers
	
- **Primitives:** string, number, boolean, `null`, `undefined`, bigint, symbol.

## 4. `==` versus `===`

This is one of the most important concepts to master.

- `==` compares values after possible type conversion.
    
- `===` compares without type conversion.

``` javascript
console.log(5 == "5");  // true
console.log(5 === "5"); // false

console.log(0 == false);  // true
console.log(0 === false); // false

console.log(null == undefined);  // true
console.log(null === undefined); /
```
## 5. Functions and arrow functions
---

There are several ways to create functions.

```javascript
// Function declaration
function add(a, b) {
    return a + b;
}

// Function expression
const multiply = function(a, b) {
    return a * b;
};

// Arrow function
const subtract = (a, b) => a - b;

console.log(add(2, 3));       // 5
console.log(multiply(2, 3));  // 6
console.log(subtract(5, 2));  // 3
```

**`this` rule:** in a normal function, `this` is the object _before the dot_ when called. Arrow functions take `this` from the surrounding code. Losing `this` (e.g. passing `obj.normal` as a callback) is a common bug — fix with `.bind(obj)` or an arrow function.

### 2.4 Scope and closures

**Closure** = a function that remembers the variables around it, even after the outer function has finished.

```javascript
function makeCounter() {
  let count = 0;            // private variable
  return () => ++count;     // inner function "closes over" count
}
const next = makeCounter();
next(); // 1
next(); // 2
```

Used for private data, event handlers, and debounce/throttle (section 8).

### 2.5 Arrays and objects you must know

```js
const orders = [{ id: 1, total: 500 }, { id: 2, total: 1500 }];

orders.map(o => o.id);                 // [1, 2]       transform each
orders.filter(o => o.total > 1000);    // [{id:2...}]  keep some
orders.find(o => o.id === 2);          // {id:2...}    first match

const { id, total } = orders[0];       // destructuring
const copy = { ...orders[0], total: 600 }; // spread: copy + override
const safe = response?.data?.items ?? [];  // optional chaining + default
```

### Reduce() function
```js
const numbers = [1, 2, 3, 4];

const sum = numbers.reduce((acc, curr) => {return acc + curr;}, 0);

console.log(sum); // 10

/*
curr = current value
0 = initial value
if not initial value is give, js takes first values as initial values and starts, if array is empty, it throws error */

```
### Shallow Copy vs Deep Copy
`We use [...aryname] to make a shalow copy of any object`

| Aspect                    | Shallow Copy                                                                                                  | Deep Copy                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Definition**            | Copies only the first level of an object/array. Nested objects/arrays are shared by reference.                | Copies everything recursively, including all nested objects/arrays.            |
| **What is copied?**       | Top-level primitive values are copied. Nested objects/arrays are not copied — only their reference is copied. | All levels are copied. No references are shared with the original.             |
| **Nested objects/arrays** | Shared between original and copy.                                                                             | Completely separate copies.                                                    |
### Methods
#### Shallow 
```js
{ ...obj }
Object.assign({}, obj)  
[...arr]
arr.slice()  
[].concat(arr)  
Array.from(arr)
```

```js
structuredClone(obj)
JSON.parse(JSON.stringify(obj))
_.cloneDeep(obj)  
Custom recursive function
```
## Spread vs Rest
---


***Spread*** :Spread takes an array, object, or iterable and **expands it into individual elements/properties**.

***Rest :***Rest does the opposite: it **collects multiple values into one array or object**

| Spread                                                  | Rest                                                    |
| ------------------------------------------------------- | ------------------------------------------------------- |
| Expands values                                          | Collects values                                         |
| Used in function calls, array literals, object literals | Used in function parameters, array/object destructuring |
| `fn(...arr)`                                            | `function fn(...args)`                                  |
| `[...arr]`                                              | `const [a, ...b] = arr`                                 |
| `{ ...obj }`                                            | `const { a, ...rest } = obj`                            |
### 2.6 Asynchronous JavaScript 

**Why:** JS runs on one thread. Slow work (network, timers, files) is done _in the background_; JS continues and handles the result later.

Three styles, same idea:

```js
// 1. Callback (old)
setTimeout(() => console.log("later"), 1000);

// 2. Promise — an object that will hold a future value/state of the object: pending → fulfilled / rejected
function checkAge(age){
            return new Promise((resolve,reject)=>{
                if (age>= 18){
                resolve("you are eligible")
            }else{
                reject('you are not eligible')
            }
        })}

checkAge(19)
.then((res) => console.log(res))
.catch((err)=> console.log("Error",err))
.finally(()=> console.log("Done"));

// 3. async/await — cleaner syntax over promises
async function loadUser() {
  try {
    const res = await fetch("/api/user");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error("Failed:", err.message);
  }
}
```

**Running in parallel:**

```js
const [user, orders] = await Promise.all([getUser(), getOrders()]); // fails if ANY fails
const results = await Promise.allSettled([a(), b()]);               // never fails; check each status
```

**Common bugs:** forgetting `await` (you get a `Promise {<pending>}` instead of data), no `.catch` ("Uncaught (in promise)" error), using `await` inside `forEach` (it doesn't wait — use `for...of`).

### 2.7 The event loop 

Order of execution: **1) all synchronous code → 2) microtasks (promise callbacks) → 3) macrotasks (setTimeout, events)**.

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
// Output: A D C B
```

**Explanation:** A and D are synchronous. The promise callback (C) is a microtask and runs before the timer (B), even with a 0 ms delay.

### 2.8 The DOM and events
---
**DOM (Document Object Model)** = the browser's tree of objects built from the HTML. JS reads and changes the page through it.

```js
const btn = document.querySelector("#buy");        // select (null if not found!)
btn.textContent = "Buy now";                         // change text
btn.classList.add("active");                         // change style via class
btn.addEventListener("click", (e) => {
  e.preventDefault();                                // stop default (e.g. form submit)
  console.log("clicked", e.target);
});

// Event delegation: one listener on the parent handles many children (even ones added later)
document.querySelector("#list").addEventListener("click", (e) => {
  if (e.target.matches("li")) console.log("item", e.target.dataset.id);
});

// Run code only after the HTML is parsed
document.addEventListener("DOMContentLoaded", init);
```

**Event bubbling:** an event fires on the clicked element, then on each parent up to `document`. `e.stopPropagation()` stops it.

### 2.9 Errors you will see in tickets

|   |   |   |
|---|---|---|
|Error|Meaning|Usual fix|
|`TypeError: Cannot read properties of undefined (reading 'x')`|You did `obj.x` where `obj` is undefined|Check the data exists; use `obj?.x`|
|`TypeError: x is not a function`|Calling something that isn't a function|Typo, wrong import, script not loaded|
|`ReferenceError: x is not defined`|Variable never declared / script order|Load scripts in the right order|
|`SyntaxError: Unexpected token '<'` in JSON|Got HTML (an error page) when JSON was expected|Check URL and status code|
|`Uncaught (in promise)`|A rejected promise had no catch|Add try/catch or `.catch`|
|`Failed to fetch`|Network failure, CORS block, or bad URL|Check Network tab and console|

### 2.10 Storage, JSON and modules (quick facts)

- `JSON.stringify(obj)` → string; `JSON.parse(str)` → object (throws on invalid JSON).
- **localStorage** — 5–10 MB, survives restarts, same origin only. **sessionStorage** — cleared when the tab closes. **Cookies** — small, sent to the server with each request.
- **Modules:** `import { x } from "./a.js"` / `export`. In Node, older code uses `require()`.
- Debugging: `console.log`, `console.table`, `debugger;` statement, breakpoints in the Sources tab.