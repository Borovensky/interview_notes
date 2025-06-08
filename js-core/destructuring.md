# JavaScript Destructuring Guide

## Table of Contents
- [What is Destructuring?](#what-is-destructuring)
- [Object Destructuring](#object-destructuring)
- [Array Destructuring](#array-destructuring)
- [Function Parameter Destructuring](#function-parameter-destructuring)
- [Rest and Spread Operators](#rest-and-spread-operators)
- [Advanced Destructuring Patterns](#advanced-destructuring-patterns)
- [ES5 vs ES6+ Approaches](#es5-vs-es6-approaches)
- [Practical Use Cases](#practical-use-cases)
- [Best Practices](#best-practices)

## What is Destructuring?

Destructuring is a JavaScript expression that allows you to extract values from arrays or properties from objects into distinct variables. It provides a more concise and readable way to access nested data.

## Object Destructuring

### Basic Object Destructuring
```javascript
const person = {
    name: 'John',
    age: 30,
    city: 'New York'
};

// ES6 destructuring
const { name, age, city } = person;
console.log(name); // "John"
console.log(age);  // 30
console.log(city); // "New York"

// ES5 equivalent
var name = person.name;
var age = person.age;
var city = person.city;
```

### Renaming Variables
```javascript
const options = {
    repeat: true,
    save: false
};

// Rename 'save' to 'write'
const { save: write, repeat: shouldRepeat } = options;
console.log(write);       // false
console.log(shouldRepeat); // true

// ES5 equivalent
var write = options.save;
var shouldRepeat = options.repeat;
```

### Default Values
```javascript
const settings = {
    theme: 'dark'
};

// Provide default values for missing properties
const { theme, fontSize = 14, language = 'en' } = settings;
console.log(theme);    // "dark"
console.log(fontSize); // 14 (default)
console.log(language); // "en" (default)
```

### Combining Renaming and Defaults
```javascript
const config = {
    host: 'localhost'
};

const { 
    host: serverHost, 
    port: serverPort = 3000,
    ssl: useSSL = false 
} = config;

console.log(serverHost); // "localhost"
console.log(serverPort); // 3000 (default)
console.log(useSSL);     // false (default)
```

### Nested Object Destructuring
```javascript
const user = {
    id: 1,
    profile: {
        name: 'Alice',
        address: {
            street: '123 Main St',
            city: 'Boston'
        }
    }
};

// Extract nested properties
const {
    profile: {
        name,
        address: { street, city }
    }
} = user;

console.log(name);   // "Alice"
console.log(street); // "123 Main St"
console.log(city);   // "Boston"

// Note: 'profile' and 'address' are not available as variables
// console.log(profile); // ReferenceError

// To get both the nested object and its properties
const {
    profile,
    profile: { name: userName }
} = user;
console.log(profile);  // { name: 'Alice', address: {...} }
console.log(userName); // "Alice"
```

### Dynamic Property Names
```javascript
const key = 'dynamicKey';
const obj = {
    dynamicKey: 'value'
};

// Use computed property names
const { [key]: value } = obj;
console.log(value); // "value"

// With functions
function extractProperty(obj, propName) {
    const { [propName]: result } = obj;
    return result;
}
```

## Array Destructuring

### Basic Array Destructuring
```javascript
const colors = ['red', 'green', 'blue'];

// ES6 destructuring
const [first, second, third] = colors;
console.log(first);  // "red"
console.log(second); // "green"
console.log(third);  // "blue"

// ES5 equivalent
var first = colors[0];
var second = colors[1];
var third = colors[2];
```

### Skipping Elements
```javascript
const numbers = [1, 2, 3, 4, 5];

// Skip elements using empty slots
const [first, , third, , fifth] = numbers;
console.log(first); // 1
console.log(third); // 3
console.log(fifth); // 5
```

### Default Values in Arrays
```javascript
const arr = [1];

const [a, b = 2, c = 3] = arr;
console.log(a); // 1
console.log(b); // 2 (default)
console.log(c); // 3 (default)
```

### Swapping Variables
```javascript
let a = 1;
let b = 2;

// Swap without temporary variable
[a, b] = [b, a];
console.log(a); // 2
console.log(b); // 1

// ES5 equivalent
var temp = a;
a = b;
b = temp;
```

### Nested Array Destructuring
```javascript
const nested = [[1, 2], [3, 4]];

const [[a, b], [c, d]] = nested;
console.log(a, b, c, d); // 1 2 3 4
```

## Function Parameter Destructuring

### Object Parameters
```javascript
// Instead of passing multiple parameters
function createUser(name, age, email, isActive) {
    // ...
}

// Use object destructuring
function createUser({ name, age, email, isActive = true }) {
    console.log(`Creating user: ${name}, age: ${age}`);
    return { name, age, email, isActive };
}

// Usage
const userData = { name: 'John', age: 30, email: 'john@email.com' };
const user = createUser(userData);

// Or inline
const user2 = createUser({ 
    name: 'Jane', 
    age: 25, 
    email: 'jane@email.com' 
});
```

### Array Parameters
```javascript
function sumFirstTwo([a, b]) {
    return a + b;
}

console.log(sumFirstTwo([1, 2, 3, 4])); // 3
```

### Mixed Destructuring in Functions
```javascript
function processOrder({ 
    orderId, 
    customer: { name, email }, 
    items: [firstItem, ...restItems] 
}) {
    console.log(`Order ${orderId} for ${name} (${email})`);
    console.log(`First item: ${firstItem}`);
    console.log(`Remaining items: ${restItems.length}`);
}

const order = {
    orderId: '12345',
    customer: { name: 'Alice', email: 'alice@email.com' },
    items: ['laptop', 'mouse', 'keyboard']
};

processOrder(order);
```

## Rest and Spread Operators

### Rest in Array Destructuring
```javascript
const numbers = [1, 2, 3, 4, 5];

const [first, second, ...rest] = numbers;
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]
```

### Rest in Object Destructuring
```javascript
const person = {
    name: 'John',
    age: 30,
    city: 'New York',
    country: 'USA'
};

const { name, ...details } = person;
console.log(name);    // "John"
console.log(details); // { age: 30, city: 'New York', country: 'USA' }
```

### Function Arguments with Rest Parameters
```javascript
// ES5 way - converting arguments to array
function numbersES5() {
    // arguments is array-like object: { '0': 1, '1': 2, '2': 3 }
    const params = [].slice.call(arguments);
    // or Array.prototype.slice.call(arguments)
    console.log(params); // [1, 2, 3]
    
    return params.reduce((sum, num) => sum + num, 0);
}

// ES6+ way - using rest parameters
function numbersES6(...params) {
    console.log(params); // [1, 2, 3] - real array
    return params.reduce((sum, num) => sum + num, 0);
}

console.log(numbersES5(1, 2, 3)); // 6
console.log(numbersES6(1, 2, 3)); // 6

// Mixed parameters with rest
function mixedParams(first, second, ...rest) {
    console.log('First:', first);   // 1
    console.log('Second:', second); // 2
    console.log('Rest:', rest);     // [3, 4, 5]
}
mixedParams(1, 2, 3, 4, 5);
```

### Spread Operator
```javascript
// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spreading
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Function calls
function sum(a, b, c) {
    return a + b + c;
}
const numbers = [1, 2, 3];
console.log(sum(...numbers)); // 6

// ES5 equivalent for function calls
console.log(sum.apply(null, numbers)); // 6
```

## Advanced Destructuring Patterns

### Destructuring in Loops
```javascript
const users = [
    { name: 'John', age: 30 },
    { name: 'Jane', age: 25 },
    { name: 'Bob', age: 35 }
];

// Destructure in for...of loop
for (const { name, age } of users) {
    console.log(`${name} is ${age} years old`);
}

// Destructure array entries
for (const [index, { name }] of users.entries()) {
    console.log(`${index}: ${name}`);
}
```

### Destructuring with Regular Expressions
```javascript
const url = 'https://api.example.com/users/123';
const match = url.match(/https:\/\/(.+)\/(.+)\/(\d+)/);

if (match) {
    const [, domain, endpoint, id] = match;
    console.log(`Domain: ${domain}`); // api.example.com
    console.log(`Endpoint: ${endpoint}`); // users
    console.log(`ID: ${id}`); // 123
}
```

### Destructuring Return Values
```javascript
function getCoordinates() {
    return { x: 10, y: 20 };
}

function getColors() {
    return ['red', 'green', 'blue'];
}

// Destructure function returns
const { x, y } = getCoordinates();
const [primary, secondary] = getColors();
```

### Conditional Destructuring
```javascript
const data = { user: { name: 'John' } };

// Safe destructuring with optional chaining (ES2020)
const { user: { name } = {} } = data || {};

// Or with default values
const { user = {} } = data;
const { name: userName = 'Anonymous' } = user;
```

## ES5 vs ES6+ Approaches

### Extracting Multiple Properties
```javascript
const config = {
    host: 'localhost',
    port: 3000,
    ssl: true,
    timeout: 5000
};

// ES5 approach
var host = config.host;
var port = config.port;
var ssl = config.ssl;
var timeout = config.timeout;

// ES6+ destructuring
const { host, port, ssl, timeout } = config;
```

### Handling Function Arguments
```javascript
// ES5 - verbose and error-prone
function createConnection(options) {
    var host = options.host || 'localhost';
    var port = options.port || 3000;
    var ssl = options.ssl || false;
    // ...
}

// ES6+ - clean and declarative
function createConnection({ 
    host = 'localhost', 
    port = 3000, 
    ssl = false 
} = {}) {
    // ...
}
```

### Working with Arrays
```javascript
const point = [10, 20];

// ES5
var x = point[0];
var y = point[1];

// ES6+
const [x, y] = point;
```

## Practical Use Cases

### API Response Handling
```javascript
async function fetchUser(id) {
    const response = await fetch(`/api/users/${id}`);
    const { 
        data: { 
            name, 
            email, 
            profile: { avatar, bio } 
        },
        status 
    } = await response.json();
    
    return { name, email, avatar, bio, status };
}
```

### Configuration Objects
```javascript
function initializeApp({
    apiUrl = '/api',
    version = '1.0',
    features: {
        authentication = true,
        logging = false
    } = {}
} = {}) {
    console.log(`App v${version} connecting to ${apiUrl}`);
    if (authentication) console.log('Auth enabled');
    if (logging) console.log('Logging enabled');
}

// Usage
initializeApp({
    apiUrl: '/api/v2',
    features: { logging: true }
});
```

### Module Imports
```javascript
// Named imports are destructuring
import { useState, useEffect } from 'react';
import { map, filter, reduce } from 'lodash';

// Equivalent to:
// const { useState, useEffect } = require('react');
```

### Event Handling
```javascript
function handleFormSubmit(event) {
    event.preventDefault();
    
    const formData = new FormData(event.target);
    const { name, email, message } = Object.fromEntries(formData);
    
    // Process form data...
}

// Mouse event handling
function handleClick({ target, clientX, clientY }) {
    console.log(`Clicked ${target.tagName} at (${clientX}, ${clientY})`);
}
```

### Working with Maps and Sets
```javascript
const userMap = new Map([
    ['john', { age: 30, city: 'NYC' }],
    ['jane', { age: 25, city: 'LA' }]
]);

// Destructure Map entries
for (const [username, { age, city }] of userMap) {
    console.log(`${username}: ${age} years old, lives in ${city}`);
}
```

## Best Practices

### 1. Use Meaningful Variable Names
```javascript
// Good
const { firstName, lastName, email } = user;

// Less clear
const { a, b, c } = user;
```

### 2. Provide Default Values
```javascript
// Protect against undefined/null
function processOptions({ 
    timeout = 5000, 
    retries = 3, 
    debug = false 
} = {}) {
    // Safe to use timeout, retries, debug
}
```

### 3. Limit Nesting Depth
```javascript
// Too deeply nested - hard to read
const { a: { b: { c: { d } } } } = complexObject;

// Better - break it down
const { a } = complexObject;
const { b } = a;
const { c } = b;
const { d } = c;
```

### 4. Use Rest Operator Wisely
```javascript
// Good - clear intent
const { id, ...userDetails } = user;

// Good - collecting remaining items
const [first, ...remaining] = items;

// Avoid - unclear what's being collected
const { ...everything } = someObject;
```

### 5. Consider Performance
```javascript
// For frequently called functions, destructuring has minimal overhead
function fastFunction({ x, y }) {
    return x + y; // Fast
}

// But excessive destructuring in hot paths might impact performance
function hotPath(data) {
    // If called millions of times, consider:
    return data.x + data.y; // Instead of destructuring
}
```

### Key Takeaways

1. **Destructuring improves readability** and reduces boilerplate code
2. **Use default values** to handle missing properties safely
3. **Rest parameters** are better than the `arguments` object
4. **Combine destructuring** with default parameters for robust functions
5. **Don't over-nest** - keep destructuring patterns readable
6. **Use meaningful names** when renaming destructured properties
7. **Destructuring works everywhere** - parameters, returns, loops, imports 