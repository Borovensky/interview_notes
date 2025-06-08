# JavaScript Reflect and Proxy Guide

## Table of Contents
- [Reflect Object Fundamentals](#reflect-object-fundamentals)
- [Reflect.construct](#reflectconstruct)
- [new.target with Reflect](#newtarget-with-reflect)
- [Reflect.apply](#reflectapply)
- [Complete Reflect API](#complete-reflect-api)
- [Proxy Fundamentals](#proxy-fundamentals)
- [Property Protection with Proxy](#property-protection-with-proxy)
- [Proxy Handler Methods](#proxy-handler-methods)
- [Prototype Chain with Proxy](#prototype-chain-with-proxy)
- [Proxy.revocable](#proxyrevocable)
- [Advanced Proxy Patterns](#advanced-proxy-patterns)
- [Reflect + Proxy Best Practices](#reflect--proxy-best-practices)
- [Real-world Use Cases](#real-world-use-cases)
- [Performance Considerations](#performance-considerations)

## Reflect Object Fundamentals

**Reflect** is a built-in object that provides methods for intercepted JavaScript operations. As noted in your code, it provides programmatic access to operations that are normally done with operators.

### Basic Reflect Usage
Based on your simple example:

```javascript
// Пример создания Reflect. но делать так все контекста Proxi не имеет смысла...просто пример
class Track {}
const t = Reflect.construct(Track, []); // аналогично => new Car();
console.log(t instanceof Track); // true
```

### Reflect vs Traditional Operations
```javascript
// Traditional vs Reflect approach
const obj = { name: 'John', age: 30 };

// Traditional property access
console.log(obj.name); // 'John'
obj.city = 'NYC';
delete obj.age;

// Reflect equivalent
console.log(Reflect.get(obj, 'name')); // 'John'
Reflect.set(obj, 'city', 'NYC');
Reflect.deleteProperty(obj, 'age');

// Benefits of Reflect:
// 1. Consistent API for meta-operations
// 2. Returns success/failure instead of throwing
// 3. Works well with Proxy handlers
// 4. More functional programming style
```

### Why Use Reflect?
```javascript
// Reflect provides more predictable behavior
const obj = {};

// Traditional approach - might throw or behave unexpectedly
try {
    Object.defineProperty(obj, 'prop', {
        value: 'test',
        configurable: false
    });
    delete obj.prop; // Fails silently in non-strict mode
} catch (e) {
    console.log('Error:', e.message);
}

// Reflect approach - returns boolean
const success = Reflect.defineProperty(obj, 'prop', {
    value: 'test',
    configurable: false
});

const deleted = Reflect.deleteProperty(obj, 'prop');
console.log('Property defined:', success); // true
console.log('Property deleted:', deleted); // false
```

## Reflect.construct

Reflect.construct allows you to call a constructor function with a specific `new.target`.

### Basic Constructor Usage
Based on your example:

```javascript
// class Car {
//     constructor(model, color) {
//         console.log(`${model} and ${color}`)
//     }
// }
// const c = Reflect.construct(Car, ['Audi', 'red']);
```

### Reflect.construct vs new Operator
```javascript
class Vehicle {
    constructor(type, brand) {
        this.type = type;
        this.brand = brand;
        console.log(`Creating ${type} by ${brand}`);
    }
}

// Traditional constructor call
const car1 = new Vehicle('car', 'Toyota');

// Reflect.construct equivalent
const car2 = Reflect.construct(Vehicle, ['car', 'Honda']);

console.log(car1 instanceof Vehicle); // true
console.log(car2 instanceof Vehicle); // true

// Both create equivalent objects
console.log(car1.type, car1.brand); // 'car' 'Toyota'
console.log(car2.type, car2.brand); // 'car' 'Honda'
```

### Dynamic Constructor Calls
```javascript
// Reflect.construct is useful for dynamic object creation
const constructors = {
    car: class Car {
        constructor(brand) {
            this.type = 'car';
            this.brand = brand;
        }
    },
    
    bike: class Bike {
        constructor(brand) {
            this.type = 'bike';
            this.brand = brand;
        }
    }
};

function createVehicle(type, brand) {
    const Constructor = constructors[type];
    if (!Constructor) {
        throw new Error(`Unknown vehicle type: ${type}`);
    }
    
    return Reflect.construct(Constructor, [brand]);
}

const tesla = createVehicle('car', 'Tesla');
const harley = createVehicle('bike', 'Harley-Davidson');

console.log(tesla.type, tesla.brand); // 'car' 'Tesla'
console.log(harley.type, harley.brand); // 'bike' 'Harley-Davidson'
```

## new.target with Reflect

The third parameter of Reflect.construct allows you to control what `new.target` points to inside the constructor.

### Custom new.target Example
Based on your detailed example:

```javascript
// мы построили r на основании конструктора Car => внутри конструктора Car, new.target указывает на функцию carMaker
// потому что мы передали ее 3-м параметром в Reflect.construct
class Car {
    constructor(model, color) {
        console.log(new.target === carMaker);
        console.log(`${model} and ${color}`)
    }
    test() {
        console.log('test car')
    }
}

function carMaker() {
    console.log('in carMaker');
}

carMaker.prototype.read = function() {
    console.log('read method')
}

const r = Reflect.construct(Car, ['Audi', 'red'], carMaker);
console.log(r instanceof Car); // false
r.read() // Works because prototype is carMaker.prototype
// после этого все свойства и методы у нас будут искаться в carMaker
```

### Understanding new.target Behavior
```javascript
class BaseClass {
    constructor() {
        console.log('Base constructor called');
        console.log('new.target:', new.target.name);
        this.constructorName = new.target.name;
    }
}

class DerivedClass extends BaseClass {
    constructor() {
        super();
        console.log('Derived constructor called');
    }
}

function CustomConstructor() {}

// Normal instantiation
const normal = new DerivedClass();
console.log(normal.constructorName); // 'DerivedClass'
console.log(normal instanceof DerivedClass); // true
console.log(normal instanceof BaseClass); // true

// Using Reflect.construct with custom new.target
const custom = Reflect.construct(BaseClass, [], CustomConstructor);
console.log(custom.constructorName); // 'CustomConstructor'
console.log(custom instanceof BaseClass); // false!
console.log(custom instanceof CustomConstructor); // true
```

### Practical new.target Applications
```javascript
// Factory pattern with controlled instantiation
class Vehicle {
    constructor(type) {
        if (new.target === Vehicle) {
            throw new Error('Vehicle cannot be instantiated directly');
        }
        this.type = type;
    }
}

class Car extends Vehicle {
    constructor(brand) {
        super('car');
        this.brand = brand;
    }
}

class Motorcycle extends Vehicle {
    constructor(brand) {
        super('motorcycle');
        this.brand = brand;
    }
}

// This works
const car = new Car('BMW');

// This throws error
// const vehicle = new Vehicle('generic'); // Error!

// But with Reflect.construct, we can bypass the check
const sneakyVehicle = Reflect.construct(Vehicle, ['generic'], Car);
console.log(sneakyVehicle instanceof Car); // true
console.log(sneakyVehicle.type); // 'generic'
```

## Reflect.apply

Reflect.apply provides a functional way to call functions with a specific `this` context and arguments.

### Basic Reflect.apply Usage
Based on your example:

```javascript
class Car1 {
    constructor() {
        this.id = 33;
    }
    show() {
        console.log(this.id);
    }
}

const a = new Car1();
a.show(); // 33
Reflect.apply(Car1.prototype.show, {id: 99}, []); // 99
```

### Reflect.apply vs Function Methods
```javascript
function greet(greeting, punctuation) {
    return `${greeting}, my name is ${this.name}${punctuation}`;
}

const person = { name: 'Alice' };
const args = ['Hello', '!'];

// Traditional approaches
const result1 = greet.call(person, 'Hello', '!');
const result2 = greet.apply(person, args);

// Reflect.apply approach
const result3 = Reflect.apply(greet, person, args);

console.log(result1); // "Hello, my name is Alice!"
console.log(result2); // "Hello, my name is Alice!"
console.log(result3); // "Hello, my name is Alice!"

// All produce the same result, but Reflect.apply is more functional
```

### Dynamic Method Calls
```javascript
class Calculator {
    add(a, b) { return a + b; }
    subtract(a, b) { return a - b; }
    multiply(a, b) { return a * b; }
    divide(a, b) { return a / b; }
}

const calc = new Calculator();

function performOperation(operation, ...args) {
    const method = calc[operation];
    
    if (typeof method !== 'function') {
        throw new Error(`Unknown operation: ${operation}`);
    }
    
    return Reflect.apply(method, calc, args);
}

console.log(performOperation('add', 5, 3)); // 8
console.log(performOperation('multiply', 4, 7)); // 28
console.log(performOperation('divide', 10, 2)); // 5

// Error handling
try {
    performOperation('power', 2, 3); // Error: Unknown operation: power
} catch (e) {
    console.log(e.message);
}
```

### Safe Method Calls
```javascript
// Reflect.apply is safer when dealing with potentially corrupted objects
const obj = {
    name: 'Test',
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

// Corrupt the apply method (malicious or accidental)
obj.greet.apply = null;

try {
    // This would fail
    // obj.greet.apply(obj, []);
} catch (e) {
    console.log('Traditional apply failed');
}

// Reflect.apply still works
const result = Reflect.apply(obj.greet, obj, []);
console.log(result); // "Hello, I'm Test"
```

## Complete Reflect API

All Reflect methods correspond to Proxy handler methods.

### Property Operations
```javascript
const obj = { name: 'John', age: 30 };

// Reflect.get(target, propertyKey, receiver)
console.log(Reflect.get(obj, 'name')); // 'John'

// Reflect.set(target, propertyKey, value, receiver)
const success = Reflect.set(obj, 'city', 'NYC');
console.log(success); // true
console.log(obj.city); // 'NYC'

// Reflect.has(target, propertyKey)
console.log(Reflect.has(obj, 'name')); // true
console.log(Reflect.has(obj, 'salary')); // false

// Reflect.deleteProperty(target, propertyKey)
const deleted = Reflect.deleteProperty(obj, 'age');
console.log(deleted); // true
console.log(obj.age); // undefined
```

### Property Descriptor Operations
```javascript
const obj = {};

// Reflect.defineProperty(target, propertyKey, descriptor)
const defined = Reflect.defineProperty(obj, 'readonly', {
    value: 'immutable',
    writable: false,
    enumerable: true,
    configurable: false
});
console.log(defined); // true

// Reflect.getOwnPropertyDescriptor(target, propertyKey)
const descriptor = Reflect.getOwnPropertyDescriptor(obj, 'readonly');
console.log(descriptor);
// { value: 'immutable', writable: false, enumerable: true, configurable: false }
```

### Object Operations
```javascript
const parent = { inherited: 'value' };
const child = Object.create(parent);
child.own = 'property';

// Reflect.getPrototypeOf(target)
console.log(Reflect.getPrototypeOf(child) === parent); // true

// Reflect.setPrototypeOf(target, prototype)
const newParent = { newInherited: 'newValue' };
const success = Reflect.setPrototypeOf(child, newParent);
console.log(success); // true
console.log(child.newInherited); // 'newValue'

// Reflect.ownKeys(target)
const keys = Reflect.ownKeys(child);
console.log(keys); // ['own']

// Reflect.isExtensible(target)
console.log(Reflect.isExtensible(child)); // true

// Reflect.preventExtensions(target)
const prevented = Reflect.preventExtensions(child);
console.log(prevented); // true
console.log(Reflect.isExtensible(child)); // false
```

## Proxy Fundamentals

Proxy allows you to intercept and customize operations performed on objects.

### Basic Proxy Structure
Based on your Employee example:

```javascript
function Employee() {
    this.name = 'John';
    this.salary = 0;
}

const e = new Employee();
// минусом тут есть то что все свойства в Employee у нас можно легко прочитать/изменить поскольку они написаны через this.

const p = new Proxy(e, {
    get: function(target, property, receiver) {
        if (property === 'salary') {
            return void 0
            // return `You have no access to prop ${ property }` 
        }
        return Reflect.get(target, property, receiver); // возвращаем стандартное поведение! 
    },
})

console.log(p.name); // John
console.log(p.salary); // undefined
```

### Understanding Proxy Components
```javascript
// Proxy structure: new Proxy(target, handler)
const target = { data: 'original' };

const handler = {
    get(target, property, receiver) {
        console.log(`Getting property: ${property}`);
        return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
        console.log(`Setting property: ${property} = ${value}`);
        return Reflect.set(target, property, value, receiver);
    }
};

const proxy = new Proxy(target, handler);

// All operations are intercepted
proxy.data; // "Getting property: data"
proxy.newProp = 'value'; // "Setting property: newProp = value"
console.log(proxy.newProp); // "Getting property: newProp"
```

## Property Protection with Proxy

### Access Control
```javascript
class SecureUser {
    constructor(name, role) {
        this.name = name;
        this.role = role;
        this._password = 'secret';
        this._sessions = [];
        
        return new Proxy(this, {
            get(target, property, receiver) {
                // Block access to private properties
                if (property.startsWith('_')) {
                    throw new Error(`Access denied to private property: ${property}`);
                }
                
                // Log property access
                console.log(`Accessing: ${property}`);
                return Reflect.get(target, property, receiver);
            },
            
            set(target, property, value, receiver) {
                // Prevent modification of role
                if (property === 'role') {
                    throw new Error('Role cannot be modified');
                }
                
                // Block setting private properties
                if (property.startsWith('_')) {
                    throw new Error(`Cannot set private property: ${property}`);
                }
                
                console.log(`Setting: ${property} = ${value}`);
                return Reflect.set(target, property, value, receiver);
            }
        });
    }
}

const user = new SecureUser('Alice', 'admin');
console.log(user.name); // "Accessing: name" then "Alice"
user.name = 'Bob'; // "Setting: name = Bob"

try {
    console.log(user._password); // Error: Access denied to private property: _password
} catch (e) {
    console.log(e.message);
}

try {
    user.role = 'user'; // Error: Role cannot be modified
} catch (e) {
    console.log(e.message);
}
```

### Validation and Transformation
```javascript
function createValidatedObject(validationRules = {}) {
    const data = {};
    
    return new Proxy(data, {
        set(target, property, value, receiver) {
            const rule = validationRules[property];
            
            if (rule) {
                // Type checking
                if (rule.type && typeof value !== rule.type) {
                    throw new Error(`${property} must be of type ${rule.type}`);
                }
                
                // Range checking
                if (rule.min !== undefined && value < rule.min) {
                    throw new Error(`${property} must be >= ${rule.min}`);
                }
                
                if (rule.max !== undefined && value > rule.max) {
                    throw new Error(`${property} must be <= ${rule.max}`);
                }
                
                // Custom validator
                if (rule.validator && !rule.validator(value)) {
                    throw new Error(`${property} failed custom validation`);
                }
                
                // Transform value
                if (rule.transform) {
                    value = rule.transform(value);
                }
            }
            
            return Reflect.set(target, property, value, receiver);
        }
    });
}

const userProfile = createValidatedObject({
    age: { type: 'number', min: 0, max: 150 },
    email: { 
        type: 'string',
        validator: (value) => value.includes('@'),
        transform: (value) => value.toLowerCase()
    },
    name: {
        type: 'string',
        transform: (value) => value.trim()
    }
});

userProfile.name = '  Alice  '; // Trimmed to 'Alice'
userProfile.email = 'ALICE@EXAMPLE.COM'; // Transformed to 'alice@example.com'
userProfile.age = 25; // Valid

console.log(userProfile); // { name: 'Alice', email: 'alice@example.com', age: 25 }

try {
    userProfile.age = -5; // Error: age must be >= 0
} catch (e) {
    console.log(e.message);
}
```

## Proxy Handler Methods

### Complete Handler Interface
```javascript
const target = { data: 'test' };

const comprehensiveHandler = {
    // Property access
    get(target, property, receiver) {
        console.log(`get: ${property}`);
        return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
        console.log(`set: ${property} = ${value}`);
        return Reflect.set(target, property, value, receiver);
    },
    
    has(target, property) {
        console.log(`has: ${property}`);
        return Reflect.has(target, property);
    },
    
    deleteProperty(target, property) {
        console.log(`delete: ${property}`);
        return Reflect.deleteProperty(target, property);
    },
    
    // Property descriptors
    defineProperty(target, property, descriptor) {
        console.log(`defineProperty: ${property}`);
        return Reflect.defineProperty(target, property, descriptor);
    },
    
    getOwnPropertyDescriptor(target, property) {
        console.log(`getOwnPropertyDescriptor: ${property}`);
        return Reflect.getOwnPropertyDescriptor(target, property);
    },
    
    // Object operations
    ownKeys(target) {
        console.log('ownKeys called');
        return Reflect.ownKeys(target);
    },
    
    getPrototypeOf(target) {
        console.log('getPrototypeOf called');
        return Reflect.getPrototypeOf(target);
    },
    
    setPrototypeOf(target, prototype) {
        console.log('setPrototypeOf called');
        return Reflect.setPrototypeOf(target, prototype);
    },
    
    isExtensible(target) {
        console.log('isExtensible called');
        return Reflect.isExtensible(target);
    },
    
    preventExtensions(target) {
        console.log('preventExtensions called');
        return Reflect.preventExtensions(target);
    },
    
    // Function calls
    apply(target, thisArg, argumentsList) {
        console.log('apply called');
        return Reflect.apply(target, thisArg, argumentsList);
    },
    
    construct(target, argumentsList, newTarget) {
        console.log('construct called');
        return Reflect.construct(target, argumentsList, newTarget);
    }
};

const proxy = new Proxy(target, comprehensiveHandler);

// Test various operations
proxy.data; // get: data
proxy.newProp = 'value'; // set: newProp = value
'data' in proxy; // has: data
delete proxy.newProp; // delete: newProp
Object.keys(proxy); // ownKeys called
```

## Prototype Chain with Proxy

### Proxy in Prototype Chain
Based on your example:

```javascript
const h = new Proxy({}, {
    get: function(target, property, receiver) {
        return `Property ${ property } doesn't exist`;
    }
});

const data = {
    id: 3000
}

Object.setPrototypeOf(data, h);  

console.log(data.id); // 3000
console.log(data.name); // Property name doesn't exist
```

### Advanced Prototype Proxy
```javascript
// Create a proxy that provides default values for missing properties
const defaultsProxy = new Proxy({}, {
    get(target, property, receiver) {
        // Provide sensible defaults based on property name
        if (property.endsWith('Count')) return 0;
        if (property.endsWith('List')) return [];
        if (property.endsWith('Map')) return new Map();
        if (property.startsWith('is')) return false;
        if (property.startsWith('has')) return false;
        
        return `[default: ${property}]`;
    }
});

const userSettings = {
    username: 'alice'
};

Object.setPrototypeOf(userSettings, defaultsProxy);

console.log(userSettings.username); // 'alice'
console.log(userSettings.loginCount); // 0
console.log(userSettings.friendsList); // []
console.log(userSettings.isActive); // false
console.log(userSettings.customProp); // '[default: customProp]'
```

### Proxy Chain
```javascript
// Multiple proxies in the chain
const logger = new Proxy({}, {
    get(target, property, receiver) {
        console.log(`Logger: accessing ${property}`);
        return Reflect.get(target, property, receiver);
    }
});

const validator = new Proxy(logger, {
    set(target, property, value, receiver) {
        if (typeof value === 'string' && value.length === 0) {
            throw new Error('Empty strings not allowed');
        }
        console.log(`Validator: setting ${property} = ${value}`);
        return Reflect.set(target, property, value, receiver);
    }
});

const cache = new Proxy(validator, {
    get(target, property, receiver) {
        const value = Reflect.get(target, property, receiver);
        console.log(`Cache: retrieved ${property} = ${value}`);
        return value;
    }
});

// Operations go through all proxies
cache.test = 'value'; // Validator: setting test = value
console.log(cache.test); // Logger: accessing test, Cache: retrieved test = value
```

## Proxy.revocable

Proxy.revocable creates a proxy that can be disabled.

### Basic Revocable Proxy
Based on your example:

```javascript
// Proxy.revocable (proxy revoke). Крутая штука. тут у нас есть возможнось контралировать точку в которой мы сможет уничтожить (стиреть наш объект)
const obj = {
    id: 3000
};

const { proxy, revoke } = Proxy.revocable(obj, {
    get: function(target, property, receiver) {
        return Reflect.get(target, property, receiver) + 1000;
    }
});

console.log(proxy.id); // 4000
revoke(); // уничтожаем наш объект
// console.log(proxy.id); // Cannot perform 'get' on a proxy that has been revoked
```

### Temporary Access Control
```javascript
class TemporaryAccess {
    constructor(data, timeoutMs) {
        this.data = data;
        this.active = true;
        
        const { proxy, revoke } = Proxy.revocable(this.data, {
            get: (target, property, receiver) => {
                if (!this.active) {
                    throw new Error('Access expired');
                }
                return Reflect.get(target, property, receiver);
            },
            
            set: (target, property, value, receiver) => {
                if (!this.active) {
                    throw new Error('Access expired');
                }
                return Reflect.set(target, property, value, receiver);
            }
        });
        
        this.proxy = proxy;
        this.revoke = revoke;
        
        // Auto-revoke after timeout
        setTimeout(() => {
            this.active = false;
            this.revoke();
            console.log('Access automatically revoked');
        }, timeoutMs);
    }
    
    getProxy() {
        return this.proxy;
    }
    
    revokeNow() {
        this.active = false;
        this.revoke();
    }
}

const sensitiveData = { secret: 'classified', id: 123 };
const tempAccess = new TemporaryAccess(sensitiveData, 2000); // 2 second access
const proxy = tempAccess.getProxy();

console.log(proxy.secret); // 'classified'
proxy.newField = 'allowed';

// Wait for auto-revoke or manually revoke
setTimeout(() => {
    try {
        console.log(proxy.secret); // Error: Cannot perform 'get' on a proxy that has been revoked
    } catch (e) {
        console.log('Access denied:', e.message);
    }
}, 3000);
```

### Resource Management
```javascript
class DatabaseConnection {
    constructor() {
        this.connected = true;
        this.queries = [];
        
        const { proxy, revoke } = Proxy.revocable(this, {
            get(target, property, receiver) {
                if (!target.connected) {
                    throw new Error('Database connection closed');
                }
                
                const value = Reflect.get(target, property, receiver);
                
                // Log method calls
                if (typeof value === 'function') {
                    return function(...args) {
                        console.log(`DB Call: ${property}(${args.join(', ')})`);
                        return value.apply(target, args);
                    };
                }
                
                return value;
            }
        });
        
        this.proxy = proxy;
        this.revoke = revoke;
    }
    
    query(sql) {
        this.queries.push(sql);
        return `Result for: ${sql}`;
    }
    
    close() {
        this.connected = false;
        this.revoke();
        console.log('Database connection closed and proxy revoked');
    }
    
    getProxy() {
        return this.proxy;
    }
}

const db = new DatabaseConnection();
const dbProxy = db.getProxy();

console.log(dbProxy.query('SELECT * FROM users')); // DB Call: query(SELECT * FROM users)
console.log(dbProxy.queries); // ['SELECT * FROM users']

db.close(); // Database connection closed and proxy revoked

try {
    dbProxy.query('SELECT * FROM posts'); // Error: Database connection closed
} catch (e) {
    console.log(e.message);
}
```

## Advanced Proxy Patterns

### Observable Objects
```javascript
function createObservable(target, callback) {
    return new Proxy(target, {
        set(obj, property, value, receiver) {
            const oldValue = obj[property];
            const result = Reflect.set(obj, property, value, receiver);
            
            if (result && oldValue !== value) {
                callback(property, value, oldValue);
            }
            
            return result;
        },
        
        deleteProperty(obj, property) {
            const oldValue = obj[property];
            const result = Reflect.deleteProperty(obj, property);
            
            if (result && obj.hasOwnProperty(property)) {
                callback(property, undefined, oldValue);
            }
            
            return result;
        }
    });
}

const data = { name: 'John', age: 30 };
const observable = createObservable(data, (property, newValue, oldValue) => {
    console.log(`Property ${property} changed from ${oldValue} to ${newValue}`);
});

observable.name = 'Jane'; // Property name changed from John to Jane
observable.city = 'NYC'; // Property city changed from undefined to NYC
delete observable.age; // Property age changed from 30 to undefined
```

### Virtual Properties
```javascript
function createVirtualObject(computedProperties = {}) {
    const data = {};
    
    return new Proxy(data, {
        get(target, property, receiver) {
            // Check if it's a computed property
            if (property in computedProperties) {
                return computedProperties[property].call(receiver);
            }
            
            return Reflect.get(target, property, receiver);
        },
        
        has(target, property) {
            return property in computedProperties || Reflect.has(target, property);
        },
        
        ownKeys(target) {
            return [...Reflect.ownKeys(target), ...Object.keys(computedProperties)];
        }
    });
}

const person = createVirtualObject({
    fullName() {
        return `${this.firstName} ${this.lastName}`;
    },
    
    initials() {
        return `${this.firstName?.[0] || ''}${this.lastName?.[0] || ''}`.toUpperCase();
    },
    
    age() {
        if (!this.birthYear) return null;
        return new Date().getFullYear() - this.birthYear;
    }
});

person.firstName = 'John';
person.lastName = 'Doe';
person.birthYear = 1990;

console.log(person.fullName); // 'John Doe'
console.log(person.initials); // 'JD'
console.log(person.age); // Current year - 1990
console.log(Object.keys(person)); // ['firstName', 'lastName', 'birthYear', 'fullName', 'initials', 'age']
```

### API Client Proxy
```javascript
function createAPIClient(baseURL) {
    return new Proxy({}, {
        get(target, property, receiver) {
            // Return a function that makes HTTP requests
            return function(id = '', options = {}) {
                const url = id ? `${baseURL}/${property}/${id}` : `${baseURL}/${property}`;
                const method = options.method || 'GET';
                
                console.log(`${method} ${url}`);
                
                // Simulate API call
                return Promise.resolve({
                    url,
                    method,
                    data: `Mock data for ${property}${id ? ` with id ${id}` : ''}`
                });
            };
        }
    });
}

const api = createAPIClient('https://api.example.com');

// These all work dynamically
api.users().then(console.log); // GET https://api.example.com/users
api.users(123).then(console.log); // GET https://api.example.com/users/123
api.posts(456, { method: 'DELETE' }).then(console.log); // DELETE https://api.example.com/posts/456
```

## Reflect + Proxy Best Practices

### Always Use Reflect in Handlers
```javascript
// ✅ Good - using Reflect ensures proper behavior
const goodProxy = new Proxy(target, {
    get(target, property, receiver) {
        // Reflect maintains proper 'this' binding
        return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
        // Reflect handles edge cases correctly
        return Reflect.set(target, property, value, receiver);
    }
});

// ❌ Bad - direct property access can break 'this' binding
const badProxy = new Proxy(target, {
    get(target, property, receiver) {
        return target[property]; // May break with getters
    },
    
    set(target, property, value, receiver) {
        target[property] = value; // May break with setters
        return true;
    }
});
```

### Preserve Invariants
```javascript
// Proxy handlers must respect object invariants
function createSecureProxy(target) {
    return new Proxy(target, {
        defineProperty(target, property, descriptor) {
            // Respect non-configurable properties
            const existing = Reflect.getOwnPropertyDescriptor(target, property);
            if (existing && !existing.configurable) {
                console.log(`Cannot redefine non-configurable property: ${property}`);
                return false;
            }
            
            return Reflect.defineProperty(target, property, descriptor);
        },
        
        deleteProperty(target, property) {
            // Respect non-configurable properties
            const descriptor = Reflect.getOwnPropertyDescriptor(target, property);
            if (descriptor && !descriptor.configurable) {
                console.log(`Cannot delete non-configurable property: ${property}`);
                return false;
            }
            
            return Reflect.deleteProperty(target, property);
        }
    });
}
```

### Error Handling
```javascript
function createRobustProxy(target, options = {}) {
    return new Proxy(target, {
        get(target, property, receiver) {
            try {
                const value = Reflect.get(target, property, receiver);
                
                if (options.logAccess) {
                    console.log(`Accessed: ${property}`);
                }
                
                return value;
            } catch (error) {
                if (options.onError) {
                    return options.onError(error, 'get', property);
                }
                throw error;
            }
        },
        
        set(target, property, value, receiver) {
            try {
                if (options.validateSet) {
                    const isValid = options.validateSet(property, value);
                    if (!isValid) {
                        console.log(`Validation failed for ${property} = ${value}`);
                        return false;
                    }
                }
                
                return Reflect.set(target, property, value, receiver);
            } catch (error) {
                if (options.onError) {
                    options.onError(error, 'set', property, value);
                    return false;
                }
                throw error;
            }
        }
    });
}
```

## Real-world Use Cases

### Framework-like Reactivity
```javascript
class ReactiveSystem {
    constructor() {
        this.dependencies = new Map();
        this.currentComputation = null;
    }
    
    reactive(obj) {
        const deps = this.dependencies;
        const system = this;
        
        return new Proxy(obj, {
            get(target, property, receiver) {
                // Track dependency
                if (system.currentComputation) {
                    if (!deps.has(target)) {
                        deps.set(target, new Map());
                    }
                    if (!deps.get(target).has(property)) {
                        deps.get(target).set(property, new Set());
                    }
                    deps.get(target).get(property).add(system.currentComputation);
                }
                
                return Reflect.get(target, property, receiver);
            },
            
            set(target, property, value, receiver) {
                const result = Reflect.set(target, property, value, receiver);
                
                // Trigger effects
                if (deps.has(target) && deps.get(target).has(property)) {
                    deps.get(target).get(property).forEach(effect => {
                        effect();
                    });
                }
                
                return result;
            }
        });
    }
    
    computed(fn) {
        const effect = () => {
            this.currentComputation = effect;
            const result = fn();
            this.currentComputation = null;
            return result;
        };
        
        return effect;
    }
}

const system = new ReactiveSystem();
const state = system.reactive({ count: 0, name: 'Vue-like' });

const doubleCount = system.computed(() => {
    console.log('Computing double count...');
    return state.count * 2;
});

console.log(doubleCount()); // Computing double count... 0
console.log(doubleCount()); // 0 (computed again due to access)

state.count = 5; // Computing double count... (triggered automatically)
```

### Configuration Management
```javascript
function createConfig(defaults = {}, options = {}) {
    const config = { ...defaults };
    const { frozen = false, required = [], validators = {} } = options;
    
    return new Proxy(config, {
        get(target, property, receiver) {
            if (required.includes(property) && !(property in target)) {
                throw new Error(`Required configuration property missing: ${property}`);
            }
            
            return Reflect.get(target, property, receiver);
        },
        
        set(target, property, value, receiver) {
            if (frozen) {
                throw new Error('Configuration is frozen');
            }
            
            if (validators[property]) {
                const isValid = validators[property](value);
                if (!isValid) {
                    throw new Error(`Invalid value for ${property}: ${value}`);
                }
            }
            
            return Reflect.set(target, property, value, receiver);
        }
    });
}

const appConfig = createConfig(
    { port: 3000, debug: false },
    {
        required: ['apiKey'],
        validators: {
            port: (value) => typeof value === 'number' && value > 0 && value < 65536,
            apiKey: (value) => typeof value === 'string' && value.length > 0
        }
    }
);

appConfig.apiKey = 'abc123';
appConfig.port = 8080;

try {
    console.log(appConfig.port); // 8080
    appConfig.port = -1; // Error: Invalid value for port: -1
} catch (e) {
    console.log(e.message);
}
```

## Performance Considerations

### Proxy Overhead
```javascript
// Proxies add overhead to property access
const plainObject = { x: 1, y: 2 };
const proxiedObject = new Proxy(plainObject, {
    get(target, property, receiver) {
        return Reflect.get(target, property, receiver);
    }
});

// Benchmark property access
const iterations = 1000000;

console.time('Plain object access');
for (let i = 0; i < iterations; i++) {
    const value = plainObject.x;
}
console.timeEnd('Plain object access');

console.time('Proxied object access');
for (let i = 0; i < iterations; i++) {
    const value = proxiedObject.x;
}
console.timeEnd('Proxied object access');

// Proxied access is significantly slower
```

### Optimization Strategies
```javascript
// Cache frequently accessed properties
function createCachedProxy(target) {
    const cache = new Map();
    
    return new Proxy(target, {
        get(target, property, receiver) {
            if (cache.has(property)) {
                return cache.get(property);
            }
            
            const value = Reflect.get(target, property, receiver);
            
            // Cache non-function values
            if (typeof value !== 'function') {
                cache.set(property, value);
            }
            
            return value;
        },
        
        set(target, property, value, receiver) {
            // Invalidate cache on write
            cache.delete(property);
            return Reflect.set(target, property, value, receiver);
        }
    });
}

// Use selective proxying - only wrap what needs interception
function createSelectiveProxy(target, interceptedProperties) {
    const propertySet = new Set(interceptedProperties);
    
    return new Proxy(target, {
        get(target, property, receiver) {
            if (propertySet.has(property)) {
                console.log(`Intercepted access to: ${property}`);
            }
            return Reflect.get(target, property, receiver);
        }
    });
}
```

### Key Takeaways

1. **Reflect provides programmatic access** to meta-operations with consistent API
2. **Reflect.construct allows custom new.target** - Useful for framework internals
3. **Proxy enables interception** of object operations for customization
4. **Always use Reflect in Proxy handlers** - Maintains correct behavior and invariants
5. **Proxy.revocable enables access control** - Great for temporary permissions
6. **Proxies add performance overhead** - Use selectively and optimize when needed
7. **Handler methods correspond to operations** - get/set for property access, apply for function calls
8. **Respect object invariants** - Non-configurable properties have restrictions
9. **Error handling is crucial** - Proxy operations can fail in unexpected ways
10. **Modern JS features rely on these APIs** - Understanding helps with frameworks and tooling 