# Chapter 3 Classes, Objects and Variables

Classes constructors are a defined `initialize` method, with instance variables defined on the fly with an `@` symbol followed by their desired name.

		class Test
			def initialize(word)
				@word = word
			end
		end

Instance variables are private by default. If we want to define an accessor, we can do:

		def word
			@word
		end

within the class to get it.

Because defining accessor methods is so common, there is a shortcut to do so:

		attr_reader :isbn, :price

Defines these methods automatically within the class for us. It does not define the variables like in C++ land but just adds the accessor methods to get at them.

Setters are methods of the class ending with an equals sign:

		def word=(word)
			@word = word
		end

If you create a method whose name ends with an equals sign, that name can appear as an lvalue.

If you **just** want to be able to write to the data member and never query against it, you can use `attr_writer`, otherwise to combine `attr_reader` and `attr_writer` use `attr_accessor`.

In this context accessor means you'll be able to both access and set that instance member. Don't confuse this with strictly an accessor, those methods are defined with attr_reader.

For any setter that wants to be able to appear as an lvalue, you must have the class method end in `=`.

ex.

		def custom_setter=(word)
			@word = word
		end

		a = Word.new("1")
		a.custom_setter = "2"

This would print 2 if we printed `a`'s instance member `@word`. These virtual attribute methods which beyond setting instance members to what is passed, manipulate what is passed, is called the **Uniform Access Principle**. 

When setting the values, we don't know if custom_setter is an attribute of the class or if custom_setter is a virtual attribute that sets an instance variable of a different name, but we do either access in the same uniform way.

By hiding the difference between straight instance variable setting methods (attributes) and calculated values setting methods (virtual attributes) you are shielding the rest of the world from the implementation of the class.

The class can then later be changed without needing to update any external lines of code.

An **attribute** is just a method. For further clarification:

> When you design a class, you decide what internal state it has and also decide what state is to appear on the outside to users. The internal state is held in instance variables. The external state is exposed through methods called *attributes*. Other actions your class performs are just called regular methods.

Attributes are methods of the class that expose instance variables to observation or change.

## Importing
Classes can import other classes or modules using the `require_relative` line:

		require_relative 'other_file.rb'

## Access Control
Since instance members are only accessible via accessors, all we need to be concerned about for control access are:

1. **Public Methods**: can be called by anyone, default is public except for initialize which is private.

2. **Protected Methods**: can be invoked only by objects of the defining class and its subclasses.

3. **Private Methods**: cannot be called from an instance such as `a.private_method`, instead the receiver (the object on which the method is called) is always `self`. An example would be `self.private_method`.

Ruby's protected and private are different from several other OO languages. If a method is protected, it can be called by *any* instance of the defining class or its subclasses.

If a method is private, it can **only** be called in the context of the current object, it is never possible to access another object's private methods directly, even if the object is of the same class as the caller.

So if we made objects a and b both of class Word, a.print(b) cannot call b.word, even though print is method from another instance of the Word class. Only the current object b can call its methods using `self`. This is different from C++ for example where the described situation would be allowed.



























