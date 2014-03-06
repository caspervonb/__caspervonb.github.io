---
published: false
---

---
layout: post
title: 'JavaScript 2.0/ECMAScript 6 Features, Syntax and Tools'
---

> This article is a work in progress, and might not contain all the features in the spesification,
> and will change to reflect the specification as the specification changes.
> Please direct any feedback [@caspervonb](http://twitter.com/caspervonb).

## Introduction
ECMAScript 6 "Harmony" the upcoming upcoming version of the ECMAScript standard, the standard previously said to be rattified in late 2013, is now targgeting rafficaition in late 2014.
However, it is unlikelyyou can use it right now, lets examine some of its new language features, library changes and available tools.

## Features
### Block Scoping
Variables declared with `var` have a function-level scope, block scoping introduces new forms of declaration for defining variables scoped to a single block.
These include `var`, `const` and `function`.

Using `let` in place of `var` allows you to define block-local variables without having to worry about them clashing with variables elsewhere within the same function body.

Example:
```javascript
for (var i = 0; i < 3; i++) {
   let j = i * i;
   console.log(j);
}
console.log(j); // => error, j is undefined
```

Using `const` in place of `var` follow the same rules, except that the value is immutable so you can only assign to it once.

Example:
```javascript
const PI = 3.14159265359;
PI = 0; // => error, const already defined.
```

### Destructuring Assignment
Destructuring allows you to assign multiple variables at once.

```javascript
var [a, b, c] = ['hello', ', ', 'world'];
console.log(a + b + c); // hello, world
```

It can also be used to unpack the values of an object.
```
var {x, y} = new Point(3, 4);
```

### Default Parameter Values
Default parameter values allows us to initialize parameters when they were not explicitly provided. This means that we no longer have to handle `undefined` just in order to provide default values.

```javascript
function Point(x = 0, y = 0) {
   this.x = x;
   this.y = y;
}

var p = new Point(); // { 0, 0 }
```

### Rest Parameters
Rest parameters provides a cleaner way of dealing with variadic functions, that is functions that take a arbitrary number of parameters.

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

```javascript
var a = [0, 1, 2];
var b = [3, 4, 5];

a.push(...b); // => [0, 1, 2, 3, 4, 5]
```
### Modules
```javascript
module "point" {
    export class Point {
        constructor (x, y) {
            public x = x;
            public y = y;
        }
    }
}
```

```javascript
module point from "/point.js";
import Point from "point";
 
var origin = new Point(0, 0);
console.log(origin);
```

### Method definition
Provides shorthand syntax for method definitions.

```javascript
obj = {
   toString() {
      return "obj";
   }
}
```

### Classes
Classes provide simple declarative syntax for defining prototypes and inheritance chains.

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
Symbols are a new kind of object that can be used as a unique property name in objects.
Using symbols instead of strings allows you to create properties that don't conflict with one another.
Symbols can also be made private, so that their properties can't be accessed by anyone who doesn't already have direct access to the symbol.

```javascript
var dead = new Symbol('dead');

class Character {
   constructor() {
      dead[dead] = false;
   }

   get dead() {
      return this[dead];
   }

   kill() {
      this[dead] = true;
   }
}

var char = new Character();
char.dead = true; // => undefined
```

### Iterators
Iterators allow any object to be iterable with the `for-of` loop, provided they define an `iterator()` method, an iterator is any object that has a `next()` method.

```
collection = {
   iterator() {
      next: () {
      },
   },
}
```

### For Of Loop
The `for-of` loop allows you to conveniently loop over iterable objects.
```javascript
for (let word of ['one', 'two', 'three']) {
   console.log(word); // => one two three
}
```

### Generators
Generators make it easy to create iterators. Instead of tracking state yourself and implementing `iterator`, you just use yield (or yield* to yield each element in an iterator): 

### Array Comprehensions
Array comprehensions provide a convenient, declarative form for creating computed arrays with a literal syntax that reads naturally. 

Filtering an array:
```javascript
[ x for (x of a) if (x.color === ‘blue’) ]
```

Mapping an array:

```javascript
[ square(x) for (x of [1,2,3,4,5]) ]
```

### Generator Expressions
Generator expressions provide a convenient, declarative form for creating generators with a syntax based on array comprehensions.

```javascript
(x for (x of generateValues()) if (x.color === ‘blue’))
```

### Arrow Functions
Arrow functions provide a more convinient syntax for one-liner functions.

```javascript
element.addEventListener('click' (e) => console.log(e));
```

### Template Strings
Template strings allows us to use string literals with embedded expressions within them.

##### String Interpolation.

```javascript
var x = 1;
var y = 2;
`${ x } + ${ y } = ${ x + y}`  // => "1 + 2 = 3"
```

Using template strings lets us do multiline strings.

```
var s = `a
   b
   c`;

assert(s === 'a\n   b\n   c');
```

## Transpilation
Currently, it's a bit mix and match of what browser implements which features, and it's going to be a long time before vendors catch up and become feature complete. Even then there is that **one** browser that you know will take even longer to catch up so running this directly harmony in the browser is not feasable.

What we can do however is [transpile](http://en.wikipedia.org/wiki/Source-to-source_compiler) our source code to run in ES5 compatible browsers. Google has a project called [Traceur](https://github.com/google/traceur-compiler) which has been around since 2011, in the early days it was experimentation with possible future syntax, now that future syntax is here and going forward it looks like they are focusing on getting together an ES6 compatible transpiler.

There is also [es6-transpiler](https://github.com/termi/es6-transpiler) which tries to have cleaner output than Traceur, primarily Traceur has a runtime library, es6-transpiler does not.