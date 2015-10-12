# JavaScript is Compiled

While people think it is a dynamic, interpreted language, it is in fact compiled, although not well in advance and not in a way such that you can make it portable across different systems.

JavaScript doesn't have much time to optimize during compilation, as there is no build step ahead of time. For JavaScript, the compilation occurs microseconds before the code is executed.

## Understanding Scope

When traversing scope, starts at the currently executing scope, looks for the variable there, and if not found, goes up one level up until the outermost global scope where it either finds it or doesn't.

LHS variable = variable on the left of the statement that is being assigned to.
RHS variable = variable not on the LHS, so usually being retrieved as part of the statement. Ex. `a = b + c`, `b` and `c` are retrieved as RHS variables.

For a RHS variable, if the variable is is never found, this results in a `ReferenceError` being thrown.

But for an LHS variable, if it doesn't find it, it will crate a new variable of that name in the **global scope**. This can be turned off by using Strict Mode and a `ReferenceError` would instead be thrown.

If a variable is found for a RHS look-up but you try to do something with it that is impossible, like trying to execute a number as a function, or reference a property on a null or undefined value, then a `TypeError` is thrown.

ReferenceError is Scope resolution-failure, whereas TypeError implies that Scope resolution was successful, but there was in illegal action.

## Lexical Scope

Scope look-up stops once it finds the first match, beginning at the innermost scope being executed at the time.

### Cheating Lexical Scope

Leads to poor performance, but can be done:

1. Eval

Takes a string as an argument, treats the contents of the string as if it had actually been authored code at that point in the program.

If you use `eval("var b = 3")` in a function that has no variable `b`, but a global variable of `var b = 2` exists, then `b = 3` will be taken, since it has been dynamically interpreted and modified the lexical scope environment in the function so that a `b` now exists. When `b` is searched for, it will be found in the function and the scope search stops.

In Strict Mode, however, `eval` runs in its own lexical scope, meaning declarations made do not actually modify the enclosing scope.

The use-case for dynamically generating code in the program is small and it should not be done frequently.

You lose optimizations if `eval` is used, since static code analysis cannot be performed because of the dynamic lexical scoping possible with `eval`.

## Function vs Block Scope

The Principle of Least Privilege, or Least Authority, states that in the design of software, such as the API for a module/object, you should only expose what is minimally necessary and hide everything else.

This prevents collisions of two different identifiers with the same name but different intended usages.

```
var a = 2;

(function foo() { // <-- insert this

    var a = 3;
    console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

This is a self-invoking function expression, and it does not declare `foo` in the global namespace, and allows us to enclose variables within a separate scope, without polluting the global one. The function expression is immediately executed as well, without having to separate the declaration and invocation. Immediate invocation has a name, **IFFE**, Immediately Invoked Function Expression.

Function expressions can also be anonymous, we don't have to name `foo` in the above expression example.

## Drawbacks of Anonymous Functions:

1. Anonymous functions have no useful name to display in stack traces, which can be bad for debugging.

2. Without a name, the function cannot refer to itself for recursion, etc, without using the **deprecated** `arguments.callee` reference. The same case occurs when an event handler function wants to unbind itself after firing.

3. Anonymous functions omit a name which is often helpful in providing more readable and understandable code.

The best practice is to always name function expressions.

Another pattern is UMD (Universal Module Definition), where the function to execute is given second, as a parameter to the first IIFE function. The IIFE then calls the function to execute, passing in the global environment.

```
var a = 2;

(function IIFE( def ){
    def( window );
})(function def( global ){

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

});
```

## Blocks as Scopes

While functions are the most common units of scope, block scope is also supported.

Take the common case:

```
for (var i=0; i<10; i++) {
    console.log( i );
}
```

The intention is to use `i` in the scope of the loop, but what happens is that `i` pollutes the scope in which the loop is created.

Block-scoping for the `i` variable would make it only available for the loop, causing an error if it was used anywhere else.

Block scoping is not, however, very commonly used or available.

It is a little-known fact that in ES3, the variables declared in the `catch` clause of a `try/catch` are block-scoped to the `catch` block.

While there are these niche ways of using block scopes, so far it is not very useful.

ES6 changes that with the `let` keyword. The `let` keyword attaches the variable declaration to the scope of whatever block it's contained in. Cool!

Now if we do:

```
var foo = true;

if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}

console.log( bar ); // ReferenceError
```

`bar` is only exposed in the block scope and is not available in the outer scope. This prevents pollution of the outer scope for what is meant to be strictly a block declaration.

To be explicit, you can do:

```
if (foo) {
  {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
  }
}
```

Note, declarations with `let` will **not** hoist to the entire scope of the block they appear in. So:

```
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

Whereas normally, bar would be hoisted.

Another example of the usefulness block scope:

```
function process(data) {
    // do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```

In the above example, the `click` function is a closure that captures the entire scope here. So, after `process` runs, while `someReallyBigData` is never used again and could be garbage collected, it probably won't be because it is captured by the `click` closure.

To fix this, we can do:

```
function process(data) {
    // do something interesting
}

// anything declared inside this block can go away after!
{
    let someReallyBigData = { .. };

    process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```

Here the `someReallyBigData` variable is declared with `let` in the block scope, so when that scope ends, it ceases to exist.

### Let Loops

```
for (let i=0; i<10; i++) {
    console.log( i );
}

console.log( i ); // ReferenceError
```

Let results in a reference error as you'd expect, since the block scope of the for loop has ended. `let` actually re-binds to each iteration of the loop, re-assinging it the value from the end of the previous loop iteration each time.

### Const

ES6 introduces `const` which also creates a block-scoped variable, but whose value is fixed.

```
var foo = true;

if (foo) {
    var a = 2;
    const b = 3; // block-scoped to the containing `if`

    a = 3; // just fine!
    b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

The `const b` goes out of scope with the block, and cannot be changed. `a` remains in scope, since it is declared with `var`.

## Conclusion

Functions enclose variables within the scope of the function, but they are not the only way to use scopes. Block scopes allow variables and functions to belong to an arbitrary block rather than an enclosing function.

# Hoisting

All declarations, both variables and functions, are processed first, before any part of the code is executed.

For example,

```
a = 2;
var a;
console.log(a); // 2
```

Will during the compile phase, hoist `var a` up to make:

```
var a;
a = 2;
console.log(a);
```

Whereas

```
console.log(a); // undefined
var a = 2;
```

is processed as:

```
var a;
console.log(a); // undefined
a = 2;
```

The compiler considers `var a = 2` as two statements, a compile-time declaration and a runtime assignment. While declaration is processed during the compilation phase and hoisted, the second statement, the assignment, is left in place for execution. That means that while the declaration is hoisted above the logging, the assignment is still below and `undefined` is printed.

You can think of it as variable and function declarations are *moved* from where they appear in the flow of the code to the top of the scope. This is called **hoisting**.

Function declarations are also hoisted, but function expressions are not:

```
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
    // ...
};
```

We get a TypeError because the function declaration `foo` is hoisted above, so it isn't a ReferenceError, but the assignment part of the function expression is not hoisted, it stays below. Therefore `foo` is undefined, and when invoked, it gets a TypeError, since its trying to run `undefined`.

An important note is that functions are hoisted above variables.

```
foo(); // 1

var foo;

function foo() {
    console.log( 1 );
}

foo = function() {
    console.log( 2 );
};
```

This prints 1, since `foo` is hoisted above the invocation. `var foo` is ignored, because subsequent `var` declarations are ignored, because the function is hoisted above it.

While duplicate `var` declarations are ignored, subsequent function declarations **do override previous ones**.

## Conclusion

Declarations are hoisted to the top of their respective scopes, while assignment, even of function expressions, are not.

# Scope closure

Closures aren't magical, they are in fact all over your code. By definition, a closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

```
function foo() {
    var a = 2;

    function bar() {
        console.log( a );
    }

    return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```

In the above code, `bar` has a closure over the scope of `foo`. Why? Because `bar` appears nested inside of `foo`. **An inner function closes over an outer function**.

A closure remembers its lexical scope when executd outside of its lexical scope. Here, `bar` is executed, but it is executed outside of its declared lexical scope. It still prints out 2, and is able to remember the environment it was constructed in.

You might think that after `foo` is executed, the scope of `foo` would get garbage collected, since the contents of the `foo` scope is no longer in use. But that scope is *still* in use by `bar` itself.

`bar` has a lexical scope closure over the scope of `foo`, which keeps that scope alive for `bar` to reference later at any time. **`bar` still has a reference to that scope, and that reference is called a closure.**
