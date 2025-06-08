# JavaScript Arrays Guide

## Table of Contents
- [Array Creation](#array-creation)
- [Array Comparison](#array-comparison)
- [Array Methods](#array-methods)
  - [Mutating Methods](#mutating-methods)
  - [Non-Mutating Methods](#non-mutating-methods)
  - [Search Methods](#search-methods)
  - [Iteration Methods](#iteration-methods)
- [Array Properties](#array-properties)
- [Common Use Cases](#common-use-cases)

## Array Creation

### Constructor Methods
```javascript
// Using Array constructor
const arr1 = new Array(2, 3);    // [2, 3]
const arr2 = new Array(2);       // [undefined, undefined] - creates array with length 2

// Using Array.of() - more predictable than Array constructor
const arr3 = Array.of();         // []
const arr4 = Array.of(4);        // [4]
const arr5 = Array.of(1, 2);     // [1, 2]
const arr6 = Array.of([1, 2]);   // [[1, 2]]

// Using Array.from() - creates array from array-like or iterable objects
const arr7 = Array.from([1, 2, 3]);           // [1, 2, 3]
const arr8 = Array.from('test');              // ['t', 'e', 's', 't']
const arr9 = Array.from({ length: 3 });       // [undefined, undefined, undefined]
const arr10 = Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]
```

### Array Literal
```javascript
const arr = [1, 2, 3];
const emptyArr = [];
```

## Array Comparison

Arrays in JavaScript are objects, so they can't be compared using `==` or `===` operators directly:

```javascript
const a = [1];
const b = [1];
console.log(a == b);   // false
console.log(a === b);  // false
```

To compare arrays, you need to compare their elements:
```javascript
// Method 1: Using JSON.stringify
const areEqual = JSON.stringify(a) === JSON.stringify(b);

// Method 2: Using every() method
const areEqual2 = a.length === b.length && a.every((val, index) => val === b[index]);
```

## Array Methods

### Mutating Methods
These methods modify the original array:

- `push()`: Adds elements to the end
- `pop()`: Removes the last element
- `shift()`: Removes the first element
- `unshift()`: Adds elements to the beginning
- `splice()`: Adds/removes elements at specified position
- `reverse()`: Reverses array order
- `sort()`: Sorts array elements
- `fill()`: Fills array with static value
```javascript
const arr = [1, 2, 3, 4, 5];
arr.fill(1, 3, 4);  // [1, 2, 3, 1, 5]
```

### Non-Mutating Methods
These methods return a new array without modifying the original:

- `concat()`: Merges arrays
- `slice()`: Extracts portion of array
- `join()`: Joins array elements into string
- `flat()`: Flattens nested arrays
```javascript
const arr = [1, 2, 3, [4, [5]]];
arr.flat();     // [1, 2, 3, 4, [5]]
arr.flat(2);    // [1, 2, 3, 4, 5]
```

- `flatMap()`: Maps and flattens result
```javascript
const arr = [1, 2, 3];
const result = arr.flatMap(x => [x * 2]);  // [2, 4, 6]
```

### Search Methods
- `indexOf()`: Returns first index of element
- `lastIndexOf()`: Returns last index of element
- `includes()`: Checks if array includes element
- `find()`: Returns first element that satisfies condition
- `findIndex()`: Returns index of first element that satisfies condition

### Iteration Methods
- `forEach()`: Executes function for each element
- `map()`: Creates new array with results of function
- `filter()`: Creates new array with elements that pass test
- `reduce()`: Reduces array to single value
- `reduceRight()`: Reduces array from right to left
- `every()`: Tests if all elements pass test
- `some()`: Tests if any element passes test

## Array Properties

- `length`: Number of elements in array
- `constructor`: Returns array's constructor function
- `prototype`: Allows adding properties to all array objects

## Common Use Cases

### Removing Duplicates
```javascript
const unique = [...new Set(array)];
// or
const unique = Array.from(new Set(array));
```

### Array Destructuring
```javascript
const [first, second, ...rest] = [1, 2, 3, 4, 5];
```

### Array Spread
```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = [...arr1, ...arr2];  // [1, 2, 3, 4]
```

### Array-like Objects
```javascript
// Convert arguments to array
function example() {
    const args = Array.from(arguments);
}

// Convert NodeList to array
const elements = Array.from(document.querySelectorAll('div'));
```

### Performance Tips
1. Use `for...of` or `forEach` for simple iterations
2. Use `map` when you need to transform elements
3. Use `filter` when you need to select elements
4. Use `reduce` when you need to accumulate values
5. Avoid using `delete` operator on arrays (use `splice` instead)
6. Pre-allocate arrays when possible for better performance 