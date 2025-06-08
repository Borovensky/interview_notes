# JavaScript Closures Guide

## Table of Contents
- [Closure Fundamentals](#closure-fundamentals)
- [Lexical Environment and Scope](#lexical-environment-and-scope)
- [The Counter Example](#the-counter-example)
- [How Closures Work Internally](#how-closures-work-internally)
- [Closure Creation Process](#closure-creation-process)
- [Variable Lookup Chain](#variable-lookup-chain)
- [Practical Closure Examples](#practical-closure-examples)
- [Closure Patterns](#closure-patterns)
- [Memory and Performance](#memory-and-performance)
- [Common Pitfalls](#common-pitfalls)
- [Advanced Closure Concepts](#advanced-closure-concepts)
- [Best Practices](#best-practices)

## Closure Fundamentals

Based on your detailed explanation, a closure is fundamentally about functions and their access to external variables.

### Definition and Core Concepts
From your comprehensive explanation:

```javascript
// Closure (Замыкание) - это функция вместе со всеми внешними переменными, которые ей доступны. - wikipedia 
// Closure = LexicalEnvironment (создается когда вызывается функция) + Scope (создается когда объявляется функция)

// Technically, any function creates a closure...
// Читехнически любая фунцкия создает замыкание...
```

### Basic Closure Example
```javascript
function outerFunction(x) {
    // This variable is in the outer function's scope
    const outerVariable = x;
    
    // Inner function has access to outer function's variables
    function innerFunction(y) {
        console.log(outerVariable + y); // Accessing outerVariable from closure
    }
    
    return innerFunction;
}

const closure = outerFunction(10);
closure(5); // 15

// Even after outerFunction has finished executing,
// innerFunction still has access to outerVariable
```

### Every Function Creates a Closure
```javascript
// Even simple functions create closures
const globalVar = 'global';

function simpleFunction() {
    console.log(globalVar); // Accesses global via closure
}

simpleFunction(); // 'global'

// The closure includes the global scope
// Function + accessible variables = closure
```

## Lexical Environment and Scope

### The Technical Breakdown
Based on your technical explanation:

```javascript
// - Каждая функция при создании получает ссылку Scope.
// - Scope указывает на объект с переменными, в контексте которого была создана функция.
// - При запуске функции создается новый объект LexicalEnvironment.
// - Поиск переменных осуществляется сначала в текущем LexicalEnvironment.
// - LexicalEnvironment => Scope => LexicalEnvironment.
```

### Scope Reference Creation
```javascript
const globalVar = 'I am global';

function createFunctionWithScope() {
    const outerVar = 'I am in outer scope';
    
    // When this function is DECLARED, it gets a reference to current scope
    function innerFunction() {
        const innerVar = 'I am local';
        
        // Variable lookup order:
        // 1. Current LexicalEnvironment (innerVar)
        // 2. Scope reference (outerVar)  
        // 3. Scope's scope reference (globalVar)
        console.log(innerVar);  // Found in step 1
        console.log(outerVar);  // Found in step 2
        console.log(globalVar); // Found in step 3
    }
    
    return innerFunction;
}

const closureFunction = createFunctionWithScope();
// The returned function carries its scope chain with it
closureFunction();
```

### Lexical Environment Creation
```javascript
function demonstrateLexicalEnvironment() {
    let count = 0;
    
    return function increment() {
        // When this function RUNS, new LexicalEnvironment is created
        // This environment contains:
        // 1. Local variables (none in this case)
        // 2. Reference to outer scope (where 'count' lives)
        
        count++; // Found via scope chain
        console.log(count);
    };
}

const incrementer = demonstrateLexicalEnvironment();
incrementer(); // 1 - new LexicalEnvironment created
incrementer(); // 2 - another new LexicalEnvironment created
incrementer(); // 3 - yet another new LexicalEnvironment created

// Each call creates new LexicalEnvironment but shares same scope reference
```

## The Counter Example

### Classic Counter Implementation
Based on your example:

```javascript
function makeCounter() {
    var currentCount = 1; // currentCount находится в замыкании

    return function() {
        return currentCount++;
    }
}

var counter = makeCounter();
counter(); // 1
counter(); // 2
var counter2 = makeCounter();
counter2(); // 1
```

### Understanding Counter Behavior
```javascript
function makeCounter() {
    let currentCount = 1;
    
    // The returned function closes over currentCount
    return function() {
        return currentCount++;
    };
}

// Each call to makeCounter creates a new scope
const counter1 = makeCounter();
const counter2 = makeCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter1()); // 3

console.log(counter2()); // 1 - independent counter!
console.log(counter2()); // 2

// counter1 and counter2 have separate closures
// Each has its own currentCount variable
```

### Enhanced Counter with Multiple Operations
```javascript
function makeAdvancedCounter(initialValue = 0) {
    let count = initialValue;
    
    return {
        increment() {
            return ++count;
        },
        
        decrement() {
            return --count;
        },
        
        getValue() {
            return count;
        },
        
        reset() {
            count = initialValue;
            return count;
        }
    };
}

const advancedCounter = makeAdvancedCounter(10);
console.log(advancedCounter.increment()); // 11
console.log(advancedCounter.increment()); // 12
console.log(advancedCounter.decrement()); // 11
console.log(advancedCounter.getValue());  // 11
console.log(advancedCounter.reset());     // 10
```

## How Closures Work Internally

### Memory Model
```javascript
function createClosure() {
    let data = 'I am preserved';
    let unused = 'I might be garbage collected';
    
    return function() {
        console.log(data); // Only 'data' is referenced
        // 'unused' might be optimized away by JS engine
    };
}

const closure = createClosure();
// The returned function maintains reference to 'data'
// Original function execution context is gone, but variables live on

closure(); // 'I am preserved'

// Even though createClosure finished executing,
// 'data' is kept alive because closure references it
```

### Scope Chain Visualization
```javascript
const global = 'global level';

function level1() {
    const var1 = 'level 1';
    
    function level2() {
        const var2 = 'level 2';
        
        function level3() {
            const var3 = 'level 3';
            
            // This function's closure includes all outer scopes
            return function() {
                console.log(var3);  // Direct scope
                console.log(var2);  // Parent scope
                console.log(var1);  // Grandparent scope
                console.log(global); // Global scope
            };
        }
        
        return level3();
    }
    
    return level2();
}

const deepClosure = level1();
deepClosure(); // Has access to entire scope chain
```

## Closure Creation Process

### Step-by-Step Process
```javascript
// Step 1: Function declaration creates scope reference
function outer() {
    const outerVar = 'outer';
    
    // Step 2: Inner function declaration gets reference to current scope
    function inner() {
        console.log(outerVar);
    }
    
    // Step 3: Return inner function (with its scope reference)
    return inner;
}

// Step 4: Call outer function
const closureFunction = outer();
// outer() execution context is destroyed
// But outerVar is kept alive because inner() references it

// Step 5: Call the closure
closureFunction(); // 'outer'
// New LexicalEnvironment created for this call
// Variable lookup follows scope chain
```

### Multiple Closures Sharing Scope
```javascript
function createSharedScope() {
    let sharedVariable = 0;
    
    const increment = function() {
        sharedVariable++;
        console.log('Incremented:', sharedVariable);
    };
    
    const decrement = function() {
        sharedVariable--;
        console.log('Decremented:', sharedVariable);
    };
    
    const getValue = function() {
        return sharedVariable;
    };
    
    // All three functions share the same scope
    return { increment, decrement, getValue };
}

const { increment, decrement, getValue } = createSharedScope();

increment(); // Incremented: 1
increment(); // Incremented: 2
decrement(); // Decremented: 1
console.log(getValue()); // 1

// All three functions share the same sharedVariable
```

## Variable Lookup Chain

### Lookup Process Implementation
Based on your explanation of the lookup chain:

```javascript
// LexicalEnvironment => Scope => LexicalEnvironment
function demonstrateLookupChain() {
    const level1Var = 'level 1';
    
    function level2() {
        const level2Var = 'level 2';
        
        return function level3() {
            const level3Var = 'level 3';
            
            // Variable lookup demonstration:
            
            // 1. Look in current LexicalEnvironment
            console.log(level3Var); // Found immediately
            
            // 2. Not found? Look in Scope reference (level2)
            console.log(level2Var); // Found in parent scope
            
            // 3. Not found? Look in Scope's Scope reference (level1)
            console.log(level1Var); // Found in grandparent scope
            
            // 4. Continue until found or reach global
            console.log(typeof window !== 'undefined' ? 'browser' : 'node'); // Found in global scope
        };
    }
    
    return level2();
}

const lookupDemo = demonstrateLookupChain();
lookupDemo();
```

### Optimized Lookup
```javascript
function createOptimizedClosure() {
    const frequently_used = 'I am accessed often';
    const rarely_used = 'I am rarely accessed';
    const never_used = 'I am never accessed';
    
    return function() {
        // Modern JS engines optimize closure variables
        // Only variables actually used are kept in closure
        console.log(frequently_used);
        
        if (Math.random() > 0.9) {
            console.log(rarely_used);
        }
        
        // never_used might be optimized away
    };
}

const optimizedClosure = createOptimizedClosure();
optimizedClosure();
```

## Practical Closure Examples

### Module Pattern
```javascript
const myModule = (function() {
    // Private variables and functions
    let privateVariable = 0;
    
    function privateFunction() {
        return 'This is private';
    }
    
    // Public API
    return {
        publicMethod() {
            privateVariable++;
            return privateFunction() + ` (called ${privateVariable} times)`;
        },
        
        getCount() {
            return privateVariable;
        }
    };
})();

console.log(myModule.publicMethod()); // "This is private (called 1 times)"
console.log(myModule.getCount()); // 1
// console.log(myModule.privateVariable); // undefined - truly private!
```

### Event Handlers with State
```javascript
function createButtonHandler(buttonName) {
    let clickCount = 0;
    
    return function(event) {
        clickCount++;
        console.log(`${buttonName} clicked ${clickCount} times`);
        
        if (clickCount >= 5) {
            console.log(`${buttonName} is very popular!`);
        }
    };
}

// Each button gets its own click counter
const loginHandler = createButtonHandler('Login Button');
const signupHandler = createButtonHandler('Signup Button');

// Simulate clicks
loginHandler(); // "Login Button clicked 1 times"
loginHandler(); // "Login Button clicked 2 times"
signupHandler(); // "Signup Button clicked 1 times"
```

### Partial Application and Currying
```javascript
function multiply(a) {
    return function(b) {
        return function(c) {
            return a * b * c;
        };
    };
}

const multiplyBy2 = multiply(2);
const multiplyBy2And3 = multiplyBy2(3);
const result = multiplyBy2And3(4);
console.log(result); // 24

// More practical example
function createValidator(validationType) {
    const validators = {
        email: (value) => /\S+@\S+\.\S+/.test(value),
        phone: (value) => /^\d{10}$/.test(value),
        required: (value) => value != null && value !== ''
    };
    
    return function(value) {
        return validators[validationType](value);
    };
}

const validateEmail = createValidator('email');
const validatePhone = createValidator('phone');

console.log(validateEmail('test@example.com')); // true
console.log(validatePhone('1234567890')); // true
```

### Memoization
```javascript
function createMemoizedFunction(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit!');
            return cache.get(key);
        }
        
        console.log('Computing...');
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

const expensiveOperation = createMemoizedFunction(function(n) {
    // Simulate expensive computation
    let result = 0;
    for (let i = 0; i < n * 1000000; i++) {
        result += i;
    }
    return result;
});

console.log(expensiveOperation(100)); // Computing... [result]
console.log(expensiveOperation(100)); // Cache hit! [result]
```

## Closure Patterns

### Factory Pattern
```javascript
function createUser(name, role) {
    let loginCount = 0;
    let lastLogin = null;
    
    return {
        getName() {
            return name;
        },
        
        getRole() {
            return role;
        },
        
        login() {
            loginCount++;
            lastLogin = new Date();
            console.log(`${name} logged in (${loginCount} times)`);
        },
        
        getStats() {
            return {
                loginCount,
                lastLogin,
                name,
                role
            };
        }
    };
}

const admin = createUser('Alice', 'admin');
const user = createUser('Bob', 'user');

admin.login(); // Alice logged in (1 times)
user.login();  // Bob logged in (1 times)
admin.login(); // Alice logged in (2 times)

console.log(admin.getStats()); // { loginCount: 2, lastLogin: ..., name: 'Alice', role: 'admin' }
```

### Configuration Pattern
```javascript
function createConfig(initialConfig = {}) {
    let config = { ...initialConfig };
    const listeners = [];
    
    return {
        get(key) {
            return config[key];
        },
        
        set(key, value) {
            const oldValue = config[key];
            config[key] = value;
            
            // Notify listeners
            listeners.forEach(listener => {
                listener(key, value, oldValue);
            });
        },
        
        onChange(callback) {
            listeners.push(callback);
        },
        
        getAll() {
            return { ...config }; // Return copy
        }
    };
}

const appConfig = createConfig({ theme: 'light', language: 'en' });

appConfig.onChange((key, newValue, oldValue) => {
    console.log(`Config changed: ${key} from ${oldValue} to ${newValue}`);
});

appConfig.set('theme', 'dark'); // Config changed: theme from light to dark
console.log(appConfig.get('theme')); // 'dark'
```

### State Machine Pattern
```javascript
function createStateMachine(initialState, transitions) {
    let currentState = initialState;
    const history = [initialState];
    
    return {
        getState() {
            return currentState;
        },
        
        transition(action) {
            const possibleTransitions = transitions[currentState];
            
            if (!possibleTransitions || !possibleTransitions[action]) {
                throw new Error(`Invalid transition: ${action} from ${currentState}`);
            }
            
            const newState = possibleTransitions[action];
            history.push(newState);
            currentState = newState;
            
            console.log(`Transitioned from ${history[history.length - 2]} to ${currentState}`);
            return currentState;
        },
        
        getHistory() {
            return [...history];
        },
        
        canTransition(action) {
            const possibleTransitions = transitions[currentState];
            return possibleTransitions && possibleTransitions[action] !== undefined;
        }
    };
}

const doorStateMachine = createStateMachine('closed', {
    closed: { open: 'open' },
    open: { close: 'closed', lock: 'locked' },
    locked: { unlock: 'closed' }
});

console.log(doorStateMachine.getState()); // 'closed'
doorStateMachine.transition('open'); // Transitioned from closed to open
doorStateMachine.transition('lock'); // Transitioned from open to locked
doorStateMachine.transition('unlock'); // Transitioned from locked to closed
```

## Memory and Performance

### Memory Considerations
```javascript
function createMemoryExample() {
    // Large data that might be kept alive unnecessarily
    const largeData = new Array(1000000).fill('data');
    const smallData = 'small';
    
    return function() {
        // Only smallData is used, but largeData might be kept alive
        console.log(smallData);
        // Some JS engines optimize this, others don't
    };
}

// Better approach: clean up references
function createOptimizedExample() {
    const largeData = new Array(1000000).fill('data');
    const smallData = 'small';
    
    // Process large data and extract only what's needed
    const processedData = processLargeData(largeData);
    
    return function() {
        console.log(smallData);
        console.log(processedData);
        // largeData can be garbage collected
    };
}

function processLargeData(data) {
    // Extract only necessary information
    return { length: data.length, summary: 'processed' };
}
```

### Closure vs Class Performance
```javascript
// Closure approach
function createClosureCounter() {
    let count = 0;
    
    return {
        increment() { return ++count; },
        decrement() { return --count; },
        getValue() { return count; }
    };
}

// Class approach  
class ClassCounter {
    constructor() {
        this.count = 0;
    }
    
    increment() { return ++this.count; }
    decrement() { return --this.count; }
    getValue() { return this.count; }
}

// Benchmark comparison
function benchmark() {
    const iterations = 1000000;
    
    // Closure creation
    console.time('Closure creation');
    for (let i = 0; i < iterations; i++) {
        createClosureCounter();
    }
    console.timeEnd('Closure creation');
    
    // Class creation
    console.time('Class creation');
    for (let i = 0; i < iterations; i++) {
        new ClassCounter();
    }
    console.timeEnd('Class creation');
}

// Generally, classes are faster for object creation
// Closures provide better encapsulation
```

## Common Pitfalls

### Loop Closure Problem
```javascript
// ❌ Common mistake
function createBadHandlers() {
    const handlers = [];
    
    for (var i = 0; i < 3; i++) {
        handlers.push(function() {
            console.log('Handler', i); // All handlers log 3!
        });
    }
    
    return handlers;
}

const badHandlers = createBadHandlers();
badHandlers.forEach(handler => handler()); // Handler 3, Handler 3, Handler 3

// ✅ Solutions

// Solution 1: Use let
function createGoodHandlers1() {
    const handlers = [];
    
    for (let i = 0; i < 3; i++) { // let creates new binding each iteration
        handlers.push(function() {
            console.log('Handler', i);
        });
    }
    
    return handlers;
}

// Solution 2: IIFE
function createGoodHandlers2() {
    const handlers = [];
    
    for (var i = 0; i < 3; i++) {
        handlers.push((function(index) {
            return function() {
                console.log('Handler', index);
            };
        })(i));
    }
    
    return handlers;
}

// Solution 3: bind
function createGoodHandlers3() {
    const handlers = [];
    
    for (var i = 0; i < 3; i++) {
        handlers.push(function(index) {
            console.log('Handler', index);
        }.bind(null, i));
    }
    
    return handlers;
}
```

### Memory Leaks
```javascript
// ❌ Potential memory leak
function createLeakyHandler() {
    const data = new Array(1000000).fill('large data');
    
    return function() {
        // Even though we don't use 'data', it might be kept alive
        console.log('Handler called');
    };
}

// ✅ Explicit cleanup
function createCleanHandler() {
    let data = new Array(1000000).fill('large data');
    
    // Process data if needed
    const summary = data.length;
    
    // Clear reference
    data = null;
    
    return function() {
        console.log('Handler called, data size was:', summary);
    };
}

// ❌ Circular reference issue
function createCircularReference() {
    const obj = {};
    
    obj.callback = function() {
        console.log(obj); // obj references callback, callback references obj
    };
    
    return obj.callback;
}

// ✅ Break circular reference
function createSafeReference() {
    const obj = {};
    
    const callback = function() {
        console.log('Callback called');
    };
    
    obj.callback = callback;
    
    // Return callback without circular reference
    return function() {
        callback();
    };
}
```

## Advanced Closure Concepts

### Closure Composition
```javascript
function compose(...functions) {
    return function(initialValue) {
        return functions.reduceRight((acc, fn) => fn(acc), initialValue);
    };
}

const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

const composedFunction = compose(square, double, addOne);
console.log(composedFunction(3)); // ((3 + 1) * 2)^2 = 64

// Each function in the composition creates a closure
```

### Lazy Evaluation
```javascript
function createLazyValue(computation) {
    let computed = false;
    let value;
    
    return function() {
        if (!computed) {
            console.log('Computing value...');
            value = computation();
            computed = true;
        }
        return value;
    };
}

const expensiveValue = createLazyValue(() => {
    // Simulate expensive computation
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
        result += Math.random();
    }
    return result;
});

console.log('Lazy value created');
console.log(expensiveValue()); // Computing value... [result]
console.log(expensiveValue()); // [result] (no computation)
```

### Closure-based Decorators
```javascript
function createTimer(fn) {
    return function(...args) {
        const start = performance.now();
        const result = fn.apply(this, args);
        const end = performance.now();
        console.log(`Function took ${end - start} milliseconds`);
        return result;
    };
}

function createMemoizer(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Compose decorators
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const optimizedFibonacci = createTimer(createMemoizer(fibonacci));
console.log(optimizedFibonacci(40)); // Fast due to memoization, with timing
```

## Best Practices

### When to Use Closures
```javascript
// ✅ Good use cases

// 1. Private variables and methods
function createBankAccount(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit(amount) {
            if (amount > 0) {
                balance += amount;
                return balance;
            }
        },
        
        withdraw(amount) {
            if (amount > 0 && amount <= balance) {
                balance -= amount;
                return balance;
            }
        },
        
        getBalance() {
            return balance;
        }
    };
}

// 2. Function factories
function createFormatter(prefix, suffix) {
    return function(text) {
        return `${prefix}${text}${suffix}`;
    };
}

const htmlTag = createFormatter('<b>', '</b>');
const parentheses = createFormatter('(', ')');

// 3. Event handling with state
function createClickTracker() {
    let clicks = 0;
    
    return function(event) {
        clicks++;
        console.log(`Click #${clicks} at coordinates available`);
    };
}
```

### Performance Optimization
```javascript
// ✅ Optimize closure performance

// 1. Minimize closure scope
function createOptimizedClosure(data) {
    // Extract only what's needed
    const needed = data.importantProperty;
    
    return function() {
        return needed; // Only 'needed' is kept alive
    };
}

// 2. Clean up references
function createCleanClosure() {
    let data = getExpensiveData();
    const processed = processData(data);
    data = null; // Clean up
    
    return function() {
        return processed;
    };
}

function getExpensiveData() {
    return new Array(1000).fill('data');
}

function processData(data) {
    return { length: data.length };
}
```

### Key Takeaways

1. **Closures = Function + External Variables** - Functions carry their lexical environment with them
2. **Every function creates a closure** - Even simple functions have access to outer scope
3. **Scope reference vs LexicalEnvironment** - Scope created at declaration, LexicalEnvironment at execution
4. **Variable lookup chain** - Current environment → Scope → Scope's scope → Global
5. **Independent closures** - Each function call creates separate closure scope
6. **Memory implications** - Closures can prevent garbage collection of outer variables
7. **Common pitfalls** - Loop variables, memory leaks, circular references
8. **Use for encapsulation** - Private variables and state management
9. **Performance considerations** - Closures have overhead but provide powerful patterns
10. **Clean up when possible** - Null references to large objects when done 