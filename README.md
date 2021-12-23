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

## Composing Functions
###### Output to Input

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
// ["to","compose","two","functions","together","pass",
// "the","output","of","first","function","call","as",
// "input","second"]
```

###### General Composition

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
      filteredWords.push( word );
    }
  }
  return filteredWords;
}

var biggerWords = compose(skipShortWords, unique, words);
var wordsUsed = biggerWords( text );
// ["compose","functions","together","output","first",
// "function","input","second"]
```

