# Advanced JavaScript Arrays Guide

## Table of Contents
- [Advanced Array Creation](#advanced-array-creation)
- [Iterators and Array Creation](#iterators-and-array-creation)
- [Memory Management](#memory-management)
  - [ArrayBuffer](#arraybuffer)
  - [DataView](#dataview)
- [Typed Arrays](#typed-arrays)
- [Performance Considerations](#performance-considerations)
- [Use Cases](#use-cases)

## Advanced Array Creation

### Creating Arrays from Function Arguments

```javascript
// Method 1: Using for loop
const createArrayFromArgs = function() {
    const array = [];
    for(let i = 0; i < arguments.length; i++) {
        array.push(arguments[i]);
    }
    return array;
}

// Method 2: Using Array.prototype.slice.call()
const createArrayFromArgs2 = function() {
    return Array.prototype.slice.call(arguments);
}

// Method 3: Using Array.from()
const createArrayFromArgs3 = function() {
    return Array.from(arguments);
}

// Method 4: Using Array.from() with transformation
const createArrayFromArgs4 = function() {
    return Array.from(arguments, n => n * 2);
}

// Modern ES6 approach with rest parameters
const createArrayFromArgs5 = (...args) => args;

// Usage examples
createArrayFromArgs(1, 2, 3);  // [1, 2, 3]
createArrayFromArgs4(1, 2, 3); // [2, 4, 6]
```

## Iterators and Array Creation

### Creating Arrays from Custom Iterators

```javascript
// Object with custom iterator
const numbers = {
    *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
    }
}

// Create array from iterator
const array = Array.from(numbers);           // [1, 2, 3]
const doubled = Array.from(numbers, n => n * 2); // [2, 4, 6]

// Generator function example
function* numberGenerator(max) {
    let i = 0;
    while (i < max) {
        yield i++;
    }
}

const generatedArray = Array.from(numberGenerator(5)); // [0, 1, 2, 3, 4]
```

### Iterator Protocol
```javascript
// Manual iterator implementation
const customIterable = {
    data: [1, 2, 3],
    [Symbol.iterator]() {
        let index = 0;
        const data = this.data;
        
        return {
            next() {
                if (index < data.length) {
                    return { value: data[index++], done: false };
                } else {
                    return { done: true };
                }
            }
        };
    }
};

// Convert to array
const fromCustomIterable = [...customIterable]; // [1, 2, 3]
```

## Memory Management

### ArrayBuffer

ArrayBuffer represents a raw binary data buffer of fixed length:

```javascript
// Allocate 16 bytes of memory
const buffer = new ArrayBuffer(16);
console.log(buffer.byteLength); // 16

// ArrayBuffer properties and methods
console.log(buffer.constructor);     // ArrayBuffer constructor
console.log(ArrayBuffer.isView(buffer)); // false

// Slicing ArrayBuffer
const sliced = buffer.slice(8, 12); // Creates new ArrayBuffer from bytes 8-11
console.log(sliced.byteLength);     // 4
```

### DataView

DataView provides a low-level interface for reading and writing multiple number types in an ArrayBuffer:

```javascript
const buffer = new ArrayBuffer(16);
const view = new DataView(buffer);

// Writing different data types
view.setInt8(0, 127);        // Set byte at position 0
view.setInt16(1, 1000);      // Set 2 bytes starting at position 1
view.setInt32(4, 1000000);   // Set 4 bytes starting at position 4
view.setFloat32(8, 3.14);    // Set 4 bytes float at position 8
view.setFloat64(12, 2.718);  // Set 8 bytes double at position 12

// Reading data
const int8Value = view.getInt8(0);      // 127
const int16Value = view.getInt16(1);    // 1000
const int32Value = view.getInt32(4);    // 1000000
const float32Value = view.getFloat32(8); // 3.14
const float64Value = view.getFloat64(12); // 2.718

// Endianness control (little-endian vs big-endian)
view.setInt16(0, 256, true);  // little-endian
view.setInt16(0, 256, false); // big-endian (default)
```

## Typed Arrays

Typed arrays provide an array-like view of an ArrayBuffer with specific numeric types:

### Available Typed Array Types

| Type | Size (bytes) | Description | Range |
|------|--------------|-------------|-------|
| Int8Array | 1 | 8-bit signed integer | -128 to 127 |
| Uint8Array | 1 | 8-bit unsigned integer | 0 to 255 |
| Uint8ClampedArray | 1 | 8-bit unsigned integer (clamped) | 0 to 255 |
| Int16Array | 2 | 16-bit signed integer | -32,768 to 32,767 |
| Uint16Array | 2 | 16-bit unsigned integer | 0 to 65,535 |
| Int32Array | 4 | 32-bit signed integer | -2,147,483,648 to 2,147,483,647 |
| Uint32Array | 4 | 32-bit unsigned integer | 0 to 4,294,967,295 |
| Float32Array | 4 | 32-bit floating point | ±1.2×10⁻³⁸ to ±3.4×10³⁸ |
| Float64Array | 8 | 64-bit floating point | ±5.0×10⁻³²⁴ to ±1.8×10³⁰⁸ |
| BigInt64Array | 8 | 64-bit signed integer | -2⁶³ to 2⁶³-1 |
| BigUint64Array | 8 | 64-bit unsigned integer | 0 to 2⁶⁴-1 |

### Creating and Using Typed Arrays

```javascript
// From regular array
const source = [1, 2, 3, 256, -1];
const int8Array = new Int8Array(source);

console.log('Int8Array properties:');
console.log(`Length: ${int8Array.length}`);        // 5
console.log(`Bytes: ${int8Array.byteLength}`);     // 5
console.log(`Buffer: ${int8Array.buffer}`);        // ArrayBuffer
console.log(`Values: [${int8Array}]`);             // [1, 2, 3, 0, -1] (256 overflows)

// From ArrayBuffer
const buffer = new ArrayBuffer(8);
const float32View = new Float32Array(buffer);
float32View[0] = 3.14;
float32View[1] = 2.718;

// From length
const uint16Array = new Uint16Array(5); // Creates array with 5 elements, all 0

// From another typed array
const copy = new Uint16Array(int8Array);

// Subarray (creates a view, not a copy)
const subarray = int8Array.subarray(1, 3); // Elements at index 1 and 2
```

### Typed Array Methods

```javascript
const typedArray = new Float32Array([1.1, 2.2, 3.3, 4.4, 5.5]);

// Most regular array methods work
typedArray.forEach(val => console.log(val));
const doubled = typedArray.map(val => val * 2);
const filtered = typedArray.filter(val => val > 3);

// Typed array specific methods
typedArray.set([10, 20], 1); // Set values starting at index 1
const copy = typedArray.slice(0, 3);
```

## Performance Considerations

### Memory Efficiency
```javascript
// Regular array - each number uses ~8 bytes (64-bit float)
const regularArray = [1, 2, 3, 4, 5]; // ~40 bytes + overhead

// Typed array - each number uses exactly specified bytes
const int8Array = new Int8Array([1, 2, 3, 4, 5]); // 5 bytes exactly
const float32Array = new Float32Array([1, 2, 3, 4, 5]); // 20 bytes exactly
```

### Performance Benefits
1. **Memory efficiency**: Exact memory allocation
2. **Speed**: Faster arithmetic operations
3. **Predictable**: No type coercion
4. **Interoperability**: Works with WebGL, Web Audio API, etc.

## Use Cases

### Web Graphics (WebGL)
```javascript
// Vertex data for WebGL
const vertices = new Float32Array([
    -1.0, -1.0,
     1.0, -1.0,
     0.0,  1.0
]);
```

### Audio Processing
```javascript
// Audio buffer for Web Audio API
const audioContext = new AudioContext();
const sampleRate = audioContext.sampleRate;
const buffer = audioContext.createBuffer(1, sampleRate, sampleRate);
const channelData = buffer.getChannelData(0); // Returns Float32Array
```

### File I/O and Binary Data
```javascript
// Reading binary file data
fetch('data.bin')
    .then(response => response.arrayBuffer())
    .then(buffer => {
        const view = new DataView(buffer);
        // Process binary data
    });
```

### Network Communication
```javascript
// WebSocket binary data
const socket = new WebSocket('ws://example.com');
socket.binaryType = 'arraybuffer';

socket.onmessage = (event) => {
    const buffer = event.data;
    const view = new DataView(buffer);
    // Process binary message
};
```

### Image Processing
```javascript
// Canvas image data
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const pixels = imageData.data; // Uint8ClampedArray with RGBA values
``` 