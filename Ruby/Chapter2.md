# Ruby Notes

## Pass Reference By Value
**Ruby is pass reference by value.** Another way to say this is that everything is passed by value rather than by reference, but that the values passed are object references, they are a reference type. Everything in ruby is technically pass-by-value, but those values are references to objects and we will call this **pass reference by value**.

Because object references are passed to methods, methods can use those references to modify the underlying object. These modifications are then visible when the method returns.

This is different than `C++` for example because you either pass by value or reference. If you pass by value, it won't be changed, if you pass by reference, it will change.

    a = 5
    def change(b)
      b = 6
    end
    puts a # 5

This has no change, since we passed by reference value. `b` is updated to a new memory location and a is the same as before.

But in the following:

    a = 'str'
    def change(b)
      b[0] = 'S'
    end
    puts a # Str

a was changed because b is a value, and that value is a reference to the same location as a. We have changed the value at that location, not re-assigned the reference value like in the previous example.

Therefore re-assignment in functions doesn't change `a`, but mutation like the previous example will.

Another simpler example of mutation is:

    a = 'str'
    b = a
    b[0] = 'd'
    b = 'what'
    puts a # 'dtr'

Both `a` and `b` are values that reference the memory location where `str` is stored. By changing `b[0]` we change what is at that memory location, the same location `a` is also looking at. But when we do `b = 'what'` we are changing the value of b, and it no longer refers to the same memory as a. Therefore a is only affected by the first change.

## Foreign Operators

`~=` is the **match** operator and is used to match a string against a regular expression.

		line = gets
		newline = line.sub(/Perl/, 'Ruby') # Substitutes first Perl
		newerline = newline.gsub(/Python/, 'Ruby') Substitutes All Python

## Code Blocks

A method can invoke an associated block one or more times using the Ruby *yield* statement.

		def call_block
			puts "start"
			yield
			puts "end"
		end

		call_block { puts "middle" }

This snippet would print:

		start
		middle
		end

because the code in `call_block` is yielding to the block passed to the method. Think of these blocks not as parameters, but as co-routines which transfer control back and forth.

You can provide arguments to the yield call and they will be passed to the block that is getting control.

ex.1

		def who_says_what
			yield("Dan", "Mike")
			yield("Rudy", "Fraa Erasmas")
		end

		who_says_what { |person, person2| puts "#{person} is 
		hangin' with #{person2}" }

This produces "Dan is hangin' with Mike" and so on.

Did not now about `upto`:
 
		3.upto(6) { |i| puts i }











