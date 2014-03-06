---
published: false
---

---
layout: post
title: 'JavaScript 2.0/ECMAScript 6 Features, Syntax and Tools'
---

> This article is a work in progress, and might not contain all the features in the specification,
> and will change to reflect the specification as the specification changes.
> Please direct any feedback [@caspervonb](http://twitter.com/caspervonb) and i will update it accordingly.

## Introduction
ECMAScript 6 "Harmony" is the upcoming upcoming version of the ECMAScript standard, the standard previously said to be ratified in late 2013, is now targeting ratification in late 2014.
However, even tho it is not standardized yet, it has gone into a feature freeze and most of the proposed features have stabilized, so it is reasonably safe to start using it today. With that in mind let us examine some of its new language features, library changes and available tools for using it today.

## Features
Harmony implements quite a few of the features that where previously
scheduled for the abandoned ECMAScript 4 draft, these include the following.

### Block Scoping
Variables declared with `var` have a function-level scope, block scoping introduces new forms of declaration for defining variables scoped to a single block.
These include `var`, `const` and `function`.

Using `let` in place of `var` allows you to define block-local variables without having to worry about them clashing with variables defined elsewhere within the same function body.

Example:

```javascript
for (var i = 0; i < 3; i++) {
   let j = i * i;
   console.log(j);
}
console.log(j); // => error, j is undefined
```

Using `const` in place of `var` follow the same afformentioned rules, except that the value is immutable so you can only assign to it once.

Example:

```javascript
const PI = 3.14159265359;
PI = 0; // => error, const already defined.
```

### Destructuring Assignment
Destructuring assignment allows the assignment of parts of data structures to several variables at once.

Example, multiple assignment:

```javascript
var [a, b, c] = ['hello', ', ', 'world'];
console.log(a + b + c); // hello, world
```

Example, destructuring an object:

```javascript
var point = new Point();
// ...
var {y} = point;
```

### Default Parameter Values
Default parameter values allows us to initialize parameters when they were not explicitly provided. This means that we no longer have to handle `undefined` just in order to provide default values.

Example:

```javascript
function Point(x = 0, y = 0) {
   this.x = x;
   this.y = y;
}

var p = new Point(); // => { 0, 0 }
```

### Rest Parameters
Rest parameters provides a cleaner way of dealing with variadic functions, that is functions that take a arbitrary number of parameters.

Example:

```javascript
function add(...values) {
   let sum = 0;

   for (var val of values) {
      sum += val;
   }

   return sum;
}

add(2, 5, 3); // => 10
```

### Spread Operator
The spread operator allows an expression to be expanded in places where multiple arguments or multiple elements are expected. 

Example:

```javascript
var a = [0, 1, 2];
var b = [3, 4, 5];

a.push(...b); // => [0, 1, 2, 3, 4, 5]
```

### Method Definition
Provides shorthand syntax for method definitions in object literals, this syntax is also compatible with classes.

Example:

```javascript
obj = {
   toString() {
      return "obj";
   },
};
```

### Classes
Classes provide clean simple declarative syntax for defining object prototypes and inheritance chains.

Example:

```javascript
class Monster extends Character {
   constructor(name, health) {
      super();
      this.name = name;
      this._health = health;
   }

   attack(target) {
      console.log('The monster attacks ' + target);
   }

   get isAlive() {
      return this._health > 0;
   }

   set health(value) {
      if (value < 0) {
         throw new Error('Health must be non-negative.')
      }

      this._health = value;
   }
}
```

### Symbols
Symbol is a new kind of primitive value type, using symbols instead of strings allows you to create properties that don't conflict with other properties because each symbol is unique by nature.

Example:

```javascript
let dead = Symbol();

let obj = {
   [dead]: false,
};
```

### Iterators
Iterators allow iteration over arbitrary objects, any object can be an iterator as long as it defines a `next()` method, any object can be iterable as long as it defines an `iterator()` method.

Example:

```javascript
function RangeIterator(min, max) {
   this['iterator'] = function () {
      var current = min;

      return {
         next: function () {
            current++;

            if (current > max) {
               throw StopIteration;
            }

            return current;
         }
      }
   };
}
```

### For Of Loop
The `for-of` loop allows you to conveniently loop over iterable objects.

Example:

```javascript
for (let i of new RangeIterator([1, 10])) {
   console.log(i);
}
```

### Generators
Generators make it easy to create iterators. Instead of tracking state yourself and implementing `iterator`, you just use yield (or yield* to yield each element in an iterator).

Example:

```javascript
function *range(min, max) {
   for (var i = min; i < max; i++) {
      yield i;
   }
}

for (let value of range(0, 100)) {
   console.log(value);
}
```

### Array Comprehensions
Array comprehensions provide a convenient, declarative form for creating computed arrays with a literal syntax that reads naturally. 

Example, filtering an array:

```javascript
[ x for (x of a) if (x.color === ‘blue’) ]
```

Example, mapping an array:

```javascript
[ square(x) for (x of [1,2,3,4,5]) ]
```

### Generator Expressions
Generator expressions provide a convenient, declarative form for creating generators with a syntax based on array comprehensions.

```javascript
(x for (x of generateValues()) if (x.color === ‘blue’))
```

### Arrow Functions
Arrow functions provide a shorthand syntax for function expressions.

```javascript
element.addEventListener('click' (e) => console.log(e));
```

### Template Strings
Template strings allows us to use string literals with embedded expressions within them.

Using template strings, you can do string interpolation with succinct simple syntax.
Example:

```javascript
var x = 1;
var y = 2;
console.log(`${ x } + ${ y } = ${ x + y}`); // => "1 + 2 = 3"
```

Using template strings you can do literal multiline strings in a convinient manner.
Example:

```
var s = `a
   b
   c`;

assert(s === 'a\n   b\n   c');
```

## Transpilation
Currently, it's a bit mix and match as to what browser implements which features, and it's going to be a long time before vendors catch up and become feature complete. Even then there is that **one** browser that you know will take even longer to catch up so running this directly harmony in the browser is not feasible.

What we can do however is [transpile](http://en.wikipedia.org/wiki/Source-to-source_compiler) our source code to run in ES5 compatible browsers. Google has a project called [Traceur](https://github.com/google/traceur-compiler) which has been around since 2011, in the early days it was experimentation with possible future syntax, now that future syntax is here and going forward it looks like they are focusing on getting together an ES6 compatible transpiler.

There is also [es6-transpiler](https://github.com/termi/es6-transpiler) which tries to have cleaner output than Traceur, primarily Traceur has a runtime library, es6-transpiler does not.