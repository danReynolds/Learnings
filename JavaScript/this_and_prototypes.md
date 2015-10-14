# This or That

`this` is a runtime binding that is contextual based on the conditions of a function's invocation.

When a function is invoked, a stackframe is created containing info about where the function was called from, the call stack, how the function was invoked, etc. One of the properties of this stackframe is the `this` reference which will be used for the duration of that function's execution.

What it references is determined entirely by the call-site where the function is called.

## Call-site

```
function baz() {
    // call-stack is: `baz`
    // so, our call-site is in the global scope

    console.log( "baz" );
    bar(); // <-- call-site for `bar`
}

function bar() {
    // call-stack is: `baz` -> `bar`
    // so, our call-site is in `baz`

    console.log( "bar" );
    foo(); // <-- call-site for `foo`
}

function foo() {
    // call-stack is: `baz` -> `bar` -> `foo`
    // so, our call-site is in `bar`

    console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

Callstack is bottom to top: baz, bar, foo

## The Rules

Four Rules for determining `this`:

1. Default Binding:

Most common case is standalone function invocation.

```
function foo() {
    console.log( this.a );
}

var a = 2;

foo(); // 2
```

Here, without `use strict` `this` is set to the global object, the window, otherwise it is undefined.

2. Implicit Binding:

If the call-site has a context object, then the implicit binding rule says that it's the **context object** which should be used for the function call's `this` binding.

Implicitly Lost:

If a function is bound to a variable, even off of a context object like:

`var bar = obj.foo`, then `bar` will have it's `this` fall back to the default binding, which is the global scope or undefined.

Since the call-site is `bar`, which is a plain un-decorated call made by the global object, the **default binding** applies. We are not calling `obj.foo` when we invoke `bar`, we are calling `foo`, it is just a reference to `foo` not `obj.foo`.

This makes sense for callbacks too:

```
function foo() {
    console.log( this.a );
}

function doFoo(fn) {
    // `fn` is just another reference to `foo`

    fn(); // <-- call-site!
}

var obj = {
    a: 2,
    foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

We pass `obj.foo` to `doFoo` but this is just an implicit reference assignment to the `foo` function and the call-site is doFoo. It is not called with a context object and therefore we get the **default binding**.


3. Explicit Binding:

Can explicitly bind `this` to a context object with the `call`, `apply` and `bind` functions.

Hard Binding:

```
function foo() {
    console.log( this.a );
}

var obj = {
    a: 2
};

var bar = function() {
    foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

Even though we call `bar` with `window`, in its implementation it calls `foo` with `obj` and the context object cannot be changed from `obj`. No matter how you later invoke the function `bar`, it will always manually invoke `foo` with `obj`.

This hard binding is a common pattern, and has a built-in utility as of ES5 called `bind`.

```
var bar = foo.bind( obj );
bar();
```

A lot of library functions and JavaScript built-ins provide an optional parameter called `context` which can be used to hard-bind instead of having to use `bind` directly.

```
var obj {};
[1,2,3].forEach(function() {}, obj);
```

4. `New` Binding

JavaScript has a `new` operator, and the code pattern to use it looks identical to what we see in class-oriented languages. Most developers assume that JavaScript's mechanism is doing something similar. Not true.

Pretty much any function can be called with `new` in front of it.

`new` is not instantiating a class, or running a special constructor function, it is instead making a **constructor call**. There are no constructor functions, but there are **constructor calls** of functions!

When a constructor call of a function is made, the following things are done automatically:

* A brand new object is created
* The newly constructed object is prototype linked
* The new object is set as the `this` binding for that function call
* the `new`-invoked function will automatically return the newly constructed object.

```
function foo(a) {
    this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

By calling `foo` with `new` in front, we've constructed a new object and set that object as the `this` for the call to `foo`. This is called **new binding**.

The function then returns the object created.

## Priority of Rules

A call-site can have multiple eligible rules. There must be an order of precedence to the rules. First off, `default binding` is lowest priority.

```
function foo() {
    console.log( this.a );
}

var obj1 = {
    a: 2,
    foo: foo
};

var obj2 = {
    a: 3,
    foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

And now here we see that `explicit binding` takes precedence over `implicit binding`.

```
function foo(something) {
    this.a = something;
}

var obj1 = {
    foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

Finally, we see that `new binding` is greater precedence than `implicit binding`.

```
function foo(something) {
    this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

And here it shows that even hard-binding with `bind`, `new` takes precedence over explicit binding.

Therefore the order of precedence for binding is:

New Binding, Explicit Binding, Implicit Binding, Default Binding.

## Determining This

The rules for determining `this` from a function's call-site in order of precedence:

1. **New Binding**: Is the function called with `new`? If so, `this` is the newly constructed object.

```
var bar = new foo();
```

2. **Explicit Binding**: Is the function called with `call`, `apply`, or with a `bind` hard-binding? If so, `this` is the explicitly set object.

```
var bar = foo.call(obj2);
```

3. **Implicit Binding**: Is the function called with a context object? If so, `this` is that context object.

4. **Default Binding**: Otherwise, `this` is the global object, window, or `undefined` if in `strict mode`.

## Binding Exceptions

Normal functions abide by the four rules above, but ES6 introduces a special kind of function that does not use these rules: arrow-function.

Arrow functions are signified not by the `function` keyword, but by the `=>` fat-arrow operator. Instead of using the four standard rules, arrow-functions adopt the `this` binding from the enclosing function or global scope.

```
function foo() {
    // return an arrow function
    return (a) => {
        // `this` here is lexically adopted from `foo()`
        console.log( this.a );
    };
}

var obj1 = {
    a: 2
};

var obj2 = {
    a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

Here `foo` returns an arrow function. The arrow function lexically captures the `this` of the scope it was called in, so since `foo` was bound to `obj1`, even if we call the arrow function `bar` with an explicit-binding of `obj2`, it violates the four rules and **always** is bound to the `this` that was set in the lexical scope it was called in. Even with `new`!

This can replace `bind` like in the following example:

```
function foo() {
    setTimeout(() => {
        // `this` here is lexically adopted from `foo()`
        console.log( this.a );
    },100);
}

var obj = {
    a: 2
};

foo.call( obj ); // 2
```

We don't have to `bind` the timeout callback, rather the callback is a fat-arrow and will always have its `this` set to the `this` of the enclosing scope it was called in.

This is essentially a syntactic replacement of `self = this` from before ES6.
