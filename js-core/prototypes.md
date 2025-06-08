# JavaScript Prototypes Guide

## Table of Contents
- [Prototype Inheritance Fundamentals](#prototype-inheritance-fundamentals)
- [__proto__ vs prototype](#__proto__-vs-prototype)
- [hasOwnProperty and Property Ownership](#hasownproperty-and-property-ownership)
- [Object Creation and Prototype Chain](#object-creation-and-prototype-chain)
- [Object.create() and Descriptors](#objectcreate-and-descriptors)
- [Functions and Constructor Prototypes](#functions-and-constructor-prototypes)
- [Constructor Property Management](#constructor-property-management)
- [JavaScript Object Hierarchy](#javascript-object-hierarchy)
- [Prototype-based Inheritance Patterns](#prototype-based-inheritance-patterns)
- [Modern Alternatives](#modern-alternatives)
- [Encapsulation Concerns](#encapsulation-concerns)
- [Best Practices](#best-practices)

## Prototype Inheritance Fundamentals

**Important Note:** As mentioned in your code, everything written through prototypes (including Classes) has encapsulation problems! If encapsulation is important, it's better not to write with prototypes. Use functional approach instead!

### Basic Prototype Inheritance
Based on your fundamental example:

```javascript
// __proto__ => (object property) object inheritance in prototype style
// The interpreter will first look for a property in the object, if not found,
// it will look in the object from which it inherited

const animal = {
    eats: true
};

const rabbit = {
    jumps: true
};

rabbit.__proto__ = animal; // Equivalent to: Object.setPrototypeOf(rabbit, animal);
console.log(rabbit.jumps); // true
console.log(rabbit.eats); // true
console.log(rabbit); // { jumps: true } ! without eats
```

### Prototype Lookup Mechanism
```javascript
const grandparent = {
    species: 'mammal',
    breathes: true
};

const parent = {
    __proto__: grandparent,
    warmBlooded: true
};

const child = {
    __proto__: parent,
    name: 'Fluffy'
};

// Property lookup goes up the chain
console.log(child.name); // 'Fluffy' (own property)
console.log(child.warmBlooded); // true (from parent)
console.log(child.species); // 'mammal' (from grandparent)
console.log(child.breathes); // true (from grandparent)
console.log(child.nonexistent); // undefined (not found anywhere)

// The lookup process:
// 1. Check child object
// 2. Check child.__proto__ (parent)
// 3. Check parent.__proto__ (grandparent)
// 4. Check grandparent.__proto__ (Object.prototype)
// 5. Check Object.prototype.__proto__ (null)
// 6. Return undefined if not found
```

### Prototype is Only for Reading
As you emphasized, `__proto__` is a mechanism that only works for searching:

```javascript
const animal = { eats: true };
const rabbit = { jumps: true };
rabbit.__proto__ = animal;

delete rabbit.eats; // Trying to delete from rabbit
console.log(rabbit.eats); // true - still outputs true!

// The delete operation only affects own properties
delete animal.eats; // This actually deletes it
console.log(rabbit.eats); // undefined - now it's gone
```

### Property Assignment vs Inheritance
```javascript
const animal = {
    eats: true,
    speed: 0
};

const rabbit = {
    __proto__: animal,
    jumps: true
};

// Reading goes up the prototype chain
console.log(rabbit.eats); // true (from animal)

// Writing creates own property (doesn't affect prototype)
rabbit.eats = false;
console.log(rabbit.eats); // false (own property)
console.log(animal.eats); // true (unchanged)

// Now rabbit has its own eats property
console.log(rabbit.hasOwnProperty('eats')); // true
```

## __proto__ vs prototype

Understanding the difference between `__proto__` and `prototype` is crucial.

### __proto__ - Instance Property
```javascript
// __proto__ is a property of object instances
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true

// It points to the prototype object
const animal = { type: 'animal' };
const dog = { breed: 'labrador' };
dog.__proto__ = animal;

console.log(dog.__proto__ === animal); // true
console.log(dog.type); // 'animal' (inherited)
```

### prototype - Function Property
```javascript
// prototype is a property of constructor functions
function Animal(name) {
    this.name = name;
}

console.log(typeof Animal.prototype); // 'object'
console.log(Animal.prototype.constructor === Animal); // true

// When using 'new', the created object's __proto__ points to constructor's prototype
const dog = new Animal('Rex');
console.log(dog.__proto__ === Animal.prototype); // true
console.log(dog.constructor === Animal); // true
```

### The Relationship
```javascript
function User(name) {
    this.name = name;
}

User.prototype.sayHi = function() {
    console.log(`Hi, I'm ${this.name}`);
};

const user = new User('John');

// These are all the same object:
console.log(user.__proto__ === User.prototype); // true
console.log(Object.getPrototypeOf(user) === User.prototype); // true

// The method is found through prototype chain
user.sayHi(); // "Hi, I'm John"
console.log(user.hasOwnProperty('sayHi')); // false (inherited)
```

## hasOwnProperty and Property Ownership

### Basic Property Ownership Checking
Based on your example:

```javascript
// hasOwnProperty method to check if a property is in the object or not (or if it's in the prototype)
console.log(rabbit.hasOwnProperty('jumps')); // true
console.log(rabbit.hasOwnProperty('eats')); // false
```

### Iteration and Property Ownership
As you noted, prototype properties appear in iteration:

```javascript
const animal = { eats: true };
const rabbit = { jumps: true };
rabbit.__proto__ = animal;

// for...in iterates over all enumerable properties (own + inherited)
for (var key in rabbit) {
    console.log(`${key} = ${rabbit[key]}`); // jumps = true, eats = true
}

// Filter to only own properties
for (var key in rabbit) {
    if (rabbit.hasOwnProperty(key)) {
        console.log(`Own property: ${key} = ${rabbit[key]}`); // Only jumps = true
    }
}
```

### Modern Property Checking Methods
```javascript
const parent = { inherited: 'value' };
const child = { 
    __proto__: parent,
    own: 'property' 
};

// Different ways to check property existence
console.log('own' in child); // true
console.log('inherited' in child); // true
console.log('nonexistent' in child); // false

console.log(child.hasOwnProperty('own')); // true
console.log(child.hasOwnProperty('inherited')); // false

// Modern alternative (ES2022)
console.log(Object.hasOwn(child, 'own')); // true
console.log(Object.hasOwn(child, 'inherited')); // false

// Get only own properties
console.log(Object.keys(child)); // ['own']
console.log(Object.getOwnPropertyNames(child)); // ['own']

// Get all enumerable properties (including inherited)
console.log(Object.keys(child).concat(
    Object.keys(Object.getPrototypeOf(child))
)); // ['own', 'inherited']
```

### Safe hasOwnProperty Usage
```javascript
// hasOwnProperty can be overridden, so use safe approach
const obj = {
    hasOwnProperty: 'not a function',
    realProperty: 'value'
};

// Unsafe
// console.log(obj.hasOwnProperty('realProperty')); // TypeError

// Safe approach
console.log(Object.prototype.hasOwnProperty.call(obj, 'realProperty')); // true

// Or use modern Object.hasOwn
console.log(Object.hasOwn(obj, 'realProperty')); // true
```

## Object Creation and Prototype Chain

### Default Prototype Chain
Based on your observation about object literals:

```javascript
// When creating an object using literal {}, we create an object with some __proto__ chain
const data = {}; // Under the hood there will be something like {}.__proto__ = x ... ({}.__proto__ = x.__proto__ = ...)
console.log(data.toString); // [Function: toString]
```

### Creating Clean Objects
As you noted, sometimes we need absolutely clean objects:

```javascript
// If you need to create an absolutely clean object
const data2 = Object.create(null); // null as prototype
console.log(data2.toString); // undefined
console.log(data2.__proto__); // undefined

// Use cases for clean objects
const cleanConfig = Object.create(null);
cleanConfig.apiKey = 'secret';
cleanConfig.timeout = 5000;

// No inherited methods to worry about
console.log(cleanConfig.toString); // undefined
console.log(cleanConfig.hasOwnProperty); // undefined

// But you can still use Object methods
console.log(Object.keys(cleanConfig)); // ['apiKey', 'timeout']
```

### Prototype Chain Hierarchy
```javascript
// JavaScript object hierarchy: null => Object.prototype => {}
const obj = {};

console.log(obj.toString === Object.prototype.toString); // true. toString method is taken from prototype
console.log(obj.__proto__ === Object.prototype); // true
console.log(obj.__proto__.__proto__); // null

// The complete chain
console.log('Object hierarchy:');
console.log('obj ---> Object.prototype ---> null');
console.log(obj.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); // true
```

### Custom Prototype Chains
```javascript
// Creating custom prototype hierarchies
const level3 = { level: 3 };
const level2 = Object.create(level3);
level2.level = 2;
const level1 = Object.create(level2);
level1.level = 1;
const level0 = Object.create(level1);
level0.level = 0;

// Walking up the chain
function printPrototypeChain(obj) {
    let current = obj;
    let depth = 0;
    
    while (current !== null) {
        console.log(`Level ${depth}:`, current);
        current = Object.getPrototypeOf(current);
        depth++;
        if (depth > 10) break; // Safety check
    }
}

printPrototypeChain(level0);
```

## Object.create() and Descriptors

### Getting Prototype Information
Based on your example:

```javascript
// To get the prototype from which we inherited
const proto = Object.getPrototypeOf(rabbit); // => {eats: true}
```

### Creating Objects with Descriptors
As shown in your example:

```javascript
// Creating a new object immediately with descriptors
const cat = Object.create(animal, {
    name: { value: 'Cat' }
});
console.log(cat.name); // Cat
console.log(cat.eats); // true
```

### Advanced Object.create() Usage
```javascript
const animalPrototype = {
    speak() {
        return `${this.name} makes a sound`;
    },
    
    describe() {
        return `This is ${this.name}, a ${this.species}`;
    }
};

// Create object with complex descriptors
const dog = Object.create(animalPrototype, {
    name: {
        value: 'Rex',
        writable: true,
        enumerable: true,
        configurable: true
    },
    
    species: {
        value: 'dog',
        writable: false,
        enumerable: true,
        configurable: false
    },
    
    age: {
        get() { return this._age || 0; },
        set(value) { 
            if (value >= 0) this._age = value; 
        },
        enumerable: true,
        configurable: true
    }
});

dog.age = 5;
console.log(dog.speak()); // "Rex makes a sound"
console.log(dog.describe()); // "This is Rex, a dog"
console.log(dog.age); // 5
```

### Cloning Objects with Prototypes
```javascript
function cloneObject(obj) {
    // Clone with same prototype
    const clone = Object.create(Object.getPrototypeOf(obj));
    
    // Copy own properties with descriptors
    const descriptors = Object.getOwnPropertyDescriptors(obj);
    Object.defineProperties(clone, descriptors);
    
    return clone;
}

const original = {
    __proto__: animalPrototype,
    name: 'Original',
    value: 42
};

const clone = cloneObject(original);
console.log(clone.name); // 'Original'
console.log(clone.__proto__ === animalPrototype); // true
```

## Functions and Constructor Prototypes

### Working with Function Constructors
Based on your example:

```javascript
// Working with functions!
function Rabbit(name) {
    this.name = name;
    this.__proto__ = animal; // Not recommended approach
}

const rabbit2 = new Rabbit('Bob');
console.log(rabbit2.name); // Bob
console.log(rabbit2.eats); // true
```

### Proper Constructor Pattern
```javascript
// Better approach using prototype property
function Animal(name) {
    this.name = name;
}

Animal.prototype.eat = function() {
    return `${this.name} is eating`;
};

Animal.prototype.sleep = function() {
    return `${this.name} is sleeping`;
};

function Dog(name, breed) {
    Animal.call(this, name); // Call parent constructor
    this.breed = breed;
}

// Set up inheritance properly
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    return `${this.name} says woof!`;
};

const dog = new Dog('Rex', 'German Shepherd');
console.log(dog.eat()); // "Rex is eating"
console.log(dog.bark()); // "Rex says woof!"
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

## Constructor Property Management

### Constructor Property Issues
As you pointed out, overwriting prototype can cause constructor issues:

```javascript
// prototype => special property in functions. (proto & prototype solve the same task, but their mechanism is different)
console.log(Rabbit.prototype.constructor); // => [Function: Rabbit]

Rabbit.prototype = animal; // Doing this is not good because we overwrite the constructor!
// Essentially, we won't be able to track how we created new functions (new Rabbit)
console.log(Rabbit.prototype.constructor); // => [Function: Object]

// To protect yourself and prevent this, after such actions you need to explicitly specify the constructor
Rabbit.prototype.constructor = Rabbit;
```

### Default Constructor Behavior
Based on your observation:

```javascript
function Rabbit2() {}
console.log(Object.getOwnPropertyNames(Rabbit2.prototype)); // ['constructor'] => constructor will always be in the prototype!
console.log(Rabbit2.prototype.constructor === Rabbit2); // true => constructor will point to the function itself
```

### Building Objects from Constructor References
As shown in your clever example:

```javascript
// Way to build one object from another without information about the class itself
function Rabbit(name) {
    this.name = name;
}

const rabbit = new Rabbit('Marcus');
const rabbitW = new rabbit.constructor('Lisa'); // !!!!!

console.log(rabbitW instanceof Rabbit); // true
console.log(rabbitW.name); // 'Lisa'
```

### Constructor Property Best Practices
```javascript
function Vehicle(type) {
    this.type = type;
}

Vehicle.prototype.start = function() {
    return `${this.type} is starting`;
};

function Car(brand) {
    Vehicle.call(this, 'car');
    this.brand = brand;
}

// Method 1: Using Object.create (preserves constructor automatically)
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car; // Fix constructor reference

// Method 2: Using __proto__ (preserves constructor)
// Car.prototype.__proto__ = Vehicle.prototype; // Historical alternative

Car.prototype.drive = function() {
    return `${this.brand} car is driving`;
};

// Verify constructor chain
const car = new Car('Toyota');
console.log(car.constructor === Car); // true
console.log(car instanceof Car); // true
console.log(car instanceof Vehicle); // true

// Generic object creation using constructor
function createSimilar(obj, ...args) {
    return new obj.constructor(...args);
}

const anotherCar = createSimilar(car, 'Honda');
console.log(anotherCar instanceof Car); // true
console.log(anotherCar.brand); // 'Honda'
```

## JavaScript Object Hierarchy

### Built-in Prototype Chain
Based on your explanation of the hierarchy:

```javascript
// Hierarchy of clean objects in JS: null => Object.prototype => {}
const obj = {};
obj.toString === Object.prototype.toString; // true. toString method is taken from prototype
obj.__proto__ === Object.prototype; // true
console.log(obj.__proto__.__proto__); // null
```

### Complete Hierarchy Visualization
```javascript
// Visualizing the complete prototype chain
function analyzePrototypeChain(obj, name = 'obj') {
    console.log(`\nPrototype chain for ${name}:`);
    
    let current = obj;
    let level = 0;
    
    while (current !== null) {
        const constructor = current.constructor?.name || 'Unknown';
        const ownProperties = Object.getOwnPropertyNames(current);
        
        console.log(`Level ${level}: ${constructor}`);
        console.log(`  Properties: ${ownProperties.join(', ')}`);
        
        current = Object.getPrototypeOf(current);
        level++;
        
        if (level > 10) break; // Safety
    }
}

// Different object types
analyzePrototypeChain({}, 'plain object');
analyzePrototypeChain([], 'array');
analyzePrototypeChain(new Date(), 'date');
analyzePrototypeChain(function(){}, 'function');
```

### Built-in Object Hierarchies
```javascript
// Array hierarchy
const arr = [];
console.log('Array prototype chain:');
console.log(arr.__proto__ === Array.prototype); // true
console.log(Array.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); // true

// Function hierarchy
function fn() {}
console.log('Function prototype chain:');
console.log(fn.__proto__ === Function.prototype); // true
console.log(Function.prototype.__proto__ === Object.prototype); // true

// Date hierarchy
const date = new Date();
console.log('Date prototype chain:');
console.log(date.__proto__ === Date.prototype); // true
console.log(Date.prototype.__proto__ === Object.prototype); // true
```

## Prototype-based Inheritance Patterns

### Classic Inheritance Pattern
Based on your comprehensive example:

```javascript
// Example of prototype inheritance of classes
function Animal(name) { // Animal constructor
    this.name = name;
}

Animal.prototype.sayHi = function() { // method in prototype
    console.log(this.name);
};

function Dog() { // Dog constructor
    Animal.apply(this, arguments); // essentially after this, Dog will have all fields (properties) of Animal only with Dog context
    this.speed = 0;
}

// One inheritance variant
Dog.prototype = Object.create(Animal.prototype); // here methods are inherited
Dog.prototype.constructor = Dog; // after previous action we overwrite constructor! here we fix it

// Another inheritance variant
// Dog.prototype.__proto__ = Animal.prototype; // this way we would preserve constructor! this method appeared (historically) later than Dog.prototype = Object.create(Animal.prototype)
```

### Enhanced Inheritance Pattern
```javascript
// More robust inheritance implementation
function createInheritance(Child, Parent) {
    // Set up prototype chain
    Child.prototype = Object.create(Parent.prototype);
    Child.prototype.constructor = Child;
    
    // Set up static inheritance (for static methods)
    Object.setPrototypeOf(Child, Parent);
    
    return Child;
}

function Animal(name, species) {
    this.name = name;
    this.species = species;
}

Animal.prototype.speak = function() {
    return `${this.name} makes a sound`;
};

Animal.prototype.describe = function() {
    return `${this.name} is a ${this.species}`;
};

Animal.getKingdom = function() {
    return 'Animalia';
};

function Mammal(name, species, warmBlooded = true) {
    Animal.call(this, name, species);
    this.warmBlooded = warmBlooded;
}

createInheritance(Mammal, Animal);

Mammal.prototype.nurse = function() {
    return `${this.name} is nursing offspring`;
};

function Dog(name, breed) {
    Mammal.call(this, name, 'Canis lupus');
    this.breed = breed;
}

createInheritance(Dog, Mammal);

Dog.prototype.speak = function() {
    return `${this.name} barks: Woof!`;
};

Dog.prototype.fetch = function() {
    return `${this.name} fetches the ball`;
};

const dog = new Dog('Rex', 'German Shepherd');
console.log(dog.speak()); // "Rex barks: Woof!"
console.log(dog.nurse()); // "Rex is nursing offspring"
console.log(dog.describe()); // "Rex is a Canis lupus"
console.log(Dog.getKingdom()); // "Animalia" (inherited static method)
```

### Mixin Pattern
```javascript
// Mixins for multiple inheritance-like behavior
const Flyable = {
    fly() {
        return `${this.name} is flying`;
    },
    
    land() {
        return `${this.name} has landed`;
    }
};

const Swimmable = {
    swim() {
        return `${this.name} is swimming`;
    },
    
    dive() {
        return `${this.name} is diving`;
    }
};

function Bird(name, species) {
    Animal.call(this, name, species);
}

createInheritance(Bird, Animal);

// Add flying capability
Object.assign(Bird.prototype, Flyable);

function Duck(name) {
    Bird.call(this, name, 'duck');
}

createInheritance(Duck, Bird);

// Add swimming capability
Object.assign(Duck.prototype, Swimmable);

Duck.prototype.speak = function() {
    return `${this.name} quacks`;
};

const duck = new Duck('Donald');
console.log(duck.speak()); // "Donald quacks"
console.log(duck.fly()); // "Donald is flying"
console.log(duck.swim()); // "Donald is swimming"
```

## Modern Alternatives

### ES6 Classes
```javascript
// Modern class syntax (syntactic sugar over prototypes)
class ModernAnimal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
    
    describe() {
        return `${this.name} is a ${this.species}`;
    }
    
    static getKingdom() {
        return 'Animalia';
    }
}

class ModernDog extends ModernAnimal {
    constructor(name, breed) {
        super(name, 'dog');
        this.breed = breed;
    }
    
    speak() {
        return `${this.name} barks: Woof!`;
    }
    
    fetch() {
        return `${this.name} fetches the ball`;
    }
}

const modernDog = new ModernDog('Max', 'Golden Retriever');
console.log(modernDog.speak()); // "Max barks: Woof!"
console.log(modernDog instanceof ModernDog); // true
console.log(modernDog instanceof ModernAnimal); // true

// Under the hood, it's still prototypes
console.log(modernDog.__proto__ === ModernDog.prototype); // true
console.log(ModernDog.prototype.__proto__ === ModernAnimal.prototype); // true
```

### Factory Functions
```javascript
// Factory function approach (better encapsulation)
function createAnimal(name, species) {
    // Private variables
    let energy = 100;
    
    return {
        name,
        species,
        
        speak() {
            return `${this.name} makes a sound`;
        },
        
        getEnergy() {
            return energy;
        },
        
        rest() {
            energy = Math.min(100, energy + 20);
            return `${this.name} is resting`;
        },
        
        exercise() {
            energy = Math.max(0, energy - 10);
            return `${this.name} is exercising`;
        }
    };
}

function createDog(name, breed) {
    const animal = createAnimal(name, 'dog');
    
    return Object.assign(animal, {
        breed,
        
        speak() {
            return `${this.name} barks: Woof!`;
        },
        
        fetch() {
            animal.exercise();
            return `${this.name} fetches the ball`;
        }
    });
}

const factoryDog = createDog('Buddy', 'Labrador');
console.log(factoryDog.speak()); // "Buddy barks: Woof!"
console.log(factoryDog.getEnergy()); // 100
console.log(factoryDog.fetch()); // "Buddy fetches the ball"
console.log(factoryDog.getEnergy()); // 90
```

## Encapsulation Concerns

### Prototype Encapsulation Problems
As you emphasized at the beginning, prototypes have encapsulation issues:

```javascript
// Prototypes don't provide true encapsulation
function User(name) {
    this.name = name;
    this._password = 'secret'; // "private" by convention only
}

User.prototype.login = function(password) {
    return password === this._password;
};

const user = new User('John');
console.log(user._password); // 'secret' - accessible!
user._password = 'hacked'; // Can be modified!
console.log(user.login('secret')); // false - original password changed

// Prototype methods can be modified
User.prototype.login = function() { return true; }; // Security breach!
```

### Better Encapsulation with Closures
```javascript
// Functional approach with true encapsulation
function createSecureUser(name, password) {
    // Truly private variables
    let _name = name;
    let _password = password;
    let _loginAttempts = 0;
    
    return {
        getName() {
            return _name;
        },
        
        login(pwd) {
            _loginAttempts++;
            if (_loginAttempts > 3) {
                throw new Error('Too many login attempts');
            }
            return pwd === _password;
        },
        
        changePassword(oldPwd, newPwd) {
            if (oldPwd === _password) {
                _password = newPwd;
                return true;
            }
            return false;
        },
        
        getLoginAttempts() {
            return _loginAttempts;
        }
    };
}

const secureUser = createSecureUser('John', 'secret123');
console.log(secureUser.getName()); // 'John'
console.log(secureUser._password); // undefined - truly private!
console.log(secureUser.login('wrong')); // false
console.log(secureUser.login('secret123')); // true
```

### Modern Private Fields
```javascript
// ES2022 private fields (true encapsulation with classes)
class SecureUser {
    #password;
    #loginAttempts = 0;
    
    constructor(name, password) {
        this.name = name;
        this.#password = password;
    }
    
    login(password) {
        this.#loginAttempts++;
        if (this.#loginAttempts > 3) {
            throw new Error('Too many login attempts');
        }
        return password === this.#password;
    }
    
    changePassword(oldPassword, newPassword) {
        if (oldPassword === this.#password) {
            this.#password = newPassword;
            return true;
        }
        return false;
    }
    
    getLoginAttempts() {
        return this.#loginAttempts;
    }
}

const modernUser = new SecureUser('Jane', 'secure456');
console.log(modernUser.name); // 'Jane'
// console.log(modernUser.#password); // SyntaxError - truly private!
console.log(modernUser.login('secure456')); // true
```

## Best Practices

### Choosing the Right Approach
```javascript
// Decision matrix for object creation patterns

// Use prototypes when:
// - Performance is critical (shared methods)
// - You need instanceof checks
// - Working with existing prototype-based code

// Use factory functions when:
// - Encapsulation is important
// - You need flexible object composition
// - Avoiding 'this' binding issues

// Use ES6 classes when:
// - You want familiar syntax
// - Team prefers class-based patterns
// - Using TypeScript (better type support)

// Use modern private fields when:
// - You need true encapsulation with classes
// - Target environments support ES2022
```

### Prototype Modification Guidelines
```javascript
// ✅ Good practices
function MyClass() {}

// Extend your own prototypes
MyClass.prototype.myMethod = function() {
    return 'custom method';
};

// ❌ Avoid modifying built-in prototypes
// Array.prototype.myCustomMethod = function() {}; // Don't do this!

// ✅ If you must extend built-ins, check first
if (!Array.prototype.myCustomMethod) {
    Array.prototype.myCustomMethod = function() {
        // Implementation
    };
}

// ✅ Better: use utility functions
function myCustomArrayMethod(arr) {
    // Implementation
}
```

### Performance Considerations
```javascript
// Prototype methods are shared across instances (memory efficient)
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

const person1 = new Person('Alice');
const person2 = new Person('Bob');

// Same method reference (memory efficient)
console.log(person1.greet === person2.greet); // true

// vs Factory functions (each instance has its own methods)
function createPerson(name) {
    return {
        name,
        greet() {
            return `Hello, I'm ${this.name}`;
        }
    };
}

const factory1 = createPerson('Alice');
const factory2 = createPerson('Bob');

// Different method references (more memory usage)
console.log(factory1.greet === factory2.greet); // false
```

### Debugging Prototypes
```javascript
// Utility functions for prototype debugging
function getPrototypeChain(obj) {
    const chain = [];
    let current = obj;
    
    while (current !== null) {
        chain.push({
            constructor: current.constructor?.name || 'Unknown',
            properties: Object.getOwnPropertyNames(current)
        });
        current = Object.getPrototypeOf(current);
    }
    
    return chain;
}

function findMethodSource(obj, methodName) {
    let current = obj;
    
    while (current !== null) {
        if (Object.hasOwn(current, methodName)) {
            return current.constructor?.name || 'Unknown';
        }
        current = Object.getPrototypeOf(current);
    }
    
    return 'Not found';
}

// Usage
const dog = new Dog('Rex', 'Labrador');
console.log(getPrototypeChain(dog));
console.log(findMethodSource(dog, 'speak')); // 'Dog'
console.log(findMethodSource(dog, 'toString')); // 'Object'
```

### Key Takeaways

1. **Prototypes have encapsulation problems** - Use functional approaches for true privacy
2. **__proto__ is for reading only** - Property assignment creates own properties
3. **prototype vs __proto__** - Functions have prototype, instances have __proto__
4. **hasOwnProperty distinguishes** own vs inherited properties
5. **Constructor property management** - Always fix constructor after prototype assignment
6. **Object.create() provides clean inheritance** - Better than direct __proto__ assignment
7. **for...in includes inherited properties** - Use hasOwnProperty to filter
8. **Modern alternatives exist** - Classes, private fields, factory functions
9. **Performance trade-offs** - Prototypes save memory, factories provide encapsulation
10. **Choose based on requirements** - Encapsulation vs performance vs syntax preferences 