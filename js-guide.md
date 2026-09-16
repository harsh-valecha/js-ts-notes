# JavaScript Basics: End-to-End Guide

JavaScript is a programming language used to make web pages interactive. It can run in browsers, servers, mobile apps, desktop apps, and many other environments.

This guide covers JavaScript fundamentals from beginner to intermediate level with small examples.

## 1. What Is JavaScript?

JavaScript can:

- Change HTML and CSS
- Respond to user actions
- Validate forms
- Fetch data from APIs
- Build web applications
- Run backend servers using Node.js
- Create games, mobile apps, and desktop apps

Example:

```javascript
console.log("Hello, JavaScript!");
```

`console.log()` prints a value to the browser console or terminal.

## 2. How to Run JavaScript

### Inside an HTML File

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Hello</h1>

    <script>
      console.log("JavaScript is running");
    </script>
  </body>
</html>
```

### External JavaScript File

Create `script.js`:

```javascript
console.log("External JavaScript file");
```

Connect it to HTML:

```html
<script src="script.js"></script>
```

It is usually best to place the script before the closing `body` tag:

```html
<body>
  <h1>My Page</h1>

  <script src="script.js"></script>
</body>
```

You can also use `defer`:

```html
<head>
  <script src="script.js" defer></script>
</head>
```

`defer` waits until the HTML has been parsed before running the script.

## 3. Comments

Comments are ignored by JavaScript.

### Single-Line Comments

```javascript
// This is a comment
console.log("Hello");
```

### Multi-Line Comments

```javascript
/*
  This is a
  multi-line comment
*/
console.log("Hello");
```

## 4. Statements

A statement is an instruction.

```javascript
let name = "Alex";
console.log(name);
```

Semicolons are optional, but using them consistently is recommended.

```javascript
let age = 25;
```

## 5. Variables

Variables store data.

### `let`

Use `let` when the value may change.

```javascript
let score = 10;
score = 20;

console.log(score);
```

### `const`

Use `const` when the variable should not be reassigned.

```javascript
const birthYear = 2000;

// This causes an error:
// birthYear = 2001;
```

### `var`

`var` is the older way to declare variables.

```javascript
var city = "London";
```

Modern JavaScript generally prefers `let` and `const`.

### Variable Naming Rules

Valid names:

```javascript
let firstName = "Sam";
let _count = 10;
let $price = 20;
```

Invalid names:

```javascript
// let 1name = "Sam";
// let first-name = "Sam";
```

Rules:

- Names cannot start with a number.
- Names can contain letters, numbers, `_`, and `$`.
- Names are case-sensitive.
- Do not use reserved keywords.

```javascript
let age = 20;
let Age = 30;

console.log(age); // 20
console.log(Age); // 30
```

## 6. Data Types

JavaScript has primitive and non-primitive data types.

### Primitive Types

- String
- Number
- BigInt
- Boolean
- Undefined
- Null
- Symbol

### Non-Primitive Type

- Object

## 7. Strings

Strings represent text.

```javascript
let firstName = "Alex";
let message = 'Hello';
```

You can use single quotes, double quotes, or backticks.

```javascript
let a = "Hello";
let b = 'Hello';
let c = `Hello`;
```

### String Length

```javascript
let word = "JavaScript";

console.log(word.length); // 10
```

### Accessing Characters

```javascript
let word = "Hello";

console.log(word[0]); // H
console.log(word[1]); // e
```

### Common String Methods

```javascript
let text = " JavaScript ";
```

```javascript
console.log(text.toUpperCase()); // " JAVASCRIPT "
console.log(text.toLowerCase()); // " javascript "
console.log(text.trim());        // "JavaScript"
console.log(text.includes("Script")); // true
```

### `startsWith()` and `endsWith()`

```javascript
let fileName = "photo.png";

console.log(fileName.startsWith("photo")); // true
console.log(fileName.endsWith(".png"));    // true
```

### `indexOf()`

```javascript
let text = "Hello world";

console.log(text.indexOf("world")); // 6
console.log(text.indexOf("JavaScript")); // -1
```

### `slice()`

```javascript
let text = "JavaScript";

console.log(text.slice(0, 4)); // Java
console.log(text.slice(4));    // Script
```

### `replace()`

```javascript
let text = "I like cats";

let result = text.replace("cats", "dogs");

console.log(result); // I like dogs
```

### Splitting a String

```javascript
let fruits = "apple,banana,orange";

let result = fruits.split(",");

console.log(result);
// ["apple", "banana", "orange"]
```

### Template Literals

Template literals use backticks.

```javascript
let name = "Alex";
let age = 25;

let message = `My name is ${name} and I am ${age} years old.`;

console.log(message);
```

## 8. Numbers

JavaScript uses the `number` type for integers and decimal values.

```javascript
let age = 25;
let price = 19.99;
let temperature = -5;
```

### Arithmetic Operators

```javascript
let a = 10;
let b = 3;

console.log(a + b); // 13
console.log(a - b); // 7
console.log(a * b); // 30
console.log(a / b); // 3.333...
console.log(a % b); // 1
console.log(a ** b); // 1000
```

### Increment and Decrement

```javascript
let count = 5;

count++;
console.log(count); // 6

count--;
console.log(count); // 5
```

### Assignment Operators

```javascript
let number = 10;

number += 5; // 15
number -= 3; // 12
number *= 2; // 24
number /= 4; // 6
```

### Number Methods

```javascript
let number = 12.3456;

console.log(number.toFixed(2)); // "12.35"
console.log(Number.isInteger(number)); // false
```

### Converting Strings to Numbers

```javascript
let value = "42";

console.log(Number(value));      // 42
console.log(parseInt(value));    // 42
console.log(parseFloat("3.14")); // 3.14
```

### `NaN`

`NaN` means “Not a Number”.

```javascript
let result = Number("hello");

console.log(result); // NaN
console.log(Number.isNaN(result)); // true
```

### Infinity

```javascript
console.log(10 / 0); // Infinity
```

## 9. BigInt

`BigInt` is used for very large integers.

```javascript
let largeNumber = 123456789012345678901234567890n;

console.log(largeNumber);
```

You cannot directly mix `BigInt` and regular numbers.

```javascript
let a = 10n;
let b = 5n;

console.log(a + b); // 15n
```

## 10. Boolean Values

A Boolean is either `true` or `false`.

```javascript
let isLoggedIn = true;
let hasPermission = false;
```

Booleans are often used in conditions.

```javascript
let age = 20;

console.log(age >= 18); // true
```

## 11. Undefined

A variable has the value `undefined` when it has been declared but not assigned a value.

```javascript
let username;

console.log(username); // undefined
```

A function that does not return a value also returns `undefined`.

```javascript
function sayHello() {
  console.log("Hello");
}

let result = sayHello();

console.log(result); // undefined
```

## 12. Null

`null` represents an intentional empty value.

```javascript
let selectedUser = null;
```

`undefined` usually means a value has not been assigned, while `null` usually means the value was intentionally cleared.

## 13. Symbols

Symbols create unique identifiers.

```javascript
let id = Symbol("id");

console.log(id);
```

Two symbols with the same description are still different:

```javascript
let a = Symbol("id");
let b = Symbol("id");

console.log(a === b); // false
```

## 14. Checking Data Types

Use `typeof`.

```javascript
console.log(typeof "Hello");  // string
console.log(typeof 42);       // number
console.log(typeof true);     // boolean
console.log(typeof undefined); // undefined
console.log(typeof null);     // object
```

`typeof null` returns `"object"` due to an old JavaScript behavior.

For arrays, use:

```javascript
let numbers = [1, 2, 3];

console.log(Array.isArray(numbers)); // true
```

## 15. Type Conversion

### String Conversion

```javascript
let number = 100;

let text = String(number);

console.log(text);        // "100"
console.log(typeof text); // string
```

### Number Conversion

```javascript
let text = "25";

let number = Number(text);

console.log(number); // 25
```

### Boolean Conversion

```javascript
console.log(Boolean(1));       // true
console.log(Boolean(0));       // false
console.log(Boolean("Hello")); // true
console.log(Boolean(""));      // false
```

## 16. Truthy and Falsy Values

Falsy values include:

```javascript
false
0
-0
0n
""
null
undefined
NaN
```

Almost everything else is truthy.

```javascript
if ("Hello") {
  console.log("This runs");
}
```

## 17. Operators

### Comparison Operators

```javascript
console.log(5 == "5");  // true
console.log(5 === "5"); // false
console.log(5 != "5");  // false
console.log(5 !== "5"); // true
console.log(5 > 3);     // true
console.log(5 < 3);     // false
console.log(5 >= 5);    // true
console.log(4 <= 5);    // true
```

Prefer strict equality:

```javascript
5 === 5;
5 !== 4;
```

Avoid relying on loose equality:

```javascript
5 == "5";
```

### Logical Operators

#### AND: `&&`

Both conditions must be true.

```javascript
let age = 25;
let hasId = true;

console.log(age >= 18 && hasId); // true
```

#### OR: `||`

At least one condition must be true.

```javascript
let isWeekend = false;
let isHoliday = true;

console.log(isWeekend || isHoliday); // true
```

#### NOT: `!`

Reverses a Boolean.

```javascript
let isOnline = true;

console.log(!isOnline); // false
```

### Nullish Coalescing Operator

`??` uses a fallback only when the value is `null` or `undefined`.

```javascript
let username = null;

let displayName = username ?? "Guest";

console.log(displayName); // Guest
```

Unlike `||`, it does not treat `0` or `""` as missing.

```javascript
let count = 0;

console.log(count || 10); // 10
console.log(count ?? 10); // 0
```

### Optional Chaining

Optional chaining prevents errors when accessing missing properties.

```javascript
let user = {};

console.log(user.profile?.email); // undefined
```

Without optional chaining, `user.profile.email` would cause an error.

## 18. Operator Precedence

JavaScript follows an order of operations.

```javascript
let result = 2 + 3 * 4;

console.log(result); // 14
```

Use parentheses to make the order clear:

```javascript
let result = (2 + 3) * 4;

console.log(result); // 20
```

## 19. Conditional Statements

### `if`

```javascript
let age = 20;

if (age >= 18) {
  console.log("Adult");
}
```

### `if...else`

```javascript
let age = 16;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

### `else if`

```javascript
let score = 75;

if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("Needs improvement");
}
```

### Ternary Operator

Use the ternary operator for short conditions.

```javascript
let age = 20;

let status = age >= 18 ? "Adult" : "Minor";

console.log(status);
```

### Nested Conditions

```javascript
let age = 25;
let hasLicense = true;

if (age >= 18) {
  if (hasLicense) {
    console.log("Can drive");
  }
}
```

A simpler version:

```javascript
if (age >= 18 && hasLicense) {
  console.log("Can drive");
}
```

## 20. `switch`

Use `switch` when checking one value against multiple options.

```javascript
let day = "Monday";

switch (day) {
  case "Monday":
    console.log("Start of the week");
    break;

  case "Friday":
    console.log("Almost weekend");
    break;

  default:
    console.log("Regular day");
}
```

Without `break`, execution continues into the next case.

## 21. Loops

Loops repeat code.

### `for` Loop

```javascript
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
```

Output:

```text
1
2
3
4
5
```

### `while` Loop

```javascript
let count = 1;

while (count <= 5) {
  console.log(count);
  count++;
}
```

### `do...while` Loop

A `do...while` loop runs at least once.

```javascript
let number = 10;

do {
  console.log(number);
  number++;
} while (number < 5);
```

### `break`

Stops a loop.

```javascript
for (let i = 1; i <= 10; i++) {
  if (i === 5) {
    break;
  }

  console.log(i);
}
```

### `continue`

Skips the current iteration.

```javascript
for (let i = 1; i <= 5; i++) {
  if (i === 3) {
    continue;
  }

  console.log(i);
}
```

## 22. Functions

Functions are reusable blocks of code.

```javascript
function greet() {
  console.log("Hello");
}

greet();
```

### Parameters

```javascript
function greet(name) {
  console.log(`Hello, ${name}`);
}

greet("Alex");
```

### Return Values

```javascript
function add(a, b) {
  return a + b;
}

let result = add(2, 3);

console.log(result); // 5
```

After `return`, the function stops running.

```javascript
function test() {
  return "Done";

  console.log("This never runs");
}
```

### Default Parameters

```javascript
function greet(name = "Guest") {
  console.log(`Hello, ${name}`);
}

greet(); // Hello, Guest
```

### Rest Parameters

Rest parameters collect multiple arguments into an array.

```javascript
function addAll(...numbers) {
  let total = 0;

  for (let number of numbers) {
    total += number;
  }

  return total;
}

console.log(addAll(1, 2, 3, 4)); // 10
```

### Function Expressions

```javascript
const greet = function () {
  console.log("Hello");
};

greet();
```

### Arrow Functions

```javascript
const greet = () => {
  console.log("Hello");
};

greet();
```

With one parameter:

```javascript
const square = number => {
  return number * number;
};
```

Short form:

```javascript
const square = number => number * number;

console.log(square(4)); // 16
```

### Callback Functions

A callback is a function passed to another function.

```javascript
function processUser(name, callback) {
  callback(name);
}

processUser("Alex", function (name) {
  console.log(`Hello, ${name}`);
});
```

## 23. Scope

Scope controls where variables can be accessed.

### Global Scope

```javascript
let appName = "My App";

function showName() {
  console.log(appName);
}

showName();
```

### Function Scope

```javascript
function test() {
  let message = "Hello";

  console.log(message);
}

test();

// This causes an error:
// console.log(message);
```

### Block Scope

`let` and `const` are block-scoped.

```javascript
if (true) {
  let message = "Inside block";
  console.log(message);
}

// message is not available here
```

### Lexical Scope

An inner function can access variables from its outer function.

```javascript
function outer() {
  let message = "Hello";

  function inner() {
    console.log(message);
  }

  inner();
}

outer();
```

## 24. Closures

A closure occurs when a function remembers variables from its surrounding scope.

```javascript
function createCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

The inner function still has access to `count`.

## 25. Hoisting

JavaScript moves some declarations to the top of their scope during execution.

Function declarations can be called before they appear in the code:

```javascript
sayHello();

function sayHello() {
  console.log("Hello");
}
```

Variables declared with `let` and `const` cannot be used before declaration:

```javascript
// console.log(name); // Error

let name = "Alex";
```

## 26. Arrays

Arrays store multiple values.

```javascript
let fruits = ["Apple", "Banana", "Orange"];

console.log(fruits[0]); // Apple
```

Array indexes start at `0`.

### Changing Array Values

```javascript
let fruits = ["Apple", "Banana"];

fruits[1] = "Orange";

console.log(fruits);
```

### Array Length

```javascript
let numbers = [10, 20, 30];

console.log(numbers.length); // 3
```

### Add to the End

```javascript
let fruits = ["Apple"];

fruits.push("Banana");

console.log(fruits);
```

### Remove from the End

```javascript
let fruits = ["Apple", "Banana"];

let removed = fruits.pop();

console.log(removed); // Banana
```

### Add to the Beginning

```javascript
let fruits = ["Banana"];

fruits.unshift("Apple");

console.log(fruits);
```

### Remove from the Beginning

```javascript
let fruits = ["Apple", "Banana"];

fruits.shift();

console.log(fruits);
```

### `slice()`

Returns part of an array without changing the original.

```javascript
let numbers = [1, 2, 3, 4, 5];

let result = numbers.slice(1, 4);

console.log(result); // [2, 3, 4]
```

### `splice()`

Changes the original array.

```javascript
let fruits = ["Apple", "Banana", "Orange"];

fruits.splice(1, 1);

console.log(fruits); // ["Apple", "Orange"]
```

### `includes()`

```javascript
let fruits = ["Apple", "Banana"];

console.log(fruits.includes("Apple")); // true
```

### `indexOf()`

```javascript
let fruits = ["Apple", "Banana"];

console.log(fruits.indexOf("Banana")); // 1
```

### `join()`

```javascript
let words = ["Hello", "world"];

console.log(words.join(" ")); // Hello world
```

### `reverse()`

```javascript
let numbers = [1, 2, 3];

numbers.reverse();

console.log(numbers); // [3, 2, 1]
```

### `sort()`

Strings are sorted alphabetically by default.

```javascript
let fruits = ["Orange", "Apple", "Banana"];

fruits.sort();

console.log(fruits);
```

For numbers, provide a comparison function:

```javascript
let numbers = [10, 2, 30, 4];

numbers.sort((a, b) => a - b);

console.log(numbers); // [2, 4, 10, 30]
```

## 27. Array Iteration Methods

### `forEach()`

Runs a function for every item.

```javascript
let numbers = [1, 2, 3];

numbers.forEach(function (number) {
  console.log(number);
});
```

Arrow function version:

```javascript
numbers.forEach(number => console.log(number));
```

### `map()`

Creates a new array by transforming each item.

```javascript
let numbers = [1, 2, 3];

let doubled = numbers.map(number => number * 2);

console.log(doubled); // [2, 4, 6]
```

### `filter()`

Creates a new array containing matching items.

```javascript
let numbers = [1, 2, 3, 4, 5];

let evenNumbers = numbers.filter(number => number % 2 === 0);

console.log(evenNumbers); // [2, 4]
```

### `find()`

Returns the first matching item.

```javascript
let users = [
  { id: 1, name: "Alex" },
  { id: 2, name: "Sam" }
];

let user = users.find(user => user.id === 2);

console.log(user);
```

### `findIndex()`

Returns the index of the first matching item.

```javascript
let numbers = [10, 20, 30];

let index = numbers.findIndex(number => number === 20);

console.log(index); // 1
```

### `some()`

Returns `true` if at least one item matches.

```javascript
let numbers = [1, 3, 5, 8];

console.log(numbers.some(number => number % 2 === 0)); // true
```

### `every()`

Returns `true` if all items match.

```javascript
let numbers = [2, 4, 6];

console.log(numbers.every(number => number % 2 === 0)); // true
```

### `reduce()`

Combines array values into one result.

```javascript
let numbers = [1, 2, 3, 4];

let total = numbers.reduce((sum, number) => {
  return sum + number;
}, 0);

console.log(total); // 10
```

### Chaining Methods

```javascript
let numbers = [1, 2, 3, 4, 5, 6];

let result = numbers
  .filter(number => number % 2 === 0)
  .map(number => number * 10);

console.log(result); // [20, 40, 60]
```

## 28. Objects

Objects store data using key-value pairs.

```javascript
let user = {
  name: "Alex",
  age: 25,
  isActive: true
};

console.log(user.name);
console.log(user.age);
```

### Bracket Notation

```javascript
console.log(user["name"]);
```

Bracket notation is useful when the property name is stored in a variable:

```javascript
let property = "age";

console.log(user[property]);
```

### Changing Properties

```javascript
user.age = 26;
```

### Adding Properties

```javascript
user.email = "alex@example.com";
```

### Deleting Properties

```javascript
delete user.isActive;
```

### Methods

An object can contain functions.

```javascript
let person = {
  name: "Alex",

  greet() {
    console.log(`Hello, I am ${this.name}`);
  }
};

person.greet();
```

### Nested Objects

```javascript
let user = {
  name: "Alex",
  address: {
    city: "London",
    country: "UK"
  }
};

console.log(user.address.city);
```

### Objects in Arrays

```javascript
let products = [
  { name: "Laptop", price: 900 },
  { name: "Phone", price: 500 }
];

console.log(products[0].name);
```

## 29. Object Methods

### `Object.keys()`

```javascript
let user = {
  name: "Alex",
  age: 25
};

console.log(Object.keys(user));
// ["name", "age"]
```

### `Object.values()`

```javascript
console.log(Object.values(user));
// ["Alex", 25]
```

### `Object.entries()`

```javascript
console.log(Object.entries(user));
// [["name", "Alex"], ["age", 25]]
```

### `Object.assign()`

```javascript
let defaults = {
  color: "blue",
  size: "medium"
};

let settings = Object.assign({}, defaults, {
  size: "large"
});

console.log(settings);
```

### Spread Syntax

```javascript
let user = {
  name: "Alex",
  age: 25
};

let updatedUser = {
  ...user,
  age: 26
};

console.log(updatedUser);
```

## 30. Destructuring

Destructuring extracts values from arrays or objects.

### Object Destructuring

```javascript
let user = {
  name: "Alex",
  age: 25
};

let { name, age } = user;

console.log(name);
console.log(age);
```

### Renaming Variables

```javascript
let user = {
  name: "Alex"
};

let { name: userName } = user;

console.log(userName);
```

### Default Values

```javascript
let user = {
  name: "Alex"
};

let { age = 18 } = user;

console.log(age); // 18
```

### Array Destructuring

```javascript
let colors = ["red", "green", "blue"];

let [first, second] = colors;

console.log(first);  // red
console.log(second); // green
```

### Skipping Values

```javascript
let numbers = [1, 2, 3];

let [, , third] = numbers;

console.log(third); // 3
```

## 31. Spread Syntax

Spread expands arrays or objects.

### Arrays

```javascript
let first = [1, 2];
let second = [3, 4];

let combined = [...first, ...second];

console.log(combined); // [1, 2, 3, 4]
```

### Copying Arrays

```javascript
let original = [1, 2, 3];
let copy = [...original];
```

### Function Arguments

```javascript
let numbers = [10, 20, 30];

console.log(Math.max(...numbers)); // 30
```

## 32. Rest Syntax

Rest collects remaining values.

```javascript
let [first, ...others] = [1, 2, 3, 4];

console.log(first);  // 1
console.log(others); // [2, 3, 4]
```

Object rest:

```javascript
let user = {
  name: "Alex",
  age: 25,
  city: "London"
};

let { name, ...details } = user;

console.log(details);
```

## 33. The `this` Keyword

`this` refers to the object that is calling a method.

```javascript
let user = {
  name: "Alex",

  sayName() {
    console.log(this.name);
  }
};

user.sayName(); // Alex
```

In arrow functions, `this` is inherited from the surrounding scope.

```javascript
let user = {
  name: "Alex",

  sayName: () => {
    console.log(this.name);
  }
};
```

Arrow functions should usually not be used as object methods when you need `this`.

## 34. Constructor Functions

Constructor functions create multiple similar objects.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

let person1 = new Person("Alex", 25);
let person2 = new Person("Sam", 30);

console.log(person1.name);
```

## 35. Classes

Classes provide a cleaner syntax for creating objects.

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hello, I am ${this.name}`);
  }
}

let person = new Person("Alex", 25);

person.greet();
```

### Class Inheritance

```javascript
class Animal {
  speak() {
    console.log("Animal sound");
  }
}

class Dog extends Animal {
  speak() {
    console.log("Woof");
  }
}

let dog = new Dog();

dog.speak();
```

### `super`

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
}

let dog = new Dog("Buddy", "Labrador");
```

## 36. Prototypes

Objects can inherit properties and methods from prototypes.

```javascript
let person = {
  greet() {
    console.log("Hello");
  }
};

let user = Object.create(person);

user.greet();
```

Classes use prototypes internally.

## 37. Date Objects

Create a date:

```javascript
let now = new Date();

console.log(now);
```

### Getting Date Parts

```javascript
let date = new Date();

console.log(date.getFullYear());
console.log(date.getMonth()); // 0 means January
console.log(date.getDate());
console.log(date.getDay());
```

### Creating a Specific Date

```javascript
let birthday = new Date("2000-05-15");

console.log(birthday);
```

### Formatting Dates

```javascript
let date = new Date();

console.log(date.toISOString());
console.log(date.toDateString());
```

## 38. Math Object

```javascript
console.log(Math.PI);
console.log(Math.round(4.6)); // 5
console.log(Math.floor(4.9)); // 4
console.log(Math.ceil(4.1));  // 5
console.log(Math.abs(-10));   // 10
console.log(Math.max(1, 5, 3)); // 5
console.log(Math.min(1, 5, 3)); // 1
```

### Random Numbers

```javascript
let random = Math.random();

console.log(random); // Between 0 and 1
```

Random integer from 1 to 10:

```javascript
let number = Math.floor(Math.random() * 10) + 1;

console.log(number);
```

## 39. JSON

JSON is a text format commonly used for exchanging data.

### JavaScript Object to JSON

```javascript
let user = {
  name: "Alex",
  age: 25
};

let json = JSON.stringify(user);

console.log(json);
```

### JSON to JavaScript Object

```javascript
let json = '{"name":"Alex","age":25}';

let user = JSON.parse(json);

console.log(user.name);
```

## 40. Sets

A `Set` stores unique values.

```javascript
let numbers = new Set();

numbers.add(1);
numbers.add(2);
numbers.add(2);

console.log(numbers); // Set {1, 2}
```

### Set Methods

```javascript
let colors = new Set(["red", "green", "blue"]);

console.log(colors.has("red")); // true
console.log(colors.size);       // 3

colors.delete("green");
colors.clear();
```

### Removing Duplicate Array Values

```javascript
let numbers = [1, 2, 2, 3, 3];

let uniqueNumbers = [...new Set(numbers)];

console.log(uniqueNumbers); // [1, 2, 3]
```

## 41. Maps

A `Map` stores key-value pairs.

```javascript
let userRoles = new Map();

userRoles.set("Alex", "Admin");
userRoles.set("Sam", "Editor");

console.log(userRoles.get("Alex")); // Admin
```

### Map Methods

```javascript
console.log(userRoles.has("Alex")); // true
console.log(userRoles.size);        // 2

userRoles.delete("Sam");
userRoles.clear();
```

Maps can use objects as keys:

```javascript
let user = {};
let map = new Map();

map.set(user, "User data");

console.log(map.get(user));
```

## 42. Error Handling

Errors can stop a program. Use `try...catch` to handle them.

```javascript
try {
  let result = unknownFunction();
} catch (error) {
  console.log("Something went wrong");
}
```

### Accessing the Error

```javascript
try {
  throw new Error("Custom error");
} catch (error) {
  console.log(error.message);
}
```

### `finally`

`finally` always runs.

```javascript
try {
  console.log("Trying");
} catch (error) {
  console.log("Error");
} finally {
  console.log("Finished");
}
```

### Throwing Errors

```javascript
function divide(a, b) {
  if (b === 0) {
    throw new Error("Cannot divide by zero");
  }

  return a / b;
}
```

## 43. Strict Mode

Strict mode catches certain common mistakes.

```javascript
"use strict";

x = 10; // Error because x was not declared
```

You can enable it for a function:

```javascript
function test() {
  "use strict";

  // Strict mode applies here
}
```

JavaScript modules automatically use strict mode.

## 44. The DOM

The DOM, or Document Object Model, represents an HTML document as objects.

Example HTML:

```html
<h1 id="title">Old Title</h1>
```

Select the element:

```javascript
let title = document.getElementById("title");

console.log(title);
```

### Selecting Elements

```javascript
document.getElementById("title");
document.querySelector(".item");
document.querySelector("#title");
document.querySelectorAll(".item");
```

### Changing Text

```javascript
let title = document.querySelector("h1");

title.textContent = "New Title";
```

### Changing HTML

```javascript
let container = document.querySelector("#container");

container.innerHTML = "<p>Hello</p>";
```

Avoid inserting untrusted user input with `innerHTML`. Prefer `textContent` when possible.

### Changing Styles

```javascript
let title = document.querySelector("h1");

title.style.color = "blue";
title.style.fontSize = "30px";
```

### Changing Classes

```javascript
let box = document.querySelector(".box");

box.classList.add("active");
box.classList.remove("hidden");
box.classList.toggle("selected");

console.log(box.classList.contains("active"));
```

### Attributes

```javascript
let image = document.querySelector("img");

image.setAttribute("alt", "A landscape");
image.setAttribute("src", "photo.jpg");

console.log(image.getAttribute("src"));
```

## 45. Creating and Removing Elements

### Creating an Element

```javascript
let paragraph = document.createElement("p");

paragraph.textContent = "New paragraph";

document.body.appendChild(paragraph);
```

### Adding an Element

```javascript
let list = document.querySelector("ul");
let item = document.createElement("li");

item.textContent = "New item";

list.append(item);
```

### Removing an Element

```javascript
let item = document.querySelector("li");

item.remove();
```

## 46. Events

Events happen when users interact with a page.

### Inline Event

```html
<button onclick="sayHello()">Click me</button>

<script>
  function sayHello() {
    alert("Hello!");
  }
</script>
```

### `addEventListener()`

```html
<button id="button">Click me</button>
```

```javascript
let button = document.querySelector("#button");

button.addEventListener("click", function () {
  console.log("Button clicked");
});
```

### Event Object

```javascript
button.addEventListener("click", function (event) {
  console.log(event);
});
```

### Common Events

```javascript
button.addEventListener("click", handler);
input.addEventListener("input", handler);
form.addEventListener("submit", handler);
window.addEventListener("load", handler);
document.addEventListener("keydown", handler);
```

### Keyboard Events

```javascript
document.addEventListener("keydown", function (event) {
  console.log(event.key);
});
```

### Preventing Default Behavior

```javascript
let link = document.querySelector("a");

link.addEventListener("click", function (event) {
  event.preventDefault();

  console.log("Link action stopped");
});
```

## 47. Forms

HTML:

```html
<form id="signupForm">
  <input id="email" type="email" />
  <button type="submit">Submit</button>
</form>
```

JavaScript:

```javascript
let form = document.querySelector("#signupForm");
let email = document.querySelector("#email");

form.addEventListener("submit", function (event) {
  event.preventDefault();

  console.log(email.value);
});
```

### Basic Validation

```javascript
form.addEventListener("submit", function (event) {
  event.preventDefault();

  if (email.value.trim() === "") {
    console.log("Email is required");
    return;
  }

  console.log("Form submitted");
});
```

## 48. Event Bubbling

Events usually move from the target element upward through its parent elements.

```html
<div id="parent">
  <button id="child">Click</button>
</div>
```

```javascript
let parent = document.querySelector("#parent");
let child = document.querySelector("#child");

parent.addEventListener("click", () => {
  console.log("Parent clicked");
});

child.addEventListener("click", () => {
  console.log("Child clicked");
});
```

Clicking the button triggers both handlers.

### Stopping Propagation

```javascript
child.addEventListener("click", function (event) {
  event.stopPropagation();

  console.log("Only child handler runs");
});
```

## 49. Event Delegation

Event delegation uses one parent listener to handle child events.

```javascript
let list = document.querySelector("ul");

list.addEventListener("click", function (event) {
  if (event.target.tagName === "LI") {
    console.log(event.target.textContent);
  }
});
```

This is useful for dynamically created elements.

## 50. Timers

### `setTimeout()`

Runs code once after a delay.

```javascript
setTimeout(() => {
  console.log("Runs after two seconds");
}, 2000);
```

### `setInterval()`

Runs code repeatedly.

```javascript
let count = 0;

let intervalId = setInterval(() => {
  count++;
  console.log(count);

  if (count === 3) {
    clearInterval(intervalId);
  }
}, 1000);
```

### Clearing a Timeout

```javascript
let timeoutId = setTimeout(() => {
  console.log("This will not run");
}, 3000);

clearTimeout(timeoutId);
```

## 51. Synchronous and Asynchronous JavaScript

Synchronous code runs line by line.

```javascript
console.log("One");
console.log("Two");
console.log("Three");
```

Asynchronous code can finish later.

```javascript
console.log("One");

setTimeout(() => {
  console.log("Two");
}, 1000);

console.log("Three");
```

Output:

```text
One
Three
Two
```

## 52. Promises

A Promise represents a future result.

```javascript
let promise = new Promise((resolve, reject) => {
  resolve("Success");
});
```

### Consuming a Promise

```javascript
promise
  .then(result => {
    console.log(result);
  })
  .catch(error => {
    console.log(error);
  });
```

### Promise Rejection

```javascript
let promise = new Promise((resolve, reject) => {
  reject("Something failed");
});

promise
  .then(result => {
    console.log(result);
  })
  .catch(error => {
    console.log(error);
  });
```

## 53. `async` and `await`

`async` functions return promises.

```javascript
async function getMessage() {
  return "Hello";
}

getMessage().then(message => {
  console.log(message);
});
```

`await` waits for a promise.

```javascript
function wait() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve("Finished");
    }, 1000);
  });
}

async function run() {
  let result = await wait();

  console.log(result);
}

run();
```

### Error Handling with `async/await`

```javascript
async function run() {
  try {
    let result = await wait();

    console.log(result);
  } catch (error) {
    console.log(error);
  }
}
```

## 54. Fetch API

`fetch()` makes HTTP requests.

### GET Request

```javascript
fetch("https://example.com/data")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.log(error);
  });
```

### Using `async/await`

```javascript
async function getData() {
  try {
    let response = await fetch("https://example.com/data");
    let data = await response.json();

    console.log(data);
  } catch (error) {
    console.log(error);
  }
}

getData();
```

### Checking Response Status

```javascript
async function getData() {
  let response = await fetch("https://example.com/data");

  if (!response.ok) {
    throw new Error("Request failed");
  }

  let data = await response.json();

  return data;
}
```

### POST Request

```javascript
async function createUser() {
  let response = await fetch("https://example.com/users", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      name: "Alex",
      age: 25
    })
  });

  let data = await response.json();

  console.log(data);
}
```

## 55. Promise Utilities

### `Promise.all()`

Waits for all promises.

```javascript
let one = Promise.resolve(1);
let two = Promise.resolve(2);

let results = await Promise.all([one, two]);

console.log(results); // [1, 2]
```

If one promise fails, the whole operation fails.

### `Promise.allSettled()`

Waits for all promises, whether they succeed or fail.

```javascript
let results = await Promise.allSettled([
  Promise.resolve("Success"),
  Promise.reject("Failure")
]);

console.log(results);
```

### `Promise.race()`

Returns the first settled promise.

```javascript
let result = await Promise.race([
  fetch("/slow-request"),
  fetch("/fast-request")
]);
```

## 56. Modules

Modules allow code to be split across files.

### Exporting

`math.js`:

```javascript
export function add(a, b) {
  return a + b;
}

export const pi = 3.14159;
```

### Importing

`app.js`:

```javascript
import { add, pi } from "./math.js";

console.log(add(2, 3));
console.log(pi);
```

HTML:

```html
<script type="module" src="app.js"></script>
```

### Default Export

`greet.js`:

```javascript
export default function greet(name) {
  console.log(`Hello, ${name}`);
}
```

Import:

```javascript
import greet from "./greet.js";

greet("Alex");
```

## 57. Local Storage

Local storage saves data in the browser.

### Save Data

```javascript
localStorage.setItem("username", "Alex");
```

### Read Data

```javascript
let username = localStorage.getItem("username");

console.log(username);
```

### Remove Data

```javascript
localStorage.removeItem("username");
```

### Clear All Data

```javascript
localStorage.clear();
```

### Saving Objects

Local storage only stores strings.

```javascript
let user = {
  name: "Alex",
  age: 25
};

localStorage.setItem("user", JSON.stringify(user));
```

Reading it:

```javascript
let storedUser = JSON.parse(localStorage.getItem("user"));

console.log(storedUser.name);
```

## 58. Session Storage

Session storage works like local storage but usually lasts only for the current browser tab session.

```javascript
sessionStorage.setItem("theme", "dark");

let theme = sessionStorage.getItem("theme");

console.log(theme);
```

## 59. Cookies

Cookies store small pieces of data.

```javascript
document.cookie = "username=Alex";
```

A cookie with an expiration date:

```javascript
document.cookie = "theme=dark; max-age=3600";
```

Cookies are commonly used for sessions and preferences.

## 60. Regular Expressions

Regular expressions search for text patterns.

### Basic Match

```javascript
let pattern = /hello/i;

console.log(pattern.test("Hello world")); // true
```

The `i` flag makes the search case-insensitive.

### Digits

```javascript
let pattern = /\d+/;

console.log(pattern.test("Age: 25")); // true
```

### Email-Like Pattern

```javascript
let emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailPattern.test("alex@example.com")); // true
```

Regular expressions can be useful for validation, but complex validation should generally be handled carefully.

## 61. WeakMap and WeakSet

`WeakMap` stores key-value pairs where keys must be objects.

```javascript
let user = {};
let data = new WeakMap();

data.set(user, {
  loggedIn: true
});

console.log(data.get(user));
```

`WeakSet` stores objects without preventing garbage collection.

```javascript
let visitedUsers = new WeakSet();

let user = {};

visitedUsers.add(user);

console.log(visitedUsers.has(user)); // true
```

## 62. Iterators and Generators

A generator can pause and resume execution.

```javascript
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

let generator = numbers();

console.log(generator.next());
console.log(generator.next());
console.log(generator.next());
console.log(generator.next());
```

Output objects include:

```javascript
{
  value: 1,
  done: false
}
```

## 63. Symbols and Iteration

Objects can define custom iteration behavior using `Symbol.iterator`.

```javascript
let collection = {
  values: [1, 2, 3],

  *[Symbol.iterator]() {
    yield* this.values;
  }
};

for (let value of collection) {
  console.log(value);
}
```

## 64. Memory and Garbage Collection

JavaScript automatically removes objects that are no longer reachable.

```javascript
let user = {
  name: "Alex"
};

user = null;
```

After the object is no longer referenced, it may eventually be removed from memory.

Avoid unnecessary global variables and remove event listeners when they are no longer needed.

## 65. The Event Loop

JavaScript uses:

- Call stack
- Web APIs or runtime APIs
- Task queue
- Microtask queue
- Event loop

Example:

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise");
});

console.log("End");
```

Output:

```text
Start
End
Promise
Timeout
```

Promise callbacks usually run before timer callbacks.

## 66. Debugging

### Console Methods

```javascript
console.log("Normal message");
console.warn("Warning");
console.error("Error");
console.table([
  { name: "Alex", age: 25 },
  { name: "Sam", age: 30 }
]);
```

### Debugger Statement

```javascript
function add(a, b) {
  debugger;

  return a + b;
}
```

When developer tools are open, execution pauses at `debugger`.

### Common Debugging Steps

1. Read the error message.
2. Check the line number.
3. Print variables with `console.log()`.
4. Verify variable types.
5. Test smaller parts of the code.
6. Use breakpoints in developer tools.

## 67. Common Errors

### Reference Error

```javascript
console.log(username);
```

This happens when a variable does not exist.

### Syntax Error

```javascript
if (true {
  console.log("Hello");
}
```

This happens when the code structure is invalid.

### Type Error

```javascript
let number = 10;

// number.toUpperCase();
```

This happens when an operation is used with the wrong type.

### Logical Error

```javascript
let total = 10 - 5;

console.log(total); // The code runs, but perhaps addition was intended
```

Logical errors produce incorrect results without necessarily throwing an error.

## 68. Common JavaScript Patterns

### Check for Empty Input

```javascript
if (input.trim() === "") {
  console.log("Input is empty");
}
```

### Toggle a Boolean

```javascript
let isOpen = false;

isOpen = !isOpen;
```

### Count Items

```javascript
let items = ["a", "b", "c"];

console.log(items.length);
```

### Find the Largest Number

```javascript
let numbers = [4, 9, 2, 7];

let largest = Math.max(...numbers);

console.log(largest);
```

### Remove Duplicates

```javascript
let values = [1, 2, 2, 3, 3];

let uniqueValues = [...new Set(values)];

console.log(uniqueValues);
```

### Check an Object Property

```javascript
let user = {
  name: "Alex"
};

if ("name" in user) {
  console.log("Name exists");
}
```

### Use a Fallback

```javascript
let name = "";
let displayName = name || "Guest";

console.log(displayName);
```

## 69. Simple Project: Counter

HTML:

```html
<button id="decrease">−</button>
<span id="count">0</span>
<button id="increase">+</button>
```

JavaScript:

```javascript
let count = 0;

let countElement = document.querySelector("#count");
let increaseButton = document.querySelector("#increase");
let decreaseButton = document.querySelector("#decrease");

increaseButton.addEventListener("click", () => {
  count++;
  countElement.textContent = count;
});

decreaseButton.addEventListener("click", () => {
  count--;
  countElement.textContent = count;
});
```

## 70. Simple Project: Todo List

HTML:

```html
<input id="todoInput" placeholder="Enter a task" />
<button id="addTodo">Add</button>

<ul id="todoList"></ul>
```

JavaScript:

```javascript
let input = document.querySelector("#todoInput");
let addButton = document.querySelector("#addTodo");
let list = document.querySelector("#todoList");

addButton.addEventListener("click", () => {
  let task = input.value.trim();

  if (task === "") {
    return;
  }

  let item = document.createElement("li");

  item.textContent = task;

  item.addEventListener("click", () => {
    item.remove();
  });

  list.append(item);

  input.value = "";
});
```

## 71. Simple Project: Digital Clock

HTML:

```html
<h1 id="clock"></h1>
```

JavaScript:

```javascript
let clock = document.querySelector("#clock");

function updateClock() {
  let now = new Date();

  clock.textContent = now.toLocaleTimeString();
}

updateClock();

setInterval(updateClock, 1000);
```

## 72. Simple Project: Random Color Generator

HTML:

```html
<button id="changeColor">Change Color</button>
```

JavaScript:

```javascript
let button = document.querySelector("#changeColor");

button.addEventListener("click", () => {
  let color = `#${Math.floor(Math.random() * 16777215).toString(16)}`;

  document.body.style.backgroundColor = color;
});
```

## 73. JavaScript Best Practices

### Use `const` by Default

```javascript
const name = "Alex";
```

Use `let` only when reassignment is needed.

### Use Descriptive Names

```javascript
let userAge = 25;
```

Prefer this over:

```javascript
let x = 25;
```

### Keep Functions Small

```javascript
function calculateTotal(price, tax) {
  return price + tax;
}
```

### Avoid Global Variables

Keep variables inside functions or modules when possible.

### Use Strict Equality

```javascript
if (value === 10) {
  // ...
}
```

### Avoid Unnecessary Nesting

Instead of:

```javascript
if (user) {
  if (user.isActive) {
    console.log("Active");
  }
}
```

Use:

```javascript
if (user && user.isActive) {
  console.log("Active");
}
```

### Handle Errors

```javascript
try {
  // Code that may fail
} catch (error) {
  console.error(error);
}
```

### Avoid Repeating Code

Create reusable functions:

```javascript
function printMessage(message) {
  console.log(message);
}
```

### Use `textContent` for User Text

```javascript
element.textContent = userInput;
```

This is safer than inserting untrusted input with `innerHTML`.

## 74. JavaScript Naming Conventions

Use camelCase for variables and functions:

```javascript
let firstName = "Alex";

function calculateTotal() {}
```

Use PascalCase for classes:

```javascript
class UserAccount {}
```

Use uppercase names for constants that represent fixed configuration values:

```javascript
const MAX_RETRIES = 3;
```

## 75. JavaScript Execution Order

JavaScript generally executes from top to bottom.

```javascript
console.log("First");
console.log("Second");
console.log("Third");
```

Function calls can change when code runs:

```javascript
function showMessage() {
  console.log("Message");
}

console.log("Before");
showMessage();
console.log("After");
```

Output:

```text
Before
Message
After
```

## 76. Equality and Object References

Primitive values are compared by value:

```javascript
console.log(5 === 5); // true
```

Objects are compared by reference:

```javascript
let a = { value: 1 };
let b = { value: 1 };

console.log(a === b); // false
```

Both objects contain the same data, but they are different objects.

```javascript
let a = { value: 1 };
let b = a;

console.log(a === b); // true
```

## 77. Shallow and Deep Copies

### Shallow Copy

```javascript
let original = {
  name: "Alex",
  address: {
    city: "London"
  }
};

let copy = { ...original };
```

Nested objects are still shared.

### Deep Copy

For simple JSON-compatible data:

```javascript
let deepCopy = JSON.parse(JSON.stringify(original));
```

Modern JavaScript also supports:

```javascript
let deepCopy = structuredClone(original);
```

## 78. Optional Chaining and Safe Access

```javascript
let response = {
  user: {
    profile: {
      name: "Alex"
    }
  }
};

console.log(response.user?.profile?.name);
```

If one property is missing, the result is `undefined` instead of an error.

## 79. Nullish Assignment

```javascript
let username = null;

username ??= "Guest";

console.log(username); // Guest
```

The value is assigned only if it is `null` or `undefined`.

## 80. Logical Assignment

```javascript
let name = "";

name ||= "Guest";

console.log(name); // Guest
```

```javascript
let isLoggedIn = false;

isLoggedIn ||= true;

console.log(isLoggedIn); // true
```

```javascript
let value = true;

value &&= false;

console.log(value); // false
```

## 81. Browser APIs

Browsers provide many APIs besides the DOM.

Examples include:

```javascript
window.location.href;
window.innerWidth;
navigator.language;
navigator.onLine;
```

### Scroll Position

```javascript
console.log(window.scrollY);
```

### Browser Alert

```javascript
alert("Hello");
```

### Confirmation

```javascript
let confirmed = confirm("Are you sure?");

console.log(confirmed);
```

### User Input

```javascript
let name = prompt("What is your name?");

console.log(name);
```

## 82. URL API

```javascript
let url = new URL("https://example.com/products?page=2");

console.log(url.hostname);
console.log(url.pathname);
console.log(url.searchParams.get("page"));
```

### Creating Query Parameters

```javascript
let params = new URLSearchParams();

params.set("search", "javascript");
params.set("page", "1");

console.log(params.toString());
```

## 83. Internationalization

The `Intl` object formats numbers, dates, and currencies.

### Currency

```javascript
let price = 1234.5;

let formatted = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
}).format(price);

console.log(formatted);
```

### Date Formatting

```javascript
let date = new Date();

let formatted = new Intl.DateTimeFormat("en-US", {
  dateStyle: "long"
}).format(date);

console.log(formatted);
```

## 84. Testing Basics

A test checks whether code works as expected.

```javascript
function add(a, b) {
  return a + b;
}

console.assert(add(2, 3) === 5);
console.assert(add(1, 1) === 2);
```

If the condition is false, the browser displays an assertion error.

## 85. Security Basics

Avoid inserting untrusted input into HTML:

```javascript
element.innerHTML = userInput;
```

Prefer:

```javascript
element.textContent = userInput;
```

Other good practices:

- Validate input on both client and server.
- Do not place secret API keys in frontend code.
- Use HTTPS for network requests.
- Avoid using `eval()`.
- Do not trust user-provided data.
- Escape or sanitize HTML when HTML insertion is necessary.

## 86. Performance Basics

Improve performance by:

- Avoiding unnecessary DOM updates
- Using event delegation
- Debouncing frequent events
- Caching DOM elements
- Avoiding expensive loops
- Loading scripts with `defer`
- Splitting large code into modules

### Debouncing Example

```javascript
function debounce(callback, delay) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      callback(...args);
    }, delay);
  };
}

let search = debounce(() => {
  console.log("Searching...");
}, 500);
```

## 87. A Practical Learning Order

A useful order for learning JavaScript is:

1. Variables and constants
2. Data types
3. Operators
4. Conditions
5. Loops
6. Functions
7. Arrays
8. Objects
9. Array methods
10. DOM manipulation
11. Events
12. Forms
13. Asynchronous JavaScript
14. Promises and `async/await`
15. Fetch and APIs
16. Modules
17. Classes
18. Browser storage
19. Error handling
20. Testing and project structure

## 88. Final Practice Program

This example combines variables, functions, arrays, objects, DOM events, and array methods.

HTML:

```html
<input id="searchInput" placeholder="Search products" />

<ul id="productList"></ul>
```

JavaScript:

```javascript
const products = [
  { name: "Laptop", price: 900 },
  { name: "Phone", price: 500 },
  { name: "Keyboard", price: 80 }
];

const searchInput = document.querySelector("#searchInput");
const productList = document.querySelector("#productList");

function displayProducts(items) {
  productList.innerHTML = "";

  items.forEach(product => {
    const item = document.createElement("li");

    item.textContent = `${product.name} - $${product.price}`;

    productList.append(item);
  });
}

searchInput.addEventListener("input", () => {
  const searchText = searchInput.value.toLowerCase();

  const filteredProducts = products.filter(product =>
    product.name.toLowerCase().includes(searchText)
  );

  displayProducts(filteredProducts);
});

displayProducts(products);
```

This program demonstrates:

- Constants
- Arrays
- Objects
- Functions
- DOM selection
- Event listeners
- `filter()`
- `forEach()`
- String methods
- Dynamic HTML elements

## Conclusion

The most important JavaScript concepts are:

- Variables store values.
- Data types describe values.
- Operators manipulate values.
- Conditions make decisions.
- Loops repeat actions.
- Functions organize reusable logic.
- Arrays store collections.
- Objects represent structured data.
- The DOM lets JavaScript interact with HTML.
- Events respond to user actions.
- Promises handle asynchronous work.
- `async` and `await` make asynchronous code easier to read.
- Modules organize larger applications.
- Classes support object-oriented programming.
- Browser APIs connect JavaScript to web features.

The best way to learn JavaScript is to practice each topic by building small projects such as a calculator, todo list, quiz app, weather app, timer, or shopping cart.