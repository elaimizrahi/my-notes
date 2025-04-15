## JavaScript Basics

### Overview

- Compiled at runtime (no type errors or checking)
- Java-like syntax, almost functional language
- Dynamic typing - similar to Python (not like Java/C++)

### Variables

- Two ways of declaring with `let` and `const`
    - `var` could be used, but is non-block scope
- Primitive types include `boolean`, `number`, `string`, `null`, and `undefined`
    - Everything else is an `Object` type
- Auto type conversion, we can use `typeof` to return the primitive types of variables as a string, usually comes back as an `Object`

### Equality

- `==` comparator can have odd behaviour

```js
0 == "" // true
1 == "1" // true
```

- Usually should use `===` for strict equality comparisons

```js
0 === "" // false
1 === "1" // false
```

- `||` operator is logical OR

```js
let v = 0;
v = v || 456 // returns 456
```

- `??` operator is nullish coalescing operator

```js
let v = 0;
v = v ?? 456 // v = 0 since v wasn't undefined or null
```

### Functions

Closures allow an inner function to reference the state of an outer function

```js
function makeRandomGreeting() { 
	const sal = Math.random() > 0.5 ? "Hi" : "Hello"; 
	return function (name) { return sal + " " + name; } 
} 
const greeting = makeRandomGreeting(); 
console.log(greeting("Sam")) // ?? Sam
```

Can also create factory functions that pass through a function

```js
function makeGreeting(msg) { 
	return function (name) { return msg() + name; }
} 
function sayHello() { return "Hello, "; } 
const greeting2 = makeGreeting(sayHello); 
console.log(greeting2("Sam")); // Hello, Sam
```

Lambda functions allow us to write a single-use function

```js
const greeting3 = makeGreeting(() => "Howdy! ");
```

### Prototype Chain using Constructor Function

```js
// a constructor function
function Shape(colour) {
 this.colour = colour;
 this.draw = function () {
 return `A ${this.colour} shape.`;
 }
}
function Square(colour, size) {
 Shape.call(this, colour);
 this.size = size;
 this.draw = function () {
 return `A ${this.colour} square with size ${this.size}`;
 }
}
const square = new Square("red", 10);
```

- `this` refers to object context
- `call` prototype constructor and link to this object
- Creates a "shadow" property

### Class (like a “template” for creating objects)

```js
class Shape {
 constructor(colour) { this.colour = colour; }
 draw() { return `A ${this.colour} shape.`; }
}
class Square extends Shape {
 constructor(colour, size) {
 super(colour);
 this.size = size;
 }
 draw() {
 return `A ${this.colour} square size ${this.size}`;
 }
}
const square = new Square("red", 10);
```

### Arrays

- Arrays are an example of an iterable object
- Some ways to declare an Array:

```js
let arr1 = [] // empty array with length 0
let arr2 = Array(5); // empty array with length 5
let arr3 = [1, 2, 3, 4, 5]; // populated array
let arr4 = Array(5).fill(99); // 5 elements, all 99
```

- Some ways to iterate over an array:

```js
for (let i = 0; i < arr3.length; i++) {
 console.log(arr3[i])
}
for (const x of arr3) { console.log(x) }
arr3.forEach((x) => console.log(x));
```

### Array Methods

- `forEach`, `sort`, `reverse`, `splice`, `indexOf`, and many more
- Note that some methods mutate the array, so check the docs

### Common Functional Array Methods

```js
let arr3 = [1, 2, 3, 4, 5];
// map returns array with transformed elements
const arr4 = arr3.map((x) => x * 10);
// [10, 20, 30, 40, 50]

// find returns first element that satisfies condition
const a = arr3.find((x) => x % 2 == 0);
// 2

// filter returns all elements that satisfy condition
const arr5 = arr3.filter((x) => x % 2 == 0);
// [2, 4]

// reduce executes a function that accumulates a single return value
const arr6 = arr3.reduce((acc, x) => acc + x, 0);
// 15
```

### Destructuring Assignment

- Unpack array elements or object properties into distinct variables

```js
// From Arrays
let arr3 = [1, 2, 3, 4, 5];
let [a, b] = arr3; // a = 1, b = 2

// From Objects
let obj = { "a": 1, "b": 2, "c": 3 };
let { a, b } = obj; // a = 1, b = 2

// Can rename destructured variables from objects
let { a: x, b: y } = obj; // x = 1, y = 2
```

### Spread Syntax and Rest Syntax

- Spread expands an iterable object (i.e., array, string)

```js
let arr3 = [1, 2, 3, 4, 5];
let arr4 = [-1, 0, ...arr3, 6, 7];
console.log(arr4); // [-1, 0, 1, 2, 3, 4, 5, 6, 7]
```

- Rest condenses multiple elements into single element

```js
let [a, b, ...c] = arr3;
console.log(c); // [3, 4, 5]

const obj = { a: 1, b: 2, c: 3 };
let { a, ...x } = obj;
console.log(x); // {b: 2, c: 3}

let { b, ...y } = obj; // what's y?
```

### Create a Prepopulated Array using Spread and Map

```js
let arr5 = [...Array(5)].map((_, i) => i * 10);
console.log(arr5); // [0, 10, 20, 30, 40]
```

### Resources for Learning JavaScript

- [MDN Introduction to JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction)
- [Strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
