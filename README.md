# Why Functional Programming?

# The Nature of Functions

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

# Managing Function Inputs

# Composing Functions

## Output to Input

Example:
```
function words(str) {
  return String(str)
    .toLowerCase()
    .split( /\s|\b/ )
    .filter(function alpha(v) {
      return /^[\w]+$/.test(v);
    );
}

function unique(list) {
  var uniqList = [];
  for (let v of list) {
    if (uniqList.indexOf(v) === -1) {
      uniqList.push(v);
    }
  }
  return uniqList;
}

var text = "To compose two functions together, pass the \ output of the first function call as the input of the \ second function call.";
var wordsUsed = unique(words(text));
// ["to","compose","two","functions","together","pass","the","output","of","first","function","call","as","input","second"]
```

## General Composition

Example:
```
function compose(...fns) {
  return function composed(result) {
    var list = [...fns];
    while (list.length > 0) {
      result = list.pop()(result);
    }
    return result;
  };
}

function skipShortWords(words) {
  var filteredWords = [];
  for (let word of words) {
    if (word.length > 4) {
      filteredWords.push(word);
    }
  }
  return filteredWords;
}

var biggerWords = compose(skipShortWords, unique, words);
var wordsUsed = biggerWords( text );      // ["compose","functions","together","output","first", "function","input","second"]
```

# Reducing Side Effects

# Value Immutability

# Closure vs. Object
Objects and closures are isomorphic to each other, which means that they can be used somewhat interchangeably to represent state and behavior in your program.

## State
It may not be obvious how closures and objects are related. So let’s explore their similarities first.
To frame this discussion, let me just briefly assert two things:
1. A programming language without closures can simulate them with objects instead.
2. A programming language without objects can simulate them with closures instead.
In other words, we can think of closures and objects as two different representations of a thing.
```
function outer() {
  var one = 1;
  var two = 2;
  return function inner() {
    return one + two;
  };
}
var three = outer();
three();        // 3
```
vs.
```
var obj = {
  one: 1,
  two: 2
};
function three(outer) {
  return outer.one + outer.two;
}
three(obj);     // 3
```

## Behavior
It’s not just that objects and closures represent ways to express collections of state, but also that they can include behavior via functions/methods. Bundling data with its behavior has a fancy name: encapsulation.
Another way to analyze this relationship: a closure associates a single function with a set of state, whereas an object holding the same state can have any number of functions to operate on that state.
```
var person = {
  firstName: "Kyle",
  lastName: "Simpson",
  first() {
    return this.firstName;
  },
  last() {
    return this.lastName;
  }
}
person.first() + " " + person.last();      // Kyle Simpson
```
vs.
```
function createPerson(firstName,lastName) {
  return API;
  function API(methodName) {
    switch (methodName) {
      case "first":
        return first();
        break;
      case "last":
        return last();
        break;
    };
  }
  function first() {
    return firstName;
  }
  function last() {
    return lastName;
  }
}
var person = createPerson("Kyle", "Simpson");
person("first") + " " + person("last");      // Kyle Simpson
```

## Structural Mutability
Conceptually, the structure of a closure is not mutable.
However, objects by default are quite mutable. You can freely add or remove (delete) properties/indices from an object, as long as that object hasn’t been frozen (Object.freeze(..)).

Example:
```
function trackEvent(evt,keypresses = []) {
  return [ ...keypresses, evt ];
}
```
Did you spot why I didn’t push(..) directly to keypresses? Because in FP, we typically want to treat arrays as immutable data structures that can be re-created and added to, but not directly changed.

# Recursion
Recursion is when a function calls itself, and that call does the same, and this cycle continues until a base condition is satisfied and the call loop unwinds.

## Mutual Recursion
When two or more functions call each other in a recursive cycle, this is referred to as mutual recursion.

Example:

```
function isOdd(v) {
  if (v === 0) return false;
  return isEven(Math.abs(v) - 1);
}
function isEven(v) {
  if (v === 0) return true;
  return isOdd(Math.abs(v) - 1);
}
```

## Why Recursion
```
function sum(total,...nums) {
  for (let num of nums) {
    total = total + num;
  }
  return total;
}
```
vs.
```
function sum(num1,...nums) {
  if (nums.length == 0) return num1;
  return num1 + sum(...nums);
}
```
It’s not just that the for-loop is eliminated in favor of the call stack, but that the incremental partial sums (the intermittent state of total) are tracked implicitly across the returns of the call stack instead of reassigning total each iteration.

## Tail Calls
```
function foo() {
  var z = "foo!";
}
function bar() {
  var y = "bar!";
  foo();
}
function baz() {
  var x = "baz!";
  bar();
}
baz();
```
Recursion far predates JS, and so do these memory limitations. Fortunately, a powerful observation was made in those early days that still offers hope. The technique is called tail calls. The idea is that if a call from function baz() to function bar() happens at the very end of function baz()’s execution – referred to as a tail call – the stack frame for baz() isn’t needed anymore. That means that either the memory can be reclaimed,
or even better, simply reused to handle function bar()’s execution.

These sorts of techniques are often referred to as Tail Call Optimizations (TCO).

##### Proper Tail Calls (PTC)
First, PTC in JavaScript requires strict mode.
Second, a proper tail call looks like this:
```
return foo( .. );
```
In other words, the function call is the last thing to execute in the surrounding function.

From:
```
function sum(num1,...nums) {
  if (nums.length == 0) return num1;
  return num1 + sum(...nums);
}
```
To:
```
"use strict";
function sum(num1, num2, ...nums) {
  num1 = num1 + num2;
  if (nums.length == 0) return result;
  return sum(num1, ...nums);
}
sum(/*initialResult=*/0, 3, 1, 17, 94, 8);      // 123
```

# List Operations

# Functional Async

# Putting It All Together
