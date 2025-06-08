# JavaScript Maps and Sets Guide

## Table of Contents
- [Set Fundamentals](#set-fundamentals)
- [Set Methods and Properties](#set-methods-and-properties)
- [Set Iteration and Behavior](#set-iteration-and-behavior)
- [Set Memory Management](#set-memory-management)
- [WeakSet](#weakset)
- [Map Fundamentals](#map-fundamentals)
- [Map Methods and Properties](#map-methods-and-properties)
- [Map vs Object](#map-vs-object)
- [WeakMap](#weakmap)
- [Practical Use Cases](#practical-use-cases)
- [Performance Considerations](#performance-considerations)
- [Best Practices](#best-practices)

## Set Fundamentals

A Set is a collection of **unique values** - no duplicates are allowed.

### Basic Set Creation and Usage
Based on your examples:

```javascript
// Creating a Set
const set = new Set();
set.add(5);
set.add('5');
console.log(set); // Set { 5, '5' }

// Set uses Object.is() comparison under the hood
// 5 === '5' is false, so both values are stored
console.log(5 === '5'); // false
console.log(Object.is(5, '5')); // false
```

### Set Value Equality Rules
```javascript
const set = new Set();

// Primitive values
set.add(1);
set.add(1); // Duplicate - ignored
set.add('1'); // Different type - added
console.log(set); // Set { 1, '1' }

// Special number values
set.add(NaN);
set.add(NaN); // NaN === NaN is false, but Set treats them as equal
console.log(set.has(NaN)); // true
console.log(set.size); // 3 (1, '1', NaN)

// Zero handling
set.add(0);
set.add(-0); // +0 and -0 are treated as the same value
console.log(set.size); // 4 (not 5)

// Objects - compared by reference
const obj1 = { a: 1 };
const obj2 = { a: 1 };
set.add(obj1);
set.add(obj2); // Different objects - both added
set.add(obj1); // Same reference - ignored
console.log(set.size); // 6
```

### Set from Iterable
```javascript
// From array
const set1 = new Set(['a', 'b', 'c']);
console.log(set1); // Set { 'a', 'b', 'c' }

// From string
const set2 = new Set('hello');
console.log(set2); // Set { 'h', 'e', 'l', 'o' } - note: only one 'l'

// From another Set
const set3 = new Set(set1);
console.log(set3); // Set { 'a', 'b', 'c' }

// From any iterable
const set4 = new Set(new Map([['key1', 'val1'], ['key2', 'val2']]).keys());
console.log(set4); // Set { 'key1', 'key2' }
```

## Set Methods and Properties

### Core Methods
```javascript
const set = new Set();

// Adding values
set.add('hello');
set.add('world');
set.add('hello'); // Ignored - duplicate
console.log(set.size); // 2

// Checking existence
console.log(set.has('hello')); // true
console.log(set.has('goodbye')); // false

// Deleting values
console.log(set.delete('hello')); // true - successfully deleted
console.log(set.delete('goodbye')); // false - wasn't there
console.log(set.size); // 1

// Clearing all values
set.clear();
console.log(set.size); // 0
```

### Method Chaining
```javascript
const set = new Set();

// add() returns the Set instance, allowing chaining
set.add(1).add(2).add(3).add(2); // Last add is ignored
console.log(set); // Set { 1, 2, 3 }

// Useful for initialization
const chainedSet = new Set()
    .add('red')
    .add('green')
    .add('blue');
```

### Size Property
```javascript
const set = new Set([1, 2, 3, 3, 3]);
console.log(set.size); // 3 (not 5)

// size is read-only
// set.size = 10; // This doesn't work
console.log(set.size); // Still 3

// Dynamically changes as Set is modified
set.add(4);
console.log(set.size); // 4
set.delete(1);
console.log(set.size); // 3
```

## Set Iteration and Behavior

### forEach Method
Based on your example with Russian comments explanation:

```javascript
const set1 = new Set(['a', 'b']);
set1.forEach((value, key, ownSet) => {
    console.log(`${key} ${value}`); // a a, b b
    console.log(set === ownSet); // true, true
});

// The key parameter exists for consistency with Map.forEach
// In Sets, key and value are the same because Sets don't have keys
// This allows the same callback to work with both Maps and Sets
```

### Iterator Methods
As noted in your comments, keys(), values(), and entries() return essentially the same iterator:

```javascript
const set3 = new Set([1, 2, 3]);

console.log(set3.keys());    // [Set Iterator] { 1, 2, 3 }
console.log(set3.values());  // [Set Iterator] { 1, 2, 3 }
console.log(set3.entries()); // [Set Entries] { [1, 1], [2, 2], [3, 3] }

// They're the same because Sets don't have separate keys
console.log(set3.keys === set3.values); // true (same method reference)

// Default iterator is values()
console.log(set3[Symbol.iterator] === set3.values); // true
```

### Iteration Patterns
```javascript
const set = new Set(['red', 'green', 'blue']);

// for...of (most common)
for (const value of set) {
    console.log(value); // red, green, blue
}

// Using entries() to get [value, value] pairs
for (const [key, value] of set.entries()) {
    console.log(key, value); // red red, green green, blue blue
}

// Converting to array
const array = [...set];
console.log(array); // ['red', 'green', 'blue']

// Array.from()
const array2 = Array.from(set);
console.log(array2); // ['red', 'green', 'blue']
```

### Duplicate Removal
As shown in your concise example:

```javascript
// Very concise way to remove duplicates
const set2 = new Set([1, 2, 3, 3, 3, 3, 3, 4, 5]);
const arr = [...set2];
console.log(arr); // [1, 2, 3, 4, 5]

// Works with any array
function removeDuplicates(array) {
    return [...new Set(array)];
}

console.log(removeDuplicates([1, 1, 2, 2, 3])); // [1, 2, 3]
console.log(removeDuplicates(['a', 'b', 'a', 'c'])); // ['a', 'b', 'c']

// Preserves order (first occurrence)
console.log(removeDuplicates([3, 1, 2, 1, 3])); // [3, 1, 2]
```

## Set Memory Management

### Object References in Sets
Based on your detailed memory explanation:

```javascript
// Interesting memory behavior...
const set4 = new Set();
let item = { a: 'b' };
set4.add(item); // We're not adding the content, we're adding the reference
console.log(set4.size); // 1

item = null; // After we 'kill' our item element, the set size still remains 1
console.log(set4.size); // 1
// After this we have no connection to item

item = [...set4][0]; // But we can still retrieve it from the set...despite having 'killed' it
console.log(item); // { a: 'b' }
```

### Memory Management Problems
```javascript
function demonstrateMemoryIssue() {
    const set = new Set();
    
    // Create many objects and add them to the set
    for (let i = 0; i < 1000; i++) {
        const obj = { id: i, data: new Array(1000).fill(i) };
        set.add(obj);
    }
    
    console.log(set.size); // 1000
    
    // Even if we lose references to objects, they remain in the set
    // The set keeps them alive, potentially causing memory leaks
    
    return set; // Objects stay in memory as long as set exists
}

const memoryHeavySet = demonstrateMemoryIssue();
// All 1000 objects are still in memory, even though we have no direct references
```

### Manual Cleanup Required
```javascript
class UserManager {
    constructor() {
        this.activeUsers = new Set();
        this.premiumUsers = new Set();
        this.moderators = new Set();
    }
    
    addUser(user) {
        this.activeUsers.add(user);
        if (user.isPremium) this.premiumUsers.add(user);
        if (user.isModerator) this.moderators.add(user);
    }
    
    removeUser(user) {
        // Problem: Need to manually check and remove from all sets
        this.activeUsers.delete(user);
        this.premiumUsers.delete(user);
        this.moderators.delete(user);
        // If we forget any set, we have a memory leak!
    }
}

// This problem is solved by WeakSet
```

## WeakSet

WeakSet solves the memory management problems of regular Sets.

### WeakSet Characteristics
Based on your explanation:

```javascript
// WeakSet can only store objects: {}, [], () => {}, function() {}
// We can only store reference content in WeakSet
// For primitive values like 1, no references point to them (different memory management)

const weakset = new WeakSet();
let key = {}; // We create an element, let's say its memory address is 123456
weakset.add(key); // Add it to WeakSet
console.log(weakset.has(key)); // true - it's there

key = null; // After we 'kill' it here, we change its memory value...say to 32142
// The trick is we can't check if it's in memory or not! Because our memory address changed
// We can only trust the specification :)
```

### WeakSet Methods (Limited)
```javascript
const weakSet = new WeakSet();

// Only three methods available
const obj1 = { id: 1 };
const obj2 = { id: 2 };

// add - adds object reference
weakSet.add(obj1);
weakSet.add(obj2);

// has - checks if object exists
console.log(weakSet.has(obj1)); // true
console.log(weakSet.has(obj2)); // true

// delete - removes object reference
console.log(weakSet.delete(obj1)); // true
console.log(weakSet.has(obj1)); // false

// No size property!
// console.log(weakSet.size); // undefined
```

### Why No Size Property?
As explained in your comments:

```javascript
// WeakSet/WeakMap don't have a size method
// This is related to how garbage collection works in JS
// We can't be sure that when we use .size, all references to objects will be removed
// Therefore .size could theoretically show different values for the same set

const weakSet = new WeakSet();
let obj = { data: 'test' };
weakSet.add(obj);

// At this point, size would be 1 if it existed
// But garbage collection is unpredictable

obj = null; // Reference removed

// Now size could be 0 or 1 depending on when GC runs
// That's why size property doesn't exist - it would be unreliable
```

### WeakSet Use Cases
```javascript
// 1. Tracking object membership without preventing GC
class PrivacyManager {
    constructor() {
        this.privateObjects = new WeakSet();
    }
    
    markPrivate(obj) {
        this.privateObjects.add(obj);
    }
    
    isPrivate(obj) {
        return this.privateObjects.has(obj);
    }
    
    // No need for cleanup - objects are automatically removed when GC'd
}

// 2. Preventing circular references in recursive functions
function processTree(node, visited = new WeakSet()) {
    if (visited.has(node)) {
        return; // Prevent infinite recursion
    }
    
    visited.add(node);
    
    if (node.children) {
        node.children.forEach(child => processTree(child, visited));
    }
    
    // No need to clean up visited set
}
```

### WeakSet Limitations
```javascript
// ❌ Can't store primitives
const weakSet = new WeakSet();
// weakSet.add(1); // TypeError: Invalid value used in weak set
// weakSet.add('string'); // TypeError: Invalid value used in weak set
// weakSet.add(true); // TypeError: Invalid value used in weak set

// ❌ Not iterable
// for (const item of weakSet) {} // TypeError: weakSet is not iterable
// [...weakSet] // TypeError: weakSet is not iterable

// ❌ No size or clear methods
// console.log(weakSet.size); // undefined
// weakSet.clear(); // TypeError: weakSet.clear is not a function

// ✅ Only objects
const obj = {};
const arr = [];
const func = function() {};
const arrow = () => {};

weakSet.add(obj);
weakSet.add(arr);
weakSet.add(func);
weakSet.add(arrow); // All valid
```

## Map Fundamentals

Map is an associative array - a collection of key-value pairs where keys can be any type.

### Basic Map Usage
Based on your examples:

```javascript
// Creating a Map
const map = new Map();
const john = {};

// Keys can be any type, not just strings
map.set(john, 24); // Object as key
map.set('name', 'John'); // String as key
map.set(1, 'number key'); // Number as key
map.set(true, 'boolean key'); // Boolean as key

console.log(map.get(john)); // 24
console.log(map.get('name')); // 'John'
```

### Map Constructor
As you noted, Maps can be created with initial data:

```javascript
// Map can be created with parameters passed directly to constructor
const map1 = new Map([['name', 'Jon'], ['age', 25]]);
console.log(map1.size); // 2

// From any iterable of [key, value] pairs
const map2 = new Map([
    ['key1', 'value1'],
    ['key2', 'value2'],
    [42, 'number key'],
    [true, 'boolean key']
]);

// From another Map
const map3 = new Map(map1);
console.log(map3.get('name')); // 'Jon'
```

### Key Types and Equality
```javascript
const map = new Map();

// Different types as keys
map.set(1, 'number one');
map.set('1', 'string one');
map.set(true, 'boolean true');
map.set({}, 'empty object');
map.set([], 'empty array');
map.set(() => {}, 'function');

console.log(map.size); // 6 - all different keys

// Key equality uses SameValueZero comparison
map.set(NaN, 'not a number');
map.set(NaN, 'updated NaN'); // Overwrites previous
console.log(map.get(NaN)); // 'updated NaN'

// +0 and -0 are treated as same key
map.set(0, 'zero');
map.set(-0, 'negative zero');
console.log(map.size); // Still 6 (not 7)
console.log(map.get(0)); // 'negative zero'
```

## Map Methods and Properties

### Core Methods
```javascript
const map = new Map();

// set(key, value) - adds/updates key-value pair, returns Map
map.set('name', 'Alice');
map.set('age', 30);
console.log(map.set('city', 'NYC')); // Returns the Map instance

// get(key) - retrieves value by key
console.log(map.get('name')); // 'Alice'
console.log(map.get('nonexistent')); // undefined

// has(key) - checks if key exists
console.log(map.has('name')); // true
console.log(map.has('salary')); // false

// delete(key) - removes key-value pair, returns boolean
console.log(map.delete('age')); // true
console.log(map.delete('salary')); // false
console.log(map.size); // 2

// clear() - removes all entries
map.clear();
console.log(map.size); // 0

// size - read-only property
console.log(map.size); // Current number of entries
```

### Method Chaining
```javascript
const map = new Map();

// set() returns the Map, allowing chaining
map
    .set('name', 'Bob')
    .set('age', 25)
    .set('city', 'London')
    .set('occupation', 'Developer');

console.log(map.size); // 4
```

### Iteration Methods
```javascript
const map = new Map([
    ['name', 'Charlie'],
    ['age', 28],
    ['city', 'Paris']
]);

// keys() - returns iterator of keys
for (const key of map.keys()) {
    console.log(key); // 'name', 'age', 'city'
}

// values() - returns iterator of values
for (const value of map.values()) {
    console.log(value); // 'Charlie', 28, 'Paris'
}

// entries() - returns iterator of [key, value] pairs
for (const [key, value] of map.entries()) {
    console.log(key, value); // 'name' 'Charlie', etc.
}

// Default iterator is entries()
for (const [key, value] of map) {
    console.log(key, value); // Same as map.entries()
}

// forEach method
map.forEach((value, key, ownMap) => {
    console.log(`${key}: ${value}`);
    console.log(map === ownMap); // true
});
```

## Map vs Object

### When to Use Map vs Object
```javascript
// Object - best for records with known string/symbol keys
const userRecord = {
    name: 'John',
    age: 30,
    email: 'john@example.com'
};

// Map - best for dynamic key-value storage with any key types
const userSettings = new Map();
userSettings.set('theme', 'dark');
userSettings.set('notifications', true);
userSettings.set(Symbol('secret'), 'hidden value');

// Key differences comparison
const obj = {};
const map = new Map();

// 1. Key types
obj['string'] = 'value';        // String keys only (symbols too)
obj[1] = 'number becomes string'; // 1 becomes '1'

map.set('string', 'value');     // Any type as key
map.set(1, 'actual number');    // Number stays number
map.set({}, 'object key');      // Object as key

// 2. Size
console.log(Object.keys(obj).length); // Manual calculation
console.log(map.size);                 // Built-in property

// 3. Iteration order
// Object: No guarantee in older versions, insertion order in modern JS
// Map: Always insertion order

// 4. Prototype
console.log('toString' in obj);     // true (inherited)
console.log(map.has('toString'));  // false (no default keys)

// 5. Performance
// Object: Optimized for property access
// Map: Better for frequent additions/deletions
```

### Performance Comparison
```javascript
// Object performance test
const obj = {};
console.time('Object operations');
for (let i = 0; i < 100000; i++) {
    obj[i] = i;
}
for (let i = 0; i < 100000; i++) {
    const value = obj[i];
}
for (let i = 0; i < 100000; i++) {
    delete obj[i];
}
console.timeEnd('Object operations');

// Map performance test
const map = new Map();
console.time('Map operations');
for (let i = 0; i < 100000; i++) {
    map.set(i, i);
}
for (let i = 0; i < 100000; i++) {
    const value = map.get(i);
}
for (let i = 0; i < 100000; i++) {
    map.delete(i);
}
console.timeEnd('Map operations');

// Generally:
// - Object: Faster property access with string keys
// - Map: Faster for frequent insertions/deletions, any key type
```

## WeakMap

WeakMap follows the same logic as WeakSet but stores key-value pairs.

### WeakMap Characteristics
```javascript
// WeakMap methods: set, get, has, delete
// Keys must be objects (same restriction as WeakSet)

const weakMap = new WeakMap();
let keyObj = { id: 1 };
let valueObj = { data: 'sensitive information' };

// set(key, value) - key must be an object
weakMap.set(keyObj, valueObj);
weakMap.set(keyObj, 'string value'); // Can overwrite

// get(key) - retrieve value by object key
console.log(weakMap.get(keyObj)); // 'string value'

// has(key) - check if key exists
console.log(weakMap.has(keyObj)); // true

// delete(key) - remove key-value pair
console.log(weakMap.delete(keyObj)); // true
console.log(weakMap.has(keyObj)); // false
```

### WeakMap Limitations
```javascript
const weakMap = new WeakMap();

// ❌ Can't use primitives as keys
// weakMap.set('string', 'value'); // TypeError
// weakMap.set(1, 'value');        // TypeError
// weakMap.set(true, 'value');     // TypeError

// ❌ Not iterable
// for (const [key, value] of weakMap) {} // TypeError
// [...weakMap] // TypeError

// ❌ No size property
// console.log(weakMap.size); // undefined

// ❌ No clear method
// weakMap.clear(); // TypeError

// ✅ Only objects as keys
const obj1 = {};
const obj2 = [];
const obj3 = () => {};

weakMap.set(obj1, 'value1');
weakMap.set(obj2, 'value2');
weakMap.set(obj3, 'value3'); // All valid
```

### WeakMap Use Cases
```javascript
// 1. Private data storage
const privateData = new WeakMap();

class User {
    constructor(name, ssn) {
        this.name = name;
        // Store sensitive data in WeakMap
        privateData.set(this, { ssn, secrets: [] });
    }
    
    getSSN() {
        return privateData.get(this).ssn;
    }
    
    addSecret(secret) {
        privateData.get(this).secrets.push(secret);
    }
}

const user = new User('John', '123-45-6789');
console.log(user.getSSN()); // '123-45-6789'
console.log(user.ssn); // undefined - not accessible directly

// When user is garbage collected, private data is automatically cleaned up

// 2. Metadata storage
const elementMetadata = new WeakMap();

function attachMetadata(element, data) {
    elementMetadata.set(element, data);
}

function getMetadata(element) {
    return elementMetadata.get(element);
}

// DOM elements can be garbage collected without memory leaks
const button = document.createElement('button');
attachMetadata(button, { clickCount: 0, createdAt: Date.now() });

// 3. Caching computed values
const computeCache = new WeakMap();

function expensiveComputation(obj) {
    if (computeCache.has(obj)) {
        return computeCache.get(obj);
    }
    
    const result = /* expensive calculation based on obj */;
    computeCache.set(obj, result);
    return result;
}
```

## Practical Use Cases

### Set Use Cases
```javascript
// 1. Tracking unique visitors
const uniqueVisitors = new Set();

function trackVisitor(userId) {
    uniqueVisitors.add(userId);
    console.log(`Total unique visitors: ${uniqueVisitors.size}`);
}

// 2. Permission checking
const adminUsers = new Set(['admin1', 'admin2', 'superadmin']);

function isAdmin(userId) {
    return adminUsers.has(userId);
}

// 3. Tag system
class Article {
    constructor(title) {
        this.title = title;
        this.tags = new Set();
    }
    
    addTag(tag) {
        this.tags.add(tag);
        return this; // For chaining
    }
    
    removeTag(tag) {
        return this.tags.delete(tag);
    }
    
    hasTag(tag) {
        return this.tags.has(tag);
    }
    
    getTags() {
        return [...this.tags];
    }
}

const article = new Article('JavaScript Tips');
article.addTag('javascript').addTag('programming').addTag('tips');

// 4. Mathematical set operations
function setUnion(setA, setB) {
    return new Set([...setA, ...setB]);
}

function setIntersection(setA, setB) {
    return new Set([...setA].filter(x => setB.has(x)));
}

function setDifference(setA, setB) {
    return new Set([...setA].filter(x => !setB.has(x)));
}

const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);
console.log(setUnion(setA, setB)); // Set {1, 2, 3, 4}
console.log(setIntersection(setA, setB)); // Set {2, 3}
console.log(setDifference(setA, setB)); // Set {1}
```

### Map Use Cases
```javascript
// 1. Caching/Memoization
const cache = new Map();

function fibonacci(n) {
    if (cache.has(n)) {
        return cache.get(n);
    }
    
    const result = n <= 1 ? n : fibonacci(n - 1) + fibonacci(n - 2);
    cache.set(n, result);
    return result;
}

// 2. Counting occurrences
function countOccurrences(array) {
    const counts = new Map();
    
    for (const item of array) {
        counts.set(item, (counts.get(item) || 0) + 1);
    }
    
    return counts;
}

const words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'];
const wordCounts = countOccurrences(words);
console.log(wordCounts); // Map { 'apple' => 3, 'banana' => 2, 'cherry' => 1 }

// 3. Object mapping
const userPreferences = new Map();
const user1 = { id: 1, name: 'Alice' };
const user2 = { id: 2, name: 'Bob' };

userPreferences.set(user1, { theme: 'dark', language: 'en' });
userPreferences.set(user2, { theme: 'light', language: 'es' });

function getUserTheme(user) {
    return userPreferences.get(user)?.theme || 'default';
}

// 4. Event system
const eventListeners = new Map();

function addEventListener(event, callback) {
    if (!eventListeners.has(event)) {
        eventListeners.set(event, new Set());
    }
    eventListeners.get(event).add(callback);
}

function removeEventListener(event, callback) {
    eventListeners.get(event)?.delete(callback);
}

function emit(event, data) {
    const listeners = eventListeners.get(event);
    if (listeners) {
        listeners.forEach(callback => callback(data));
    }
}
```

## Performance Considerations

### Set vs Array Performance
```javascript
// Set lookup: O(1) average case
// Array.includes(): O(n)

const largeArray = Array.from({ length: 100000 }, (_, i) => i);
const largeSet = new Set(largeArray);

// Searching in array - slow for large arrays
console.time('Array includes');
const foundInArray = largeArray.includes(99999);
console.timeEnd('Array includes');

// Searching in set - fast
console.time('Set has');
const foundInSet = largeSet.has(99999);
console.timeEnd('Set has');

// Set is much faster for membership testing
```

### Map vs Object Performance
```javascript
// Key access patterns
const obj = {};
const map = new Map();

// Initialization
console.time('Object property assignment');
for (let i = 0; i < 100000; i++) {
    obj[`key${i}`] = `value${i}`;
}
console.timeEnd('Object property assignment');

console.time('Map set');
for (let i = 0; i < 100000; i++) {
    map.set(`key${i}`, `value${i}`);
}
console.timeEnd('Map set');

// Access
console.time('Object property access');
for (let i = 0; i < 100000; i++) {
    const value = obj[`key${i}`];
}
console.timeEnd('Object property access');

console.time('Map get');
for (let i = 0; i < 100000; i++) {
    const value = map.get(`key${i}`);
}
console.timeEnd('Map get');

// Deletion
console.time('Object delete');
for (let i = 0; i < 100000; i++) {
    delete obj[`key${i}`];
}
console.timeEnd('Object delete');

console.time('Map delete');
for (let i = 0; i < 100000; i++) {
    map.delete(`key${i}`);
}
console.timeEnd('Map delete');
```

### Memory Usage Considerations
```javascript
// Regular Set/Map keep strong references
function memoryLeakExample() {
    const cache = new Map();
    const users = new Set();
    
    for (let i = 0; i < 1000000; i++) {
        const user = { id: i, data: new Array(1000).fill(i) };
        users.add(user);
        cache.set(user, `cached_${i}`);
    }
    
    // Even if we lose references to individual users,
    // they're kept alive by the Set and Map
    return { cache, users };
}

// WeakSet/WeakMap allow garbage collection
function memoryEfficientExample() {
    const cache = new WeakMap();
    const privateData = new WeakMap();
    
    function createUser(id) {
        const user = { id };
        cache.set(user, `cached_${id}`);
        privateData.set(user, new Array(1000).fill(id));
        return user;
    }
    
    // When user objects are no longer referenced,
    // they can be garbage collected along with their cached data
    return { createUser };
}
```

## Best Practices

### Choosing the Right Collection
```javascript
// ✅ Use Set for unique values
const uniqueIds = new Set();
uniqueIds.add(userId);

// ✅ Use Map for key-value pairs with any key type
const userSessions = new Map();
userSessions.set(userObject, sessionData);

// ✅ Use WeakSet for object tracking without preventing GC
const processedNodes = new WeakSet();

// ✅ Use WeakMap for private/metadata storage
const privateFields = new WeakMap();

// ❌ Don't use regular Object when you need:
// - Non-string keys
// - Frequent additions/deletions
// - Size information
// - Guaranteed iteration order

// ❌ Don't use Map when you need:
// - JSON serialization (Map doesn't serialize)
// - Property access syntax (obj.prop vs map.get('prop'))
// - Prototype inheritance
```

### Common Patterns
```javascript
// 1. Default values with Map
function getWithDefault(map, key, defaultValue) {
    if (map.has(key)) {
        return map.get(key);
    }
    map.set(key, defaultValue);
    return defaultValue;
}

// Or using logical OR with careful consideration
function getOrCreate(map, key, factory) {
    return map.get(key) || (map.set(key, factory()), map.get(key));
}

// 2. Grouping with Map
function groupBy(array, keyFn) {
    const groups = new Map();
    
    for (const item of array) {
        const key = keyFn(item);
        if (!groups.has(key)) {
            groups.set(key, []);
        }
        groups.get(key).push(item);
    }
    
    return groups;
}

// 3. Set operations utility
class SetUtils {
    static union(setA, setB) {
        return new Set([...setA, ...setB]);
    }
    
    static intersection(setA, setB) {
        return new Set([...setA].filter(x => setB.has(x)));
    }
    
    static difference(setA, setB) {
        return new Set([...setA].filter(x => !setB.has(x)));
    }
    
    static isSubset(subset, superset) {
        return [...subset].every(x => superset.has(x));
    }
}

// 4. Safe WeakMap access
function safeWeakMapGet(weakMap, key, defaultValue) {
    try {
        return weakMap.has(key) ? weakMap.get(key) : defaultValue;
    } catch (error) {
        // Key might have been garbage collected
        return defaultValue;
    }
}
```

### Error Handling
```javascript
// Set/Map operations are generally safe, but watch for:

// 1. WeakSet/WeakMap type errors
function safeWeakSetAdd(weakSet, item) {
    if (item && typeof item === 'object') {
        weakSet.add(item);
        return true;
    }
    console.warn('WeakSet can only store objects');
    return false;
}

// 2. Iteration during modification
function safeSetIteration(set, processor) {
    // Create a copy to avoid modification during iteration
    const items = [...set];
    
    for (const item of items) {
        const shouldRemove = processor(item);
        if (shouldRemove) {
            set.delete(item);
        }
    }
}

// 3. Map key comparison gotchas
const map = new Map();
const key1 = { id: 1 };
const key2 = { id: 1 }; // Different object, same content

map.set(key1, 'value1');
console.log(map.get(key2)); // undefined - different objects!

// Use a consistent key strategy
function createUserKey(user) {
    return `user_${user.id}`;
}
```

### Key Takeaways

1. **Set stores unique values** using `Object.is()` equality
2. **Set.forEach** has key parameter for consistency with Map, but key equals value
3. **Set methods** `keys()`, `values()`, `entries()` return similar iterators
4. **Memory management**: Regular Set/Map keep strong references, WeakSet/WeakMap allow GC
5. **WeakSet/WeakMap limitations**: Only objects as keys/values, not iterable, no size
6. **Map allows any key type**, unlike objects which convert keys to strings
7. **Choose collections wisely**: Set for uniqueness, Map for key-value, Weak* for memory efficiency
8. **Performance**: Set/Map are faster for frequent additions/deletions than Array/Object