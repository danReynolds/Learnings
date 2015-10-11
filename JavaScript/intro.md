# JS Intro

`==` checks for value equality with coercion allowed.
`===` checks for value equality without allowing coercion. It is called strict equality for this reason.

In short, don't use `==` if either side is boolean or 0, "", or [].

Objects, including functions and arrays are held by reference, so `==` and `===` check whether references
match, not anything about the values.

That's why:

```
var a = [1,2,3]
var b = [1,2,3]
var c = "1,2,3"

a == c // True, since it coerces a to "1,2,3". Arrays are automatically coerced to a string joined by commas
a == b // False, since the two references are different.
```

ES6 has block scoping using `let` instead of var, so that if you say `let a = 5` inside of a conditional or a loop, then the enclosing function cannot access `a`, it is scoped to the block it was declared in.

## Strict mode

ES5 added strict mode. Makes code more optimizable in addition to tightening rules.

## Immediately invoked function expression (IIFE)

```
(function IIFE() {
  console.log("This happens now.");
})();
```

## Closure

A function with an environment. Allows the function to access captured variables through the closure's reference to them, even when the function is invoked outside of the variable's scope.

```
function a(x) {
  function b(y) {
    return x + y;
  }
  return b;
}
var add1 = a(1);
console.log(add1(2)); // 3
```

The reference to the inner `b` function gets returned with each call to the outer function. Then when the reference is invoked, through add1, it remembers the environment it had before, using 1 as `x` and adding `y` which is 3.

There is a closure over the `x` parameter of the outer function, and `x` is captured and re-usable.

## Module pattern

Takes advantage of closures, lets you define private implementations.

Module pattern:

```
function User(){
 var username, password;
 function doLogin(user,pw) {
   username = user;
   password = pw;
 }
 var publicAPI = {
   login: doLogin
 };
 return publicAPI;
}
var fred = User();
fred.login( "fred", "12Battery34!" );
```

The outer scope is the `User` function and `fred` references the returned object, which reveals the public API of the User module.

The inner variables `username` and `password` are captured by the closure and now are retained in the environment and can be changed or accessed as needed.


## Using This

```
function foo() {
 console.log( this.bar );
}
var bar = "global";
var obj1 = {
 bar: "obj1",
 foo: foo
};
var obj2 = {
 bar: "obj2"
};
foo(); // 1: "global"
obj1.foo(); // 2: "obj1"
foo.call( obj2 ); // 3: "obj2"
new foo(); // 4: undefined
```

The four rules of `this`:

1. `foo()` ends up setting `this` to the global object in non-strict mode, and in strict mode, `this` is undefined and an error would be given for trying to access a property off of it.

2. `obj1.foo()` sets `this` to the `obj1` object, since it is `obj1` that invokes its `foo` method.

3. `foo.call(obj2)` sets `this` to obj2 since it is explicitly specified by the `call` method.

4. `this` is set to a new empty object.

## Prototypes

When you reference a property on an object, if that property doesn't exist, JavaScript will automatically use that object's internal prototype reference to find another object to look for the property on. It is a fallback if the property is missing.

```
var foo = {
 a: 42
};
// create `bar` and link it to `foo`
var bar = Object.create( foo );
bar.b = "hello world";
bar.b; // "hello world"
bar.a; // 42 <-- delegated to `foo`
```

Here, the fallback for `a`, which is not a property of `bar` is `foo`, who is checked and has an `a` property.

Since `bar` is prototype-linked to `foo`, JavaScript will automatically fallback to looking for `a` on `foo`.

The most common way this feature is used is as prototypical inheritance.

A more natural way of thinking about prototypes is as behaviour delegation. Linked objects can delegate from one to the other for parts needed.

## Default params

Another thing that ES6 adds is default parameter values, as in other languages. Look up more ES6 features later.

## Transpiling

Transpiling is taking code and compiling it into another language at a similar level of abstraction. So if some of these new ES6 features are not available on older browsers, you can transpile it to ES5 compatible code.

Compiling is taking code and compiling it into another language at a different level of abstraction. So taking C code and turning it into machine code is changing the level of abstraction.
