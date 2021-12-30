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

# Recursion

# List Operations

# Functional Async

# Putting It All Together
