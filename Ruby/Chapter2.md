# Ruby Notes

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











