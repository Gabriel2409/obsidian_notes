#js

Behavior where variables, functions, or classes are moved to the top of their scope before the execution of the code. 


 ### Variable Hoisting

var:  hoisted, but their value is not initialized. They are set to undefined until the code assigns them a value.

```js
console.log(x); // Output: undefined
var x = 10;
console.log(x); // Output: 10
```

let and const: also hoisted, but they are in a "temporal dead zone" from the start of the block until the declaration is encountered. 

```js
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 20;
```
### Function Hoisting

Function Declarations: Entire functions are hoisted, meaning you can call the function before it is declared.

```js
greet(); // Output: Hello!
function greet() {
    console.log("Hello!");
}
```

Function Expressions (using var, let, or const): Only the variable declaration is hoisted, but the function itself is not initialized. 

```js
console.log(sayHi); // Output: undefined (if using var)
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() {
    console.log("Hi!");
};
```
Note : arrow functions behave the same way

### Class Hoisting

Classes in JavaScript are not hoisted in the same way as functions. They exist in the temporal dead zone like let and const.

```js
const obj = new MyClass(); // ReferenceError: Cannot access 'MyClass' before initialization
class MyClass {
    constructor() {
        this.name = "Class";
    }
}
```
