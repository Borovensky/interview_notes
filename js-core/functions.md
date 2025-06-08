# JavaScript Functions Guide

## Table of Contents
- [Function Patterns](#function-patterns)
  - [IIFE (Immediately Invoked Function Expression)](#iife-immediately-invoked-function-expression)
  - [Module Pattern](#module-pattern)
- [Function Arguments](#function-arguments)
- [Recursion](#recursion)
- [Arrow Functions](#arrow-functions)
- [Function Parameters](#function-parameters)
- [Function Properties](#function-properties)
- [Function Scope and Hoisting](#function-scope-and-hoisting)
- [Constructor Functions](#constructor-functions)
- [Higher-Order Functions](#higher-order-functions)
- [Advanced Function Techniques](#advanced-function-techniques)
- [Best Practices](#best-practices)

## Function Patterns

### IIFE (Immediately Invoked Function Expression)

IIFE creates a private scope that executes immediately:

```javascript
// Basic IIFE pattern
(function() {
    var a = 'IIFE';
    console.log(a); // 'IIFE'
})();

// 'a' is not accessible here - private scope
// console.log(a); // ReferenceError
```

#### IIFE Benefits
```javascript
// Problem: Global namespace pollution
var userName = 'John';
var userAge = 30;

// Solution: IIFE creates private scope
(function() {
    var userName = 'John';
    var userAge = 30;
    
    // Only expose what's needed
    window.MyApp = {
        init: function() { /* private initialization */ }
    };
})();
```

### Module Pattern

The module pattern provides encapsulation - always returns an object:

```javascript
var module = (function() {
    var name; // Private variable
    
    return {
        setName: function(value) {
            name = value;
        },
        getName: function() {
            return name;
        }
    };
})();

module.setName('Bob');
console.log(module.getName()); // 'Bob'
// console.log(module.name); // undefined (private)
```

#### Enhanced Module Pattern
```javascript
const UserModule = (function() {
    // Private variables
    let users = [];
    let currentUser = null;
    
    // Private methods
    function validateUser(user) {
        return user && user.name && user.email;
    }
    
    // Public API
    return {
        addUser(user) {
            if (validateUser(user)) {
                user.id = Date.now();
                users.push(user);
                return user;
            }
            throw new Error('Invalid user data');
        },
        
        getCurrentUser() {
            return currentUser ? { ...currentUser } : null;
        }
    };
})();
```

## Function Arguments

### The Arguments Object
```javascript
var f = function() {
    console.log('Type:', typeof arguments); // 'object'
    console.log('Is array:', Array.isArray(arguments)); // false
    
    // Convert to real array
    const argsArray = Array.from(arguments);
    
    for (let item of arguments) {
        console.log(item);
    }
};

f(1, 's', 3);
```

### Arguments vs Rest Parameters
```javascript
// Old way - using arguments
function oldSum() {
    let total = 0;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;
}

// Modern way - using rest parameters
function modernSum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}

// Rest parameters are real arrays
function testRest(...args) {
    console.log(Array.isArray(args)); // true
    return args.map(x => x * 2);
}
```

## Recursion

```javascript
function pow(x, y) {
    if (y != 1) {
        return x * pow(x, y - 1);
    } else {
        return x;
    }
}
console.log(pow(2, 3)); // 8
```

### Better Recursion Examples
```javascript
// Improved with edge cases
function power(base, exponent) {
    if (exponent === 0) return 1;
    if (exponent === 1) return base;
    if (exponent < 0) return 1 / power(base, -exponent);
    
    return base * power(base, exponent - 1);
}

// Tail recursion (more memory efficient)
function factorialTail(n, accumulator = 1) {
    if (n <= 1) return accumulator;
    return factorialTail(n - 1, n * accumulator);
}

// Fibonacci with memoization
function fibonacciMemo() {
    const cache = {};
    
    return function fib(n) {
        if (n in cache) return cache[n];
        if (n <= 1) return n;
        
        cache[n] = fib(n - 1) + fib(n - 2);
        return cache[n];
    };
}
```

## Arrow Functions

Arrow function characteristics:
- No binding of this, super, new.target
- Cannot be called with 'new' keyword  
- No prototype property
- Cannot change 'this'
- No arguments object
- No duplicate parameter names

```javascript
// Basic syntax
const add = (a, b) => a + b;
const square = x => x * x;
const greet = () => 'Hello!';

// With block body
const complexOperation = (x, y) => {
    const temp = x * 2;
    return temp + y;
};

// Returning objects (need parentheses)
const createUser = (name, age) => ({ name, age });
```

### Arrow vs Regular Functions
```javascript
const obj = {
    name: 'Test',
    
    regularMethod: function() {
        console.log(this.name); // 'Test'
        
        // Arrow function preserves 'this'
        const arrowInner = () => {
            console.log(this.name); // 'Test'
        };
        arrowInner();
    },
    
    // Arrow method - 'this' is not the object!
    arrowMethod: () => {
        console.log(this.name); // undefined
    }
};
```

## Function Parameters

### Default Parameters and Arguments
As shown in your example, arguments ignores default parameters:

```javascript
function f(a, b = 10) {
    console.log('Parameters:', a, b);
    console.log('Arguments:', arguments);
}

f(1, 2); // Arguments: {'0': 1, '1': 2}
f(1);    // Arguments: {'0': 1} - default not in arguments!
```

### Advanced Parameter Patterns
```javascript
// Default parameters with expressions
function greet(name, greeting = 'Hello', punctuation = '!') {
    return `${greeting}, ${name}${punctuation}`;
}

// Destructuring parameters
function processUser({ name, age, email = 'not provided' }) {
    console.log(`Name: ${name}, Age: ${age}, Email: ${email}`);
}

// Rest parameters
function createMessage(primary, ...details) {
    return `${primary}: ${details.join(', ')}`;
}
```

## Function Properties

### The Name Property
```javascript
function test() {}
console.log(test.name); // 'test'

// Name property is read-only
test.name = 'newName';
console.log(test.name); // Still 'test'

// Function expressions
const f = function f1() {};
// console.log(f1.name); // ReferenceError: f1 is not defined
console.log(f.name);    // 'f1' - uses the function expression name!

// Anonymous functions get variable name
const anonymous = function() {};
console.log(anonymous.name); // 'anonymous'
```

### Other Function Properties
```javascript
function example(a, b, c) {
    return a + b + c;
}

console.log(example.length);    // 3 (number of parameters)
console.log(example.name);      // 'example'

// Custom properties
example.version = '1.0';
console.log(example.version);   // '1.0'
```

## Function Scope

### Block Scope for Functions
```javascript
if (true) {
    function f() {
        console.log(true);
    }
}

f(); // Works in non-strict mode!

// But in strict mode:
'use strict';
if (true) {
    function strictF() {
        console.log(true);
    }
}
// strictF(); // ReferenceError: strictF is not defined
```

### Function Hoisting
```javascript
// Function declarations are hoisted
console.log(hoisted()); // "I'm hoisted!"

function hoisted() {
    return "I'm hoisted!";
}

// Function expressions are NOT hoisted
// console.log(notHoisted()); // TypeError: notHoisted is not a function

var notHoisted = function() {
    return "I'm not hoisted!";
};
```

## Constructor Functions

### Dual Nature of Functions
```javascript
// Using instanceof (can be fooled)
function Person(name) {
    if (this instanceof Person) {
        this.name = name;
    } else {
        throw new Error('Use new');
    }
}

const alice = new Person('Alice'); // Works

// But instanceof can be bypassed!
const p = {};
Person.call(p, 'Charlie'); // Works! No error thrown

// Better solution: new.target (ES6)
function BetterPerson(name) {
    if (typeof new.target !== 'undefined') {
        // 100% reliable check for 'new' keyword
        this.name = name;
    } else {
        throw new Error('Use new');
    }
}

// Check specific constructor
function SpecificPerson(name) {
    if (new.target === SpecificPerson) {
        this.name = name;
    } else {
        throw new Error('Use new SpecificPerson');
    }
}
```

## Higher-Order Functions

HOF are functions that take other functions as arguments or return functions:

```javascript
// Function that takes another function as argument
function withLogging(fn) {
    return function(...args) {
        console.log(`Calling ${fn.name} with:`, args);
        const result = fn(...args);
        console.log('Result:', result);
        return result;
    };
}

// Function that returns another function
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
console.log(double(5)); // 10

// Common HOF examples
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);
```

## Advanced Function Techniques

### Memoization
Based on your memoization example:

```javascript
const memo = () => {
    let cache = {};
    return (n) => {
        if (n in cache) {
            return cache[n];
        } else {
            let res = n + 10;
            cache[n] = res;
            return res;
        }
    };
};

const addMemo = memo();
console.log(addMemo(9)); // 19
console.log(addMemo(9)); // 19 (from cache)
```

### Generic Memoization
```javascript
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Expensive fibonacci calculation
const fibonacci = memoize(function(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});
```

### Currying
```javascript
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...args2) {
                return curried.apply(this, args.concat(args2));
            };
        }
    };
}

const add3 = (a, b, c) => a + b + c;
const curriedAdd = curry(add3);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
```

### Function Composition
```javascript
// Compose functions from right to left
function compose(...fns) {
    return function(value) {
        return fns.reduceRight((acc, fn) => fn(acc), value);
    };
}

const add1 = x => x + 1;
const multiply2 = x => x * 2;
const square = x => x * x;

const composed = compose(square, multiply2, add1);
console.log(composed(3)); // square(multiply2(add1(3))) = 64
```

## Best Practices

### 1. Choose the Right Function Type
```javascript
// Use arrow functions for short, simple operations
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);

// Use regular functions for methods that need 'this'
const obj = {
    name: 'Test',
    greet() { // Not: greet: () => {
        return `Hello, ${this.name}`;
    }
};

// Use function declarations for functions that need hoisting
function utilityFunction() {
    // Can be called before declaration
}
```

### 2. Parameter Validation
```javascript
function processUser(user = {}) {
    // Destructuring with defaults
    const { 
        name = '', 
        age = 0, 
        email = '' 
    } = user;
    
    // Validation
    if (!name || !email) {
        throw new Error('Name and email are required');
    }
    
    if (age < 0 || age > 150) {
        throw new Error('Invalid age');
    }
    
    return { name, age, email };
}
```

### 3. Pure Functions When Possible
```javascript
// ❌ Impure - modifies external state
let total = 0;
function addToTotal(value) {
    total += value;
    return total;
}

// ✅ Pure - no side effects
function add(a, b) {
    return a + b;
}

// ✅ Pure - returns new array instead of modifying
function addToArray(arr, item) {
    return [...arr, item];
}
```

### 4. Use Modern Function Features
```javascript
// ✅ Modern approach
const processData = ({ data = [], transformer = x => x } = {}) => {
    return data
        .filter(item => item != null)
        .map(transformer)
        .filter(result => result !== undefined);
};

// ✅ Async function handling
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}
```

### Key Takeaways

1. **IIFE and Module patterns** provide encapsulation
2. **Arrow functions** have different `this` binding behavior
3. **Arguments object** vs rest parameters have important differences
4. **new.target** is more reliable than `instanceof` for constructors
5. **Default parameters** don't appear in arguments object
6. **Function hoisting** behavior differs between declarations and expressions
7. **Memoization** can optimize expensive recursive functions
8. **Higher-order functions** enable powerful functional programming patterns 