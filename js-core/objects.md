# JavaScript Objects Guide

## Table of Contents
- [Property Descriptors](#property-descriptors)
- [Object.defineProperty](#objectdefineproperty)
- [Enumerable Properties](#enumerable-properties)
- [Getters and Setters](#getters-and-setters)
- [Multiple Property Definition](#multiple-property-definition)
- [Property Introspection](#property-introspection)
- [Object Protection Methods](#object-protection-methods)
- [Object State Checking](#object-state-checking)
- [Advanced Object Concepts](#advanced-object-concepts)
- [Object Creation Patterns](#object-creation-patterns)
- [Best Practices](#best-practices)

## Property Descriptors

Property descriptors allow you to describe how a property will behave when certain operations are performed on it.

### Descriptor Properties
Based on your detailed explanation:

```javascript
// Property descriptors have these attributes:
// value — the value of the property
// writable — if true, the property value can be changed
// configurable — if true, the property can be deleted and descriptor can be changed
// enumerable — if true, the property will be enumerated in for...in loops and Object.keys
// get — function called when property is accessed
// set — function called when property is assigned
```

### Data Descriptors vs Accessor Descriptors
```javascript
// Data descriptor (has value and/or writable)
const dataDescriptor = {
    value: 'Hello',
    writable: true,
    enumerable: true,
    configurable: true
};

// Accessor descriptor (has get and/or set)
const accessorDescriptor = {
    get() { return this._value; },
    set(val) { this._value = val; },
    enumerable: true,
    configurable: true
};

// ❌ Cannot mix data and accessor descriptors
const invalidDescriptor = {
    value: 'Hello',
    get() { return 'World'; } // TypeError: Invalid property descriptor
};
```

### Default Descriptor Values
```javascript
const obj = {};

// When using Object.defineProperty without specifying attributes
Object.defineProperty(obj, 'prop1', {
    value: 'test'
    // writable: false (default)
    // enumerable: false (default)
    // configurable: false (default)
});

// When creating properties normally
obj.prop2 = 'test';
// This is equivalent to:
Object.defineProperty(obj, 'prop2', {
    value: 'test',
    writable: true,    // different default!
    enumerable: true,  // different default!
    configurable: true // different default!
});

console.log(Object.getOwnPropertyDescriptor(obj, 'prop1'));
// { value: 'test', writable: false, enumerable: false, configurable: false }

console.log(Object.getOwnPropertyDescriptor(obj, 'prop2'));
// { value: 'test', writable: true, enumerable: true, configurable: true }
```

## Object.defineProperty

### Basic Usage with Strict Mode
As you emphasized, strict mode is crucial for seeing errors:

```javascript
'use strict'; // Important! Without strict mode, you won't see errors!

var user = {};
Object.defineProperty(user, 'name', {
    value: 'Pitter',
    writable: false,
    configurable: false
});

// user.name = 'Alex'; // TypeError: Cannot assign to read only property 'name'
console.log(user.name); // 'Pitter'
```

### Writable Property
```javascript
'use strict';

const obj = {};
Object.defineProperty(obj, 'readOnly', {
    value: 'Cannot change me',
    writable: false
});

Object.defineProperty(obj, 'mutable', {
    value: 'Can change me',
    writable: true
});

console.log(obj.readOnly); // 'Cannot change me'
// obj.readOnly = 'New value'; // TypeError in strict mode

obj.mutable = 'Changed!';
console.log(obj.mutable); // 'Changed!'
```

### Configurable Property
```javascript
'use strict';

const obj = {};
Object.defineProperty(obj, 'permanent', {
    value: 'Forever',
    configurable: false
});

Object.defineProperty(obj, 'flexible', {
    value: 'Changeable',
    configurable: true
});

// Can't delete non-configurable property
// delete obj.permanent; // TypeError: Cannot delete property 'permanent'

// Can't redefine non-configurable property
// Object.defineProperty(obj, 'permanent', {
//     value: 'New value'
// }); // TypeError: Cannot redefine property: permanent

// But configurable properties can be deleted and redefined
delete obj.flexible; // OK
console.log(obj.flexible); // undefined

// Redefine flexible property
Object.defineProperty(obj, 'flexible', {
    value: 'Redefined',
    writable: false
});
```

### Special Cases with Configurable
```javascript
'use strict';

const obj = {};
Object.defineProperty(obj, 'special', {
    value: 'test',
    writable: true,
    configurable: false
});

// Even though configurable is false, you can:
// 1. Change writable from true to false (but not false to true)
Object.defineProperty(obj, 'special', {
    writable: false
}); // OK

// 2. Change the value if writable is true
obj.special = 'new value'; // This worked before we set writable to false

// But now:
// obj.special = 'another value'; // TypeError: Cannot assign to read only property
```

## Enumerable Properties

### Enumerable Behavior in Loops
Based on your example:

```javascript
var person = {
    name: 'Petter',
    toString() {
        return this.name;
    }
};

// Initially, toString is enumerable
for (var key in person) {
    console.log(key); // 'name', 'toString'
}

// Make toString non-enumerable
Object.defineProperty(person, 'toString', {
    enumerable: false
});

// Now toString won't appear in enumeration
for (var key in person) {
    console.log(key); // Only 'name'
}

console.log(Object.keys(person)); // ['name']
console.log(person.toString()); // Still works! 'Petter'
```

### Enumerable vs Non-Enumerable Detection
```javascript
const obj = {
    visible: 'I appear in loops',
    hidden: 'I am hidden'
};

Object.defineProperty(obj, 'hidden', {
    enumerable: false
});

// Different ways to get properties
console.log(Object.keys(obj)); // ['visible']
console.log(Object.getOwnPropertyNames(obj)); // ['visible', 'hidden']
console.log(Object.getOwnPropertyDescriptors(obj));
// {
//   visible: { value: 'I appear in loops', writable: true, enumerable: true, configurable: true },
//   hidden: { value: 'I am hidden', writable: true, enumerable: false, configurable: true }
// }

// for...in only shows enumerable properties
for (const key in obj) {
    console.log(key); // Only 'visible'
}

// JSON.stringify only includes enumerable properties
console.log(JSON.stringify(obj)); // {"visible":"I appear in loops"}
```

### Built-in Non-Enumerable Properties
```javascript
const arr = [1, 2, 3];

// Array length is non-enumerable
for (const key in arr) {
    console.log(key); // '0', '1', '2' (not 'length')
}

console.log(Object.keys(arr)); // ['0', '1', '2']
console.log(Object.getOwnPropertyNames(arr)); // ['0', '1', '2', 'length']

// Check if property is enumerable
console.log(Object.propertyIsEnumerable.call(arr, 'length')); // false
console.log(arr.propertyIsEnumerable('0')); // true
```

## Getters and Setters

### Object Literal Syntax
Based on your getter/setter example:

```javascript
var user = {
    firstName: null,
    lastName: null,
    
    get fullName() {
        return this.firstName + ' ' + this.lastName;
    },
    
    set fullName(value) {
        var split = value.split(' ');
        this.firstName = split[0];
        this.lastName = split[1];
    }
};

user.fullName = "Alex Leon";
console.log(user.firstName); // 'Alex'
console.log(user.lastName);  // 'Leon'
console.log(user.fullName);  // 'Alex Leon'
```

### defineProperty with Getters/Setters
```javascript
const user = {
    _age: 0
};

Object.defineProperty(user, 'age', {
    get() {
        console.log('Getting age');
        return this._age;
    },
    
    set(value) {
        console.log('Setting age to', value);
        if (value < 0) {
            throw new Error('Age cannot be negative');
        }
        this._age = value;
    },
    
    enumerable: true,
    configurable: true
});

user.age = 25; // 'Setting age to 25'
console.log(user.age); // 'Getting age', then 25
// user.age = -5; // Error: Age cannot be negative
```

### Computed Properties with Getters
```javascript
class Rectangle {
    constructor(width, height) {
        this.width = width;
        this.height = height;
    }
    
    get area() {
        return this.width * this.height;
    }
    
    get perimeter() {
        return 2 * (this.width + this.height);
    }
    
    set dimensions({width, height}) {
        this.width = width;
        this.height = height;
    }
}

const rect = new Rectangle(10, 5);
console.log(rect.area); // 50
console.log(rect.perimeter); // 30

rect.dimensions = {width: 20, height: 10};
console.log(rect.area); // 200
```

### Lazy Loading with Getters
```javascript
const expensiveObject = {
    get expensiveProperty() {
        if (!this._expensiveValue) {
            console.log('Computing expensive value...');
            this._expensiveValue = this.computeExpensiveValue();
        }
        return this._expensiveValue;
    },
    
    computeExpensiveValue() {
        // Simulate expensive computation
        let result = 0;
        for (let i = 0; i < 1000000; i++) {
            result += Math.random();
        }
        return result;
    }
};

// First access - computes the value
console.log(expensiveObject.expensiveProperty); // 'Computing expensive value...'

// Subsequent accesses - returns cached value
console.log(expensiveObject.expensiveProperty); // No computation
```

## Multiple Property Definition

### Object.defineProperties
Based on your example:

```javascript
var user = {};
Object.defineProperties(user, {
    firstName: { 
        value: 'Bob',
        writable: true,
        enumerable: true,
        configurable: true
    },
    lastName: { 
        value: 'Lian',
        writable: true,
        enumerable: true,
        configurable: true
    },
    fullName: { 
        get: function() { 
            return this.firstName + ' ' + this.lastName; 
        },
        enumerable: true,
        configurable: true
    }
});

console.log(user.fullName); // 'Bob Lian'
```

### Mixed Descriptors
```javascript
const obj = {};
Object.defineProperties(obj, {
    // Data property
    name: {
        value: 'JavaScript',
        writable: false,
        enumerable: true
    },
    
    // Accessor property
    upperName: {
        get() { return this.name.toUpperCase(); },
        enumerable: true
    },
    
    // Hidden property
    secret: {
        value: 'hidden',
        enumerable: false
    },
    
    // Read-only with getter/setter
    version: {
        get() { return this._version || '1.0'; },
        set(val) { 
            if (typeof val === 'string') {
                this._version = val; 
            }
        },
        enumerable: true
    }
});

console.log(obj.name); // 'JavaScript'
console.log(obj.upperName); // 'JAVASCRIPT'
console.log(Object.keys(obj)); // ['name', 'upperName', 'version']
obj.version = '2.0';
console.log(obj.version); // '2.0'
```

## Property Introspection

### Object.getOwnPropertyNames vs Object.keys
Based on your example:

```javascript
var obj = {
    a: 1,
    b: 2,
    internal: 3,
};

Object.defineProperty(obj, 'internal', {
    enumerable: false,
});

console.log(Object.keys(obj)); // ['a', 'b'] - only enumerable
console.log(Object.getOwnPropertyNames(obj)); // ['a', 'b', 'internal'] - all own properties
```

### Complete Property Introspection
```javascript
const obj = {
    public: 'visible',
    [Symbol('sym')]: 'symbol property'
};

Object.defineProperty(obj, 'hidden', {
    value: 'not enumerable',
    enumerable: false
});

// Different ways to get properties
console.log(Object.keys(obj)); // ['public']
console.log(Object.getOwnPropertyNames(obj)); // ['public', 'hidden']
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(sym)]

// Get ALL own property keys
console.log(Reflect.ownKeys(obj)); // ['public', 'hidden', Symbol(sym)]

// Get property descriptors
console.log(Object.getOwnPropertyDescriptor(obj, 'public'));
// { value: 'visible', writable: true, enumerable: true, configurable: true }

console.log(Object.getOwnPropertyDescriptors(obj));
// Returns descriptors for all own properties
```

### Property Existence Checking
```javascript
const obj = {
    prop: 'value',
    nullProp: null,
    undefinedProp: undefined
};

Object.defineProperty(obj, 'hidden', {
    value: 'secret',
    enumerable: false
});

// Different ways to check property existence
console.log('prop' in obj); // true
console.log('hidden' in obj); // true
console.log('nonexistent' in obj); // false

console.log(obj.hasOwnProperty('prop')); // true
console.log(obj.hasOwnProperty('hidden')); // true
console.log(obj.hasOwnProperty('toString')); // false (inherited)

console.log(Object.hasOwn(obj, 'prop')); // true (newer alternative)
console.log(Object.hasOwn(obj, 'toString')); // false

// Value vs existence
console.log(obj.nullProp); // null
console.log('nullProp' in obj); // true
console.log(obj.undefinedProp); // undefined
console.log('undefinedProp' in obj); // true
console.log(obj.nonexistent); // undefined
console.log('nonexistent' in obj); // false
```

## Object Protection Methods

### Object.preventExtensions
Based on your example:

```javascript
var person = {
    name: 'Bob',
};

Object.preventExtensions(person);

person.name = 'Teta'; // OK - can modify existing properties
delete person.name; // OK - can delete properties
person.name = 'Bob'; // TypeError: Cannot add property - after deletion, can't add back!
// person.age = 19; // TypeError: Cannot add property age
```

### Detailed preventExtensions Behavior
```javascript
'use strict';

const obj = { existing: 'property' };
Object.preventExtensions(obj);

// ✅ Can modify existing properties
obj.existing = 'modified';
console.log(obj.existing); // 'modified'

// ✅ Can delete existing properties
delete obj.existing;
console.log(obj.existing); // undefined

// ❌ Cannot add new properties
// obj.newProp = 'value'; // TypeError: Cannot add property newProp

// ❌ Cannot re-add deleted properties
// obj.existing = 'back'; // TypeError: Cannot add property existing

// Check extensibility
console.log(Object.isExtensible(obj)); // false
```

### Object.seal
Based on your example:

```javascript
const person = { name: 'Bob' };
Object.seal(person);

person.name = 'Teta'; // OK - can modify values
// person.age = 19; // TypeError: Cannot add property age
// delete person.name; // TypeError: Cannot delete property name
```

### Object.freeze
Based on your example:

```javascript
const person = { name: 'Bob' };
Object.freeze(person);

// person.name = 'Teta'; // TypeError: Cannot assign to read only property
// person.age = 19; // TypeError: Cannot add property age
// delete person.name; // TypeError: Cannot delete property name
```

### Protection Levels Comparison
```javascript
const obj1 = { a: 1 };
const obj2 = { a: 1 };
const obj3 = { a: 1 };

// Level 1: preventExtensions
Object.preventExtensions(obj1);
obj1.a = 2;        // ✅ Can modify
delete obj1.a;     // ✅ Can delete
// obj1.b = 2;     // ❌ Cannot add

// Level 2: seal
Object.seal(obj2);
obj2.a = 2;        // ✅ Can modify
// delete obj2.a;  // ❌ Cannot delete
// obj2.b = 2;     // ❌ Cannot add

// Level 3: freeze
Object.freeze(obj3);
// obj3.a = 2;     // ❌ Cannot modify
// delete obj3.a;  // ❌ Cannot delete
// obj3.b = 2;     // ❌ Cannot add

console.log('preventExtensions allows:', 'modify ✅, delete ✅, add ❌');
console.log('seal allows:', 'modify ✅, delete ❌, add ❌');
console.log('freeze allows:', 'modify ❌, delete ❌, add ❌');
```

### Shallow Protection Warning
As you noted, these methods only work on one level:

```javascript
const obj = {
    level1: {
        level2: {
            deep: 'value'
        }
    }
};

Object.freeze(obj);

// ❌ Cannot modify top level
// obj.level1 = 'new'; // TypeError

// ✅ But can modify nested objects!
obj.level1.level2.deep = 'modified';
console.log(obj.level1.level2.deep); // 'modified'

// For deep freezing, you need a recursive approach
function deepFreeze(obj) {
    Object.getOwnPropertyNames(obj).forEach(prop => {
        const value = obj[prop];
        if (value && typeof value === 'object') {
            deepFreeze(value);
        }
    });
    return Object.freeze(obj);
}

const deepObj = {
    level1: {
        level2: { deep: 'value' }
    }
};

deepFreeze(deepObj);
// Now nested modifications will also fail
// deepObj.level1.level2.deep = 'modified'; // TypeError
```

## Object State Checking

### Checking Protection Status
Based on your reference:

```javascript
const obj = { name: 'test' };

// Initially extensible
console.log(Object.isExtensible(obj)); // true
console.log(Object.isSealed(obj)); // false
console.log(Object.isFrozen(obj)); // false

// After preventExtensions
Object.preventExtensions(obj);
console.log(Object.isExtensible(obj)); // false
console.log(Object.isSealed(obj)); // false (can still delete/modify)
console.log(Object.isFrozen(obj)); // false (can still modify)

// After seal
Object.seal(obj);
console.log(Object.isExtensible(obj)); // false
console.log(Object.isSealed(obj)); // true
console.log(Object.isFrozen(obj)); // false (can still modify values)

// After freeze
Object.freeze(obj);
console.log(Object.isExtensible(obj)); // false
console.log(Object.isSealed(obj)); // true
console.log(Object.isFrozen(obj)); // true
```

### Relationship Between States
```javascript
// isFrozen implies isSealed
// isSealed implies !isExtensible

function analyzeObject(obj, name) {
    const extensible = Object.isExtensible(obj);
    const sealed = Object.isSealed(obj);
    const frozen = Object.isFrozen(obj);
    
    console.log(`${name}:`);
    console.log(`  Extensible: ${extensible}`);
    console.log(`  Sealed: ${sealed}`);
    console.log(`  Frozen: ${frozen}`);
    
    // Logical relationships
    console.log(`  Frozen → Sealed: ${!frozen || sealed}`); // Always true
    console.log(`  Sealed → !Extensible: ${!sealed || !extensible}`); // Always true
}

const normal = {};
const prevented = {};
const sealed = {};
const frozen = {};

Object.preventExtensions(prevented);
Object.seal(sealed);
Object.freeze(frozen);

analyzeObject(normal, 'Normal');
analyzeObject(prevented, 'PreventExtensions');
analyzeObject(sealed, 'Sealed');
analyzeObject(frozen, 'Frozen');
```

## Advanced Object Concepts

### Property Descriptors and Inheritance
```javascript
function Parent() {
    this.parentProp = 'parent';
}

Parent.prototype.inheritedMethod = function() {
    return 'inherited';
};

function Child() {
    Parent.call(this);
    this.childProp = 'child';
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

const child = new Child();

// Own properties vs inherited
console.log(Object.getOwnPropertyNames(child)); // ['parentProp', 'childProp']
console.log('inheritedMethod' in child); // true
console.log(child.hasOwnProperty('inheritedMethod')); // false

// Property descriptors for own properties only
console.log(Object.getOwnPropertyDescriptor(child, 'childProp'));
// { value: 'child', writable: true, enumerable: true, configurable: true }

console.log(Object.getOwnPropertyDescriptor(child, 'inheritedMethod')); // undefined
```

### Proxies and Property Descriptors
```javascript
const target = {};
const handler = {
    defineProperty(target, property, descriptor) {
        console.log(`Defining property: ${property}`);
        console.log('Descriptor:', descriptor);
        return Reflect.defineProperty(target, property, descriptor);
    },
    
    getOwnPropertyDescriptor(target, property) {
        console.log(`Getting descriptor for: ${property}`);
        return Reflect.getOwnPropertyDescriptor(target, property);
    }
};

const proxy = new Proxy(target, handler);

Object.defineProperty(proxy, 'name', {
    value: 'proxy object',
    writable: false
});

console.log(Object.getOwnPropertyDescriptor(proxy, 'name'));
```

### Symbol Properties and Descriptors
```javascript
const sym = Symbol('description');
const obj = {};

Object.defineProperty(obj, sym, {
    value: 'symbol value',
    enumerable: false
});

// Symbol properties are not enumerable by default anyway
console.log(Object.keys(obj)); // []
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(description)]
console.log(Object.getOwnPropertyDescriptor(obj, sym));
// { value: 'symbol value', writable: false, enumerable: false, configurable: false }
```

## Object Creation Patterns

### Factory Functions with Descriptors
```javascript
function createUser(name, age) {
    const user = {};
    
    Object.defineProperties(user, {
        name: {
            value: name,
            enumerable: true,
            configurable: false
        },
        age: {
            get() { return this._age; },
            set(value) {
                if (value < 0) throw new Error('Age cannot be negative');
                this._age = value;
            },
            enumerable: true,
            configurable: true
        },
        id: {
            value: Math.random().toString(36).substr(2, 9),
            enumerable: true,
            configurable: false,
            writable: false
        }
    });
    
    user.age = age; // Use setter
    return user;
}

const user = createUser('Alice', 25);
console.log(user.name); // 'Alice'
console.log(user.id); // Random ID
// user.name = 'Bob'; // Cannot change (configurable: false, writable: false)
// user.id = 'new-id'; // Cannot change (writable: false)
```

### Mixins with Property Descriptors
```javascript
const Timestamped = {
    addTimestamps(obj) {
        Object.defineProperties(obj, {
            createdAt: {
                value: new Date(),
                enumerable: true,
                configurable: false,
                writable: false
            },
            updatedAt: {
                get() { return this._updatedAt || this.createdAt; },
                set(value) { this._updatedAt = value; },
                enumerable: true,
                configurable: true
            }
        });
        return obj;
    }
};

const Validatable = {
    addValidation(obj, validators = {}) {
        Object.keys(validators).forEach(prop => {
            const originalDescriptor = Object.getOwnPropertyDescriptor(obj, prop);
            const validator = validators[prop];
            
            Object.defineProperty(obj, prop, {
                get: originalDescriptor.get || function() { 
                    return this[`_${prop}`]; 
                },
                set(value) {
                    if (!validator(value)) {
                        throw new Error(`Invalid value for ${prop}: ${value}`);
                    }
                    if (originalDescriptor.set) {
                        originalDescriptor.set.call(this, value);
                    } else {
                        this[`_${prop}`] = value;
                    }
                },
                enumerable: true,
                configurable: true
            });
        });
        return obj;
    }
};

// Usage
const user = { name: 'John', email: 'john@example.com' };
Timestamped.addTimestamps(user);
Validatable.addValidation(user, {
    email: (value) => value.includes('@')
});

console.log(user.createdAt); // Current date
// user.email = 'invalid'; // Error: Invalid value for email
```

## Best Practices

### When to Use Property Descriptors
```javascript
// ✅ Good use cases for descriptors

// 1. Creating immutable properties
function createConfig(options) {
    const config = {};
    Object.keys(options).forEach(key => {
        Object.defineProperty(config, key, {
            value: options[key],
            writable: false,
            enumerable: true,
            configurable: false
        });
    });
    return config;
}

// 2. Computed properties with caching
class ExpensiveCalculator {
    constructor(data) {
        this._data = data;
        this._cache = new Map();
    }
    
    get result() {
        if (!this._cache.has('result')) {
            console.log('Computing expensive result...');
            this._cache.set('result', this._data.reduce((a, b) => a + b, 0));
        }
        return this._cache.get('result');
    }
}

// 3. Property validation
function createValidatedObject(props, validators) {
    const obj = {};
    Object.keys(props).forEach(key => {
        const validator = validators[key];
        Object.defineProperty(obj, key, {
            get() { return this[`_${key}`]; },
            set(value) {
                if (validator && !validator(value)) {
                    throw new Error(`Invalid ${key}: ${value}`);
                }
                this[`_${key}`] = value;
            },
            enumerable: true,
            configurable: true
        });
        obj[key] = props[key]; // Initial value
    });
    return obj;
}
```

### Common Pitfalls and Solutions
```javascript
// ❌ Forgetting strict mode
function badExample() {
    // Without 'use strict', assignment to non-writable property fails silently
    const obj = {};
    Object.defineProperty(obj, 'readonly', {
        value: 'test',
        writable: false
    });
    obj.readonly = 'changed'; // Silently fails!
    console.log(obj.readonly); // Still 'test'
}

// ✅ Always use strict mode
function goodExample() {
    'use strict';
    const obj = {};
    Object.defineProperty(obj, 'readonly', {
        value: 'test',
        writable: false
    });
    // obj.readonly = 'changed'; // TypeError: Cannot assign to read only property
}

// ❌ Mixing data and accessor descriptors
function invalidDescriptor() {
    const obj = {};
    // Object.defineProperty(obj, 'prop', {
    //     value: 'test',
    //     get() { return 'getter'; } // TypeError: Invalid property descriptor
    // });
}

// ✅ Use one type consistently
function validDescriptors() {
    const obj = {};
    
    // Data descriptor
    Object.defineProperty(obj, 'data', {
        value: 'test',
        writable: true
    });
    
    // Accessor descriptor
    Object.defineProperty(obj, 'accessor', {
        get() { return this._value; },
        set(val) { this._value = val; }
    });
}

// ❌ Assuming deep protection
function shallowProtection() {
    const obj = { nested: { value: 'changeable' } };
    Object.freeze(obj);
    obj.nested.value = 'changed'; // This works!
    console.log(obj.nested.value); // 'changed'
}

// ✅ Implement deep protection if needed
function deepProtection() {
    function deepFreeze(obj) {
        Object.getOwnPropertyNames(obj).forEach(prop => {
            const value = obj[prop];
            if (value && typeof value === 'object') {
                deepFreeze(value);
            }
        });
        return Object.freeze(obj);
    }
    
    const obj = { nested: { value: 'protected' } };
    deepFreeze(obj);
    // obj.nested.value = 'changed'; // TypeError
}
```

### Performance Considerations
```javascript
// Property access performance
const obj = {};
const iterations = 1000000;

// Regular property
obj.regular = 'test';

// Property with getter
Object.defineProperty(obj, 'getter', {
    get() { return this._value || 'default'; }
});

// Benchmark regular property access
console.time('Regular property');
for (let i = 0; i < iterations; i++) {
    const value = obj.regular;
}
console.timeEnd('Regular property');

// Benchmark getter access
console.time('Getter property');
for (let i = 0; i < iterations; i++) {
    const value = obj.getter;
}
console.timeEnd('Getter property');

// Regular properties are faster for simple value access
// Use getters/setters only when you need the functionality
```

### Modern Alternatives
```javascript
// Modern class syntax often preferred over defineProperty
class ModernClass {
    #private = 'truly private';
    
    constructor(name) {
        this.name = name;
    }
    
    get displayName() {
        return `Mr/Ms ${this.name}`;
    }
    
    set displayName(value) {
        // Extract name from display format
        this.name = value.replace(/^Mr\/Ms\s+/, '');
    }
    
    get private() {
        return this.#private;
    }
}

// Proxy for more dynamic behavior
function createValidatedProxy(target, validators) {
    return new Proxy(target, {
        set(obj, prop, value) {
            const validator = validators[prop];
            if (validator && !validator(value)) {
                throw new Error(`Invalid ${String(prop)}: ${value}`);
            }
            obj[prop] = value;
            return true;
        }
    });
}
```

### Key Takeaways

1. **Use strict mode** - Essential for seeing descriptor-related errors
2. **Property descriptors control behavior** - `writable`, `enumerable`, `configurable`, `get`, `set`
3. **Default values differ** - `defineProperty` defaults to `false`, regular assignment to `true`
4. **Cannot mix descriptor types** - Use either data or accessor descriptors, not both
5. **Protection methods are shallow** - Only affect the immediate object level
6. **enumerable controls visibility** - Affects `for...in`, `Object.keys()`, `JSON.stringify()`
7. **configurable controls mutability** - Once `false`, descriptor cannot be changed
8. **getters/setters enable computed properties** - With validation, caching, and side effects
9. **Modern alternatives exist** - Classes, private fields, Proxies often more appropriate
10. **Performance implications** - Getters/setters slower than direct property access
``` 