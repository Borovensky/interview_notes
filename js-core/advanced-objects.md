# Advanced JavaScript Objects Guide

## Table of Contents
- [Property Names with Spaces](#property-names-with-spaces)
- [Property Enumeration Checking](#property-enumeration-checking)
- [Property Existence with in Operator](#property-existence-with-in-operator)
- [JSON Serialization Control](#json-serialization-control)
- [Object String Conversion](#object-string-conversion)
- [Custom JSON Serialization](#custom-json-serialization)
- [Primitive Conversion with valueOf](#primitive-conversion-with-valueof)
- [Precise Equality with Object.is](#precise-equality-with-objectis)
- [Super Keyword and Prototype Methods](#super-keyword-and-prototype-methods)
- [Advanced Object Patterns](#advanced-object-patterns)
- [Type Conversion Deep Dive](#type-conversion-deep-dive)
- [Best Practices](#best-practices)

## Property Names with Spaces

JavaScript allows property names with spaces and special characters when using bracket notation.

### Basic Usage
Based on your example:

```javascript
const data = {
    'type information': null // можно использовать пробелы (can use spaces)
};
console.log(data['type information']); // null
```

### Complex Property Names
```javascript
const obj = {
    'normal property': 'value1',
    'property with spaces': 'value2',
    'property-with-dashes': 'value3',
    'property.with.dots': 'value4',
    '123numeric': 'value5',
    '': 'empty string key',
    'multi\nline\nkey': 'multiline value'
};

// Access with bracket notation
console.log(obj['property with spaces']); // 'value2'
console.log(obj['property-with-dashes']); // 'value3'
console.log(obj['123numeric']); // 'value5'
console.log(obj['']); // 'empty string key'

// Cannot use dot notation for these
// console.log(obj.property with spaces); // SyntaxError
// console.log(obj.123numeric); // SyntaxError

// But normal properties work with both
console.log(obj['normal property']); // 'value1'
// obj.normalProperty would be different property!
```

### Dynamic Property Names
```javascript
const dynamicObj = {};
const propertyName = 'dynamic property name';
const computedName = 'computed-' + Date.now();

// Setting properties dynamically
dynamicObj[propertyName] = 'dynamic value';
dynamicObj[computedName] = 'computed value';

// Using symbols as property keys
const symbolKey = Symbol('description');
dynamicObj[symbolKey] = 'symbol value';

// Computed property names in object literals (ES6)
const prefix = 'app';
const settings = {
    [`${prefix}_theme`]: 'dark',
    [`${prefix}_language`]: 'en',
    [Symbol.iterator]: function* () {
        yield this[`${prefix}_theme`];
        yield this[`${prefix}_language`];
    }
};

console.log(settings.app_theme); // 'dark'
console.log([...settings]); // ['dark', 'en']
```

### Special Characters and Unicode
```javascript
const unicodeObj = {
    '🔑': 'key emoji',
    '属性': 'Chinese property',
    'свойство': 'Russian property',
    'خاصية': 'Arabic property',
    '$special': 'dollar sign',
    '_underscore': 'underscore',
    '@symbol': 'at symbol'
};

// All valid property names
Object.keys(unicodeObj).forEach(key => {
    console.log(`${key}: ${unicodeObj[key]}`);
});
```

## Property Enumeration Checking

Understanding which properties appear in enumeration is crucial for object iteration.

### propertyIsEnumerable Method
Based on your example:

```javascript
Object.prototype.propertyIsEnumerable('toString'); // false - проверяем является ли свойство enumerable (устанавливается через дискриптор)
```

### Detailed Enumeration Checking
```javascript
const obj = {
    visible: 'appears in loops',
    hidden: 'does not appear'
};

// Make 'hidden' non-enumerable
Object.defineProperty(obj, 'hidden', {
    enumerable: false
});

// Check enumerability
console.log(obj.propertyIsEnumerable('visible')); // true
console.log(obj.propertyIsEnumerable('hidden')); // false
console.log(obj.propertyIsEnumerable('toString')); // false (inherited)
console.log(obj.propertyIsEnumerable('nonexistent')); // false

// Built-in properties are typically non-enumerable
const arr = [1, 2, 3];
console.log(arr.propertyIsEnumerable('length')); // false
console.log(arr.propertyIsEnumerable('0')); // true (array indices)
console.log(arr.propertyIsEnumerable('push')); // false (inherited method)
```

### Enumeration vs Visibility
```javascript
const obj = {
    enumerable: 'visible in for...in',
    nonEnumerable: 'hidden from for...in'
};

Object.defineProperty(obj, 'nonEnumerable', {
    enumerable: false
});

// Different ways properties appear
console.log('for...in loop:');
for (const key in obj) {
    console.log(key); // Only 'enumerable'
}

console.log('Object.keys():', Object.keys(obj)); // ['enumerable']
console.log('Object.getOwnPropertyNames():', Object.getOwnPropertyNames(obj)); // ['enumerable', 'nonEnumerable']

// JSON.stringify only includes enumerable properties
console.log('JSON.stringify():', JSON.stringify(obj)); // {"enumerable":"visible in for...in"}

// But property still exists and accessible
console.log('Direct access:', obj.nonEnumerable); // "hidden from for...in"
```

### Prototype Chain Enumeration
```javascript
function Parent() {
    this.parentOwn = 'parent own property';
}
Parent.prototype.parentProto = 'parent prototype property';

function Child() {
    Parent.call(this);
    this.childOwn = 'child own property';
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.childProto = 'child prototype property';

const child = new Child();

// Check enumerability across prototype chain
console.log(child.propertyIsEnumerable('childOwn')); // true (own property)
console.log(child.propertyIsEnumerable('parentOwn')); // true (own property)
console.log(child.propertyIsEnumerable('childProto')); // false (inherited)
console.log(child.propertyIsEnumerable('parentProto')); // false (inherited)

// for...in includes inherited enumerable properties
console.log('for...in includes:');
for (const key in child) {
    console.log(`${key}: ${child.propertyIsEnumerable(key) ? 'own' : 'inherited'}`);
}
```

## Property Existence with in Operator

The `in` operator checks for property existence throughout the prototype chain.

### Basic in Operator Usage
Based on your example:

```javascript
console.log('toString' in data); // true - оператор in чекает все свойства, даже те которые находятся в прототипах! (он их не переберает - просто проверяет на наличие)
```

### in vs hasOwnProperty vs Object.hasOwn
```javascript
function Parent() {}
Parent.prototype.inheritedProp = 'inherited';

function Child() {
    this.ownProp = 'own';
}
Child.prototype = Object.create(Parent.prototype);

const child = new Child();

// Different ways to check property existence
console.log('ownProp' in child); // true
console.log('inheritedProp' in child); // true
console.log('toString' in child); // true (from Object.prototype)
console.log('nonexistent' in child); // false

console.log(child.hasOwnProperty('ownProp')); // true
console.log(child.hasOwnProperty('inheritedProp')); // false
console.log(child.hasOwnProperty('toString')); // false

// Modern alternative (ES2022)
console.log(Object.hasOwn(child, 'ownProp')); // true
console.log(Object.hasOwn(child, 'inheritedProp')); // false

// Checking specific prototype levels
console.log(Child.prototype.hasOwnProperty('constructor')); // true
console.log(Parent.prototype.hasOwnProperty('inheritedProp')); // true
```

### Practical Applications
```javascript
// Feature detection
function supportsLocalStorage() {
    return 'localStorage' in window && window.localStorage !== null;
}

// API availability checking
function checkAPI(obj, apiName) {
    if (apiName in obj) {
        console.log(`${apiName} is available`);
        return typeof obj[apiName] === 'function';
    }
    return false;
}

// Safe property access with inheritance check
function getPropertySource(obj, prop) {
    if (!(prop in obj)) {
        return 'not found';
    }
    
    if (Object.hasOwn(obj, prop)) {
        return 'own property';
    }
    
    // Walk up prototype chain to find where property is defined
    let current = Object.getPrototypeOf(obj);
    while (current) {
        if (Object.hasOwn(current, prop)) {
            return `inherited from ${current.constructor.name || 'Anonymous'}`;
        }
        current = Object.getPrototypeOf(current);
    }
    
    return 'found but source unknown';
}

const obj = { own: 'value' };
console.log(getPropertySource(obj, 'own')); // "own property"
console.log(getPropertySource(obj, 'toString')); // "inherited from Object"
console.log(getPropertySource(obj, 'missing')); // "not found"
```

### Checking for Undefined vs Non-existent
```javascript
const obj = {
    prop1: 'value',
    prop2: null,
    prop3: undefined
};

// Value checks
console.log(obj.prop1); // 'value'
console.log(obj.prop2); // null
console.log(obj.prop3); // undefined
console.log(obj.prop4); // undefined

// Existence checks
console.log('prop1' in obj); // true
console.log('prop2' in obj); // true
console.log('prop3' in obj); // true
console.log('prop4' in obj); // false

// Distinguishing undefined value from non-existent property
function hasDefinedProperty(obj, prop) {
    return prop in obj && obj[prop] !== undefined;
}

console.log(hasDefinedProperty(obj, 'prop1')); // true
console.log(hasDefinedProperty(obj, 'prop2')); // false (null)
console.log(hasDefinedProperty(obj, 'prop3')); // false (undefined)
console.log(hasDefinedProperty(obj, 'prop4')); // false (doesn't exist)
```

## JSON Serialization Control

JSON.stringify provides powerful options for controlling serialization.

### Replacer Function and Spacing
Based on your example:

```javascript
const obj = {a: 1, b: 2, c: 3};
const serialized = JSON.stringify(obj, function(key, value) {
    if (value < 2) return void 0; // Remove values less than 2
    return value;
}, 4); // 4 пробела в выводе (4 spaces in output)

console.log(serialized);
// без 4 => {"b":2,"c":3}
// c 4 =>
// {
//     "b": 2,
//     "c": 3
// }
```

### Replacer Function Patterns
```javascript
const data = {
    name: 'John',
    age: 30,
    password: 'secret123',
    email: 'john@example.com',
    preferences: {
        theme: 'dark',
        notifications: true
    }
};

// Filter out sensitive data
const publicJSON = JSON.stringify(data, function(key, value) {
    // First call has empty key for root object
    if (key === '') return value;
    
    // Filter out sensitive properties
    if (key === 'password') return undefined;
    
    // Transform specific values
    if (key === 'email') return value.split('@')[0] + '@***';
    
    return value;
}, 2);

console.log(publicJSON);
// {
//   "name": "John",
//   "age": 30,
//   "email": "john@***",
//   "preferences": {
//     "theme": "dark",
//     "notifications": true
//   }
// }
```

### Replacer Array (Property Whitelist)
```javascript
const user = {
    id: 1,
    name: 'Alice',
    password: 'secret',
    email: 'alice@example.com',
    preferences: {
        theme: 'light',
        language: 'en'
    }
};

// Only include specific properties
const limitedJSON = JSON.stringify(user, ['id', 'name', 'email']);
console.log(limitedJSON); // {"id":1,"name":"Alice","email":"alice@example.com"}

// Nested properties need to be included explicitly
const nestedJSON = JSON.stringify(user, ['id', 'name', 'preferences', 'theme']);
console.log(nestedJSON); // {"id":1,"name":"Alice","preferences":{"theme":"light"}}
```

### Advanced Replacer Patterns
```javascript
// Type-based filtering
function createTypeFilter(allowedTypes) {
    return function(key, value) {
        if (key === '') return value; // Root object
        
        const type = typeof value;
        if (!allowedTypes.includes(type)) {
            return `[${type} filtered]`;
        }
        return value;
    };
}

const mixed = {
    string: 'text',
    number: 42,
    boolean: true,
    func: () => 'hello',
    symbol: Symbol('test'),
    date: new Date()
};

const numbersAndStrings = JSON.stringify(mixed, createTypeFilter(['string', 'number']), 2);
console.log(numbersAndStrings);

// Circular reference handler
function createCircularReplacer() {
    const seen = new WeakSet();
    return function(key, value) {
        if (typeof value === 'object' && value !== null) {
            if (seen.has(value)) {
                return '[Circular Reference]';
            }
            seen.add(value);
        }
        return value;
    };
}

const obj1 = { name: 'obj1' };
const obj2 = { name: 'obj2', ref: obj1 };
obj1.ref = obj2; // Create circular reference

const safeJSON = JSON.stringify(obj1, createCircularReplacer(), 2);
console.log(safeJSON);
```

## Object String Conversion

Objects can customize their string representation through the toString method.

### Custom toString Implementation
Based on your example:

```javascript
const person = {
    name: 'Bob',
    toString() {
        return 'person';
    }
};
console.log('Object - ' + person); // Object - person
// Без метода toString будет => Object - [object Object]
```

### toString in Different Contexts
```javascript
const customObject = {
    type: 'CustomType',
    id: 123,
    
    toString() {
        return `${this.type}#${this.id}`;
    }
};

// String concatenation
console.log('Item: ' + customObject); // "Item: CustomType#123"

// Template literals call toString
console.log(`Current item: ${customObject}`); // "Current item: CustomType#123"

// Explicit conversion
console.log(String(customObject)); // "CustomType#123"

// Alert/console methods
console.log(customObject); // CustomType#123 (in some environments)

// Comparison (calls toString)
console.log(customObject == 'CustomType#123'); // true
console.log(customObject === 'CustomType#123'); // false
```

### toString vs Symbol.toPrimitive
```javascript
const advancedObject = {
    value: 42,
    
    toString() {
        console.log('toString called');
        return `[Object: ${this.value}]`;
    },
    
    valueOf() {
        console.log('valueOf called');
        return this.value;
    },
    
    [Symbol.toPrimitive](hint) {
        console.log(`Symbol.toPrimitive called with hint: ${hint}`);
        switch (hint) {
            case 'number':
                return this.value;
            case 'string':
                return `[${this.value}]`;
            case 'default':
                return this.value;
            default:
                throw new Error('Invalid hint');
        }
    }
};

// Symbol.toPrimitive takes precedence
console.log(+advancedObject); // Symbol.toPrimitive with 'number'
console.log(String(advancedObject)); // Symbol.toPrimitive with 'string'
console.log(advancedObject + ''); // Symbol.toPrimitive with 'default'

// Remove Symbol.toPrimitive to see toString/valueOf behavior
delete advancedObject[Symbol.toPrimitive];

console.log(+advancedObject); // valueOf called
console.log(String(advancedObject)); // toString called
console.log(advancedObject + ''); // valueOf called (+ operator prefers number)
```

### Class-based toString
```javascript
class User {
    constructor(name, role = 'user') {
        this.name = name;
        this.role = role;
    }
    
    toString() {
        return `${this.role}: ${this.name}`;
    }
    
    // For debugging
    [Symbol.for('nodejs.util.inspect.custom')]() {
        return `User { name: '${this.name}', role: '${this.role}' }`;
    }
}

const admin = new User('Alice', 'admin');
console.log('Current user: ' + admin); // "Current user: admin: Alice"
console.log(admin.toString()); // "admin: Alice"
```

## Custom JSON Serialization

The toJSON method allows objects to control their JSON representation.

### Basic toJSON Implementation
Based on your example:

```javascript
const data2 = {
    name: 'Bob',
    age: 23,
    toJSON() {
        return {
            'value': this.name
        };
    }
};
console.log(JSON.stringify(data2)); // {"value":"Bob"}
```

### Advanced toJSON Patterns
```javascript
class User {
    constructor(name, email, password) {
        this.name = name;
        this.email = email;
        this._password = password;
        this.createdAt = new Date();
    }
    
    toJSON() {
        // Return only public data
        return {
            name: this.name,
            email: this.email,
            createdAt: this.createdAt.toISOString(),
            // Exclude password and other sensitive data
        };
    }
}

const user = new User('John', 'john@example.com', 'secret123');
console.log(JSON.stringify(user));
// {"name":"John","email":"john@example.com","createdAt":"2023-..."}
```

### Conditional Serialization
```javascript
class APIResponse {
    constructor(data, includeMetadata = false) {
        this.data = data;
        this.metadata = {
            timestamp: Date.now(),
            version: '1.0',
            requestId: Math.random().toString(36)
        };
        this.includeMetadata = includeMetadata;
    }
    
    toJSON() {
        const result = { data: this.data };
        
        if (this.includeMetadata) {
            result.metadata = this.metadata;
        }
        
        return result;
    }
}

const response1 = new APIResponse({ users: [] });
const response2 = new APIResponse({ users: [] }, true);

console.log(JSON.stringify(response1)); // {"data":{"users":[]}}
console.log(JSON.stringify(response2)); // {"data":{"users":[]},"metadata":{...}}
```

### Nested toJSON
```javascript
class Address {
    constructor(street, city, country) {
        this.street = street;
        this.city = city;
        this.country = country;
    }
    
    toJSON() {
        return `${this.street}, ${this.city}, ${this.country}`;
    }
}

class Person {
    constructor(name, address) {
        this.name = name;
        this.address = address;
        this.id = Math.random().toString(36);
    }
    
    toJSON() {
        return {
            name: this.name,
            address: this.address, // Will call address.toJSON()
            // Exclude internal ID
        };
    }
}

const person = new Person('Alice', new Address('123 Main St', 'NYC', 'USA'));
console.log(JSON.stringify(person));
// {"name":"Alice","address":"123 Main St, NYC, USA"}
```

### Dynamic toJSON
```javascript
function createSerializableObject(data, options = {}) {
    const obj = { ...data };
    
    obj.toJSON = function() {
        const result = {};
        
        for (const [key, value] of Object.entries(this)) {
            if (key === 'toJSON') continue; // Skip the toJSON method itself
            
            // Apply transformations based on options
            if (options.excludePrivate && key.startsWith('_')) {
                continue;
            }
            
            if (options.lowercaseKeys) {
                result[key.toLowerCase()] = value;
            } else {
                result[key] = value;
            }
        }
        
        return result;
    };
    
    return obj;
}

const obj = createSerializableObject({
    Name: 'John',
    _private: 'secret',
    public: 'visible'
}, { excludePrivate: true, lowercaseKeys: true });

console.log(JSON.stringify(obj)); // {"name":"John","public":"visible"}
```

## Primitive Conversion with valueOf

The valueOf method controls how objects convert to primitive values.

### Basic valueOf Usage
Based on your example:

```javascript
const wallet = {
    amount: 150,
    valueOf() {
        return this.amount;
    }
};
console.log(wallet > 100); // true
```

### valueOf in Different Contexts
```javascript
const price = {
    value: 29.99,
    currency: 'USD',
    
    valueOf() {
        return this.value;
    },
    
    toString() {
        return `$${this.value}`;
    }
};

// Numeric operations use valueOf
console.log(price + 10); // 39.99
console.log(price * 2); // 59.98
console.log(price < 30); // true
console.log(+price); // 29.99

// String operations use toString
console.log('Price: ' + price); // "Price: $29.99"
console.log(`Cost: ${price}`); // "Cost: $29.99"

// Some operations prefer valueOf over toString
console.log(price + ''); // "29.99" (uses valueOf, then converts to string)
console.log(String(price)); // "$29.99" (explicitly calls toString)
```

### Complex valueOf Examples
```javascript
class Duration {
    constructor(milliseconds) {
        this.ms = milliseconds;
    }
    
    valueOf() {
        return this.ms;
    }
    
    toString() {
        const seconds = Math.floor(this.ms / 1000);
        const minutes = Math.floor(seconds / 60);
        const hours = Math.floor(minutes / 60);
        
        if (hours > 0) {
            return `${hours}h ${minutes % 60}m ${seconds % 60}s`;
        } else if (minutes > 0) {
            return `${minutes}m ${seconds % 60}s`;
        } else {
            return `${seconds}s`;
        }
    }
}

const duration1 = new Duration(5000); // 5 seconds
const duration2 = new Duration(65000); // 1 minute 5 seconds

// Numeric comparison uses valueOf
console.log(duration1 < duration2); // true
console.log(duration1 + duration2); // 70000

// String conversion uses toString
console.log(`Duration: ${duration1}`); // "Duration: 5s"
console.log(`Total: ${duration2}`); // "Total: 1m 5s"
```

### valueOf with Date Objects
```javascript
// Date objects have valueOf that returns timestamp
const date1 = new Date('2023-01-01');
const date2 = new Date('2023-12-31');

console.log(date1 < date2); // true (compares timestamps)
console.log(date2 - date1); // milliseconds difference

// Custom date-like object
class SimpleDate {
    constructor(year, month, day) {
        this.date = new Date(year, month - 1, day);
    }
    
    valueOf() {
        return this.date.getTime();
    }
    
    toString() {
        return this.date.toDateString();
    }
}

const customDate1 = new SimpleDate(2023, 1, 1);
const customDate2 = new SimpleDate(2023, 12, 31);

console.log(customDate1 < customDate2); // true
console.log(customDate2 - customDate1); // milliseconds difference
console.log(`Start: ${customDate1}`); // "Start: Sun Jan 01 2023"
```

## Precise Equality with Object.is

Object.is provides more precise equality comparison than == or ===.

### Object.is vs Other Comparisons
Based on your comprehensive examples:

```javascript
// Zero comparison
console.log(+0 == -0);            // true
console.log(+0 === -0);           // true
console.log(Object.is(+0, -0));   // false - более точное сравнение!

// NaN comparison
console.log(NaN == NaN);          // false
console.log(NaN === NaN);         // false
console.log(Object.is(NaN, NaN)); // true - более точное сравнение!

// Regular values
console.log(5 == 5);              // true
console.log(5 === 5);             // true
console.log(5 == '5');            // true (type coercion)
console.log(5 === '5');           // false (no coercion)
console.log(Object.is(5, 5));     // true
console.log(Object.is(5, '5'));   // false (no coercion)
```

### Practical Applications of Object.is
```javascript
// Safe equality checking function
function safeEquals(a, b) {
    return Object.is(a, b);
}

// Array searching with NaN support
function findIndexStrict(array, searchElement) {
    for (let i = 0; i < array.length; i++) {
        if (Object.is(array[i], searchElement)) {
            return i;
        }
    }
    return -1;
}

const arr = [1, NaN, 3, -0, 5];
console.log(findIndexStrict(arr, NaN)); // 1
console.log(findIndexStrict(arr, -0)); // 3
console.log(findIndexStrict(arr, +0)); // -1

// Built-in Array.includes uses Object.is internally for NaN
console.log(arr.includes(NaN)); // true
console.log(arr.indexOf(NaN)); // -1 (indexOf uses ===)
```

### Object.is in Map and Set
```javascript
// Map uses Object.is for key comparison
const map = new Map();
map.set(NaN, 'not a number');
map.set(+0, 'positive zero');
map.set(-0, 'negative zero');

console.log(map.get(NaN)); // 'not a number'
console.log(map.get(+0)); // 'negative zero' (overwrote +0)
console.log(map.get(-0)); // 'negative zero'
console.log(map.has(NaN)); // true

// Set uses Object.is for value comparison
const set = new Set([NaN, +0, -0]);
console.log(set.size); // 2 (NaN and one zero)
console.log(set.has(NaN)); // true
console.log(set.has(+0)); // true
console.log(set.has(-0)); // true
```

### Custom Equality Functions
```javascript
// Different equality strategies
const equalityStrategies = {
    loose: (a, b) => a == b,
    strict: (a, b) => a === b,
    sameValue: (a, b) => Object.is(a, b),
    sameValueZero: (a, b) => a === b || (Number.isNaN(a) && Number.isNaN(b))
};

function testEquality(a, b) {
    console.log(`Comparing ${a} and ${b}:`);
    Object.entries(equalityStrategies).forEach(([name, fn]) => {
        console.log(`  ${name}: ${fn(a, b)}`);
    });
}

testEquality(+0, -0);
testEquality(NaN, NaN);
testEquality(5, '5');
```

## Super Keyword and Prototype Methods

The super keyword provides clean access to parent object methods.

### Modern super vs Legacy Prototype Access
Based on your example:

```javascript
const person1 = {
    getType() {
        return 'person';
    }
};

const friend = {
    __proto__: person1,
    print() {
        console.log(super.getType()); // super всегда указывает на прототип
        // console.log(Object.getPrototypeOf(this).getType.call(this)); // более старый вариант
    }
    // print: function() { // если будет function присваиваться к методу...то super работать не будет
    //     console.log(super.getType()); 
    // }
};
friend.print(); // 'person'
```

### super in Object Literals
```javascript
const animal = {
    name: 'Animal',
    speak() {
        return `${this.name} makes a sound`;
    },
    describe() {
        return `This is a ${this.name}`;
    }
};

const dog = {
    __proto__: animal,
    name: 'Dog',
    
    speak() {
        return super.speak() + ' - Woof!';
    },
    
    // Method shorthand is required for super to work
    describe() {
        return super.describe() + ' and it\'s friendly';
    }
    
    // This would NOT work:
    // describe: function() {
    //     return super.describe() + ' and it\'s friendly'; // SyntaxError
    // }
};

console.log(dog.speak()); // "Dog makes a sound - Woof!"
console.log(dog.describe()); // "This is a Dog and it's friendly"
```

### super in Classes
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
    
    static getSpecies() {
        return 'Unknown species';
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Call parent constructor
        this.breed = breed;
    }
    
    speak() {
        return super.speak() + ' - Woof!'; // Call parent method
    }
    
    getInfo() {
        return `${super.speak()}, breed: ${this.breed}`;
    }
    
    static getSpecies() {
        return super.getSpecies() + ' - Canine'; // Call parent static method
    }
}

const dog = new Dog('Rex', 'German Shepherd');
console.log(dog.speak()); // "Rex makes a sound - Woof!"
console.log(dog.getInfo()); // "Rex makes a sound, breed: German Shepherd"
console.log(Dog.getSpecies()); // "Unknown species - Canine"
```

### super vs Explicit Prototype Access
```javascript
const parent = {
    value: 'parent',
    getValue() {
        return this.value;
    }
};

const child = {
    __proto__: parent,
    value: 'child',
    
    // Modern approach with super
    getValueSuper() {
        return super.getValue(); // Clean and clear
    },
    
    // Legacy approach
    getValueLegacy() {
        return Object.getPrototypeOf(this).getValue.call(this);
    },
    
    // Wrong approach (doesn't bind 'this' correctly)
    getValueWrong() {
        return Object.getPrototypeOf(this).getValue(); // 'this' is parent, not child
    }
};

console.log(child.getValueSuper()); // 'child' (correct)
console.log(child.getValueLegacy()); // 'child' (correct but verbose)
console.log(child.getValueWrong()); // 'parent' (incorrect!)
```

### super with Getters and Setters
```javascript
const base = {
    get value() {
        return this._value || 'default';
    },
    
    set value(val) {
        console.log('Base setter called');
        this._value = val;
    }
};

const derived = {
    __proto__: base,
    
    get value() {
        return `[${super.value}]`; // Call parent getter
    },
    
    set value(val) {
        console.log('Derived setter called');
        super.value = val.toUpperCase(); // Call parent setter
    }
};

derived.value = 'hello';
// "Derived setter called"
// "Base setter called"
console.log(derived.value); // "[HELLO]"
```

## Advanced Object Patterns

### Mixins with super
```javascript
const Loggable = {
    log(message) {
        console.log(`[${this.constructor.name}] ${message}`);
    }
};

const Timestamped = {
    getTimestamp() {
        return new Date().toISOString();
    },
    
    logWithTime(message) {
        super.log(`${this.getTimestamp()}: ${message}`);
    }
};

// Set up prototype chain: Timestamped -> Loggable
Object.setPrototypeOf(Timestamped, Loggable);

const MyObject = {
    __proto__: Timestamped,
    name: 'MyObject',
    
    doSomething() {
        this.logWithTime('Doing something important');
    }
};

MyObject.doSomething(); // [Object] 2023-...: Doing something important
```

### Factory Pattern with Prototypes
```javascript
function createUser(name, role) {
    const UserPrototype = {
        getName() {
            return this.name;
        },
        
        getRole() {
            return this.role;
        },
        
        toString() {
            return `${this.getRole()}: ${this.getName()}`;
        }
    };
    
    const AdminPrototype = {
        __proto__: UserPrototype,
        
        getRole() {
            return `Admin (${super.getRole()})`;
        },
        
        hasAdminAccess() {
            return true;
        }
    };
    
    const prototype = role === 'admin' ? AdminPrototype : UserPrototype;
    
    return Object.create(prototype, {
        name: { value: name, enumerable: true },
        role: { value: role, enumerable: true }
    });
}

const user = createUser('Alice', 'user');
const admin = createUser('Bob', 'admin');

console.log(user.toString()); // "user: Alice"
console.log(admin.toString()); // "Admin (admin): Bob"
console.log(admin.hasAdminAccess()); // true
```

## Type Conversion Deep Dive

### Conversion Order and Precedence
```javascript
const conversionTest = {
    name: 'ConversionTest',
    
    toString() {
        console.log('toString called');
        return 'stringValue';
    },
    
    valueOf() {
        console.log('valueOf called');
        return 42;
    },
    
    [Symbol.toPrimitive](hint) {
        console.log(`Symbol.toPrimitive called with hint: ${hint}`);
        switch (hint) {
            case 'number': return 100;
            case 'string': return 'primitiveString';
            case 'default': return 'defaultValue';
        }
    }
};

// Test different conversion contexts
console.log('Testing conversions:');
console.log(Number(conversionTest)); // Symbol.toPrimitive with 'number'
console.log(String(conversionTest)); // Symbol.toPrimitive with 'string'
console.log(conversionTest + ''); // Symbol.toPrimitive with 'default'
console.log(+conversionTest); // Symbol.toPrimitive with 'number'

// Remove Symbol.toPrimitive to test toString/valueOf
delete conversionTest[Symbol.toPrimitive];

console.log('\nWithout Symbol.toPrimitive:');
console.log(Number(conversionTest)); // valueOf called
console.log(String(conversionTest)); // toString called
console.log(conversionTest + ''); // valueOf called (+ prefers number)
console.log(conversionTest == 42); // valueOf called
```

### Custom Conversion Logic
```javascript
class SmartNumber {
    constructor(value) {
        this.value = value;
    }
    
    [Symbol.toPrimitive](hint) {
        switch (hint) {
            case 'number':
                return Number(this.value);
            case 'string':
                return String(this.value);
            case 'default':
                // For most operators, behave like a number
                return Number(this.value);
            default:
                throw new Error(`Invalid conversion hint: ${hint}`);
        }
    }
    
    toString() {
        return `SmartNumber(${this.value})`;
    }
}

const smart = new SmartNumber('42');
console.log(smart + 8); // 50 (number conversion)
console.log(`Value: ${smart}`); // "Value: 42" (string conversion)
console.log(smart == 42); // true (default conversion)
console.log(smart === 42); // false (no conversion)
```

## Best Practices

### Object Property Management
```javascript
// ✅ Good practices for property access
function safePropertyAccess(obj, path, defaultValue) {
    const keys = path.split('.');
    let current = obj;
    
    for (const key of keys) {
        if (current == null || !(key in current)) {
            return defaultValue;
        }
        current = current[key];
    }
    
    return current;
}

const data = {
    user: {
        profile: {
            name: 'John'
        }
    }
};

console.log(safePropertyAccess(data, 'user.profile.name', 'Unknown')); // 'John'
console.log(safePropertyAccess(data, 'user.profile.age', 'Unknown')); // 'Unknown'

// ✅ Enumeration best practices
function getPublicProperties(obj) {
    const result = {};
    
    for (const key in obj) {
        if (obj.hasOwnProperty(key) && !key.startsWith('_')) {
            result[key] = obj[key];
        }
    }
    
    return result;
}

// ✅ Safe JSON serialization
function safeJSONStringify(obj, space) {
    try {
        return JSON.stringify(obj, function(key, value) {
            // Handle circular references
            if (typeof value === 'object' && value !== null) {
                if (this._seen && this._seen.has(value)) {
                    return '[Circular]';
                }
                if (!this._seen) this._seen = new WeakSet();
                this._seen.add(value);
            }
            
            // Filter out functions and symbols
            if (typeof value === 'function' || typeof value === 'symbol') {
                return undefined;
            }
            
            return value;
        }, space);
    } catch (error) {
        return `[Serialization Error: ${error.message}]`;
    }
}
```

### Performance Considerations
```javascript
// Property existence checking performance
const obj = { a: 1, b: 2, c: 3 };
const iterations = 1000000;

console.time('hasOwnProperty');
for (let i = 0; i < iterations; i++) {
    obj.hasOwnProperty('a');
}
console.timeEnd('hasOwnProperty');

console.time('Object.hasOwn');
for (let i = 0; i < iterations; i++) {
    Object.hasOwn(obj, 'a');
}
console.timeEnd('Object.hasOwn');

console.time('in operator');
for (let i = 0; i < iterations; i++) {
    'a' in obj;
}
console.timeEnd('in operator');

// Generally: in operator > Object.hasOwn > hasOwnProperty
```

### Error Handling
```javascript
function robustObjectOperation(obj, operation) {
    try {
        // Validate input
        if (obj == null) {
            throw new Error('Object cannot be null or undefined');
        }
        
        if (typeof obj !== 'object') {
            throw new Error('Input must be an object');
        }
        
        return operation(obj);
    } catch (error) {
        console.error(`Object operation failed: ${error.message}`);
        return null;
    }
}

// Usage
const result = robustObjectOperation(someObj, (obj) => {
    return JSON.stringify(obj);
});
```

### Key Takeaways

1. **Property names can contain spaces** - Use bracket notation for special characters
2. **propertyIsEnumerable checks own enumerable properties** - Different from `in` operator
3. **`in` operator checks entire prototype chain** - Use for feature detection
4. **JSON.stringify accepts replacer and spacing** - Powerful for data filtering and formatting
5. **toString controls string conversion** - Used in string contexts and concatenation
6. **toJSON customizes JSON serialization** - Return what should be serialized
7. **valueOf controls primitive conversion** - Used in numeric contexts and operators
8. **Object.is provides precise equality** - Handles NaN and signed zeros correctly
9. **super requires method shorthand syntax** - Cannot use function expressions
10. **Conversion methods have precedence** - Symbol.toPrimitive > valueOf/toString