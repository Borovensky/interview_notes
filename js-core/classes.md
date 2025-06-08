# JavaScript Classes Guide

## Table of Contents
- [Basic Class Syntax](#basic-class-syntax)
- [How Classes Work Under the Hood](#how-classes-work-under-the-hood)
- [Class Instantiation Patterns](#class-instantiation-patterns)
- [Static Methods](#static-methods)
- [Class Inheritance](#class-inheritance)
- [Extending Built-in Classes](#extending-built-in-classes)
- [Advanced Class Features](#advanced-class-features)
- [Best Practices](#best-practices)

## Basic Class Syntax

### ES6 Class Declaration
```javascript
class PersonClass {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
}

// Usage
const jon = new PersonClass('John');
jon.sayName(); // "John"

// Important: Classes must be called with 'new'
// PersonClass('John'); // TypeError: Class constructor PersonClass cannot be invoked without 'new'

// Type checking
console.log(jon instanceof PersonClass); // true
console.log(jon instanceof Object);      // true
console.log(typeof PersonClass);         // "function"
```

### Class Expression
```javascript
// Named class expression
const PersonClass = class PersonClass {
    constructor(name) {
        this.name = name;
    }
};

// Anonymous class expression
const PersonClass2 = class {
    constructor(name) {
        this.name = name;
    }
};
```

## How Classes Work Under the Hood

Classes in JavaScript are syntactic sugar over prototype-based inheritance. Here's how the above class would look using functions:

```javascript
let PersonType = (function() {
    'use strict';
    
    const PersonType = function(name) {
        // Ensure constructor is called with 'new'
        if (typeof new.target === 'undefined') {
            throw new Error('Constructor must be called with "new" keyword');
        }
        this.name = name;
    }
    
    // Define method on prototype
    Object.defineProperty(PersonType.prototype, 'sayName', {
        value: function() {
            // Prevent method from being called as constructor
            if (typeof new.target !== 'undefined') {
                throw new Error('Method cannot be called with "new" keyword');
            }
            console.log(this.name);
        },
        enumerable: false,   // Method won't appear in for...in loops
        writable: true,      // Method can be overwritten
        configurable: true   // Property descriptor can be changed
    });
    
    return PersonType;
})();

// Usage is identical to class
const person = new PersonType('Alice');
person.sayName(); // "Alice"
```

### Key Differences Between Classes and Functions
| Feature | Class | Function Constructor |
|---------|-------|---------------------|
| Hoisting | No (temporal dead zone) | Yes |
| Strict Mode | Always in strict mode | Depends on context |
| new.target | Always defined in constructor | May be undefined |
| Enumerable Methods | Non-enumerable by default | Enumerable by default |
| Call without new | Throws TypeError | Returns undefined or pollutes global |

## Class Instantiation Patterns

### Singleton Pattern with Classes
```javascript
// Create a single instance using anonymous class
let person = new class {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
    
    greet() {
        console.log(`Hello, I'm ${this.name}`);
    }
}('Peter');

person.sayName(); // "Peter"
person.greet();   // "Hello, I'm Peter"

// Alternative singleton pattern
class Singleton {
    constructor() {
        if (Singleton.instance) {
            return Singleton.instance;
        }
        Singleton.instance = this;
        this.data = [];
    }
    
    addData(item) {
        this.data.push(item);
    }
}

const instance1 = new Singleton();
const instance2 = new Singleton();
console.log(instance1 === instance2); // true
```

## Static Methods

Static methods belong to the class itself, not to instances.

### Modern ES6 Static Methods
```javascript
class PersonClass {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
    
    // Static method
    static create(name) {
        return new PersonClass(name);
    }
    
    static getSpecies() {
        return 'Homo sapiens';
    }
    
    // Static getter
    static get defaultName() {
        return 'Anonymous';
    }
}

// Usage
const bob = PersonClass.create('Bob');
bob.sayName(); // "Bob"
console.log(PersonClass.getSpecies()); // "Homo sapiens"
console.log(PersonClass.defaultName);  // "Anonymous"
```

### Pre-ES6 Static Methods (Function Approach)
```javascript
function PersonType(name) {
    this.name = name;
}

// Static method - attached to constructor function
PersonType.create = name => new PersonType(name);
PersonType.getSpecies = () => 'Homo sapiens';

// Instance method - attached to prototype
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

// Usage
const bob = PersonType.create('Bob');
bob.sayName(); // "Bob"
console.log(PersonType.getSpecies()); // "Homo sapiens"
```

## Class Inheritance

### Basic Inheritance with extends
```javascript
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
    
    getArea() {
        return this.length * this.width;
    }
    
    getPerimeter() {
        return 2 * (this.length + this.width);
    }
}

class Square extends Rectangle {
    constructor(length) {
        // Must call super() before using 'this'
        super(length, length); // Call parent constructor
    }
    
    // Override parent method (polymorphism)
    getArea() {
        return `${super.getArea()} square units`;
    }
    
    // New method specific to Square
    getDiagonal() {
        return Math.sqrt(2) * this.length;
    }
}

const square = new Square(3);
console.log(square.getArea());      // "9 square units"
console.log(square.getPerimeter()); // 12 (inherited method)
console.log(square.getDiagonal());  // 4.242640687119285
```

### Advanced Inheritance Concepts
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        console.log(`${this.name} makes a sound`);
    }
    
    static getKingdom() {
        return 'Animalia';
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);
        this.breed = breed;
    }
    
    speak() {
        console.log(`${this.name} barks`);
    }
    
    // Static methods are also inherited
    static getDomesticationDate() {
        return '15,000 years ago';
    }
}

const dog = new Dog('Rex', 'German Shepherd');
dog.speak(); // "Rex barks"
console.log(Dog.getKingdom()); // "Animalia" (inherited static method)
console.log(Dog.getDomesticationDate()); // "15,000 years ago"
```

## Extending Built-in Classes

### Custom Error Classes
```javascript
class FetchError extends Error {
    constructor(message, status) {
        super(message);
        this.name = this.constructor.name; // Set error name to class name
        this.status = status;
        
        // Maintain proper stack trace for where error was thrown
        if (typeof Error.captureStackTrace === 'function') {
            Error.captureStackTrace(this, this.constructor);
        } else {
            this.stack = (new Error(message)).stack;
        }
    }
    
    toString() {
        return `${this.name}: ${this.message} (Status: ${this.status})`;
    }
}

// Usage
try {
    throw new FetchError('Failed to fetch data', 404);
} catch (err) {
    console.log(err instanceof FetchError); // true
    console.log(err instanceof Error);      // true
    console.log(err.toString()); // "FetchError: Failed to fetch data (Status: 404)"
}
```

### Custom Array Class
```javascript
class MyArray extends Array {
    constructor(...items) {
        super(...items);
    }
    
    // Add custom method
    first() {
        return this[0];
    }
    
    last() {
        return this[this.length - 1];
    }
    
    // Override existing method
    toString() {
        return `MyArray: [${super.toString()}]`;
    }
}

const arr = new MyArray(1, 2, 3, 4, 5);
console.log(arr.first()); // 1
console.log(arr.last());  // 5
console.log(arr.toString()); // "MyArray: [1,2,3,4,5]"
console.log(arr instanceof Array); // true
```

## Advanced Class Features

### Private Fields and Methods (ES2022)
```javascript
class BankAccount {
    // Private fields
    #balance = 0;
    #accountNumber;
    
    constructor(accountNumber) {
        this.#accountNumber = accountNumber;
    }
    
    // Private method
    #validateAmount(amount) {
        return amount > 0 && typeof amount === 'number';
    }
    
    // Public methods
    deposit(amount) {
        if (this.#validateAmount(amount)) {
            this.#balance += amount;
            return this.#balance;
        }
        throw new Error('Invalid amount');
    }
    
    getBalance() {
        return this.#balance;
    }
    
    // Private fields are not accessible from outside
    // account.#balance; // SyntaxError
}
```

### Getters and Setters
```javascript
class Temperature {
    constructor(celsius = 0) {
        this._celsius = celsius;
    }
    
    get celsius() {
        return this._celsius;
    }
    
    set celsius(value) {
        if (typeof value !== 'number') {
            throw new Error('Temperature must be a number');
        }
        this._celsius = value;
    }
    
    get fahrenheit() {
        return (this._celsius * 9/5) + 32;
    }
    
    set fahrenheit(value) {
        if (typeof value !== 'number') {
            throw new Error('Temperature must be a number');
        }
        this._celsius = (value - 32) * 5/9;
    }
}

const temp = new Temperature(25);
console.log(temp.celsius);    // 25
console.log(temp.fahrenheit); // 77
temp.fahrenheit = 86;
console.log(temp.celsius);    // 30
```

### Mixins Pattern
```javascript
// Mixin functions
const Flyable = {
    fly() {
        console.log(`${this.name} is flying`);
    }
};

const Swimmable = {
    swim() {
        console.log(`${this.name} is swimming`);
    }
};

// Apply mixins to class
class Duck {
    constructor(name) {
        this.name = name;
    }
}

// Add mixin methods
Object.assign(Duck.prototype, Flyable, Swimmable);

const duck = new Duck('Donald');
duck.fly();  // "Donald is flying"
duck.swim(); // "Donald is swimming"
```

## Best Practices

### 1. Constructor Best Practices
```javascript
class User {
    constructor(name, email, options = {}) {
        // Validate required parameters
        if (!name || !email) {
            throw new Error('Name and email are required');
        }
        
        // Use default values
        this.name = name;
        this.email = email;
        this.isActive = options.isActive ?? true;
        this.role = options.role || 'user';
        
        // Initialize arrays/objects to avoid sharing references
        this.permissions = [...(options.permissions || [])];
    }
}
```

### 2. Method Chaining
```javascript
class Calculator {
    constructor(value = 0) {
        this.value = value;
    }
    
    add(num) {
        this.value += num;
        return this; // Return this for chaining
    }
    
    multiply(num) {
        this.value *= num;
        return this;
    }
    
    result() {
        return this.value;
    }
}

const result = new Calculator(5)
    .add(3)
    .multiply(2)
    .result(); // 16
```

### 3. Error Handling in Constructors
```javascript
class DatabaseConnection {
    constructor(config) {
        try {
            this.validateConfig(config);
            this.connect(config);
        } catch (error) {
            this.cleanup();
            throw error; // Re-throw after cleanup
        }
    }
    
    validateConfig(config) {
        if (!config.host || !config.port) {
            throw new Error('Host and port are required');
        }
    }
    
    connect(config) {
        // Connection logic
    }
    
    cleanup() {
        // Cleanup resources
    }
}
```

### Key Takeaways
1. **Always use `new`** with classes (enforced automatically)
2. **Call `super()`** before using `this` in derived class constructors
3. **Use static methods** for utility functions related to the class
4. **Private fields** (`#field`) for true encapsulation in modern environments
5. **Getters/setters** for controlled property access
6. **Method chaining** for fluent APIs
7. **Proper error handling** in constructors and methods 