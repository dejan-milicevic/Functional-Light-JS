# Functional-Light-JS
Functional Light JS Concepts

## Functions of Functions
Functions can receive and return values of any type. A function that receives or returns one or more other function values has the special name: higher-order function.

Closure is when a function remembers and accesses variables from outside of its own scope, even when that function is executed in a different scope.

Example I:
```
function makeAdder(x) {
  return function sum(y) {
    return x + y;
  }
}

var addTo10 = makeAdder(10);
var addTo37 = makeAdder(37);

addTo10(3);      // 13
addTo37(13);     // 50
```

Example II: 
```
function person(name) {
  return function identify() {
    console.log(`I am ${name}`);
  };
}

var fred = person("Fred");
var susan = person("Susan");

fred();       // I am Fred
susan();      // I am Susan
```

