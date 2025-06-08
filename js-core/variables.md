# JavaScript Variables Guide (var, let, const)

## Table of Contents
- [Variable Declaration without Keywords](#variable-declaration-without-keywords)
- [Scope Chain and Lexical Environment](#scope-chain-and-lexical-environment)
- [Temporal Dead Zone](#temporal-dead-zone)
- [typeof Operator with TDZ](#typeof-operator-with-tdz)
- [Global Scope Behavior](#global-scope-behavior)
- [Hoisting Differences](#hoisting-differences)
- [Block Scope vs Function Scope](#block-scope-vs-function-scope)
- [const Immutability](#const-immutability)
- [Variable Declaration Best Practices](#variable-declaration-best-practices)
- [Common Pitfalls](#common-pitfalls)

## Variable Declaration without Keywords

### Implicit Global Variables
Based on your interesting behavior example:

```javascript
// интересное поведение
function updateN() {
    n = 2; // будет работать только если объявляем переменную без клучевого слова!
    // это два разных n но происходит это из за того что внутри функции мы пытаемся не создать переменную а найти ее
    // по ссылке которая называется scope мы переходим от lexicalEnv (функции) к верхнему lexicalEnv.
    // var let const = 2 => в консоли будет null;
}
let n = null;
updateN();
console.log(n); // 2
```

### Understanding the Scope Chain Behavior
```javascript
// When no keyword is used, JavaScript looks up the scope chain
function demonstrateScope() {
    // Without declaration keyword, JavaScript searches up the scope chain
    undeclaredVar = 'global assignment';
    
    // With keywords, creates local variable
    var localVar = 'local var';
    let localLet = 'local let';
    const localConst = 'local const';
}

// Before function call
console.log(typeof undeclaredVar); // 'undefined'

demonstrateScope();

// After function call
console.log(undeclaredVar); // 'global assignment' - created as global!
console.log(typeof localVar); // 'undefined' - function scoped
console.log(typeof localLet); // 'undefined' - block scoped
console.log(typeof localConst); // 'undefined' - block scoped
```

### Lexical Environment Lookup Process
```javascript
let outerVariable = 'outer';

function outerFunction() {
    let middleVariable = 'middle';
    
    function innerFunction() {
        // Assignment without declaration triggers scope chain lookup:
        // 1. Check innerFunction's lexical environment
        // 2. Check outerFunction's lexical environment  
        // 3. Check global lexical environment
        // 4. If not found, create global property (or error in strict mode)
        
        outerVariable = 'modified outer'; // Found in step 3, modifies existing
        middleVariable = 'modified middle'; // Found in step 2, modifies existing
        newGlobal = 'created global'; // Not found, creates global (step 4)
        
        // With declaration keywords, creates local binding
        let localBinding = 'local to inner';
    }
    
    innerFunction();
    console.log(middleVariable); // 'modified middle'
}

outerFunction();
console.log(outerVariable); // 'modified outer'
console.log(newGlobal); // 'created global'
```

### Strict Mode Impact
```javascript
'use strict';

function strictExample() {
    try {
        // In strict mode, assignment without declaration throws error
        undeclaredVariable = 'this will fail';
    } catch (error) {
        console.log(error.message); // "undeclaredVariable is not defined"
    }
    
    // Must use declaration keywords
    let properlyDeclared = 'this works';
    return properlyDeclared;
}

strictExample();

// Global variables still need to be explicitly created
globalThis.explicitGlobal = 'explicitly global';
console.log(explicitGlobal); // 'explicitly global'
```

## Scope Chain and Lexical Environment

### Lexical Environment Structure
```javascript
// Each execution context has a lexical environment with:
// 1. Environment Record (stores variables)
// 2. Reference to outer lexical environment

let globalVar = 'global';

function outerFunc(param1) {
    let outerVar = 'outer';
    
    function innerFunc(param2) {
        let innerVar = 'inner';
        
        // Access hierarchy:
        console.log(innerVar);  // Own environment
        console.log(param2);    // Own environment (parameter)
        console.log(outerVar);  // Outer function environment
        console.log(param1);    // Outer function environment
        console.log(globalVar); // Global environment
        
        // Assignment affects the environment where variable is found
        outerVar = 'modified from inner';
        globalVar = 'modified global';
    }
    
    return innerFunc;
}

const inner = outerFunc('outer param');
inner('inner param');
```

### Variable Resolution Algorithm
```javascript
function demonstrateResolution() {
    // Variable resolution steps:
    
    // 1. Check current lexical environment
    let localVar = 'local';
    
    function nested() {
        // 2. If not found, check outer environment
        console.log(localVar); // Found in outer (step 2)
        
        // 3. Continue up the chain until found or reach global
        console.log(globalVar); // Found in global environment
        
        // 4. If not found anywhere, ReferenceError (strict) or create global (non-strict)
        try {
            console.log(nonExistentVar);
        } catch (e) {
            console.log('Variable not found in scope chain');
        }
    }
    
    nested();
}

var globalVar = 'global';
demonstrateResolution();
```

## Temporal Dead Zone

### TDZ with let and const
Based on your TDZ example:

```javascript
// Временная мертвая зона
if (true) {
    console.log(typeof value); // ReferenceError! 
    // в слущае с var будет undefined
    let value = 1; 
}
```

### Understanding TDZ Behavior
```javascript
// TDZ exists from start of scope until declaration is reached

function demonstrateTDZ() {
    // TDZ starts here for letVar and constVar
    
    console.log(varVar); // undefined (hoisted, initialized with undefined)
    
    try {
        console.log(letVar); // ReferenceError: Cannot access before initialization
    } catch (e) {
        console.log('TDZ Error for let:', e.message);
    }
    
    try {
        console.log(constVar); // ReferenceError: Cannot access before initialization  
    } catch (e) {
        console.log('TDZ Error for const:', e.message);
    }
    
    var varVar = 'var value';
    let letVar = 'let value';     // TDZ ends here for letVar
    const constVar = 'const value'; // TDZ ends here for constVar
    
    console.log(varVar);   // 'var value'
    console.log(letVar);   // 'let value'
    console.log(constVar); // 'const value'
}

demonstrateTDZ();
```

### TDZ in Different Contexts
```javascript
// Block scope TDZ
{
    // TDZ for blockLet starts here
    
    const getValue = () => blockLet; // Function created in TDZ
    
    try {
        getValue(); // ReferenceError when called
    } catch (e) {
        console.log('TDZ in closure:', e.message);
    }
    
    let blockLet = 'value'; // TDZ ends here
    console.log(getValue()); // Now works: 'value'
}

// Loop TDZ
for (let i = 0; i < 3; i++) {
    // Each iteration creates new lexical environment
    // TDZ for i exists until loop initialization
    
    setTimeout(() => {
        console.log(i); // Each closure captures different 'i'
    }, 100);
}
// Prints: 0, 1, 2 (compare with var which would print 3, 3, 3)

// Class field TDZ
class Example {
    field1 = this.field2; // TDZ error if field2 is not yet declared
    field2 = 'value';
    
    constructor() {
        // By constructor time, all fields are initialized
        console.log(this.field1, this.field2);
    }
}
```

### Function TDZ Behavior
```javascript
// Function declarations are fully hoisted (no TDZ)
console.log(hoistedFunction()); // Works: 'I am hoisted'

function hoistedFunction() {
    return 'I am hoisted';
}

// Function expressions follow TDZ rules
try {
    console.log(funcExpression()); // ReferenceError
} catch (e) {
    console.log('Function expression TDZ:', e.message);
}

const funcExpression = () => 'I have TDZ';

// Class declarations have TDZ
try {
    new MyClass(); // ReferenceError
} catch (e) {
    console.log('Class TDZ:', e.message);
}

class MyClass {
    constructor() {
        console.log('Class created');
    }
}
```

## typeof Operator with TDZ

### Special typeof Behavior
Based on your example showing the interesting typeof behavior:

```javascript
// есть одно очень интересное / нестандартное поведение:
console.log(typeof value); // undefined. Такое работает только в случае с typeof!!! прото стоит принять это)
if (true) {
    let value = 1; 
}
```

### typeof Safety with Undeclared Variables
```javascript
// typeof is "safe" with completely undeclared variables
console.log(typeof completelyUndeclared); // 'undefined' (no error)

// But NOT safe with variables in TDZ
function typeofTDZDemo() {
    try {
        console.log(typeof inTDZ); // ReferenceError!
    } catch (e) {
        console.log('typeof TDZ error:', e.message);
    }
    
    let inTDZ = 'value';
}

typeofTDZDemo();

// Global scope typeof behavior
console.log(typeof globalUndeclared); // 'undefined' - safe

if (false) {
    // This block never executes, so variable is never declared
    let neverDeclared = 'never';
}
console.log(typeof neverDeclared); // 'undefined' - safe because not in scope

// Block scope typeof
{
    console.log(typeof outsideBlock); // 'undefined' - safe
}
let outsideBlock = 'value';
```

### typeof vs Direct Access
```javascript
// Feature detection pattern using typeof safety
function featureDetection() {
    // Safe way to check for global variables
    if (typeof window !== 'undefined') {
        console.log('Browser environment');
    }
    
    if (typeof process !== 'undefined') {
        console.log('Node.js environment');
    }
    
    if (typeof jQuery !== 'undefined') {
        console.log('jQuery is available');
    }
    
    // These would throw errors if variables don't exist:
    // if (window) { ... }     // ReferenceError if window doesn't exist
    // if (process) { ... }    // ReferenceError if process doesn't exist
}

featureDetection();

// TDZ checking utility
function isInTDZ(variableName, checkFunction) {
    try {
        checkFunction();
        return false;
    } catch (e) {
        return e instanceof ReferenceError && 
               e.message.includes('Cannot access');
    }
}

function tdzExample() {
    const inTDZ = isInTDZ('testVar', () => testVar);
    console.log('Variable in TDZ:', inTDZ); // true
    
    let testVar = 'initialized';
    
    const notInTDZ = isInTDZ('testVar', () => testVar);
    console.log('Variable in TDZ:', notInTDZ); // false
}
```

## Global Scope Behavior

### var vs let/const in Global Scope
Based on your browser-specific example:

```javascript
// let & const в глобальной области видимости
// only in browser
var a = 1; // переменная вызванная через var будет в глобальной области видимости window. let & const не записываются в window
if (window.a) {
    // var will be here. 
} else {
    // let & const will be here
}
```

### Global Object Property Creation
```javascript
// In browser environment
var globalVar = 'var value';
let globalLet = 'let value';
const globalConst = 'const value';

// var creates property on global object (window in browser)
console.log(window.globalVar); // 'var value' (in browser)
console.log(globalVar === window.globalVar); // true (in browser)

// let and const do NOT create properties on global object
console.log(window.globalLet); // undefined (in browser)
console.log(window.globalConst); // undefined (in browser)

// All are still accessible as variables
console.log(globalVar);   // 'var value'
console.log(globalLet);   // 'let value'
console.log(globalConst); // 'const value'
```

### Environment-Agnostic Global Access
```javascript
// Cross-environment global object access
const getGlobalObject = () => {
    if (typeof globalThis !== 'undefined') return globalThis; // ES2020
    if (typeof window !== 'undefined') return window;         // Browser
    if (typeof global !== 'undefined') return global;         // Node.js
    if (typeof self !== 'undefined') return self;             // Web Workers
    throw new Error('Unable to locate global object');
};

const globalObj = getGlobalObject();

// Safe global variable detection
function hasGlobalProperty(name) {
    return name in globalObj;
}

// Test different declaration types
var testVar = 'test';
let testLet = 'test';
const testConst = 'test';

console.log(hasGlobalProperty('testVar'));   // true
console.log(hasGlobalProperty('testLet'));   // false
console.log(hasGlobalProperty('testConst')); // false

// Explicit global assignment
globalObj.explicitGlobal = 'explicit';
console.log(explicitGlobal); // 'explicit'
```

### Global Scope Pollution
```javascript
// var can accidentally create globals
function varPollution() {
    for (var i = 0; i < 3; i++) {
        // var is function-scoped, so i is accessible outside loop
    }
    console.log('i after loop:', i); // 3
}

varPollution();

// let/const prevent accidental globals
function noPollution() {
    for (let j = 0; j < 3; j++) {
        // let is block-scoped
    }
    try {
        console.log('j after loop:', j); // ReferenceError
    } catch (e) {
        console.log('j is not accessible outside loop');
    }
}

noPollution();

// Global scope best practices
(function() {
    // Use IIFE to avoid global pollution
    var localToIIFE = 'hidden';
    
    // Only expose what's necessary
    globalObj.myLibrary = {
        publicMethod() {
            return 'public API';
        }
    };
})();

console.log(typeof localToIIFE); // 'undefined'
console.log(myLibrary.publicMethod()); // 'public API'
```

## Hoisting Differences

### var Hoisting
```javascript
function demonstrateVarHoisting() {
    console.log(hoistedVar); // undefined (not ReferenceError)
    
    var hoistedVar = 'initialized';
    
    console.log(hoistedVar); // 'initialized'
}

// Equivalent to:
function hoistingEquivalent() {
    var hoistedVar; // Declaration hoisted to top
    console.log(hoistedVar); // undefined
    
    hoistedVar = 'initialized'; // Assignment stays in place
    console.log(hoistedVar); // 'initialized'
}
```

### let/const Hoisting with TDZ
```javascript
function demonstrateLetConstHoisting() {
    // Variables are hoisted but in TDZ
    
    try {
        console.log(hoistedLet); // ReferenceError
    } catch (e) {
        console.log('let TDZ error');
    }
    
    try {
        console.log(hoistedConst); // ReferenceError
    } catch (e) {
        console.log('const TDZ error');
    }
    
    let hoistedLet = 'let value';
    const hoistedConst = 'const value';
    
    console.log(hoistedLet);   // 'let value'
    console.log(hoistedConst); // 'const value'
}

demonstrateLetConstHoisting();
```

### Function Hoisting Interaction
```javascript
// Function declarations are fully hoisted
console.log(hoistedFunction()); // 'I work!'

function hoistedFunction() {
    return 'I work!';
}

// var function expressions
console.log(varFunc); // undefined
console.log(typeof varFunc); // 'undefined'

var varFunc = function() {
    return 'var function';
};

// let/const function expressions
try {
    console.log(letFunc); // ReferenceError
} catch (e) {
    console.log('let function in TDZ');
}

let letFunc = function() {
    return 'let function';
};

// Arrow functions follow same rules
const arrowFunc = () => 'arrow function';
```

## Block Scope vs Function Scope

### Scope Comparison
```javascript
function scopeComparison() {
    // var is function-scoped
    if (true) {
        var functionScoped = 'accessible outside block';
    }
    console.log(functionScoped); // 'accessible outside block'
    
    // let/const are block-scoped
    if (true) {
        let blockScoped = 'only in block';
        const alsoBlockScoped = 'also only in block';
        
        console.log(blockScoped);     // 'only in block'
        console.log(alsoBlockScoped); // 'also only in block'
    }
    
    try {
        console.log(blockScoped); // ReferenceError
    } catch (e) {
        console.log('blockScoped not accessible');
    }
    
    try {
        console.log(alsoBlockScoped); // ReferenceError
    } catch (e) {
        console.log('alsoBlockScoped not accessible');
    }
}

scopeComparison();
```

### Loop Scope Behavior
```javascript
// Classic var loop problem
console.log('var loop:');
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log('var i:', i); // 3, 3, 3
    }, 100);
}

// let creates new binding for each iteration
console.log('let loop:');
for (let j = 0; j < 3; j++) {
    setTimeout(() => {
        console.log('let j:', j); // 0, 1, 2
    }, 200);
}

// const in loops (with objects)
const objects = [{id: 0}, {id: 1}, {id: 2}];
for (const obj of objects) {
    setTimeout(() => {
        console.log('const obj:', obj.id); // 0, 1, 2
    }, 300);
}

// const in for loop (error)
try {
    for (const k = 0; k < 3; k++) { // TypeError: Assignment to constant variable
        console.log(k);
    }
} catch (e) {
    console.log('const increment error:', e.message);
}
```

### Switch Statement Scope
```javascript
function switchScope(value) {
    switch (value) {
        case 1:
            let x = 'case 1'; // Block scoped to entire switch
            break;
        case 2:
            // let x = 'case 2'; // SyntaxError: Identifier 'x' has already been declared
            x = 'modified in case 2'; // Can modify existing x
            break;
    }
    
    // x is not accessible here
    try {
        console.log(x);
    } catch (e) {
        console.log('x not accessible outside switch');
    }
}

// Better switch scope management
function betterSwitchScope(value) {
    switch (value) {
        case 1: {
            let x = 'case 1';
            console.log(x);
            break;
        }
        case 2: {
            let x = 'case 2'; // Now allowed due to block scope
            console.log(x);
            break;
        }
    }
}

betterSwitchScope(1); // 'case 1'
betterSwitchScope(2); // 'case 2'
```

## const Immutability

### const Binding Immutability
```javascript
// const prevents reassignment of binding
const immutableBinding = 'cannot reassign';

try {
    immutableBinding = 'new value'; // TypeError
} catch (e) {
    console.log('const reassignment error:', e.message);
}

// But object contents can be modified
const mutableObject = { value: 'original' };
mutableObject.value = 'modified'; // Allowed
mutableObject.newProperty = 'added'; // Allowed
console.log(mutableObject); // { value: 'modified', newProperty: 'added' }

// Array modification is allowed
const mutableArray = [1, 2, 3];
mutableArray.push(4); // Allowed
mutableArray[0] = 'changed'; // Allowed
console.log(mutableArray); // ['changed', 2, 3, 4]
```

### Creating Truly Immutable Objects
```javascript
// Object.freeze for shallow immutability
const shallowFrozen = Object.freeze({
    value: 'frozen',
    nested: { canChange: true }
});

try {
    shallowFrozen.value = 'cannot change'; // Silently fails (error in strict mode)
} catch (e) {
    console.log('Frozen object modification error');
}

shallowFrozen.nested.canChange = false; // Allowed (nested not frozen)
console.log(shallowFrozen.nested.canChange); // false

// Deep freeze implementation
function deepFreeze(obj) {
    Object.getOwnPropertyNames(obj).forEach(prop => {
        const value = obj[prop];
        if (value && typeof value === 'object') {
            deepFreeze(value);
        }
    });
    return Object.freeze(obj);
}

const deepFrozen = deepFreeze({
    value: 'frozen',
    nested: { cannotChange: true }
});

try {
    deepFrozen.nested.cannotChange = false; // Error in strict mode
} catch (e) {
    console.log('Deep frozen modification error');
}

// const with destructuring
const [a, b, c] = [1, 2, 3];
// a, b, c are all const bindings

const { name, age } = { name: 'John', age: 30 };
// name, age are const bindings
```

## Variable Declaration Best Practices

### Modern Declaration Strategy
```javascript
// ✅ Preferred approach: const by default
const PI = 3.14159;
const users = ['Alice', 'Bob'];
const config = { api: 'https://api.example.com' };

// ✅ Use let when reassignment is needed
let counter = 0;
let currentUser = null;

for (let i = 0; i < users.length; i++) {
    if (users[i] === 'Alice') {
        currentUser = users[i];
        break;
    }
}

// ❌ Avoid var in modern code
// var should only be used when you specifically need function scoping
```

### Declaration Patterns
```javascript
// ✅ Declare variables close to usage
function processData(input) {
    // Declare when needed
    const processed = input.map(item => item.toUpperCase());
    
    if (processed.length > 0) {
        // Declare in relevant scope
        const first = processed[0];
        console.log('First item:', first);
    }
    
    return processed;
}

// ✅ Use meaningful names
const userAccountBalance = 1000;
const isAccountActive = true;
const lastLoginTimestamp = Date.now();

// ❌ Avoid unclear names
// const x = 1000;
// const flag = true;
// const t = Date.now();

// ✅ Group related declarations
const API_BASE_URL = 'https://api.example.com';
const API_TIMEOUT = 5000;
const API_RETRIES = 3;

// ✅ Use const for functions that don't need hoisting
const calculateTotal = (items) => {
    return items.reduce((sum, item) => sum + item.price, 0);
};

// ✅ Initialize variables when declaring
let sum = 0; // Clear initial value
const items = []; // Clear initial state
```

### Avoiding Common Mistakes
```javascript
// ❌ Common mistake: using var in loops
function badLoop() {
    for (var i = 0; i < 3; i++) {
        setTimeout(() => {
            console.log('bad:', i); // 3, 3, 3
        }, 100);
    }
}

// ✅ Correct: using let in loops
function goodLoop() {
    for (let i = 0; i < 3; i++) {
        setTimeout(() => {
            console.log('good:', i); // 0, 1, 2
        }, 100);
    }
}

// ❌ Common mistake: forgetting const immutability
function badConstUsage() {
    const settings = { theme: 'dark' };
    // Later in code...
    settings = { theme: 'light' }; // TypeError!
}

// ✅ Correct: modifying object properties
function goodConstUsage() {
    const settings = { theme: 'dark' };
    // Later in code...
    settings.theme = 'light'; // OK
    // Or create new object if immutability needed
    const newSettings = { ...settings, theme: 'light' };
}

// ❌ Common mistake: TDZ violations
function badTDZ() {
    console.log(value); // ReferenceError
    let value = 'initialized';
}

// ✅ Correct: declare before use
function goodTDZ() {
    let value = 'initialized';
    console.log(value); // 'initialized'
}
```

## Common Pitfalls

### Variable Shadowing
```javascript
const globalVar = 'global';

function demonstrateShadowing() {
    const globalVar = 'local'; // Shadows global
    console.log(globalVar); // 'local'
    
    {
        const globalVar = 'block'; // Shadows function scope
        console.log(globalVar); // 'block'
    }
    
    console.log(globalVar); // 'local'
}

demonstrateShadowing();
console.log(globalVar); // 'global'

// Accidental shadowing can hide bugs
function riskyFunction(items) {
    let result = [];
    
    for (let i = 0; i < items.length; i++) {
        if (items[i].valid) {
            // Accidentally shadow outer 'result'
            let result = processItem(items[i]);
            console.log(result); // Local result
        }
    }
    
    return result; // Returns empty array!
}
```

### Memory Leaks with Closures
```javascript
// ❌ Memory leak with var
function createLeakyHandlers() {
    const handlers = [];
    
    for (var i = 0; i < 1000; i++) {
        handlers.push(() => {
            console.log(i); // All closures reference same 'i'
        });
    }
    
    return handlers; // All handlers keep the same scope alive
}

// ✅ No leak with let
function createCleanHandlers() {
    const handlers = [];
    
    for (let i = 0; i < 1000; i++) {
        handlers.push(() => {
            console.log(i); // Each closure has its own 'i'
        });
    }
    
    return handlers; // Each handler has minimal closure
}
```

### Assignment vs Declaration
```javascript
function confusingAssignment() {
    // This looks like declaration but is assignment
    if (true) {
        var declared = 'declared';
    }
    
    // This is definitely assignment to existing variable
    declared = 'reassigned';
    
    // This creates global (in non-strict mode)
    undeclared = 'oops, global';
    
    console.log(declared); // 'reassigned'
}

confusingAssignment();
console.log(typeof undeclared); // 'string' - global created!

// ✅ Clear intent with proper declarations
function clearDeclarations() {
    let localVar = 'initial';
    
    if (true) {
        localVar = 'modified'; // Clear reassignment
    }
    
    const newVar = 'const declaration'; // Clear new binding
    
    console.log(localVar, newVar);
}
```

### Key Takeaways

1. **Assignment without keywords** - Creates globals or modifies existing variables in scope chain
2. **Temporal Dead Zone** - let/const variables cannot be accessed before declaration
3. **typeof safety** - Works with undeclared variables but not TDZ variables
4. **Global object properties** - var creates global object properties, let/const don't
5. **Block vs function scope** - let/const are block-scoped, var is function-scoped
6. **Hoisting differences** - var is initialized with undefined, let/const are in TDZ
7. **const immutability** - Prevents reassignment, not object mutation
8. **Modern best practice** - Use const by default, let when reassignment needed, avoid var
9. **Scope chain resolution** - JavaScript searches up lexical environments for variables
10. **TDZ serves a purpose** - Prevents accessing variables before proper initialization 