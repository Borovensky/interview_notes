# JavaScript Error Handling Guide

## Table of Contents
- [Types of Errors](#types-of-errors)
- [Error Objects](#error-objects)
- [Try-Catch-Finally](#try-catch-finally)
- [Asynchronous Error Handling](#asynchronous-error-handling)
- [Custom Errors](#custom-errors)
- [Error Handling Patterns](#error-handling-patterns)
- [Best Practices](#best-practices)
- [Modern Error Handling](#modern-error-handling)

## Types of Errors

JavaScript errors can be categorized into three main types:

### 1. Parsing Errors
These occur before code execution when the JavaScript engine cannot parse the code due to syntax violations.

```javascript
// Examples that would cause parsing errors:
// function test() {
//     console.log('hello';  // Missing closing parenthesis
// }

// const 123var = 'test';  // Invalid variable name
// const function = 'test'; // Reserved word usage
```

### 2. Runtime Errors
These occur during code execution when the syntax is correct but something goes wrong while running.

```javascript
// ReferenceError - accessing undefined variables
try {
    console.log(nonExistentVariable);
} catch (e) {
    console.log('Runtime error caught:', e.name);
}

// TypeError - calling methods on wrong types
try {
    const str = 'hello';
    str.push('world'); // TypeError: str.push is not a function
} catch (e) {
    console.log('Runtime error caught:', e.name);
}
```

### 3. Logical Errors
These don't throw exceptions but produce incorrect results.

```javascript
// Incorrect condition - logical error
function getDiscount(age) {
    if (age > 65) {  // Should be >= 65
        return 0.1;
    }
    return 0;
}
console.log(getDiscount(65)); // 0 (incorrect)
```

## Error Objects

### SyntaxError
Thrown when code violates JavaScript syntax rules.

```javascript
try {
    eval('const var = "test"'); // 'var' is a reserved word
} catch (e) {
    console.log(`${e.name}: ${e.message}`);
}
```

### ReferenceError
Occurs when trying to access a variable or function that doesn't exist.

```javascript
try {
    console.log(undeclaredVariable);
} catch (e) {
    console.log(`${e.name}: ${e.message}`);
}
```

### TypeError
Thrown when a value is not of the expected type.

```javascript
try {
    const obj = null;
    console.log(obj.property);
} catch (e) {
    console.log(`${e.name}: ${e.message}`);
}
```

### RangeError
Occurs when a value is outside the allowed range.

```javascript
try {
    const arr = new Array(-5);
} catch (e) {
    console.log(`${e.name}: ${e.message}`);
}
```

### URIError
Thrown when URI encoding/decoding functions are used incorrectly.

```javascript
try {
    decodeURIComponent('%E0%A4%A');
} catch (e) {
    console.log(`${e.name}: ${e.message}`);
}
```

### EvalError
Rarely used in modern JavaScript.

### InternalError
Browser-specific error for internal engine issues (non-standard).

## Try-Catch-Finally

### Finally Always Executes
Based on your original example:

```javascript
function f(a, b) {
    try {
        console.log('1'); // ✅ Executes
        return a + b;     // ✅ Executes and sets return value
        console.log('2'); // ❌ Never executes (after return)
    } finally {
        console.log('finally'); // ✅ ALWAYS executes!
    }
}

const result = f(1, 2);
console.log(result);

// Output order: 1 → finally → 3
```

### Finally with Exceptions
```javascript
try {
    console.log(1);           // ✅ Executes
    throw new Error('Test');  // ✅ Throws error
} finally {
    console.log('finally');   // ✅ Executes before error propagates
}

// Output order: 1 → finally → Error!
```

## Asynchronous Error Handling

### The Problem
As noted in your original file, traditional try-catch doesn't work with asynchronous code:

```javascript
// ❌ This doesn't work - error won't be caught!
try {
    setTimeout(function() {
        throw new Error('Async error');
    }, 0);
} catch (e) {
    console.log(e.name); // This never executes
}
console.log(1); // This executes first, then error occurs

// The error occurs after try-catch has finished executing
```

### Solutions

#### 1. Promises with .catch()
```javascript
function asyncOperation() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (Math.random() > 0.5) {
                reject(new Error('Random failure'));
            } else {
                resolve('Success!');
            }
        }, 100);
    });
}

asyncOperation()
    .then(result => console.log('Result:', result))
    .catch(error => console.log('Error:', error.message));
```

#### 2. Async/Await with Try-Catch
```javascript
async function handleAsync() {
    try {
        const result = await asyncOperation();
        console.log('Result:', result);
    } catch (error) {
        console.log('Caught async error:', error.message);
    }
}
```

## Custom Errors

### Creating Custom Error Classes
```javascript
class ValidationError extends Error {
    constructor(message, field) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
    }
}

function validateUser(user) {
    if (!user.name) {
        throw new ValidationError('Name is required', 'name');
    }
    if (!user.email) {
        throw new ValidationError('Email is required', 'email');
    }
}

try {
    validateUser({ name: 'John' }); // Missing email
} catch (error) {
    if (error instanceof ValidationError) {
        console.log(`Validation failed for ${error.field}: ${error.message}`);
    }
}
```

## Error Handling Patterns

### Retry Pattern
```javascript
async function withRetry(fn, maxRetries = 3, delay = 1000) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            lastError = error;
            
            if (attempt === maxRetries) {
                throw lastError;
            }
            
            await new Promise(resolve => setTimeout(resolve, delay));
            delay *= 2; // Exponential backoff
        }
    }
}
```

### Global Error Handling
```javascript
// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
    console.log('Unhandled promise rejection:', event.reason);
    event.preventDefault();
});

// Global error handler
window.addEventListener('error', (event) => {
    console.log('Global error:', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno
    });
});
```

## Best Practices

### 1. Specific Error Handling
```javascript
// ✅ Handle different error types specifically
try {
    await riskyOperation();
} catch (error) {
    if (error instanceof NetworkError) {
        // Handle network issues
    } else if (error instanceof ValidationError) {
        // Handle validation issues
    } else {
        // Handle unexpected errors
    }
}
```

### 2. Input Validation
```javascript
function safeJSONParse(jsonString, defaultValue = null) {
    try {
        return JSON.parse(jsonString);
    } catch (error) {
        console.warn('Invalid JSON:', error.message);
        return defaultValue;
    }
}
```

### 3. Graceful Degradation
```javascript
async function getUserProfile(userId) {
    try {
        return await fetchFullProfile(userId);
    } catch (error) {
        try {
            return await fetchBasicProfile(userId);
        } catch (fallbackError) {
            return { id: userId, name: 'Unknown User' };
        }
    }
}
```

### Key Takeaways

1. **Understand error types**: Parsing, Runtime, and Logical errors
2. **Use appropriate error objects**: Choose the right error type
3. **Handle async errors properly**: Use promises or async/await
4. **Create meaningful custom errors**: Include relevant context
5. **Implement graceful degradation**: Provide fallbacks when possible
6. **Validate inputs early**: Prevent errors before they occur
7. **Use modern JavaScript features**: Optional chaining for safer code

## Modern Error Handling

### Using Optional Chaining and Nullish Coalescing
```javascript
// ✅ Safe property access
function getNestedProperty(obj) {
    try {
        // Old way - verbose and error-prone
        return obj && obj.user && obj.user.profile && obj.user.profile.name;
    } catch (error) {
        return null;
    }
}

// ✅ Modern way - cleaner and safer
function getNestedPropertyModern(obj) {
    return obj?.user?.profile?.name ?? 'Unknown';
}

// ✅ Safe method calling
function callOptionalMethod(obj) {
    obj?.someMethod?.(); // Only calls if obj exists and someMethod is a function
}
```

### Error Handling in Different Contexts
```javascript
// React component error handling
const UserProfile = ({ userId }) => {
    const [user, setUser] = useState(null);
    const [error, setError] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        async function loadUser() {
            try {
                setLoading(true);
                setError(null);
                const userData = await fetchUser(userId);
                setUser(userData);
            } catch (err) {
                setError(err.message);
            } finally {
                setLoading(false);
            }
        }
        
        loadUser();
    }, [userId]);
    
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    if (!user) return <div>User not found</div>;
    
    return <div>Welcome, {user.name}!</div>;
};
```

### Key Takeaways

1. **Understand error types**: Parsing, Runtime, and Logical errors
2. **Use appropriate error objects**: Choose the right error type
3. **Handle async errors properly**: Use promises or async/await
4. **Create meaningful custom errors**: Include relevant context
5. **Implement graceful degradation**: Provide fallbacks when possible
6. **Validate inputs early**: Prevent errors before they occur
7. **Use modern JavaScript features**: Optional chaining for safer code 