# Why Functional Programming?

```
var numbers = [4,10,0,27,42,17,15,-6,58];
var faves = [];
var magicNumber = 0;

pickFavoriteNumbers();
calculateMagicNumber();
outputMsg();      // The magic number is: 42

function calculateMagicNumber() {
  for (let fave of faves) {
    magicNumber = magicNumber + fave;
  }
}
function pickFavoriteNumbers() {
  for (let num of numbers) {
    if (num >= 10 && num <= 20) {
      faves.push( num );
    }
  }
}
function outputMsg() {
  var msg = `The magic number is: ${magicNumber}`;
  console.log(msg);
}
```
vs.
```
var sumOnlyFavorites = FP.compose([
  FP.filterReducer(FP.gte(10)),
  FP.filterReducer(FP.lte(20))
])(sum);
var printMagicNumber = FP.pipe([
  FP.reduce(sumOnlyFavorites, 0),
  constructMsg,
  console.log
]);
var numbers = [4,10,0,27,42,17,15,-6,58];

printMagicNumber(numbers);      // The magic number is: 42

function sum(x, y) {
  return x + y;
}
function constructMsg(v) {
  return `The magic number is: ${v}`;
}
```

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

## All for One

Imagine you’re passing a function to a utility, where the utility will send multiple arguments to that function. But you may only want the function to receive a single argument.

```
function unary(fn) {
  return function onlyOneArg(arg) {
    return fn(arg);
  };
}

["1","2","3"].map(unary(parseInt));
```

## One on One

```
function identity(v) {
  return v;
}

var words = " Now is the time for all... ".split( /\s|\b/ );
words;      // ["","Now","is","the","time","for","all","...",""]
words.filter( identity );      // ["Now","is","the","time","for","all","..."]
```

Because identity(..) simply returns the value passed to it, JS coerces each value into either true or false, and that determines whether to keep or exclude each value in the final array.

## Unchanging One

Certain APIs don’t let you pass a value directly into a method, but require you to pass in a function, even if that function literally just returns the value. One such API is the then(..) method on JS Promises:

```
// doesn't work:
p1.then(foo).then(p2).then(bar);
// instead:
p1.then(foo).then(function(){ return p2; }).then(bar);

function constant(v) {
  return function value() {
    return v;
  };
}

p1.then(foo).then(constant(p2)).then(bar);
```

## Adapting Arguments to Parameters

```
function foo(x, y) {
  console.log(x + y);
}
function bar(fn) {
  fn([3, 9]);
}
bar(foo)      // fails
```

We can define a helper to adapt a function so that it spreads out a single received array as its individual arguments:

```
function spreadArgs(fn) {
  return function spreadFn(argsArr) {
    return fn(...argsArr);
  };
}
bar(spreadArgs(foo));      // 12
```

While we’re talking about a spreadArgs(..) utility, let’s also define a utility to handle the opposite action:

```
function gatherArgs(fn) {
  return function gatheredFn(...argsArr) {
    return fn(argsArr);
  };
}

function combineFirstTwo([v1, v2]) {
  return v1 + v2;
}
[1,2,3,4,5].reduce(gatherArgs(combineFirstTwo));      // 15
```

## Some Now, Some Later

If a function takes multiple arguments, you may want to specify some of those up front and leave the rest to be specified later.

```
function partial(fn, ...presetArgs) {
  return function partiallyApplied(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}
```

Example I:

```
var getPerson = partial(ajax, "http://some.api/person");

// version 1
var getCurrentUser = partial(ajax, "http://some.api/person", { user: CURRENT_USER_ID });

// version 2
var getCurrentUser = partial(getPerson, { user: CURRENT_USER_ID });
```

Example II:

```
function add(x, y) {
  return x + y;
}

[1,2,3,4,5].map(function adder(val) {
  return add(3, val);
});      // [4,5,6,7,8]
```

## Reversing Arguments

Recall that the signature for our Ajax function is: ajax( url, data, cb ). What if we wanted to partially apply the cb but wait to specify data and url later? We could create a utility that wraps a function to reverse its argument order:


```
function reverseArgs(fn) {
  return function argsReversed(...args) { 
    return fn(...args.reverse());
  };
}

var cache = {};
var cacheResult = reverseArgs(
  partial(
    reverseArgs(ajax),
    function onResult(obj) {
      cache[obj.id] = obj;
    }
  )
);
// later:
cacheResult("http://some.api/person", { user: CURRENT_USER_ID });
```

## One at a Time

Let’s examine a technique similar to partial application, where a function that expects multiple arguments is broken down into successive chained functions that each take a single argument (arity: 1) and return another function to accept the next argument. This technique is called currying.

```
function curry(fn, arity = fn.length) {
  return (function nextCurried(prevArgs) {
    return function curried(nextArg) {
      var args = [...prevArgs, nextArg]; 
      if (args.length >= arity) {
        return fn(...args);
      } else {
        return nextCurried(args);
      }
    };
  })([]);
}

[1,2,3,4,5].map(curry(add)(3));      // [4,5,6,7,8]
```

# Composing Functions

Composition is how an FPer models the flow of data through the program. In some senses, it’s the most foundational concept in all of FP, because without it, you can’t declaratively model data and state changes. In other words, everything else in FP would collapse without composition.

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

The FPer doesn’t eliminate all side effects. Rather, the goal is to limit them as much as possible. To do that, we first need to fully understand them.

Pure functions are how we best avoid side effects. A pure function is one that always returns the same output given the same input, and has no side causes or side effects. Referential transparency further states that – more as a mental exercise than a literal action – a pure function’s call could be replaced with its output and the program would not have altered behavior.

Refactoring an impure function to be pure is the preferred option. But if that’s not possible, try encapsulating the side causes/effects, or creating a pure interface against them.

# Value Immutability

ToDo

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

ToDo

# Functional Async

For operations that will be proceed over time, all of these foundational FP principles can be applied time-independently. Exactly like promises model single future values, we can model eager lists of values instead as lazy Observable (event) streams of values that may come in one-at-a-time.
A map(..) on an array runs its mapping function once for each value currently in the array, putting all the mapped values in the outcome array. A map(..) on an Observable runs its mapping function once for each value, whenever it comes in, and pushes all the mapped values to the output Observable.
In other words, if an array is an eager data structure for FP operations, an Observable is its lazy-over-time counterpart.

# Putting It All Together

ToDo
