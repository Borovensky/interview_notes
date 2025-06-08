# JavaScript Data Types Guide

## Table of Contents
- [Primitive Data Types](#primitive-data-types)
- [Reference Data Types](#reference-data-types)
- [Floating Point Arithmetic Issues](#floating-point-arithmetic-issues)
- [Type Coercion](#type-coercion)
- [Special Number Values](#special-number-values)
- [Number System Conversions](#number-system-conversions)
- [Literal vs Constructor Creation](#literal-vs-constructor-creation)
- [Working with undefined](#working-with-undefined)
- [Increment and Decrement Operators](#increment-and-decrement-operators)
- [Mathematical Operations](#mathematical-operations)
- [Type Checking Methods](#type-checking-methods)
- [Common Gotchas](#common-gotchas)

## Primitive Data Types

JavaScript has 7 primitive data types:

### 1. Number
```javascript
const integer = 42;
const float = 3.14159;
const negative = -100;
const scientific = 1e6; // 1,000,000

// Special number values
const infinity = Infinity;
const negInfinity = -Infinity;
const notANumber = NaN;

console.log(typeof integer); // "number"
console.log(Number.isInteger(42)); // true
console.log(Number.isFinite(Infinity)); // false
```

### 2. String
```javascript
const singleQuote = 'Hello';
const doubleQuote = "World";
const template = `Hello ${doubleQuote}`;
const multiline = `Line 1
Line 2`;

console.log(typeof singleQuote); // "string"
console.log('Hello'.length); // 5
```

### 3. Boolean
```javascript
const isTrue = true;
const isFalse = false;

console.log(typeof isTrue); // "boolean"

// Truthy and falsy values
console.log(Boolean(1)); // true
console.log(Boolean(0)); // false
console.log(Boolean('')); // false
console.log(Boolean('hello')); // true
```

### 4. Undefined
```javascript
let notAssigned;
console.log(notAssigned); // undefined
console.log(typeof notAssigned); // "undefined"

function noReturn() {}
console.log(noReturn()); // undefined
```

### 5. Null
```javascript
const empty = null;
console.log(typeof empty); // "object" (this is a known bug in JavaScript)
console.log(empty === null); // true
```

### 6. Symbol (ES6)
```javascript
const symbol1 = Symbol();
const symbol2 = Symbol('description');
const symbol3 = Symbol('description');

console.log(typeof symbol1); // "symbol"
console.log(symbol2 === symbol3); // false (symbols are always unique)

// Global symbol registry
const globalSymbol1 = Symbol.for('key');
const globalSymbol2 = Symbol.for('key');
console.log(globalSymbol1 === globalSymbol2); // true
```

### 7. BigInt (ES2020)
```javascript
const bigNumber = 123456789012345678901234567890n;
const bigFromNumber = BigInt(123);

console.log(typeof bigNumber); // "bigint"
console.log(bigNumber > Number.MAX_SAFE_INTEGER); // true

// Cannot mix BigInt with regular numbers
// console.log(bigNumber + 1); // TypeError
console.log(bigNumber + 1n); // 123456789012345678901234567891n
```

## Reference Data Types

### Object
```javascript
const obj = { name: 'John', age: 30 };
const arr = [1, 2, 3];
const func = function() {};
const date = new Date();

console.log(typeof obj); // "object"
console.log(typeof arr); // "object"
console.log(typeof func); // "function"
console.log(typeof date); // "object"

// More specific type checking
console.log(Array.isArray(arr)); // true
console.log(obj instanceof Object); // true
console.log(date instanceof Date); // true
```

## Floating Point Arithmetic Issues

One of the most common JavaScript gotchas involves floating point precision:

```javascript
// The Problem
const a = 0.1;
const b = 0.2;
const sum = a + b; // 0.30000000000000004 (not 0.3!)
console.log(sum === 0.3); // false

// Solutions
// Method 1: Multiply, add, then divide
const fixedSum1 = (a * 10 + b * 10) / 10; // 0.3

// Method 2: Use toFixed() and convert back
const fixedSum2 = parseFloat((a + b).toFixed(10)); // 0.3

// Method 3: Use Number.EPSILON for comparison
function isEqual(x, y) {
    return Math.abs(x - y) < Number.EPSILON;
}
console.log(isEqual(0.1 + 0.2, 0.3)); // true

// Method 4: Use a library like decimal.js for precise calculations
// const Decimal = require('decimal.js');
// const result = new Decimal(0.1).plus(0.2); // "0.3"
```

### Why This Happens
```javascript
// Binary representation of decimals
console.log((0.1).toString(2)); // "0.0001100110011001100110011001100110011001100110011001101"
console.log((0.2).toString(2)); // "0.001100110011001100110011001100110011001100110011001101"

// Some decimal numbers cannot be represented exactly in binary
const examples = [
    0.1, 0.2, 0.3, 0.6, 0.7, 0.9
];

examples.forEach(num => {
    console.log(`${num} + 0 = ${num + 0}`);
});
```

## Type Coercion

JavaScript's automatic type conversion can lead to surprising results:

### String Concatenation vs Addition
```javascript
const a = 10;
const b = 20;
const c = '30';

// Left-to-right evaluation
const sum1 = a + b + c;  // "3030" (10 + 20 = 30, then "30" + "30")
const sum2 = c + b + a;  // "302010" (string concatenation from left)
const sum3 = a + (b + c); // "102030" (10 + "2030")
const sum4 = (a + b) + c; // "3030" (30 + "30")

// Force addition
const sum5 = a + b + Number(c); // 60
const sum6 = a + b + parseInt(c); // 60
const sum7 = a + b + +c; // 60 (unary plus operator)
```

### Implicit Type Conversion Rules
```javascript
// To String
console.log(String(123)); // "123"
console.log(123 + ''); // "123"
console.log(`${123}`); // "123"

// To Number
console.log(Number('123')); // 123
console.log(+'123'); // 123
console.log('123' * 1); // 123
console.log('123' - 0); // 123

// To Boolean
console.log(Boolean('hello')); // true
console.log(!!'hello'); // true

// Falsy values
const falsyValues = [false, 0, -0, 0n, '', null, undefined, NaN];
falsyValues.forEach(val => console.log(`${val} -> ${Boolean(val)}`));
```

### Comparison Coercion
```javascript
// == vs ===
console.log(1 == '1');   // true (coercion)
console.log(1 === '1');  // false (strict)

console.log(true == 1);  // true
console.log(true === 1); // false

console.log(null == undefined);  // true
console.log(null === undefined); // false

// Array coercion in comparisons
console.log([1] == 1);     // true
console.log([1,2] == '1,2'); // true
console.log([] == 0);      // true
console.log([] == false);  // true
```

## Special Number Values

### Division by Zero
```javascript
const a = 10;
const b = 0;

const result = a / b; // Infinity
console.log(result); // Infinity
console.log(typeof result); // "number"

// Negative infinity
console.log(-10 / 0); // -Infinity

// Zero divided by zero
console.log(0 / 0); // NaN

// Checking for infinity
console.log(Number.isFinite(result)); // false
console.log(Number.isFinite(10)); // true
```

### NaN (Not a Number)
```javascript
const notANumber = 0 / 0;
console.log(notANumber); // NaN
console.log(typeof notANumber); // "number"

// NaN is not equal to anything, including itself
console.log(NaN === NaN); // false
console.log(NaN == NaN);  // false

// Checking for NaN
console.log(Number.isNaN(NaN)); // true
console.log(Number.isNaN('hello')); // false (it's not NaN, it's a string)
console.log(isNaN('hello')); // true (legacy function converts first)

// Operations that result in NaN
console.log(Math.sqrt(-1)); // NaN
console.log(parseInt('hello')); // NaN
console.log('hello' * 2); // NaN
```

## Number System Conversions

```javascript
const n = 17;

// Convert to different bases
const decimal = n.toString(10);     // "17"
const hexadecimal = n.toString(16); // "11"
const octal = n.toString(8);        // "21"
const binary = n.toString(2);       // "10001"

console.log(`Decimal: ${decimal}`);
console.log(`Hex: ${hexadecimal}`);
console.log(`Octal: ${octal}`);
console.log(`Binary: ${binary}`);

// Convert from different bases back to decimal
console.log(parseInt('11', 16));    // 17
console.log(parseInt('21', 8));     // 17
console.log(parseInt('10001', 2));  // 17

// Large numbers
const large = 255;
console.log(large.toString(16)); // "ff"
console.log(large.toString(2));  // "11111111"
```

### Working with Different Number Formats
```javascript
// Scientific notation
const scientific1 = 1e6;    // 1,000,000
const scientific2 = 2.5e-4; // 0.00025

// Hexadecimal literals
const hex = 0xFF;    // 255
const hex2 = 0x10;   // 16

// Binary literals (ES6)
const binary1 = 0b1010; // 10
const binary2 = 0b1111; // 15

// Octal literals (ES6)
const octal1 = 0o17;  // 15
const octal2 = 0o777; // 511
```

## Literal vs Constructor Creation

Understanding the difference between primitive values and object wrappers:

```javascript
// Primitive vs Object
const a = new Number(17); // Number object
const b = 17;             // primitive number

console.log(typeof a); // "object"
console.log(typeof b); // "number"

console.log(a == b);  // true (value comparison with coercion)
console.log(a === b); // false (type and value comparison)

// Same behavior for all primitive types
const str1 = new String('hello'); // String object
const str2 = 'hello';             // primitive string

const bool1 = new Boolean(true);  // Boolean object
const bool2 = true;               // primitive boolean

// Object wrappers are always truthy
console.log(Boolean(new Boolean(false))); // true!
console.log(Boolean(false)); // false

// Performance implications
// Primitives are faster and use less memory
console.log(b.toExponential()); // "1.7e+1" (auto-boxing)
```

### Auto-boxing and Auto-unboxing
```javascript
// JavaScript automatically wraps primitives when needed
const str = 'hello';
console.log(str.length); // 5 (auto-boxing)
console.log(str.toUpperCase()); // "HELLO"

// The wrapper is discarded immediately
str.customProp = 'test';
console.log(str.customProp); // undefined

// Explicit conversion
const num = new Number(42);
console.log(+num); // 42 (auto-unboxing)
console.log(num.valueOf()); // 42 (explicit)
```

## Working with undefined

### Getting undefined Safely
```javascript
// Problem: undefined can be reassigned in old JavaScript
// var undefined = 2; // This was possible in old versions

// Solution: Use void 0
const unknown = void 0; // Always returns undefined
console.log(unknown); // undefined

// The void operator evaluates expression and returns undefined
console.log(void 1); // undefined
console.log(void 'hello'); // undefined
console.log(void (2 + 3)); // undefined

// Safe undefined comparison
function isUndefined(value) {
    return value === void 0;
}

// Also useful in one-liners
const result = condition ? value : void 0;
```

### Different Ways to Get undefined
```javascript
// Declared but not assigned
let notAssigned;

// Function with no return
function noReturn() {}

// Missing object properties
const obj = {};
const missing = obj.nonExistent;

// Array holes
const arr = [1, , 3];
const hole = arr[1];

// Function parameters
function test(param) {
    return param; // undefined if not passed
}

console.log([notAssigned, noReturn(), missing, hole, test()]);
// [undefined, undefined, undefined, undefined, undefined]
```

## Increment and Decrement Operators

Understanding pre-increment vs post-increment:

```javascript
let a = 1;

// Post-increment: return current value, then increment
console.log(a++); // 1 (returns 1, then a becomes 2)
console.log(a);   // 2

// Pre-increment: increment first, then return new value
a = 1; // reset
console.log(++a); // 2 (a becomes 2, then returns 2)
console.log(a);   // 2

// Same applies to decrement
let b = 5;
console.log(b--); // 5 (returns 5, then b becomes 4)
console.log(--b); // 3 (b becomes 3, then returns 3)
```

### Practical Examples
```javascript
// Common use in loops
const arr = [1, 2, 3, 4, 5];

// Post-increment typically used in loops
for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}

// Pre-increment can be more efficient (negligible in modern JS)
for (let i = 0; i < arr.length; ++i) {
    console.log(arr[i]);
}

// In expressions - be careful!
let x = 5;
let result1 = x++ * 2; // 10 (5 * 2, then x becomes 6)
let result2 = ++x * 2; // 14 (x becomes 7, then 7 * 2)
```

## Mathematical Operations

### Exponentiation
```javascript
// ES2016 exponentiation operator
const power1 = 3 ** 2;        // 9
const power2 = 2 ** 10;       // 1024
const power3 = 16 ** (1/2);   // 4 (square root)

// Equivalent to Math.pow()
const power4 = Math.pow(3, 2); // 9

// Works with negative numbers
const power5 = (-2) ** 3;     // -8
const power6 = (-2) ** 2;     // 4

// Precedence (right-associative)
console.log(2 ** 3 ** 2);     // 512 (2 ** (3 ** 2) = 2 ** 9)
console.log((2 ** 3) ** 2);   // 64
```

### Other Mathematical Operations
```javascript
// Modulo operator
console.log(17 % 5);  // 2
console.log(-17 % 5); // -2
console.log(17 % -5); // 2

// Bitwise operations
console.log(5 & 3);   // 1 (AND)
console.log(5 | 3);   // 7 (OR)
console.log(5 ^ 3);   // 6 (XOR)
console.log(~5);      // -6 (NOT)
console.log(5 << 1);  // 10 (left shift)
console.log(5 >> 1);  // 2 (right shift)
```

## Type Checking Methods

### typeof Operator
```javascript
console.log(typeof undefined); // "undefined"
console.log(typeof null);      // "object" (bug!)
console.log(typeof true);      // "boolean"
console.log(typeof 42);        // "number"
console.log(typeof 'hello');   // "string"
console.log(typeof Symbol());  // "symbol"
console.log(typeof 123n);      // "bigint"
console.log(typeof {});        // "object"
console.log(typeof []);        // "object"
console.log(typeof function(){}); // "function"
```

### More Precise Type Checking
```javascript
function getType(value) {
    return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType(null));        // "Null"
console.log(getType([]));          // "Array"
console.log(getType(new Date()));  // "Date"
console.log(getType(/regex/));     // "RegExp"
console.log(getType(new Map()));   // "Map"

// Utility functions
const isArray = Array.isArray;
const isInteger = Number.isInteger;
const isNaN = Number.isNaN;
const isFinite = Number.isFinite;

// Custom type checkers
const isString = val => typeof val === 'string';
const isNumber = val => typeof val === 'number' && !Number.isNaN(val);
const isObject = val => val !== null && typeof val === 'object' && !Array.isArray(val);
```

## Common Gotchas

### 1. Array Type Checking
```javascript
console.log(typeof []); // "object" (not "array"!)
console.log(Array.isArray([])); // true (correct way)
```

### 2. null vs undefined
```javascript
console.log(null == undefined);  // true
console.log(null === undefined); // false
console.log(typeof null);        // "object"
console.log(typeof undefined);   // "undefined"
```

### 3. NaN Comparisons
```javascript
console.log(NaN === NaN); // false
console.log(Object.is(NaN, NaN)); // true
console.log(Number.isNaN(NaN)); // true
```

### 4. String to Number Conversion
```javascript
console.log(+'123');     // 123
console.log(+'123abc');  // NaN
console.log(parseInt('123abc')); // 123
console.log(Number('123abc'));   // NaN
```

### 5. Boolean Coercion
```javascript
// Falsy values
const falsy = [false, 0, -0, 0n, '', null, undefined, NaN];
console.log(falsy.every(val => !val)); // true

// Everything else is truthy
console.log(Boolean('0'));     // true
console.log(Boolean('false')); // true
console.log(Boolean([]));      // true
console.log(Boolean({}));      // true
```

### Key Takeaways

1. **Use strict equality (`===`)** to avoid type coercion surprises
2. **Be aware of floating point precision** issues with decimal arithmetic
3. **Use `Array.isArray()`** instead of `typeof` for arrays
4. **Use `Number.isNaN()`** instead of global `isNaN()`
5. **Use `void 0`** for safe undefined comparisons
6. **Understand operator precedence** especially with `++` and `--`
7. **Prefer primitives over object wrappers** for better performance
8. **Use proper type checking methods** for reliable code 