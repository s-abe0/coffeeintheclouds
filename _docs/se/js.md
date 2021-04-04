---
layout: default
---

## JavaScript Concepts

#### var vs. let

`var` is not block-scoped; i.e. it can be accessed outside of code blocks. 

```
  if(true) {
      var insideBlock = "insideBlock";
  }

  console.log(insideBlock);  // output 'insideBlock'
```
However, `var` is function-level scoped. The declared variable cannot be accessed outside of a function:
```
    function print() {
        var hola = "hola";
    }

    console.log(hola); // ReferenceError: hola is not defined;
```

`let` on the other hand is block-scoped; i.e. it is scoped to the closest closing block.
```
    if(true) {
        var insideBlock = "insideBlock";
    }

    console.log(insideBlock);  // ReferenceError: insideBlock is not defined'
```

`var` can be re-declared at anytime in the same scope, but `let` cannot be redeclared:
```
    var coffee = "French roast";
    var coffee = "Columbian";

    console.log(coffee); // Columbian

    let cloud = "aws";
    let cloud = "azure";  // SyntaxError: 'cloud' has already been defined
```

`var` variables are **hoisted**, meaning they are processed when the function or script starts. They can be accessed before they are declared:
```
    function printCoffee() {
        console.log(coffee); // undefined

        var coffee = "Columbian";
    }

    printCoffee();
```
This does not cause any errors because the coffee variable was *hoisted* and exists, but when the variable is logged to the console, it is not defined yet. 

#### Immediately-invoked function expressions (IIFE)

Since `var` has no block-level scope, old programmers invented ways of emulating it by using `Function Expressions`. These are functions wrapped in an expression (parenthesis), which will be immediately executed when the script runs. Variables can then be declared within the *iife* and can be private to that function.

```
    (function() {
        var coffee = "Breakfast blend";

        console.log(coffee); // Breakfast blend
    })();
```

After then iife is executed, the *coffee* variable is no longer available, as it was scoped to the function.

#### Hoisting
Hoisting in JavaScript is not a technical term, but a term that is used to describe how variable and function declarations are put into memory during the compile phase of a script. This allows functions and varaibles to be used in code before they are declared:
```
    printCoffee("French roast"); // French roast

    function printCoffee(coffee) {
        console.log(coffee);
    }
```

However, only declarations are hoisted. Variables will not be initialized until they are reached during execution
```
    console.log(coffee); // undefined
    var coffee;
    coffee = "French roast";
```

Note that only functions and `var` declared variables are hoisted. `let` and `const` are not hoisted, and will return a ReferenceError if they are access before declaration.

#### Closures

A `closure` is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). Every function declaration creates a new lexical environment, and has a reference to the outer lexical environment, all the way up to the global lexical environment. If a variable is referenced within the function, that variable is searched for first witin the current lexical environment. If not found, it searches up the lexical environment chain until its found, or if not found, throws an error. However, this cannot go in the opposite direction. 

A simple example of closure:

```
    function makeCounter(x) {
        let count = 0;

        return function() {
            return count += x;
        }
    }

    let counter = makeCounter(5);
    console.log(counter()); // 5
    console.log(counter()); // 10
    console.log(counter()); // 15
```

When makeCounter is called, it returns the anonymous function which increments the count variable by the given parameter x. The anonymous function has its own lexical environment. When *count* and *x* are referenced, its lexical environment doest have either variable, so the parent lexical environment is searched, which is the *makeCounter* environment. Here, both *x* and *count* are found, so they are used. Any modifications to these variables within the child lexical environment are reflected in the parent lexical environment, which is shown each time *counter()* is called.

A more interesting example of closures:
```
    function makeCounter(x) {
        let count = 0;

        return function() {
            return count += x;
        }
    }

    let counter = makeCounter(5);
    let secondCounter = makeCounter(8);

    console.log(counter()); // 5
    console.log(secondCounter()); // 8

    console.log(counter()); // 10
    console.log(secondCounter()); // 16

    console.log(counter()); // 15
    console.log(secondCounter()); // 24
```

Here, two closures are created, and each have its own lexical environments; i.e. a new lexical environment is created after each call to *makeCounter()*. Each lexical environment stores separate declarations of the *x* and *count* variables, and is seen after a call to *counter()* and *secondCounter()*, as each one prints different values.

This functionality can be useful for creating object-oriented like functionalities in JavaScript, such as data-hiding and private functions:
```
    let getGuitar = function(type) {
        let isTuned = false;

        return {
            tune: function() {
                if(isTuned) {
                    console.log("Already tuned!");
                } else {
                    isTuned = true;
                    console.log("Tuning guitar");
                }
            },
            playSong: function() {
                if(type === "electric") {
                    console.log("Shread like Van Halen");
                } else {
                    console.log("Play some Johnny Cash");
                }
            }
        }
    }

    let electricGuitar = getGuitar("electric");
    let acousticGuitar = getGuitar("acoustic");

    electricGuitar.tune();     // Tuning guitar
    electricGuitar.playSong(); // Shred like Van Halen

    acousticGuitar.tune();     // Tuning guitar
    acousticGuitar.playSong(); // Play some Johnny Cash
    acousticGuitar.tune();     // Already tuned!
```


Immediately Invoked Function Expressions (IIFE) can also be utilized to create closures:
```
    let adder = (function() {
        let value = 0;

        function changeValue(val) {
            value += val;
        }

        return {
            increment: function() {
                changeValue(1);
            },
            decrement: function() {
                changeValue(-1);
            },
            getValue: function() {
                return value;
            }
        }
    })();

    console.log(adder.getValue());  // 0
    adder.increment();
    adder.increment();

    console.log(adder.getValue());  // 2
    adder.decrement();

    console.log(adder.getValue());  // 1

```

#### Promises

A `Promise` is an object representing the eventual completion or failure of an asynchronous operation. Callbacks are attached to promises, instead of passed into functions. A promised is comprised of an *executor* function and *consumer* functions. The *executor* function is passed into the Promise as an argument, and takes two arguments: `resolve` and `reject`. One of the two arguments of the executor function must be called in order for the Promise to be fulfilled. `resolve()` indicated a success of an operation, whereas `reject()` indicates a failure.

Heres an example of creating a Promise:
```
    let promise = new Promise(function(resolve, reject) {
        // do something

        resolve();
    });
```

 Consumer functions can be *subscribed* to the promise using `.then`, `.catch`, or `.finally` of the promise variable. 

 `.then` takes two functions as arguments; the first function handles `resolve` (operation success), the second function handles `reject` (operation failure).

 ```
    promise.then(function(result) {
        // do something with the result
    }, function(error) {
        // indicate an error has occured
    });
 ```

 `.catch` takes only one argument; a function which handles `reject`. It essentially operates the same as `.then(null, function(error) { ... })`, but is shorter and more readable.

 ```
    promise.catch(function(error) {
        // indicate an error has occured
    });
 ```

 `.finally` takes one argument; a function which will always run when the promise is fulfilled. It acts essentially the same as the finally of a try/catch. The function handler for *finally* takes no arguments.
 
 ```
    promise.then((result) => {
        // do something
    }, (error) => {
        // handle error
    }).finally(() => {
        // logic here always runs
    });
 ```

 All three of these clauses can be used like so:

 ```
    promise.then(result => {
        // do something with result
    }).catch(error => {
        // handle error
    }).finally(() => {
        // do some general cleanup logic
    });
 ```

 The real power of Promises comes in with chaining. Whereas chaining original callbacks can be very messy, chaining promises can be as simple as tying together multiple `.then` statements. This can be achieved because a call to `.then` returns a promise. When a handler of a `.then` statement returns a value, that value becomes the result of the promise. 

 ```
    new Promise((resolve, reject) => {
        resolve(2);
    }).then((result) => {
        console.log(result);  // 2
        return result * 2;
    }).then((result) => {
        console.log(result);  // 4
    });
 ```

 A handler can also create a return a Promise, like so:
 ```
    new Promise((resolve, reject) => {
        resolve(2);
    }).then((result) => {
        console.log(result);
        
        return new Promise((resolve, reject) => {
            resolve(result * 2);
        })
    }).then((result) => {
        console.log(result);
    });
 ```

 Promises provide benefits over general callbacks as they can reduce code complexity, increase readability, and remove the problem of *callback hell* when function chaining is needed.


 #### async/await

 The `async` keyword placed before a function declaration simply means this function will return a promise

 ```
    async function f() {
        let i = 2;
        return i;
    }

    f().then((result) => {
        console.log(result);  // 2
    });
 ```

 `await` causes function execution to wait until the promise is resolved. However, only the execution of the function will be halted. Any execution after the async function call will continue.

 ```
    async function f() {
        let prom = new Promise((resolve, reject) => {
            setTimeout(() => {
                console.log("promise complete");
                resolve();
            }, 1000);
        });

        await prom;
    }

    f();

    console.log("This will run before await");
 ```

 Output:
 ```
    'This will run before await'
    'promise complete'
 ```

 `await` can only be used within an `async` function. Using `await` within a normal function will cause a Syntax error.

 It can also be used to mimic the *sleep* operation:
 ```
 
 ```