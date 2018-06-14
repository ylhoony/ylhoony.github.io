---
layout: post
title:      "Scope, Hoisting ,and Arrow functions in JavaScript"
date:       2018-06-14 19:18:10 -0400
permalink:  scope_hoisting_and_arrow_functions_in_javascript
---

 
I am quite new to JavaScript, and learned always use const and let for variable declaration, which barely leads me to problems. The code below made me quite confused when it comes to `var`, so I am writing this to clarify the definition of scope and hoistings.


```
var salary = '$100,000';

function bonus() {
  console.log('My current salary is', salary);   // logs 'My current salary is undefined'
	
  var salary = '$50,000';
	console.log('My bonus is', salary);   // logs 'My bonus is $50,000'
}

bonus();
```


## What did I miss?

When use `var`,
The main part I missed was Variable Declaration and Variable Initialization. 
While JavaScript engine reads a JavaScript file from top-to-bottom, 
- Declaration is only to save the name to the memory
- Initialization is to assign actual value to the declared value.

Declaration is hoisted, but Initializations are not hoisted!


Flow of the code above look like below: 

```
1. var salary;  // Javascript declared 'salary' variable saving to memory, meaning only container named as 'salary' is created.
2. function bonus()  is hoisted, meaning JavaScript putting function declarations into memory before it executes any code
3. bonus() function is invoked.
4. bonus function starts to run, and try to log the salary. 'salary' variable is available, it returns 'undefined' since  the value of $100,000 is not initialized in this flow. 
5. var salary = '$50,000'; // 'salary' variable is now initialized with $50,000.
6. it logs 'My bonus is $50,000'
```


**Below are the summary of the Scope and Hoisting.**


## Scope in JavaScript

According to MDN, Scope is the **current** context of execution (execution context).
The context in which values and expressions are "**visible**," or **can be referenced.** 
If a variable or other expression is not "in the current scope," then it is unavailable for use. 

Scopes can also be layered in a hierarchy, so that **child scopes have access to parent scopes**, but not vice versa.

Simply, scope determines what variables or functions we can access inside/outside the block.
So, what is the execution context, then? 


### Execution Context

The execution context is not exactly the same as the scope. 

ECMAScript defines the execution context as below: 
```
An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation.
```

Ok, I have no idea what this means. Did some more digging, below is general flow. 
I am quoting below from other blog ([link](http://ryanmorr.com/understanding-scope-and-context-in-javascript/))

```
Fundamentally, scope is function-based while context is object-based. 
In other words, scope pertains to the variable access of a function when it is invoked and is unique to each invocation. 
Context is always the value of the this keyword which is a reference to the object that “owns” the currently executing code.
```

The easiest way to see the execution context is using `this` keyword in JavaScript.
When JavaScript is executed initially, it enters into the **global execution context**, which creates a new **scope (global scope)** for variables and function. If you declare variable and funcion within global scope, it is accessible from the functional scope.
Once you declare and invoke function, it enters into the **new execution context** with its own **scope (function scope)**.  If you declare variable and funcion withinin this function scope, it will be accessible within this scope only.


```
const test = {
  getContext: function() {
    return this;
  }
}
test.getContext(); 
// returns => { getContext: [Function: getContext] }
```


Briefly, there are **global scope** and **function scope** which controls the accessibility of variables and functions within respective ** execution context**. 

There is one more scope added with ECMAScript 2015 (ES6)  - **BLOCK SCOPE**



### Block-Scoped variables

ES6 introduced the block-scoped variables `const` and `let`.
Let's see what makes difference when you declare variable with `var` or `const` and `let`.

Here is good summary of each variable declaration keywords - 

**Keyword	 |            Scope	          |  Hoisting	 |  Can Be Reassigned	 |  Can Be Redeclared**

 `var`	          |  Function scope  |	      Yes	      |                 Yes	                   |                 Yes

`let`              |     	Block scope    |      	No       |                Yes                    |	                No

`const`        |     	Block scope     |        No        |               	No                     |                 	No



#### `var`

When you declare variable with keyword `var`, variable declaration will be hoisted wherever they occur, again, initialization is not hoisted.

`var` keyword is **NOT** block-scoped, what does it mean? 

```
function x() {
  if(true) {
    var myVar = 1234;
  }
	myVar = 5678;
  console.log(myVar);
}

x(); // => returns 5678;
```

If you have declare a variable with `var` keyword inside the block statement in the function scope, you can access the variable outside the block (but in the function), and also reassign or redeclare the variable with the same name!
This is not good at all..


Below is information about `var` keyword from MDN:
```
The scope of a variable declared with var is its current execution context, which is either the enclosing function or, for variables declared outside any function, global.  If you re-declare a JavaScript variable, it will not lose its value.

Assigning a value to an undeclared variable implicitly creates it as a global variable (it becomes a property of the global object) when the assignment is executed. The differences between declared and undeclared variables are:

1. Declared variables are constrained in the execution context in which they are declared. 
2. Undeclared variables are always global.

2. Declared variables are created before any code is executed. Undeclared variables do not exist until the code assigning to them is executed.

3. Declared variables are a non-configurable property of their execution context (function or global). Undeclared variables are configurable (e.g. can be deleted).
```


### `const` - Constants

Declaring a variable with `const` keyword creates a constant those value cannot change through re-assignment, and it can't be redeclared. 
Scope of the `const` can be either global or function or local to the **block**.

`const` variable must be initialized with declaration, specifying its value in the same statement since it can't be changed later.

```
function x() {
    const varX = 1324;
    if(true) {
        console.log(varX); // returns => 1324
        const varY = 5678;
    }
    console.log(varY); // returns => Uncaught ReferenceError: varY is not defined
		
    const varZ = 9012;
    const varZ = 9000; // if you include this line, returns =>  Uncaught SyntaxError: Identifier 'varZ' has already been declared
}
x(); 
```

This example shows that you cannot access to the `const` variable declared and initialized in `if` block statement.
Also, it throws an error message when you try to reassign the value to `const` variable. 

```
function x() {
    const varX = 1324;
    if(true) {
		const varX = 5678;
        console.log(varX);  // returns => 5678
    }
    console.log(varX); // returns => 1324
}
x();
```
This example is slightly different from previous one, using same variable name inside/outside `if` statement.  They stay in their own scope and returns the value. 


### `let`

The `let` keyword declares a block scope local variable, optionally initializing it to a value.
Similar to `const` keyword, only difference is you can reassign the value to the variable.

This is unlike the var keyword, which defines a variable globally, or locally to an entire function regardless of block scope.

```
function letTest() {
  let x = 1;
  if (true) {
    let x = 2; 
    x = 4;
    console.log(x);  // returns 4
  }
  x = 9999;
  console.log(x);  // returns 9999
}
```
The example above shows the value reassignment and how block scope returns the value. 


## Hoisting

Hoisting is that 
```
the variable and function declarations are put into memory during the compile phase, but stay exactly where you typed them in your coding.
```

### Function Hoisting

One of the advantages of JavaScript putting function declarations into memory before it executes any code segment is that it allows you to use a function before you declare it in your code. 

```
catName("Chloe");

function catName(name) {
  console.log("My cat's name is " + name);
}

//=> returns 'My cat's name is Chloe'
```

What about **function expressions**? 

For the first one as below, I used `var` keyword for declaration. 
Again, only declarations are hoisted for variables when using `var` keyword.
So, JavaScript knows there is a variable for function expression, but cannot execute it since it is not yet initialized.

```
catName("Chloe");

var catName = function(name) {
  console.log("My cat's name is " + name);
}

//=> returns TypeError: catName is not a function
```

The second example below used `const` keyword instead of `var` keyword. 
It returns "ReferenceError: catName is not defined". 
As stated in the summary, `const` keyword is not hoisted, so there is no record of variable name hoisted, meaning saved to the memory.

```
catName("Chloe");

const catName = (name) => {
  console.log("My cat's name is " + name);
}

//=> returns ReferenceError: catName is not defined
```


### `var`iable Hoisting

Again, when you use `var`, variable declaration is hoisted - as it appears that the variable declaration is moved to the top of the function or global code. This also means that a variable can appear to be used before it's declared. 

This can cause confusion - 
```
function surprise() {
  console.log(call);   // undefined
  var call = "WOW!";
  console.log(call);    // "WOW!"
}
surprise();
```

This is actually the same flow as the first one on the top. `call` variable is hoisted, but when logging first time, it was not initialized with the value, so it returned `undefined`.  And, the second logging returned the correct value.

If you replace `var` with `const` or `let`, you will see that it throws an error message - ReferenceError: call is not defined.

Simple solution to this is that 
1. use `const` or `let` for variables, avoide using `var`
2. declare them on the top of the scope


### Arrow Function Expression

Below is definition of arrow function in MDN - 
```
An arrow function expression has a shorter syntax than a function expression and does not have its own this, arguments, super, or new.target. 
These function expressions are best suited for non-method functions, and they cannot be used as constructors.
```


### shorter functions

Arrow functions are shorter than regular function expressions, and also **anonymous**.
Below is one example - 

```
// function expression
var multiply = function(x, y) {
  return x * y;
};

// arrow function expression
const multiply = (x, y) => { return x * y };
```

In JavaScript, functions are 'first class objects' which can be passed, declared, handled, and have properties and methods just like any other object.
So, we can set a pointer to an arrow function, or pass an arrow function through as an argument to another function.
The return value of a function declaration is a pointer to the function object itself.

```
const square = (n => n * n);

[1, 2, 3].map(n => n * n);

[1, 2, 3].map(square);
```


### Return key word
Arrow function expressions return the result of the expression. So, `return` keyword is optional .
```
var multiply = (a, b) => a * b // returns the result
var multiply = (a, b) => { a * b } // returns undefined
var multiply = (a, b) => { return a * b } //returns the result
```

Whenever we wrap the function block body `{  }`, `return` keyword is required.



### non-binding of this

When a function is invoked from another function, the context or this value is global.
However, you don’t need to create `self = this` closures or use `bind(this)` with arrow function expression since it allows you to retain the scope of the caller inside the function.

a. there is no binding when you invoke, resulting in window as `this` context
```
const person = {
  firstName: 'bob',
  greet: function() {
    return function reallyGreet() {
      console.log(this); // returns window
      return console.log(`Hi, I'm ${this.firstName}`);
    }
  }
}
person.greet()();
// Hi, I'm undefined
```

b. `bind(this)` is added, `this` as context has changed.
```
const person = {
  firstName: 'bob',
  greet: function() {
    return function reallyGreet() {
		  console.log(this); // returns {firstName: "bob", greet: ƒ}
      return `Hi, I'm ${this.firstName}`
    }.bind(this)
  }
}
person.greet()();
// Hi, I'm bob
```

c. updated `greet` function with arrow function expression which retains the scope.
```
  const person = {
    firstName: 'bob',
    greet: function() {
      return () => {
			  console.log(this); // returns {firstName: "bob", greet: ƒ}
        return `Hi, I'm ${this.firstName}`
      }
    }
  }
  person.greet()();
  // "Hi, I'm bob"
	```




**References: **

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#Function_scope
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting
https://developer.mozilla.org/en-US/docs/Glossary/Hoisting
https://www.sitepoint.com/back-to-basics-javascript-hoisting/
https://codeburst.io/javascript-what-is-hoisting-dfa84512dd28
https://medium.com/@naveenkarippai/scoping-and-hoisting-in-javascript-2c2e82107427
https://medium.com/front-end-hacking/hoisting-in-javascript-f4a600a02a78
https://www.digitalocean.com/community/tutorials/understanding-variables-scope-hoisting-in-javascript
https://stackoverflow.com/questions/7493936/is-there-a-difference-between-the-terms-execution-context-and-scope
http://www.ecma-international.org/ecma-262/8.0/index.html#sec-execution-contexts
http://ryanmorr.com/understanding-scope-and-context-in-javascript/
https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/

