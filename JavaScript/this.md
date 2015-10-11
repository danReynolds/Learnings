# JavaScript `this` Keyword

When a function executes, it gets the `this` property with the value of the object that **invokes the function** when it is used.

If the function is invoked by a certain object, then it is the invoking object, not the object on which the function was defined that is referenced by `this`.

In strict mode, `this` has the value undefined for **global functions** and **anonymous functions** that are not bound to an object.

`this` used inside a function A contains the value of the object that invoked A.

In the global scope without strict mode, all variables and function are defined on the `window` object. Therefore `this` has the value of the `window` object.

so `var test = "on window"` can be retrieved by `window.test`.

Scenarios where `this` is less simple:

1. When borrowing a method that uses `this`

Borrowing a method is the process by which one object uses another object's method for itself, since it wants that functionality but doesn't have it.

For example: A has a sort() method, B does not. So we say that B.sortedResult = A.sort()

If A.sort() uses `this.numbers` as the numbers to sort, B.sortedResult will be A's sorted numbers, not B's, since the method is being invoked by A.

To sort B's values, we need to use:

A.sort.apply(B, numbers)

Now in this case, we are using A's sort, but changing its `this` to explicitly be set to B, so that we sort B's numbers instead of A's.

2. When assigning a method that uses `this` to a variable

The global window will be used as `this` when it is referenced in a function that has been assigned to a variable.

This can also be fixed with bind, for example.

3. When a function that uses `this` is passed as a callback

If you pass a function defined on object A as a callback handler for object B to be used onClick for example, then the object invoking invoking the function is B, so `this` is B not A. `this` is set to the object **invoking** the function.

Even though it still has to call user.clickHandler, the invoker is the button so that is the object whose `this` is used.

This problem can be fixed by using `bind`, so instead of passing `user.clickHandler`, we pass `user.clickHandler.bind(user)` so that the clickHandler gets an explicit `this` value to use.

Bind can also be used for currying by presetting parameters to functions:

var a = blah.bind(null, 'set param 1', 'set param 2');
a("this first param now sets param 3");

Bind **does not** execute immediately, rather it returns a function with `this` and parameters bound as specified.

Call can also manipulate the `this` context, but it calls the function immediately. `blah.call(differentThis, 'param 1')` will immediately invoke function blah, rather than return it.

Apply is nearly identical to call, except that apply takes an array as a second parameter, which it then turns into an argument list.

4. When `this` is used inside a closure

If you iterate over an array using `forEach` within a function, for example, `this` cannot be set to the outer function's this. It is instead bound to the global window object within an anonymous function, or undefined when strict mode is used.

This is because `this` is accessible only by a function itself, not by inner functions.
