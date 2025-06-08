# JavaScript Loops Guide

## Table of Contents
- [Loop Control Statements](#loop-control-statements)
- [Labels in Loops](#labels-in-loops)
- [For...in vs For...of](#forin-vs-forof)
- [Making Objects Iterable](#making-objects-iterable)
- [Types of Loops](#types-of-loops)
- [Array Iteration Methods](#array-iteration-methods)
- [Loop Performance](#loop-performance)
- [Common Patterns](#common-patterns)
- [Best Practices](#best-practices)

## Loop Control Statements

Understanding `continue`, `break`, and `return` is crucial for controlling loop execution.

### Continue Statement
Based on your example:

```javascript
for (var i = 0; i < 5; i++) {
    if (i === 3) {
        continue; // Skip this iteration and move to next => 0, 1, 2, 4
    }
    console.log(i); // Prints: 0, 1, 2, 4
}
```

### Break Statement
```javascript
for (var i = 0; i < 5; i++) {
    if (i === 3) {
        break; // Stop the loop completely => 0, 1, 2
    }
    console.log(i); // Prints: 0, 1, 2
}
```

### Return Statement in Functions
```javascript
function findFirstEven(numbers) {
    for (var i = 0; i < numbers.length; i++) {
        if (numbers[i] % 2 === 0) {
            return numbers[i]; // Exit function immediately
        }
        console.log('Checking:', numbers[i]);
    }
    return null; // No even number found
}

console.log(findFirstEven([1, 3, 4, 5])); // Output: Checking: 1, Checking: 3, then returns 4
```

### Control Statements Comparison
| Statement | Effect | Scope | Use Case |
|-----------|--------|-------|----------|
| `continue` | Skip current iteration | Current loop only | Skip invalid/unwanted items |
| `break` | Exit loop completely | Current loop only | Stop when condition met |
| `return` | Exit function | Entire function | Return result early |

### Nested Loop Control
```javascript
function findInMatrix(matrix, target) {
    for (let i = 0; i < matrix.length; i++) {
        for (let j = 0; j < matrix[i].length; j++) {
            if (matrix[i][j] === target) {
                console.log(`Found ${target} at [${i}][${j}]`);
                return [i, j]; // Exits entire function
            }
            
            if (matrix[i][j] < 0) {
                continue; // Skip negative numbers, continue inner loop
            }
            
            if (matrix[i][j] > 100) {
                break; // Exit inner loop only, continue outer loop
            }
        }
    }
    return null;
}

const matrix = [
    [1, 5, 10],
    [15, 200, 25], // break triggered at 200, but outer loop continues
    [30, 35, 40]
];
```

## Labels in Loops

As mentioned in your comment, labels are rarely used but can be helpful with nested loops.

### Basic Label Syntax
Based on your reference to MDN:

```javascript
outerLoop: for (let i = 0; i < 3; i++) {
    innerLoop: for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            break outerLoop; // Breaks out of the outer loop
        }
        console.log(`i: ${i}, j: ${j}`);
    }
}
// Output: i: 0, j: 0, i: 0, j: 1, i: 0, j: 2, i: 1, j: 0
// Then breaks completely
```

### Continue with Labels
```javascript
outerLoop: for (let i = 0; i < 3; i++) {
    innerLoop: for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            continue outerLoop; // Continue from next outer loop iteration
        }
        console.log(`i: ${i}, j: ${j}`);
    }
}
// Skips remaining inner iterations when i=1, j=1
```

### Practical Label Example
```javascript
function findFirstDuplicate(arrays) {
    searchArrays: for (let i = 0; i < arrays.length; i++) {
        checkElements: for (let j = 0; j < arrays[i].length; j++) {
            // Check against all previous arrays
            for (let k = 0; k < i; k++) {
                if (arrays[k].includes(arrays[i][j])) {
                    console.log(`Found duplicate: ${arrays[i][j]}`);
                    break searchArrays; // Exit all nested loops
                }
            }
        }
    }
}

findFirstDuplicate([[1, 2], [3, 4], [2, 5]]); // Found duplicate: 2
```

### When to Use Labels
```javascript
// ❌ Usually not needed - can be refactored
outer: for (let i = 0; i < 10; i++) {
    for (let j = 0; j < 10; j++) {
        if (someCondition) break outer;
    }
}

// ✅ Better - extract to function
function processMatrix() {
    for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 10; j++) {
            if (someCondition) return; // Cleaner exit
        }
    }
}
```

## For...in vs For...of

Understanding the difference is crucial for working with objects and arrays.

### For...in - Iterates over Keys
Based on your example:

```javascript
var arr = [1, 2, 3];
var i;

for (i in arr) {
    console.log(i); // 0, 1, 2 (indices/keys)
    console.log(arr[i]); // 1, 2, 3 (values)
}

// For objects - iterates over property names
const obj = { a: 'val1', b: 'val2', c: 'val3' };
for (const key in obj) {
    console.log(key); // 'a', 'b', 'c'
    console.log(obj[key]); // 'val1', 'val2', 'val3'
}
```

### For...of - Iterates over Values
```javascript
var arr = [1, 2, 3];
var i;

for (i of arr) {
    console.log(i); // 1, 2, 3 (values directly)
}

// Works with any iterable
for (const char of 'hello') {
    console.log(char); // 'h', 'e', 'l', 'l', 'o'
}

for (const item of new Set([1, 2, 3])) {
    console.log(item); // 1, 2, 3
}
```

### The Object Problem
As you noted, objects are not iterable by default:

```javascript
const obj = { a: 'val1', b: 'val2', c: 'val3' };

// ❌ This throws TypeError: obj is not iterable
// for (const value of obj) { }

// ✅ Solutions:
for (const key of Object.keys(obj)) {
    console.log(obj[key]); // 'val1', 'val2', 'val3'
}

for (const value of Object.values(obj)) {
    console.log(value); // 'val1', 'val2', 'val3'
}

for (const [key, value] of Object.entries(obj)) {
    console.log(key, value); // 'a' 'val1', 'b' 'val2', 'c' 'val3'
}
```

### for...in vs for...of Detailed Comparison
| Feature | for...in | for...of |
|---------|----------|----------|
| **Iterates over** | Property names (keys) | Property values |
| **Works with** | Objects, arrays (enumerable properties) | Iterables (arrays, strings, Maps, Sets) |
| **Array behavior** | Returns indices as strings | Returns actual values |
| **Inherited properties** | Yes (unless hasOwnProperty check) | No |
| **Order** | Not guaranteed in all environments | Insertion order |
| **Performance** | Slower (property lookup) | Faster (iterator protocol) |
| **Prototype chain** | Traverses prototype | Only own iterable |

### Array Enumeration Gotchas
```javascript
const arr = [1, 2, 3];
arr.customProperty = 'custom';
arr.length; // 3

// for...in includes custom properties!
for (const key in arr) {
    console.log(key); // '0', '1', '2', 'customProperty'
}

// for...of only iterates array elements
for (const value of arr) {
    console.log(value); // 1, 2, 3
}

// Safe for...in with arrays
for (const key in arr) {
    if (arr.hasOwnProperty(key) && !isNaN(key)) {
        console.log(arr[key]); // 1, 2, 3
    }
}
```

## Making Objects Iterable

Based on your example of making an object iterable:

### Custom Iterator Implementation
```javascript
const obj = {
    a: 'val1',
    b: 'val2',
    c: 'val3',
    [Symbol.iterator]: function() {
        const props = Object.keys(this);
        let index = 0;
        
        return {
            next: () => {
                return {
                    value: this[props[index++]],
                    done: index > props.length
                };
            }
        };
    }
};

for (const value of obj) {
    console.log(value); // 'val1', 'val2', 'val3'
}
```

### Enhanced Iterable Object
```javascript
class IterableObject {
    constructor(data) {
        Object.assign(this, data);
    }
    
    // Default iteration - values
    *[Symbol.iterator]() {
        for (const key of Object.keys(this)) {
            if (typeof this[key] !== 'function') {
                yield this[key];
            }
        }
    }
    
    // Custom iteration methods
    *keys() {
        for (const key of Object.keys(this)) {
            if (typeof this[key] !== 'function') {
                yield key;
            }
        }
    }
    
    *entries() {
        for (const key of Object.keys(this)) {
            if (typeof this[key] !== 'function') {
                yield [key, this[key]];
            }
        }
    }
}

const iterableObj = new IterableObject({ a: 1, b: 2, c: 3 });

for (const value of iterableObj) {
    console.log(value); // 1, 2, 3
}

for (const key of iterableObj.keys()) {
    console.log(key); // 'a', 'b', 'c'
}

for (const [key, value] of iterableObj.entries()) {
    console.log(key, value); // 'a' 1, 'b' 2, 'c' 3
}
```

### Multiple Iteration Strategies
```javascript
const multiIterable = {
    data: [1, 2, 3, 4, 5],
    
    // Default iterator - all values
    [Symbol.iterator]() {
        return this.data[Symbol.iterator]();
    },
    
    // Even numbers only
    *evens() {
        for (const num of this.data) {
            if (num % 2 === 0) yield num;
        }
    },
    
    // Odd numbers only
    *odds() {
        for (const num of this.data) {
            if (num % 2 === 1) yield num;
        }
    }
};

console.log([...multiIterable]); // [1, 2, 3, 4, 5]
console.log([...multiIterable.evens()]); // [2, 4]
console.log([...multiIterable.odds()]); // [1, 3, 5]
```

## Types of Loops

### For Loop Variants
```javascript
// Classic for loop
for (let i = 0; i < 5; i++) {
    console.log(i);
}

// Reverse iteration
for (let i = 4; i >= 0; i--) {
    console.log(i); // 4, 3, 2, 1, 0
}

// Step by 2
for (let i = 0; i < 10; i += 2) {
    console.log(i); // 0, 2, 4, 6, 8
}

// Multiple variables
for (let i = 0, j = 10; i < 5; i++, j--) {
    console.log(i, j); // 0 10, 1 9, 2 8, 3 7, 4 6
}

// Complex conditions
for (let i = 0; i < arr.length && arr[i] !== null; i++) {
    console.log(arr[i]);
}
```

### While Loops
```javascript
// Basic while loop
let i = 0;
while (i < 5) {
    console.log(i);
    i++;
}

// Condition checked first
let arr = [1, 2, 3];
let index = 0;
while (index < arr.length) {
    console.log(arr[index]);
    index++;
}

// Infinite loop protection
let attempts = 0;
while (someCondition && attempts < 100) {
    // Do something
    attempts++;
}
```

### Do-While Loops
```javascript
// Execute at least once
let userInput;
do {
    userInput = prompt('Enter a number greater than 10:');
} while (userInput <= 10);

// Menu system example
let choice;
do {
    choice = showMenu();
    processChoice(choice);
} while (choice !== 'quit');
```

### Loop Comparison
| Loop Type | Use Case | Execution | Best For |
|-----------|----------|-----------|----------|
| `for` | Known iterations | 0 or more times | Arrays, counters |
| `while` | Unknown iterations | 0 or more times | Conditions, input validation |
| `do-while` | At least once | 1 or more times | Menus, user input |
| `for...in` | Object properties | Over enumerable properties | Object iteration |
| `for...of` | Iterable values | Over iterable values | Arrays, strings, collections |

## Array Iteration Methods

### forEach and Friends
```javascript
const numbers = [1, 2, 3, 4, 5];

// forEach - no return value
numbers.forEach((num, index) => {
    console.log(`${index}: ${num}`);
});

// map - returns new array
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter - returns filtered array
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4]

// find - returns first match
const found = numbers.find(num => num > 3);
console.log(found); // 4

// some - returns boolean
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true

// every - returns boolean
const allPositive = numbers.every(num => num > 0);
console.log(allPositive); // true
```

### Reducing Arrays
```javascript
const numbers = [1, 2, 3, 4, 5];

// reduce - accumulate to single value
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

// Complex reduce example
const people = [
    { name: 'John', age: 25 },
    { name: 'Jane', age: 30 },
    { name: 'Bob', age: 25 }
];

const ageGroups = people.reduce((groups, person) => {
    const age = person.age;
    if (!groups[age]) {
        groups[age] = [];
    }
    groups[age].push(person);
    return groups;
}, {});
console.log(ageGroups); // { 25: [{name: 'John'...}, {name: 'Bob'...}], 30: [{name: 'Jane'...}] }
```

### Method Chaining
```javascript
const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const result = data
    .filter(num => num % 2 === 0)    // [2, 4, 6, 8, 10]
    .map(num => num * num)           // [4, 16, 36, 64, 100]
    .reduce((sum, num) => sum + num, 0); // 220

console.log(result); // 220

// With early termination using some/every
const hasLargeSquare = data
    .filter(num => num % 2 === 0)
    .map(num => num * num)
    .some(square => square > 50); // true (64 and 100 are > 50)
```

## Loop Performance

### Performance Considerations
```javascript
const largeArray = new Array(1000000).fill().map((_, i) => i);

// ❌ Slower - accessing length property each iteration
console.time('for-with-length');
for (let i = 0; i < largeArray.length; i++) {
    // Do something
}
console.timeEnd('for-with-length');

// ✅ Faster - cache length
console.time('for-cached-length');
for (let i = 0, len = largeArray.length; i < len; i++) {
    // Do something
}
console.timeEnd('for-cached-length');

// ✅ Often fastest for arrays
console.time('for-of');
for (const item of largeArray) {
    // Do something
}
console.timeEnd('for-of');

// Performance varies by JS engine and use case
console.time('forEach');
largeArray.forEach(item => {
    // Do something
});
console.timeEnd('forEach');
```

### Loop Optimization Techniques
```javascript
// 1. Minimize work inside loops
// ❌ Inefficient
for (let i = 0; i < array.length; i++) {
    const expensiveComputation = doExpensiveWork();
    array[i] = array[i] * expensiveComputation;
}

// ✅ More efficient
const expensiveComputation = doExpensiveWork();
for (let i = 0; i < array.length; i++) {
    array[i] = array[i] * expensiveComputation;
}

// 2. Use appropriate data structures
// ❌ Slow for large arrays
const items = [];
for (let i = 0; i < 10000; i++) {
    if (items.includes(someValue)) { // O(n) operation in loop!
        // do something
    }
}

// ✅ Use Set for membership testing
const itemSet = new Set();
for (let i = 0; i < 10000; i++) {
    if (itemSet.has(someValue)) { // O(1) operation
        // do something
    }
}
```

### Loop vs Array Methods Performance
```javascript
const data = new Array(100000).fill().map(() => Math.random());

// Benchmark different approaches
function benchmarkSum() {
    // Traditional for loop
    console.time('for-loop');
    let sum1 = 0;
    for (let i = 0; i < data.length; i++) {
        sum1 += data[i];
    }
    console.timeEnd('for-loop');
    
    // for...of loop
    console.time('for-of');
    let sum2 = 0;
    for (const value of data) {
        sum2 += value;
    }
    console.timeEnd('for-of');
    
    // reduce method
    console.time('reduce');
    const sum3 = data.reduce((acc, val) => acc + val, 0);
    console.timeEnd('reduce');
    
    console.log('All sums equal:', sum1 === sum2 && sum2 === sum3);
}
```

## Common Patterns

### Loop Patterns
```javascript
// 1. Finding items
function findPattern(array, predicate) {
    for (let i = 0; i < array.length; i++) {
        if (predicate(array[i], i)) {
            return { item: array[i], index: i };
        }
    }
    return null;
}

// 2. Grouping items
function groupBy(array, keyFn) {
    const groups = {};
    for (const item of array) {
        const key = keyFn(item);
        if (!groups[key]) {
            groups[key] = [];
        }
        groups[key].push(item);
    }
    return groups;
}

// 3. Processing in batches
function processBatch(array, batchSize, processor) {
    for (let i = 0; i < array.length; i += batchSize) {
        const batch = array.slice(i, i + batchSize);
        processor(batch);
    }
}

// 4. Early termination patterns
function processUntilCondition(array, processor, stopCondition) {
    for (let i = 0; i < array.length; i++) {
        const result = processor(array[i]);
        if (stopCondition(result)) {
            return result;
        }
    }
    return null;
}
```

### Nested Loop Patterns
```javascript
// 1. Matrix operations
function transposeMatrix(matrix) {
    const result = [];
    for (let i = 0; i < matrix[0].length; i++) {
        result[i] = [];
        for (let j = 0; j < matrix.length; j++) {
            result[i][j] = matrix[j][i];
        }
    }
    return result;
}

// 2. Combinations
function generatePairs(array) {
    const pairs = [];
    for (let i = 0; i < array.length; i++) {
        for (let j = i + 1; j < array.length; j++) {
            pairs.push([array[i], array[j]]);
        }
    }
    return pairs;
}

// 3. Avoiding nested loops with preprocessing
// ❌ O(n²) approach
function findCommonElements(arr1, arr2) {
    const common = [];
    for (const item1 of arr1) {
        for (const item2 of arr2) {
            if (item1 === item2) {
                common.push(item1);
            }
        }
    }
    return common;
}

// ✅ O(n) approach
function findCommonElementsOptimized(arr1, arr2) {
    const set2 = new Set(arr2);
    const common = [];
    for (const item of arr1) {
        if (set2.has(item)) {
            common.push(item);
        }
    }
    return common;
}
```

## Best Practices

### 1. Choose the Right Loop Type
```javascript
// ✅ Use for...of for values
for (const item of array) {
    console.log(item);
}

// ✅ Use for...in for object properties
for (const key in object) {
    if (object.hasOwnProperty(key)) {
        console.log(key, object[key]);
    }
}

// ✅ Use traditional for when you need index control
for (let i = 0; i < array.length; i++) {
    if (shouldSkip(i)) continue;
    process(array[i], i);
}

// ✅ Use array methods for functional approach
const results = array
    .filter(isValid)
    .map(transform)
    .reduce(accumulate, initialValue);
```

### 2. Avoid Common Pitfalls
```javascript
// ❌ Variable leaking with var
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // Prints 3, 3, 3
}

// ✅ Use let for block scope
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // Prints 0, 1, 2
}

// ❌ Modifying array while iterating
const numbers = [1, 2, 3, 4, 5];
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 === 0) {
        numbers.splice(i, 1); // This shifts indices!
        i--; // Need to adjust index
    }
}

// ✅ Filter instead
const odds = numbers.filter(num => num % 2 !== 0);

// ✅ Or iterate backwards
for (let i = numbers.length - 1; i >= 0; i--) {
    if (numbers[i] % 2 === 0) {
        numbers.splice(i, 1);
    }
}
```

### 3. Handle Edge Cases
```javascript
function safeIteration(data) {
    // Check if data exists and is iterable
    if (!data || typeof data[Symbol.iterator] !== 'function') {
        console.warn('Data is not iterable');
        return [];
    }
    
    const results = [];
    try {
        for (const item of data) {
            // Handle null/undefined items
            if (item != null) {
                results.push(process(item));
            }
        }
    } catch (error) {
        console.error('Error during iteration:', error);
        return [];
    }
    
    return results;
}

function process(item) {
    // Safe processing
    return item.toString().toUpperCase();
}
```

### 4. Performance-Aware Looping
```javascript
// ✅ Cache expensive computations
const expensiveValue = computeExpensiveValue();
for (const item of array) {
    useExpensiveValue(item, expensiveValue);
}

// ✅ Use appropriate break/continue
for (const item of largeArray) {
    if (!isRelevant(item)) continue; // Skip early
    
    const result = expensiveProcess(item);
    if (result.isComplete) break; // Stop when done
}

// ✅ Consider lazy evaluation for large datasets
function* processLargeDataset(data) {
    for (const item of data) {
        yield expensiveProcess(item);
    }
}

// Only process what you need
const processor = processLargeDataset(hugeArray);
const firstFive = [];
for (const result of processor) {
    firstFive.push(result);
    if (firstFive.length === 5) break;
}
```

### Key Takeaways

1. **Use `continue` to skip iterations**, `break` to exit loops, `return` to exit functions
2. **Labels are rarely needed** - prefer function extraction for complex nested loops
3. **`for...in` iterates over keys**, `for...of` iterates over values
4. **Objects are not iterable by default** - implement `Symbol.iterator` or use Object methods
5. **Choose the right loop type** for readability and performance
6. **Cache expensive operations** outside loops when possible
7. **Be careful with array modification** during iteration
8. **Use `let` instead of `var`** to avoid closure issues in loops 