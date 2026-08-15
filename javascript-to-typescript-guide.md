# JavaScript → TypeScript: Complete Learning Guide

> Read top to bottom. Every topic has a plain explanation + runnable example.

---

## Table of Contents

**Part 1 — JavaScript Fundamentals**
1. What is JavaScript
2. Variables (`var`, `let`, `const`)
3. Data Types
4. Operators
5. Type Conversion & Coercion
6. Control Flow (if/else, switch)
7. Loops
8. Functions
9. Arrays
10. Objects
11. Strings (methods)
12. Scope & Hoisting
13. Closures
14. `this` keyword
15. Prototypes & Prototypal Inheritance
16. Classes (ES6)

**Part 2 — Modern JavaScript (ES6+)**
17. Arrow Functions
18. Template Literals
19. Destructuring
20. Spread & Rest Operators
21. Default Parameters
22. Array Methods (map/filter/reduce/etc.)
23. Modules (import/export)
24. Promises
25. Async/Await
26. Error Handling (try/catch)
27. Map, Set, WeakMap, WeakSet
28. Optional Chaining & Nullish Coalescing
29. Iterators & Generators
30. Event Loop (how JS actually runs)

**Part 3 — TypeScript**
31. Why TypeScript
32. Setup & Compiling
33. Basic Types
34. Type Inference & Type Annotations
35. Arrays & Tuples
36. Objects & Interfaces
37. `type` vs `interface`
38. Union & Intersection Types
39. Literal Types & Enums
40. Functions in TypeScript
41. `any`, `unknown`, `never`, `void`
42. Type Assertions & Narrowing
43. Classes in TypeScript
44. Generics
45. Utility Types
46. Modules & Namespaces
47. `tsconfig.json` explained
48. Working with 3rd-party JS (`.d.ts`)
49. Final Project Idea

---

# PART 1 — JAVASCRIPT FUNDAMENTALS

## 1. What is JavaScript
JavaScript (JS) is a programming language that runs in browsers and on servers (via Node.js). It makes web pages interactive — clicks, forms, animations, API calls, etc.

```html
<script>
  console.log("Hello, JavaScript!");
</script>
```

---

## 2. Variables

Three ways to declare a variable:

```js
var name = "old way";     // function-scoped, avoid using
let age = 25;              // block-scoped, can be reassigned
const PI = 3.14;           // block-scoped, cannot be reassigned
```

- Use `const` by default.
- Use `let` when the value will change.
- Avoid `var` (legacy, causes bugs due to hoisting/scope leaks).

```js
let count = 0;
count = count + 1; // ✅ allowed

const max = 100;
max = 200; // ❌ Error: Assignment to constant variable
```

---

## 3. Data Types

**Primitive types:**
```js
let str  = "hello";       // string
let num  = 42;             // number
let big  = 123n;           // bigint
let bool = true;           // boolean
let empty = null;          // null (intentional "nothing")
let notSet;                 // undefined (not assigned)
let sym  = Symbol("id");   // symbol (unique identifier)
```

**Reference type:**
```js
let obj = { name: "Harsh" };  // object (includes arrays, functions)
```

Check type:
```js
console.log(typeof "hi");   // "string"
console.log(typeof 42);     // "number"
console.log(typeof {});     // "object"
console.log(typeof []);     // "object" (arrays are objects!)
console.log(Array.isArray([])); // true — correct way to check arrays
```

---

## 4. Operators

```js
// Arithmetic
5 + 2   // 7
5 - 2   // 3
5 * 2   // 10
5 / 2   // 2.5
5 % 2   // 1 (remainder)
5 ** 2  // 25 (power)

// Comparison
5 == "5"   // true  (loose equality — converts types, AVOID)
5 === "5"  // false (strict equality — checks type + value, USE THIS)
5 !== "5"  // true

// Logical
true && false // false (AND)
true || false // true  (OR)
!true         // false (NOT)

// Assignment shortcuts
let x = 5;
x += 1; // x = x + 1
x -= 1;
x *= 2;
```

---

## 5. Type Conversion & Coercion

```js
// Explicit conversion
String(123);      // "123"
Number("123");    // 123
Boolean(0);        // false
Boolean("");       // false
Boolean("hi");     // true

// Implicit coercion (JS does it automatically — be careful!)
"5" + 5;    // "55"  (string concatenation)
"5" - 1;    // 4     (numeric subtraction, string converted)
"5" * "2";  // 10
```

---

## 6. Control Flow

```js
let age = 20;

if (age >= 18) {
  console.log("Adult");
} else if (age >= 13) {
  console.log("Teenager");
} else {
  console.log("Child");
}

// switch
let day = "Mon";
switch (day) {
  case "Mon":
    console.log("Start of week");
    break;
  case "Fri":
    console.log("Almost weekend");
    break;
  default:
    console.log("Regular day");
}

// Ternary operator
let status = age >= 18 ? "Adult" : "Minor";
```

---

## 7. Loops

```js
// for loop
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// while loop
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}

// do...while (runs at least once)
let j = 0;
do {
  console.log(j);
  j++;
} while (j < 5);

// for...of — loop over VALUES (arrays, strings)
for (const value of [10, 20, 30]) {
  console.log(value);
}

// for...in — loop over KEYS (objects)
const person = { name: "Harsh", age: 25 };
for (const key in person) {
  console.log(key, person[key]);
}
```

---

## 8. Functions

```js
// Function declaration (hoisted — usable before definition)
function greet(name) {
  return `Hello, ${name}`;
}

// Function expression (NOT hoisted)
const add = function (a, b) {
  return a + b;
};

// Arrow function (see section 17)
const multiply = (a, b) => a * b;

// Default parameters
function greetUser(name = "Guest") {
  return `Hi ${name}`;
}

// Rest parameters (collect extra args into an array)
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}
sum(1, 2, 3); // 6
```

---

## 9. Arrays

```js
const fruits = ["apple", "banana", "mango"];

fruits[0];             // "apple"
fruits.length;          // 3
fruits.push("kiwi");     // add to end
fruits.pop();            // remove from end
fruits.shift();          // remove from start
fruits.unshift("grape"); // add to start
fruits.includes("mango"); // true
fruits.indexOf("banana"); // 1
fruits.slice(0, 2);       // ["grape", "apple"] — copy, doesn't mutate
fruits.splice(1, 1);      // removes 1 item at index 1 — mutates original
```

---

## 10. Objects

```js
const person = {
  name: "Harsh",
  age: 25,
  greet() {
    return `Hi, I'm ${this.name}`;
  }
};

person.name;          // dot notation → "Harsh"
person["age"];         // bracket notation → 25
person.city = "Ahmedabad"; // add new property
delete person.age;         // remove property

Object.keys(person);    // ["name", "greet", "city"]
Object.values(person);  // [values array]
Object.entries(person); // [ [key, value], ... ]
```

---

## 11. Strings

```js
const str = "Hello World";

str.length;              // 11
str.toUpperCase();       // "HELLO WORLD"
str.toLowerCase();       // "hello world"
str.includes("World");   // true
str.slice(0, 5);         // "Hello"
str.split(" ");           // ["Hello", "World"]
str.trim();                // removes whitespace from ends
str.replace("World", "JS"); // "Hello JS"
```

---

## 12. Scope & Hoisting

```js
// Global scope
let globalVar = "I'm global";

function demo() {
  // Function scope
  let localVar = "I'm local";
  console.log(globalVar); // accessible
}
console.log(localVar); // ❌ Error — not accessible outside function

// Block scope (let/const only)
if (true) {
  let blockVar = "only inside this block";
}
console.log(blockVar); // ❌ Error

// Hoisting: var declarations are moved to top (but not initialized)
console.log(hoistedVar); // undefined (not an error!)
var hoistedVar = "value";

console.log(hoistedLet); // ❌ Error: Cannot access before initialization
let hoistedLet = "value";
```

---

## 13. Closures

A closure is a function that "remembers" variables from where it was created, even after that outer function has finished running.

```js
function makeCounter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3
// `count` is private — only accessible via the returned function
```

Practical use: private state, function factories, `setTimeout` callbacks.

---

## 14. `this` Keyword

`this` refers to the object that is calling the function. Its value depends on **how** a function is called, not where it's defined.

```js
const obj = {
  name: "Harsh",
  greet() {
    console.log(this.name); // "Harsh" — obj called it
  }
};
obj.greet();

function standalone() {
  console.log(this); // in strict mode: undefined; browsers: window
}

// Arrow functions do NOT have their own `this` —
// they inherit `this` from where they were defined
const obj2 = {
  name: "Harsh",
  greet: () => {
    console.log(this.name); // undefined — `this` is NOT obj2
  }
};
```

---

## 15. Prototypes & Prototypal Inheritance

Every JS object has a hidden link to another object called its **prototype**, from which it inherits properties/methods.

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} makes a sound`;
};

const dog = new Animal("Dog");
dog.speak(); // "Dog makes a sound" — inherited from prototype

console.log(dog.__proto__ === Animal.prototype); // true
```

---

## 16. Classes (ES6)

Classes are "syntactic sugar" over prototypes — cleaner syntax for OOP.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak() {
    return `${this.name} barks`; // method overriding
  }
}

const d = new Dog("Rex");
console.log(d.speak()); // "Rex barks"

// static method — belongs to the class, not instances
class MathUtils {
  static double(n) {
    return n * 2;
  }
}
MathUtils.double(5); // 10
```

---

# PART 2 — MODERN JAVASCRIPT (ES6+)

## 17. Arrow Functions

```js
// Regular
function add(a, b) { return a + b; }

// Arrow — shorter syntax
const add2 = (a, b) => a + b;

// Single param — parens optional
const square = n => n * n;

// No params
const sayHi = () => "Hi!";

// Multi-line body needs { } and explicit return
const greet = (name) => {
  const msg = `Hello, ${name}`;
  return msg;
};
```

---

## 18. Template Literals

```js
const name = "Harsh";
const age = 25;

// Old way
console.log("My name is " + name + " and I am " + age);

// Template literal — backticks + ${}
console.log(`My name is ${name} and I am ${age}`);

// Multi-line strings
const multiLine = `Line 1
Line 2
Line 3`;
```

---

## 19. Destructuring

```js
// Array destructuring
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// Skip items
const [, second] = [1, 2];

// Object destructuring
const person = { name: "Harsh", age: 25 };
const { name, age } = person;

// Rename while destructuring
const { name: fullName } = person;

// Default values
const { city = "Unknown" } = person;

// Nested destructuring
const { address: { pincode } } = { address: { pincode: 380001 } };

// Function parameter destructuring
function printUser({ name, age }) {
  console.log(`${name} is ${age}`);
}
```

---

## 20. Spread & Rest Operators

Same `...` syntax, opposite jobs.

```js
// SPREAD — expands an array/object into individual items
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1,2,3,4,5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // {a:1, b:2, c:3}

function sum(a, b, c) { return a + b + c; }
sum(...[1, 2, 3]); // 6

// REST — collects multiple items into an array
function sumAll(...nums) {
  return nums.reduce((t, n) => t + n, 0);
}
sumAll(1, 2, 3, 4); // 10

const [first, ...rest] = [1, 2, 3, 4];
console.log(rest); // [2, 3, 4]
```

---

## 21. Default Parameters

```js
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}
greet();               // "Hello, Guest!"
greet("Harsh");         // "Hello, Harsh!"
greet("Harsh", "Hey");  // "Hey, Harsh!"
```

---

## 22. Array Methods (very important)

```js
const nums = [1, 2, 3, 4, 5];

// map — transform each item, returns new array
nums.map(n => n * 2); // [2,4,6,8,10]

// filter — keep items matching condition
nums.filter(n => n % 2 === 0); // [2,4]

// reduce — collapse array into a single value
nums.reduce((total, n) => total + n, 0); // 15

// find — first matching item
nums.find(n => n > 3); // 4

// findIndex — index of first match
nums.findIndex(n => n > 3); // 3

// some — true if ANY item matches
nums.some(n => n > 4); // true

// every — true if ALL items match
nums.every(n => n > 0); // true

// forEach — loop, no return value
nums.forEach(n => console.log(n));

// sort — mutates original array!
[3, 1, 2].sort(); // [1,2,3]
[3, 1, 2].sort((a, b) => b - a); // [3,2,1] descending
```

---

## 23. Modules (import/export)

```js
// mathUtils.js
export function add(a, b) { return a + b; }
export const PI = 3.14;
export default function multiply(a, b) { return a * b; }

// main.js
import multiply, { add, PI } from "./mathUtils.js";
```

Enable modules in browser: `<script type="module" src="main.js"></script>`
In Node.js: set `"type": "module"` in `package.json`.

---

## 24. Promises

A Promise represents a value that will be available **later** (async operation result).

```js
const fetchData = new Promise((resolve, reject) => {
  const success = true;
  setTimeout(() => {
    if (success) resolve("Data loaded!");
    else reject("Error loading data");
  }, 1000);
});

fetchData
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log("Done"));

// Promise states: pending → fulfilled | rejected

// Running multiple promises
Promise.all([fetchData, fetchData]).then(results => console.log(results));
```

---

## 25. Async/Await

Cleaner syntax built on top of Promises.

```js
function getData() {
  return new Promise(resolve => setTimeout(() => resolve("Data!"), 1000));
}

async function main() {
  console.log("Fetching...");
  const result = await getData(); // pauses here until resolved
  console.log(result);
}
main();

// With error handling
async function safeMain() {
  try {
    const result = await getData();
    console.log(result);
  } catch (err) {
    console.error("Failed:", err);
  }
}
```

---

## 26. Error Handling

```js
try {
  JSON.parse("invalid json");
} catch (error) {
  console.error("Error:", error.message);
} finally {
  console.log("This always runs");
}

// Custom errors
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

function validateAge(age) {
  if (age < 0) throw new ValidationError("Age can't be negative");
  return age;
}

try {
  validateAge(-5);
} catch (e) {
  console.log(e.name, e.message); // "ValidationError" "Age can't be negative"
}
```

---

## 27. Map, Set, WeakMap, WeakSet

```js
// Map — key-value pairs, keys can be ANY type
const map = new Map();
map.set("name", "Harsh");
map.set(1, "one");
map.get("name"); // "Harsh"
map.has("name"); // true
map.size;          // 2

// Set — collection of UNIQUE values
const set = new Set([1, 2, 2, 3, 3]);
console.log([...set]); // [1, 2, 3] — duplicates removed
set.add(4);
set.has(2); // true
```

---

## 28. Optional Chaining & Nullish Coalescing

```js
const user = { profile: { name: "Harsh" } };

// Optional chaining — avoid "cannot read property of undefined" errors
console.log(user?.profile?.name);   // "Harsh"
console.log(user?.address?.city);   // undefined (no error!)

// Nullish coalescing — fallback ONLY for null/undefined (not 0 or "")
const count = 0;
console.log(count ?? 10);   // 0   (0 is not null/undefined, kept as-is)
console.log(count || 10);   // 10  (0 is falsy, so || replaces it — different!)
```

---

## 29. Iterators & Generators

```js
// Generator function — can pause and resume execution
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Usable in for...of
for (const num of numberGenerator()) {
  console.log(num); // 1, 2, 3
}
```

---

## 30. Event Loop (how JS actually runs)

JavaScript is **single-threaded** but handles async operations via the event loop:

1. **Call stack** — runs synchronous code line by line.
2. **Web APIs** (browser) / **libuv** (Node) — handle `setTimeout`, network requests, etc. in the background.
3. **Microtask queue** — Promises go here (higher priority).
4. **Macrotask (callback) queue** — `setTimeout`, DOM events go here.
5. Event loop constantly checks: is the call stack empty? → push next queued task.

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");

// Output: 1, 4, 3, 2
// Sync code runs first, then microtasks (Promises), then macrotasks (setTimeout)
```

---

# PART 3 — TYPESCRIPT

## 31. Why TypeScript

TypeScript = JavaScript + static types. It catches bugs **before** you run the code, gives better autocomplete, and makes large codebases easier to maintain. TypeScript compiles ("transpiles") down to plain JavaScript.

---

## 32. Setup & Compiling

```bash
npm install -g typescript
tsc --init          # creates tsconfig.json
tsc app.ts          # compiles app.ts → app.js
tsc --watch          # auto-recompile on save
```

```ts
// app.ts
let message: string = "Hello TS";
console.log(message);
```

---

## 33. Basic Types

```ts
let isDone: boolean = true;
let age: number = 25;
let name: string = "Harsh";
let list: number[] = [1, 2, 3];
let listAlt: Array<number> = [1, 2, 3]; // same thing, generic syntax
let anything: any = "avoid this — disables type checking";
let nothing: null = null;
let notSet: undefined = undefined;
```

---

## 34. Type Inference & Type Annotations

TypeScript can often figure out types automatically — you don't always need to annotate.

```ts
let age = 25;          // inferred as number automatically
// age = "25";          // ❌ Error: Type 'string' is not assignable to type 'number'

let city: string;       // annotation without value
city = "Ahmedabad";
```

Rule of thumb: let inference handle simple cases; annotate function params and return types explicitly.

---

## 35. Arrays & Tuples

```ts
// Array
let scores: number[] = [90, 85, 70];
let names: string[] = ["Harsh", "Amit"];

// Tuple — fixed-length array with specific types per position
let user: [string, number] = ["Harsh", 25];
// user = [25, "Harsh"]; // ❌ Error: wrong order/types

// Readonly array — can't be mutated
const fixed: readonly number[] = [1, 2, 3];
// fixed.push(4); // ❌ Error
```

---

## 36. Objects & Interfaces

```ts
// Inline object type
let person: { name: string; age: number } = { name: "Harsh", age: 25 };

// Interface — reusable named shape for an object
interface Person {
  name: string;
  age: number;
  email?: string;      // optional property (? mark)
  readonly id: number;  // cannot be changed after creation
}

const user: Person = { id: 1, name: "Harsh", age: 25 };
// user.id = 2; // ❌ Error — readonly

// Interfaces can extend other interfaces
interface Employee extends Person {
  salary: number;
}
```

---

## 37. `type` vs `interface`

```ts
// interface — best for object shapes, can be extended/merged
interface Animal {
  name: string;
}
interface Animal { // declaration merging — adds to existing interface
  sound: string;
}

// type — more flexible, can represent unions, primitives, tuples too
type ID = string | number;
type Point = { x: number; y: number };
type Direction = "up" | "down" | "left" | "right";
```

**Rule of thumb:** use `interface` for object/class shapes; use `type` for unions, tuples, or complex compositions.

---

## 38. Union & Intersection Types

```ts
// Union — value can be ONE of several types
function printId(id: string | number) {
  console.log(id);
}
printId(101);
printId("abc-101");

// Intersection — combines multiple types into one
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged; // must have BOTH name and age

const p: Person = { name: "Harsh", age: 25 };
```

---

## 39. Literal Types & Enums

```ts
// Literal types — exact allowed values
let direction: "up" | "down";
direction = "up";   // ✅
// direction = "left"; // ❌ Error

// Enum — named set of constant values
enum Role {
  Admin,     // 0
  Editor,    // 1
  Viewer     // 2
}
let userRole: Role = Role.Admin;

// String enum (more readable when logging/debugging)
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE"
}
```

---

## 40. Functions in TypeScript

```ts
// Typed parameters + return type
function add(a: number, b: number): number {
  return a + b;
}

// Optional parameter
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}`;
}

// Default parameter
function multiply(a: number, b: number = 2): number {
  return a * b;
}

// Arrow function with types
const subtract = (a: number, b: number): number => a - b;

// Function type as a variable
let mathOp: (a: number, b: number) => number;
mathOp = add;

// void — function returns nothing
function logMessage(msg: string): void {
  console.log(msg);
}
```

---

## 41. `any`, `unknown`, `never`, `void`

```ts
// any — disables all type checking (avoid when possible)
let loose: any = "could be anything";
loose = 42; // no error

// unknown — safer alternative to any, forces type-checking before use
let value: unknown = "hello";
// value.toUpperCase(); // ❌ Error — must narrow first
if (typeof value === "string") {
  value.toUpperCase(); // ✅ now safe
}

// never — function never returns (throws or infinite loop)
function throwError(msg: string): never {
  throw new Error(msg);
}

// void — function returns nothing meaningful
function log(msg: string): void {
  console.log(msg);
}
```

---

## 42. Type Assertions & Narrowing

```ts
// Type assertion — "trust me, I know the type" (no runtime conversion!)
let someValue: unknown = "this is a string";
let strLength: number = (someValue as string).length;

// Type narrowing — TS automatically refines type inside conditionals
function printLength(value: string | number) {
  if (typeof value === "string") {
    console.log(value.length);  // TS knows it's a string here
  } else {
    console.log(value.toFixed(2)); // TS knows it's a number here
  }
}
```

---

## 43. Classes in TypeScript

```ts
class Animal {
  // access modifiers control visibility
  public name: string;       // accessible everywhere (default)
  private secret: string = "hidden";  // only inside this class
  protected age: number = 0;  // accessible in this class + subclasses
  readonly species: string = "unknown"; // can't be reassigned

  constructor(name: string) {
    this.name = name;
  }

  speak(): string {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak(): string {
    return `${this.name} barks`; // can access protected `age`
    // console.log(this.secret); // ❌ Error — private, not accessible
  }
}

// Shorthand constructor properties
class Point {
  constructor(public x: number, public y: number) {}
}
const p = new Point(10, 20); // x and y auto-assigned

// Implementing an interface
interface Shape {
  area(): number;
}
class Circle implements Shape {
  constructor(private radius: number) {}
  area(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

---

## 44. Generics

Generics let you write reusable code that works with multiple types while still being type-safe.

```ts
// Without generics — loses type info
function identity(value: any): any {
  return value;
}

// With generics — preserves type info
function identity2<T>(value: T): T {
  return value;
}
identity2<string>("hello");  // returns string
identity2<number>(42);        // returns number
identity2(true);                // T inferred automatically as boolean

// Generic interface
interface Box<T> {
  content: T;
}
const stringBox: Box<string> = { content: "hello" };
const numberBox: Box<number> = { content: 42 };

// Generic function with array
function getFirst<T>(arr: T[]): T {
  return arr[0];
}
getFirst<number>([1, 2, 3]); // 1

// Generic class
class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}
const numberStack = new Stack<number>();
numberStack.push(1);

// Constrained generics — T must have a `.length` property
function logLength<T extends { length: number }>(item: T): void {
  console.log(item.length);
}
logLength("hello"); // works — strings have .length
logLength([1, 2, 3]); // works — arrays have .length
```

---

## 45. Utility Types

Built-in helper types that transform existing types.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

// Partial<T> — makes all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string }

// Required<T> — makes all properties required
type RequiredUser = Required<PartialUser>;

// Pick<T, keys> — select specific properties
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit<T, keys> — remove specific properties
type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string }

// Readonly<T> — makes all properties readonly
type ReadonlyUser = Readonly<User>;

// Record<keys, valueType> — build an object type with specific keys/values
type Roles = "admin" | "editor" | "viewer";
type RolePermissions = Record<Roles, string[]>;
// { admin: string[]; editor: string[]; viewer: string[] }
```

---

## 46. Modules & Namespaces

```ts
// mathUtils.ts
export interface Point { x: number; y: number; }
export function add(a: number, b: number): number { return a + b; }
export default class Calculator {}

// main.ts
import Calculator, { add, Point } from "./mathUtils";
```

Same `import`/`export` syntax as JS — TypeScript just adds type-checking on top.

---

## 47. `tsconfig.json` explained

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",       // JS version to compile down to
    "module": "commonjs",      // module system (commonjs / esnext)
    "strict": true,             // enables all strict type-checking (recommended!)
    "outDir": "./dist",         // compiled JS output folder
    "rootDir": "./src",         // TS source folder
    "esModuleInterop": true,    // fixes import compatibility issues
    "skipLibCheck": true,        // skip type-checking node_modules (faster builds)
    "noImplicitAny": true        // error if a type can't be inferred (part of strict)
  },
  "include": ["src/**/*"]
}
```

---

## 48. Working with 3rd-party JS (`.d.ts`)

Many JS libraries don't have built-in types. Type declaration files (`.d.ts`) describe their shape to TypeScript.

```bash
# Install community-maintained type definitions
npm install --save-dev @types/lodash
```

```ts
// Writing your own .d.ts for a JS-only library
declare module "some-untyped-library" {
  export function doSomething(input: string): number;
}
```

---

## 49. Final Project Idea

Build a small **Task Manager** to combine everything:

```ts
interface Task {
  id: number;
  title: string;
  completed: boolean;
}

class TaskManager {
  private tasks: Task[] = [];
  private nextId = 1;

  addTask(title: string): Task {
    const task: Task = { id: this.nextId++, title, completed: false };
    this.tasks.push(task);
    return task;
  }

  completeTask(id: number): void {
    const task = this.tasks.find(t => t.id === id);
    if (task) task.completed = true;
  }

  getPending(): Task[] {
    return this.tasks.filter(t => !t.completed);
  }
}

const manager = new TaskManager();
manager.addTask("Learn JavaScript");
manager.addTask("Learn TypeScript");
manager.completeTask(1);
console.log(manager.getPending()); // [{ id: 2, title: "Learn TypeScript", completed: false }]
```

This project touches: classes, interfaces, generics-friendly array methods, access modifiers, and type-safe functions — the full JS → TS journey in one file.

---

## Suggested Learning Order
1. Sections 1–16 (JS fundamentals) — practice each in browser console or Node.
2. Sections 17–30 (Modern JS) — rewrite old code using these features.
3. Sections 31–49 (TypeScript) — convert a small JS project of yours into `.ts`.

Good luck — build small projects after every 3–4 sections instead of just reading.
