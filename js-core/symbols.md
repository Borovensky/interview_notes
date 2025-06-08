# JavaScript Symbols Guide

## Table of Contents
- [Symbol Fundamentals](#symbol-fundamentals)
- [Symbol Descriptions](#symbol-descriptions)
- [Global Symbol Registry](#global-symbol-registry)
- [Symbols as Object Keys](#symbols-as-object-keys)
- [Symbol Enumeration and Visibility](#symbol-enumeration-and-visibility)
- [Well-Known Symbols](#well-known-symbols)
- [Symbol.hasInstance](#symbolhasinstance)
- [Symbol.isConcatSpreadable](#symbolisconcatspreadable)
- [Symbol.iterator](#symboliterator)
- [Symbol.match and String Methods](#symbolmatch-and-string-methods)
- [Complete Well-Known Symbols](#complete-well-known-symbols)
- [Symbol Use Cases](#symbol-use-cases)
- [Symbol Limitations](#symbol-limitations)
- [Best Practices](#best-practices)

## Symbol Fundamentals

As noted in your introduction, Symbols were initially designed as private properties for objects, but they didn't fully succeed in this task. However, they serve as a unique primitive data type.

### Basic Symbol Creation
Based on your explanation:

```javascript
// Symbols => Изначально задумывались как приватные свойства для объектов, но с этой задачей не справились.
// Это новый тип данных (примитив). Symbol генерирует уникальный, неповторимый идентификатор.
const name = Symbol(); // у символа нет конструктора. new Symbol => TypeError
console.log(name);  // Symbol()
```

### Symbol Characteristics
```javascript
// Symbols are always unique
const sym1 = Symbol();
const sym2 = Symbol();
console.log(sym1 === sym2); // false - always unique!

// No constructor - Symbol is not a constructor function
try {
    const symObj = new Symbol(); // TypeError: Symbol is not a constructor
} catch (e) {
    console.log(e.message);
}

// Symbol is a primitive type
console.log(typeof Symbol()); // 'symbol'

// Symbols are immutable
const sym = Symbol('description');
// sym.description = 'new description'; // Cannot modify
```

### Symbol Uniqueness
```javascript
// Every Symbol() call creates a unique symbol
const symbols = [];
for (let i = 0; i < 1000; i++) {
    symbols.push(Symbol());
}

// All symbols are unique
const allUnique = symbols.every((sym, index) => 
    symbols.slice(index + 1).every(otherSym => sym !== otherSym)
);
console.log(allUnique); // true

// Even with same description, they're different
const a = Symbol('same');
const b = Symbol('same');
console.log(a === b); // false
console.log(a.description === b.description); // true (descriptions are the same)
```

## Symbol Descriptions

### Adding Descriptions for Debugging
Based on your example:

```javascript
// Field name (строка в параметре) это просто описание символа. Чтоб было легче дебажить...и т.д
const name2 = Symbol('Field name');
console.log(name2); // Symbol(Field name)
```

### Description Property and Methods
```javascript
const userSymbol = Symbol('user data');
const adminSymbol = Symbol('admin privileges');

// Access description
console.log(userSymbol.description); // 'user data'
console.log(adminSymbol.description); // 'admin privileges'

// Description is just for debugging
const sym1 = Symbol('test');
const sym2 = Symbol('test');
console.log(sym1.description === sym2.description); // true
console.log(sym1 === sym2); // false (still unique!)

// Description can be undefined
const noDescription = Symbol();
console.log(noDescription.description); // undefined

// toString includes description
console.log(userSymbol.toString()); // 'Symbol(user data)'
console.log(noDescription.toString()); // 'Symbol()'
```

### Debugging with Descriptions
```javascript
// Use descriptive names for better debugging
const PRIVATE_COUNTER = Symbol('private counter');
const PRIVATE_NAME = Symbol('private name');
const CACHE_KEY = Symbol('cache key');

class User {
    constructor(name) {
        this[PRIVATE_NAME] = name;
        this[PRIVATE_COUNTER] = 0;
        this[CACHE_KEY] = new Map();
    }
    
    getName() {
        return this[PRIVATE_NAME];
    }
    
    incrementCounter() {
        this[PRIVATE_COUNTER]++;
    }
    
    getCounter() {
        return this[PRIVATE_COUNTER];
    }
}

const user = new User('Alice');
user.incrementCounter();
console.log(user.getCounter()); // 1

// In debugger, you can see meaningful symbol descriptions
console.log(Object.getOwnPropertySymbols(user));
// [Symbol(private name), Symbol(private counter), Symbol(cache key)]
```

## Global Symbol Registry

### Symbol.for and Symbol.keyFor
Based on your explanation of the global registry:

```javascript
// У символов есть глобальный риестр символов (for / keyFor). Он доступен из любово места программы (кода)
const a = Symbol.for('A'); // создать запись в глобальноv риестре через for. 
const description = Symbol.keyFor(a);
console.log(description); // 'A'

// === когда мы будем сравнивать дискрипшини у символов, они уже могу быть равны
const i = Symbol.for('A');
const j = Symbol.for('A');
console.log(i === j); // true
```

### Global vs Local Symbols
```javascript
// Local symbols (always unique)
const localSym1 = Symbol('test');
const localSym2 = Symbol('test');
console.log(localSym1 === localSym2); // false

// Global symbols (shared across entire application)
const globalSym1 = Symbol.for('test');
const globalSym2 = Symbol.for('test');
console.log(globalSym1 === globalSym2); // true

// Symbol.keyFor only works with global symbols
console.log(Symbol.keyFor(globalSym1)); // 'test'
console.log(Symbol.keyFor(localSym1)); // undefined

// Check if symbol is global
function isGlobalSymbol(sym) {
    return Symbol.keyFor(sym) !== undefined;
}

console.log(isGlobalSymbol(globalSym1)); // true
console.log(isGlobalSymbol(localSym1)); // false
```

### Cross-Module Symbol Sharing
```javascript
// Module A
// const SHARED_KEY = Symbol.for('app.shared.key');
// export { SHARED_KEY };

// Module B  
// const SHARED_KEY = Symbol.for('app.shared.key'); // Same symbol!

// Both modules get the same symbol
// This enables cross-module communication

// Namespaced global symbols (recommended pattern)
const APP_NAMESPACE = 'myapp';
const createGlobalSymbol = (name) => Symbol.for(`${APP_NAMESPACE}.${name}`);

const USER_KEY = createGlobalSymbol('user');
const SESSION_KEY = createGlobalSymbol('session');
const CONFIG_KEY = createGlobalSymbol('config');

// Later, anywhere in the application
const sameUserKey = Symbol.for('myapp.user');
console.log(USER_KEY === sameUserKey); // true
```

### Global Symbol Registry Management
```javascript
// Utility for managing global symbols
class SymbolRegistry {
    static create(key, description = key) {
        if (Symbol.for(key)) {
            return Symbol.for(key);
        }
        // Note: Symbol.for always returns the same symbol for the same key
        return Symbol.for(key);
    }
    
    static get(key) {
        return Symbol.for(key);
    }
    
    static exists(key) {
        try {
            const sym = Symbol.for(key);
            return Symbol.keyFor(sym) === key;
        } catch {
            return false;
        }
    }
    
    static getKey(symbol) {
        return Symbol.keyFor(symbol);
    }
}

// Usage
const mySymbol = SymbolRegistry.create('user.id');
const sameSymbol = SymbolRegistry.get('user.id');
console.log(mySymbol === sameSymbol); // true
console.log(SymbolRegistry.getKey(mySymbol)); // 'user.id'
```

## Symbols as Object Keys

### Basic Symbol Keys
Based on your example:

```javascript
// Как использовать символ в качестве ключа
const name3 = Symbol();
const person = {};
person[name3] = 'Tom';
const value = person[name3];
console.log(value); // Tom
```

### Symbol Keys vs String Keys
```javascript
const stringKey = 'name';
const symbolKey = Symbol('name');

const obj = {};
obj[stringKey] = 'String value';
obj[symbolKey] = 'Symbol value';

console.log(obj[stringKey]); // 'String value'
console.log(obj[symbolKey]); // 'Symbol value'
console.log(obj['name']); // 'String value'

// Symbol and string keys are completely separate
console.log(Object.keys(obj)); // ['name']
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(name)]
```

### Object Literal with Symbol Keys
```javascript
const ID_KEY = Symbol('id');
const TYPE_KEY = Symbol('type');

// Using computed property names
const entity = {
    name: 'Entity',
    [ID_KEY]: 'entity-123',
    [TYPE_KEY]: 'user',
    regularProperty: 'visible'
};

console.log(entity[ID_KEY]); // 'entity-123'
console.log(entity[TYPE_KEY]); // 'user'

// Class with symbol properties
class SecureData {
    constructor(publicData, privateData) {
        this.publicData = publicData;
        this[ID_KEY] = Math.random().toString(36);
        this[TYPE_KEY] = 'secure';
        this._privateData = privateData; // Convention-based "private"
    }
    
    getId() {
        return this[ID_KEY];
    }
    
    getType() {
        return this[TYPE_KEY];
    }
}

const data = new SecureData('public info', 'secret');
console.log(data.getId()); // Random ID
console.log(data.publicData); // 'public info'
// data[ID_KEY] would require access to the symbol
```

## Symbol Enumeration and Visibility

### Symbol Non-Enumerable Nature
Based on your observation:

```javascript
// Symbol является не перечеслимым (другая разновидность ключей)! Он больше задумывался как служебное поле.
const c = Symbol();
const object = {a: 'a', b: 'b'};
object[c] = 'c';
for(let i in object) {
    console.log(i); // a b 
}
```

### Getting Symbol Properties
As you showed, there's a special method:

```javascript
// чтоб получить все символи из объекта есть метод getOwnPropertySymbols
const keys = Object.getOwnPropertySymbols(object);
console.log(keys); // [ Symbol() ]
console.log(object[keys[0]]); // c => это способ как можно обойти неперечисляемость Symbol в объекте
```

### Complete Property Enumeration
```javascript
const regularKey = 'regular';
const symbolKey = Symbol('symbol');
const nonEnumerableKey = 'nonEnumerable';

const obj = {
    [regularKey]: 'regular value',
    [symbolKey]: 'symbol value'
};

// Add non-enumerable property
Object.defineProperty(obj, nonEnumerableKey, {
    value: 'non-enumerable value',
    enumerable: false
});

// Different ways to get properties
console.log('for...in loop:');
for (const key in obj) {
    console.log(key); // Only 'regular'
}

console.log('Object.keys():', Object.keys(obj)); // ['regular']
console.log('Object.getOwnPropertyNames():', Object.getOwnPropertyNames(obj)); // ['regular', 'nonEnumerable']
console.log('Object.getOwnPropertySymbols():', Object.getOwnPropertySymbols(obj)); // [Symbol(symbol)]

// Get ALL own properties (including symbols)
console.log('Reflect.ownKeys():', Reflect.ownKeys(obj)); // ['regular', 'nonEnumerable', Symbol(symbol)]
```

### JSON Serialization and Symbols
```javascript
const obj = {
    name: 'John',
    age: 30,
    [Symbol('secret')]: 'hidden data'
};

// JSON.stringify ignores symbol properties
console.log(JSON.stringify(obj)); // {"name":"John","age":30}

// To include symbols in serialization, you need custom logic
function serializeWithSymbols(obj) {
    const result = { ...obj };
    const symbols = Object.getOwnPropertySymbols(obj);
    
    result.__symbols__ = {};
    symbols.forEach(sym => {
        result.__symbols__[sym.description || sym.toString()] = obj[sym];
    });
    
    return JSON.stringify(result);
}

function deserializeWithSymbols(jsonString) {
    const parsed = JSON.parse(jsonString);
    const { __symbols__, ...regular } = parsed;
    
    if (__symbols__) {
        Object.keys(__symbols__).forEach(desc => {
            const symbol = Symbol(desc === 'Symbol()' ? undefined : desc);
            regular[symbol] = __symbols__[desc];
        });
    }
    
    return regular;
}

const serialized = serializeWithSymbols(obj);
console.log(serialized); // Includes symbol data
const deserialized = deserializeWithSymbols(serialized);
console.log(Object.getOwnPropertySymbols(deserialized)); // Has symbols restored
```

## Well-Known Symbols

As noted in your code, most new JavaScript features use well-known symbols.

### Introduction to Well-Known Symbols
Based on your comment:

```javascript
// большинство новых вещей в js спецификации водится с использовнием так называемых well known symbols (Symbol)
```

Well-known symbols are predefined symbols that JavaScript uses internally to customize object behavior.

### List of Well-Known Symbols
```javascript
// Complete list of well-known symbols
const wellKnownSymbols = {
    iterator: Symbol.iterator,
    asyncIterator: Symbol.asyncIterator,
    hasInstance: Symbol.hasInstance,
    isConcatSpreadable: Symbol.isConcatSpreadable,
    match: Symbol.match,
    matchAll: Symbol.matchAll,
    replace: Symbol.replace,
    search: Symbol.search,
    species: Symbol.species,
    split: Symbol.split,
    toPrimitive: Symbol.toPrimitive,
    toStringTag: Symbol.toStringTag,
    unscopables: Symbol.unscopables
};

console.log('All well-known symbols:');
Object.entries(wellKnownSymbols).forEach(([name, symbol]) => {
    console.log(`${name}: ${symbol.toString()}`);
});
```

## Symbol.hasInstance

### Custom instanceof Behavior
Based on your example:

```javascript
class MyArray {
    static [Symbol.hasInstance](instance) {
        return Array.isArray(instance);
    }
}
console.log([] instanceof MyArray); // true (when uncommented)
console.log([] instanceof Array); // true 
```

### Advanced hasInstance Usage
```javascript
// Create a "type" that accepts multiple actual types
class StringOrNumber {
    static [Symbol.hasInstance](instance) {
        return typeof instance === 'string' || typeof instance === 'number';
    }
}

console.log('hello' instanceof StringOrNumber); // true
console.log(42 instanceof StringOrNumber); // true
console.log(true instanceof StringOrNumber); // false
console.log({} instanceof StringOrNumber); // false

// Duck typing with hasInstance
class Flyable {
    static [Symbol.hasInstance](instance) {
        return instance && 
               typeof instance.fly === 'function' &&
               typeof instance.land === 'function';
    }
}

const bird = {
    fly() { return 'flying'; },
    land() { return 'landed'; }
};

const airplane = {
    fly() { return 'taking off'; },
    land() { return 'landing'; }
};

const car = {
    drive() { return 'driving'; }
};

console.log(bird instanceof Flyable); // true
console.log(airplane instanceof Flyable); // true
console.log(car instanceof Flyable); // false
```

### Practical hasInstance Applications
```javascript
// Framework-like type checking
class Component {
    static [Symbol.hasInstance](instance) {
        return instance &&
               typeof instance.render === 'function' &&
               typeof instance.mount === 'function';
    }
}

class Button {
    render() { return '<button>Click me</button>'; }
    mount(parent) { parent.appendChild(this.element); }
}

class InvalidComponent {
    display() { return 'I am not a real component'; }
}

console.log(new Button() instanceof Component); // true
console.log(new InvalidComponent() instanceof Component); // false

// API validation
class APIResponse {
    static [Symbol.hasInstance](instance) {
        return instance &&
               typeof instance === 'object' &&
               'status' in instance &&
               'data' in instance &&
               typeof instance.status === 'number';
    }
}

const validResponse = { status: 200, data: { users: [] } };
const invalidResponse = { message: 'Hello' };

console.log(validResponse instanceof APIResponse); // true
console.log(invalidResponse instanceof APIResponse); // false
```

## Symbol.isConcatSpreadable

### Array Concatenation Control
Based on your example:

```javascript
const arr = [1, 2, 3];
const obj = {
    // [Symbol.isConcatSpreadable]: true,
    length: 2,
    0: 'a',
    1: 'b'
};
console.log(arr.concat(obj)); // [ 1, 2, 3, { '0': 'a', '1': 'b', length: 2 } ]
// если разкоментировать код obj то результат будет => [ 1, 2, 3, 'a', 'b' ]
```

### Controlling Spread Behavior
```javascript
// Default behavior - arrays are spreadable, objects are not
const arr1 = [1, 2];
const arr2 = [3, 4];
const obj = { 0: 'a', 1: 'b', length: 2 };

console.log([].concat(arr1, arr2)); // [1, 2, 3, 4]
console.log([].concat(obj)); // [{ 0: 'a', 1: 'b', length: 2 }]

// Make object spreadable
obj[Symbol.isConcatSpreadable] = true;
console.log([].concat(obj)); // ['a', 'b']

// Make array non-spreadable
arr1[Symbol.isConcatSpreadable] = false;
console.log([].concat(arr1, arr2)); // [[1, 2], 3, 4]
```

### Custom Array-like Classes
```javascript
class CustomArray {
    constructor(...items) {
        this.length = items.length;
        items.forEach((item, index) => {
            this[index] = item;
        });
    }
    
    // Make it spreadable by default
    get [Symbol.isConcatSpreadable]() {
        return true;
    }
    
    push(item) {
        this[this.length] = item;
        this.length++;
        return this.length;
    }
}

const customArr = new CustomArray('x', 'y', 'z');
const normalArr = [1, 2, 3];

console.log(normalArr.concat(customArr)); // [1, 2, 3, 'x', 'y', 'z']

// Conditional spreading
class ConditionalArray extends CustomArray {
    constructor(items, shouldSpread = true) {
        super(...items);
        this.shouldSpread = shouldSpread;
    }
    
    get [Symbol.isConcatSpreadable]() {
        return this.shouldSpread;
    }
}

const spreadable = new ConditionalArray(['a', 'b'], true);
const nonSpreadable = new ConditionalArray(['c', 'd'], false);

console.log([1, 2].concat(spreadable)); // [1, 2, 'a', 'b']
console.log([1, 2].concat(nonSpreadable)); // [1, 2, ConditionalArray]
```

## Symbol.iterator

### Making Objects Iterable
Based on your example:

```javascript
const data = {
    id: 'a1',
};
// console.log([...data]); // TypeError: data is not iterable. Объекты не итерируемые. 
// чтоб сделать его итерируемым можно использовать symbol
data[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
};
console.log([...data]); // [1, 2]
```

### Custom Iterator Implementation
```javascript
class NumberRange {
    constructor(start, end, step = 1) {
        this.start = start;
        this.end = end;
        this.step = step;
    }
    
    [Symbol.iterator]() {
        let current = this.start;
        const end = this.end;
        const step = this.step;
        
        return {
            next() {
                if (current < end) {
                    const value = current;
                    current += step;
                    return { value, done: false };
                } else {
                    return { done: true };
                }
            }
        };
    }
}

const range = new NumberRange(1, 10, 2);
console.log([...range]); // [1, 3, 5, 7, 9]

// Use in for...of loop
for (const num of range) {
    console.log(num); // 1, 3, 5, 7, 9
}
```

### Generator-based Iterators
```javascript
class Fibonacci {
    constructor(max = 10) {
        this.max = max;
    }
    
    * [Symbol.iterator]() {
        let prev = 0, curr = 1, count = 0;
        
        while (count < this.max) {
            yield prev;
            [prev, curr] = [curr, prev + curr];
            count++;
        }
    }
}

const fib = new Fibonacci(8);
console.log([...fib]); // [0, 1, 1, 2, 3, 5, 8, 13]

// Object with multiple iteration strategies
class DataCollection {
    constructor(data) {
        this.data = data;
    }
    
    // Default iteration - values
    * [Symbol.iterator]() {
        yield* this.values();
    }
    
    * keys() {
        yield* Object.keys(this.data);
    }
    
    * values() {
        yield* Object.values(this.data);
    }
    
    * entries() {
        yield* Object.entries(this.data);
    }
}

const collection = new DataCollection({ a: 1, b: 2, c: 3 });
console.log([...collection]); // [1, 2, 3]
console.log([...collection.keys()]); // ['a', 'b', 'c']
console.log([...collection.entries()]); // [['a', 1], ['b', 2], ['c', 3]]
```

## Symbol.match and String Methods

### Controlling String Method Behavior
Based on your example:

```javascript
const pattern = /bar/;
// pattern[Symbol.match] = false;
console.log('/bar/'.startsWith(pattern)); // TypeError: First argument to String.prototype.startsWith must not be a regular expression
// мы не можем проверить регулярное выражение. там должна быть строка! 
//  pattern[Symbol.match] = false; => меняет поведение и все будет работать => true;
```

### Symbol.match Implementation
```javascript
// Make regex work with string methods that normally reject regexes
const regex = /test/;
regex[Symbol.match] = false;

console.log('test string'.startsWith(regex)); // false (would error without Symbol.match = false)
console.log('test string'.includes(regex)); // false
console.log('regex test'.endsWith(regex)); // false

// Custom matcher object
class CustomMatcher {
    constructor(pattern) {
        this.pattern = pattern;
    }
    
    [Symbol.match](string) {
        const index = string.indexOf(this.pattern);
        if (index === -1) return null;
        
        return {
            0: this.pattern,
            index,
            input: string,
            length: 1
        };
    }
    
    toString() {
        return this.pattern;
    }
}

const matcher = new CustomMatcher('hello');
console.log('hello world'.match(matcher)); // { 0: 'hello', index: 0, input: 'hello world', length: 1 }
```

### All String Symbol Methods
```javascript
class StringProcessor {
    constructor(pattern) {
        this.pattern = pattern;
    }
    
    [Symbol.match](string) {
        console.log('match called');
        const index = string.indexOf(this.pattern);
        return index >= 0 ? [this.pattern] : null;
    }
    
    [Symbol.replace](string, replacement) {
        console.log('replace called');
        return string.replace(this.pattern, replacement);
    }
    
    [Symbol.search](string) {
        console.log('search called');
        return string.indexOf(this.pattern);
    }
    
    [Symbol.split](string) {
        console.log('split called');
        return string.split(this.pattern);
    }
    
    // Disable regex-like behavior for string methods
    get [Symbol.match]() {
        return undefined; // This makes it not a regex-like object
    }
}

const processor = new StringProcessor('o');
const text = 'hello world';

console.log(text.match(processor)); // Uses Symbol.match
console.log(text.replace(processor, 'X')); // Uses Symbol.replace  
console.log(text.search(processor)); // Uses Symbol.search
console.log(text.split(processor)); // Uses Symbol.split
```

## Complete Well-Known Symbols

### Symbol.toPrimitive
```javascript
class Temperature {
    constructor(celsius) {
        this.celsius = celsius;
    }
    
    [Symbol.toPrimitive](hint) {
        switch (hint) {
            case 'number':
                return this.celsius;
            case 'string':
                return `${this.celsius}°C`;
            case 'default':
                return this.celsius;
            default:
                throw new Error(`Invalid hint: ${hint}`);
        }
    }
}

const temp = new Temperature(25);
console.log(+temp); // 25 (number hint)
console.log(`Temperature: ${temp}`); // "Temperature: 25°C" (string hint)
console.log(temp + 5); // 30 (default hint)
```

### Symbol.toStringTag
```javascript
class CustomClass {
    get [Symbol.toStringTag]() {
        return 'CustomClass';
    }
}

const obj = new CustomClass();
console.log(Object.prototype.toString.call(obj)); // '[object CustomClass]'
console.log(obj.toString()); // '[object CustomClass]'

// Built-in examples
console.log(Object.prototype.toString.call(new Map())); // '[object Map]'
console.log(Object.prototype.toString.call(new Set())); // '[object Set]'
console.log(Object.prototype.toString.call(Promise.resolve())); // '[object Promise]'
```

### Symbol.species
```javascript
class MyArray extends Array {
    // Control what constructor is used for derived objects
    static get [Symbol.species]() {
        return Array; // Return regular Array instead of MyArray
    }
}

const myArr = new MyArray(1, 2, 3);
const mapped = myArr.map(x => x * 2);

console.log(myArr instanceof MyArray); // true
console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array); // true
```

### Symbol.unscopables
```javascript
// Control which properties are excluded from 'with' statements
const obj = {
    a: 1,
    b: 2,
    c: 3,
    
    [Symbol.unscopables]: {
        b: true // 'b' will be excluded from 'with' scope
    }
};

with (obj) {
    console.log(a); // 1
    console.log(c); // 3
    // console.log(b); // ReferenceError: b is not defined
}
```

## Symbol Use Cases

### Private-like Properties
```javascript
// Symbols provide privacy through obscurity
const PRIVATE_DATA = Symbol('private');
const PRIVATE_METHOD = Symbol('privateMethod');

class BankAccount {
    constructor(initialBalance) {
        this.accountNumber = Math.random().toString(36);
        this[PRIVATE_DATA] = {
            balance: initialBalance,
            transactions: []
        };
    }
    
    [PRIVATE_METHOD](amount, type) {
        this[PRIVATE_DATA].transactions.push({
            amount,
            type,
            timestamp: Date.now()
        });
    }
    
    deposit(amount) {
        if (amount > 0) {
            this[PRIVATE_DATA].balance += amount;
            this[PRIVATE_METHOD](amount, 'deposit');
        }
    }
    
    withdraw(amount) {
        if (amount > 0 && amount <= this[PRIVATE_DATA].balance) {
            this[PRIVATE_DATA].balance -= amount;
            this[PRIVATE_METHOD](amount, 'withdrawal');
            return true;
        }
        return false;
    }
    
    getBalance() {
        return this[PRIVATE_DATA].balance;
    }
}

const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500

// Private data is not easily accessible
console.log(Object.keys(account)); // ['accountNumber']
console.log(account.balance); // undefined

// But can be accessed if you have the symbol (not truly private)
console.log(account[PRIVATE_DATA]); // { balance: 1500, transactions: [...] }
```

### Metadata and Annotations
```javascript
const METADATA = Symbol('metadata');
const VALIDATION_RULES = Symbol('validation');
const SERIALIZE_CONFIG = Symbol('serialize');

class Entity {
    static addMetadata(target, metadata) {
        target[METADATA] = { ...target[METADATA], ...metadata };
    }
    
    static getMetadata(target) {
        return target[METADATA] || {};
    }
    
    static addValidation(target, field, rules) {
        if (!target[VALIDATION_RULES]) {
            target[VALIDATION_RULES] = {};
        }
        target[VALIDATION_RULES][field] = rules;
    }
    
    static validate(instance) {
        const rules = instance.constructor[VALIDATION_RULES] || {};
        const errors = [];
        
        Object.keys(rules).forEach(field => {
            const value = instance[field];
            const fieldRules = rules[field];
            
            if (fieldRules.required && (value === undefined || value === null)) {
                errors.push(`${field} is required`);
            }
            
            if (fieldRules.type && typeof value !== fieldRules.type) {
                errors.push(`${field} must be of type ${fieldRules.type}`);
            }
        });
        
        return errors;
    }
}

class User extends Entity {
    constructor(name, email, age) {
        super();
        this.name = name;
        this.email = email;
        this.age = age;
    }
}

// Add metadata
Entity.addMetadata(User, {
    tableName: 'users',
    version: '1.0'
});

// Add validation rules
Entity.addValidation(User, 'name', { required: true, type: 'string' });
Entity.addValidation(User, 'email', { required: true, type: 'string' });
Entity.addValidation(User, 'age', { type: 'number' });

const user = new User('Alice', 'alice@example.com', 25);
const errors = Entity.validate(user);
console.log(errors); // []

const invalidUser = new User(null, 123, 'invalid');
const invalidErrors = Entity.validate(invalidUser);
console.log(invalidErrors); // ['name is required', 'email must be of type string', 'age must be of type number']
```

### Plugin/Extension System
```javascript
const PLUGINS = Symbol('plugins');
const HOOKS = Symbol('hooks');

class ExtensibleClass {
    constructor() {
        this[PLUGINS] = new Map();
        this[HOOKS] = new Map();
    }
    
    addPlugin(name, plugin) {
        this[PLUGINS].set(name, plugin);
        
        // Initialize plugin hooks
        if (plugin.hooks) {
            Object.keys(plugin.hooks).forEach(hookName => {
                if (!this[HOOKS].has(hookName)) {
                    this[HOOKS].set(hookName, []);
                }
                this[HOOKS].get(hookName).push(plugin.hooks[hookName]);
            });
        }
        
        if (plugin.init) {
            plugin.init(this);
        }
    }
    
    async runHook(hookName, ...args) {
        const hooks = this[HOOKS].get(hookName) || [];
        const results = [];
        
        for (const hook of hooks) {
            const result = await hook(...args);
            results.push(result);
        }
        
        return results;
    }
    
    getPlugin(name) {
        return this[PLUGINS].get(name);
    }
}

// Example usage
const app = new ExtensibleClass();

const loggingPlugin = {
    init(app) {
        console.log('Logging plugin initialized');
    },
    hooks: {
        beforeAction: (action) => {
            console.log(`Before action: ${action}`);
        },
        afterAction: (action, result) => {
            console.log(`After action: ${action}, result:`, result);
        }
    }
};

const cachingPlugin = {
    cache: new Map(),
    hooks: {
        beforeAction: (action) => {
            if (this.cache.has(action)) {
                console.log(`Cache hit for: ${action}`);
                return this.cache.get(action);
            }
        },
        afterAction: (action, result) => {
            this.cache.set(action, result);
        }
    }
};

app.addPlugin('logging', loggingPlugin);
app.addPlugin('caching', cachingPlugin);
```

## Symbol Limitations

### Not Truly Private
```javascript
// Symbols are not truly private - they can be discovered
const SECRET = Symbol('secret');

class NotSoSecure {
    constructor(secret) {
        this[SECRET] = secret;
    }
    
    getPublicData() {
        return 'public';
    }
}

const obj = new NotSoSecure('top secret');

// Symbol can be discovered
const symbols = Object.getOwnPropertySymbols(obj);
console.log(obj[symbols[0]]); // 'top secret'

// All property keys including symbols
console.log(Reflect.ownKeys(obj)); // [Symbol(secret)]
```

### Performance Considerations
```javascript
// Symbol property access is slower than string property access
const symbolKey = Symbol('key');
const stringKey = 'key';

const obj = {
    [symbolKey]: 'symbol value',
    [stringKey]: 'string value'
};

// Benchmark (simplified example)
function benchmarkAccess(obj, key, iterations = 1000000) {
    const start = performance.now();
    
    for (let i = 0; i < iterations; i++) {
        const value = obj[key];
    }
    
    return performance.now() - start;
}

// String access is typically faster
const stringTime = benchmarkAccess(obj, stringKey);
const symbolTime = benchmarkAccess(obj, symbolKey);

console.log(`String access: ${stringTime}ms`);
console.log(`Symbol access: ${symbolTime}ms`);
```

### JSON Serialization Issues
```javascript
const DATA_KEY = Symbol('data');

const obj = {
    name: 'John',
    [DATA_KEY]: 'important data'
};

// Symbols are lost in JSON serialization
const json = JSON.stringify(obj);
console.log(json); // {"name":"John"}

const parsed = JSON.parse(json);
console.log(parsed[DATA_KEY]); // undefined

// Custom serialization needed
function serializeWithSymbols(obj) {
    const regular = JSON.parse(JSON.stringify(obj));
    const symbols = Object.getOwnPropertySymbols(obj);
    
    if (symbols.length > 0) {
        regular.__symbols = {};
        symbols.forEach(sym => {
            regular.__symbols[sym.toString()] = obj[sym];
        });
    }
    
    return JSON.stringify(regular);
}
```

## Best Practices

### When to Use Symbols
```javascript
// ✅ Good use cases for symbols

// 1. Creating unique constants
const STATUS_PENDING = Symbol('pending');
const STATUS_COMPLETE = Symbol('complete');
const STATUS_FAILED = Symbol('failed');

// 2. Adding metadata to objects without naming conflicts
const CREATED_AT = Symbol('createdAt');
const LAST_MODIFIED = Symbol('lastModified');

// 3. Creating private-like properties
const PRIVATE_STATE = Symbol('privateState');

// 4. Implementing well-known symbol protocols
class Collection {
    [Symbol.iterator]() {
        // Make object iterable
    }
    
    [Symbol.toStringTag] = 'Collection';
}

// 5. Plugin/extension systems
const PLUGIN_METADATA = Symbol('pluginMetadata');
```

### Symbol Naming Conventions
```javascript
// ✅ Good symbol naming
const USER_ID = Symbol('userId');
const CACHE_KEY = Symbol('cache key');
const PRIVATE_METHOD = Symbol('private method');

// ✅ Namespaced global symbols
const APP_USER_TOKEN = Symbol.for('myapp.user.token');
const APP_SESSION_ID = Symbol.for('myapp.session.id');

// ❌ Avoid unclear names
const SYM1 = Symbol();
const X = Symbol('x');

// ✅ Document symbol purpose
/**
 * Used to store private user authentication data
 * @type {symbol}
 */
const AUTH_DATA = Symbol('authentication data');
```

### Symbol Storage and Access Patterns
```javascript
// ✅ Export symbols for controlled access
// symbols.js
export const USER_ROLE = Symbol('userRole');
export const PERMISSIONS = Symbol('permissions');

// user.js
import { USER_ROLE, PERMISSIONS } from './symbols.js';

class User {
    constructor(name, role, permissions) {
        this.name = name;
        this[USER_ROLE] = role;
        this[PERMISSIONS] = permissions;
    }
    
    hasPermission(permission) {
        return this[PERMISSIONS].includes(permission);
    }
}

// ✅ Symbol registry for cross-module communication
class SymbolRegistry {
    static symbols = new Map();
    
    static get(name) {
        if (!this.symbols.has(name)) {
            this.symbols.set(name, Symbol(name));
        }
        return this.symbols.get(name);
    }
    
    static global(name) {
        return Symbol.for(name);
    }
}

// Usage across modules
const USER_DATA = SymbolRegistry.get('userData');
const SHARED_STATE = SymbolRegistry.global('app.shared.state');
```

### Error Handling
```javascript
// Safe symbol property access
function getSymbolProperty(obj, symbol, defaultValue) {
    try {
        return obj[symbol];
    } catch (error) {
        console.warn('Symbol property access failed:', error);
        return defaultValue;
    }
}

// Validate symbol usage
function validateSymbol(sym) {
    if (typeof sym !== 'symbol') {
        throw new TypeError('Expected a symbol');
    }
    
    return sym;
}

// Symbol-based feature detection
function hasSymbolSupport() {
    return typeof Symbol !== 'undefined' && 
           typeof Symbol.iterator !== 'undefined';
}

if (hasSymbolSupport()) {
    // Use symbol features
} else {
    // Fallback for older environments
}
```

### Key Takeaways

1. **Symbols are unique primitives** - Each Symbol() call creates a unique identifier
2. **Global registry enables sharing** - Symbol.for() creates or retrieves globally shared symbols
3. **Symbols are non-enumerable** - They don't appear in for...in loops or Object.keys()
4. **Well-known symbols customize behavior** - Built-in symbols control JavaScript language features
5. **Not truly private** - Symbols can be discovered with Object.getOwnPropertySymbols()
6. **Performance overhead** - Symbol property access is slower than string properties
7. **JSON doesn't support symbols** - Custom serialization needed to preserve symbol properties
8. **Great for metadata** - Ideal for adding non-conflicting properties to objects
9. **Plugin systems benefit** - Symbols provide clean extension mechanisms
10. **Use descriptions for debugging** - Symbol descriptions help during development 