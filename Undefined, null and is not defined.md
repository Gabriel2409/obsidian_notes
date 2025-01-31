#js

## undefined vs null
```js
var a;
console.log(a) // undefined
```
- `undefined` = value automatically assigned by JS when we create a variable and we have not yet given it a value
- `null` represents an explicit empty value
Explicitly assigning `undefined` can cause confusion 
```js
const obj = {}; 
obj.key = undefined; 
console.log('key' in obj); // true
console.log(obj.key === undefined); // true 
delete obj.key; // Now 'key' in obj is false
```

Json interaction: 
```js
const obj = { key: undefined, anotherKey: null }; console.log(JSON.stringify(obj)); // '{"anotherKey":null}'
```


Note: 

```js
if(a == null){console.log("HERE")} // works for undefined and null but not 0
```


Best practices:

- Use `undefined` for **uninitialized variables or missing properties** (the default behavior of JavaScript).
- Use `null` when you want to **explicitly signal "no value" or "empty."**
- Avoid assigning `undefined` explicitly. Instead:
    - Remove a property using `delete` if necessary.
    - Use `null` to explicitly clear a value if needed.

## Is not defined

```js
console.log(b) // Uncaught ReferenceError: b is not defined

var a ={}
console.log(a.b) // undefined
console.log(a.b.c) // Uncaught TypeError: a.b is undefined
```
b is never declared so it throws a reference error
a.b is undefined so when we try to access a property we get a type error because undefined is not an object