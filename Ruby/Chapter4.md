# Chapter 4 Containers, Blocks and Iterators
**Arrays** hold a collection of object references.

You can use `..` to get a range from an array. You can additionally use `...` to get a range minus the last element.

		a = [1,2,3]
		a[1...2] == [2]

You can do some clever array manipulation through array indexing:

		a = [1,3,5,7,9]

1. With two indices, the first number is the index at which to begin and the second is the length

		a[2,2] = 4 # a = [1,3,4,9]

Our array has shrunk since we were replacing two numbers beginning at position 2 with the rvalue, but we only supplied one value.

2. If 0 is passed as the length, then the value is placed before the start index:

		a[2,0] = 10 # a = [1,3,10,4,9]

3. If an array is supplied as the rvalue, then the array is treated as one element to insert into the array at the provided location:

		a[1,1] = [11,12,13] # a = [1,11,12,13,10,4,9]

4. Alternatively we can specify a range using the `..` syntax:

		a[0..5] = [] # a = [9]

5. If the range is outside the bounds of the array then it is filled with nils in between:

		a[2..4] = 8,9 # a = [9,nil,8,9]

Notice that index 4 was ignored and not assigned nil, only unfilled values in-between.

Arrays operate as Ruby's general container and can work as stacks, sets, queues, and other containers.

## Hashes

Counting frequency of values in an array:

		a = Hash.new(0)
		b = [1,2,3,1,4,4,4]
		b.each { |c| a[c] += 1 } # {1=>2,2=>1,3=>1,4=>3}

Now we can sort the hash to order by frequency:

		a.sort_by { |index, value| value }

We supply a block to sort_by telling it how to compare the values. Here we just use the value to sort and it will by default order by increasing value.

The result of `sort_by` however is no longer a hash but an array of tuples with the index and value sorted by value.

The result of sorting `a' would be:

		[[3,1],[2,1],[1,2],[4,3]]

Since calling `sort_by` or `each` puts each key and value into a two-dimensional array. Notice that this was unstable, while index 2 came before 3 in the hash it is behind after the sort.

## Built-in Testing
Ruby has a ready to go testing framework called `Test::Unit`.

To use it, just:

		require 'test/unit'
		class YourTestClass < Test::Unit::TestCase

## Scoping in Blocks
If you want to be able to use a variable of the same name in a block without changing its value outside of that scope, do it as a semicolon after  the block parameters:

		stuff = 3
		[1,2,3].each do |number; stuff|
			puts number
			stuff = stuff + 1
		end
		puts stuff # 3

## Iterators
A Ruby iterator is a method that can invoke a block of code. The `each` iterator has a special place in Ruby, each enumerable method `select`, `detect`, `first` on the enumerable module work by calling `each` and then doing a certain block. Note that class Array has its own 'first' as well.

## Enumerators
> The Enumerable mixin provides collection classes with several traversal and searching methods, and with the ability to sort. The class must provide a method each, which yields successive members of the collection.

Enumerators offer many methods to classes, all possible by first calling the class's `each` method. Since the enumerator class also has these methods, they can be chained together and called on enumerator objects. 

Enumerators are **external iterators**. In ruby, collections contain their own iterators. Ruby's included iterators fall down when you need to treat an iterator as an object like you do with C++ with `std::vector<int>::iterator it = blah.begin()` for example, which can then be traversed with `++it`.

Additionally, it is difficult to iterate over two collections in parallel in Ruby.

To combat these limitations, Ruby has a built-in Enumerator class which implements external iterators. Enumerators are **objects**. They take the act of iterating and turn it into an object.

*This is one way of dealing with infinite collections.* Instead of doing `Infinity.each { puts "hey" }`, which never returns, `Infinity.each` will allow you to manipulate the enumerator object.

Create an **enumerator** object by calling `to_enum` on a collection.

    a = [1,2,3,4]
    enum_a = a.to_enum
    puts enum_a.next # 1

The iterator is advanced using next, it is not a lookahead but an advancement.

## Ruby has so many things
So with an external enumerator, you can use a loop, which detects when *any* involved iterator has finished and will exit cleanly:

    shorty = [1,2,3].to_enum
    longman = ('a'..'z').to_enum
    
    loop do
      puts "#{shorty.next} and #{longman.next}"
    end
    # 1 and a
    # 2 and b
    # 3 and c

Then terminates because `shorty` has finished.

Enumerators are **objects.** They takes the act of iterating and turns it into an object. When you do `[1,2,3].map` you don't need to give it a block. The result is an **enumerator object.**


Enumerators have a built in `with_index` method so that you can do
`cat`.each_char.with_index` instead of having to use `each.each_with_index`.

Call `to_a` on an enumerator to get its divisions in an array. For example, `(1..10).each_slice(3).to_a` will give you `[[1,2,3],[4,5,6],[7,8,9]]`.

## Enumerators are Generators and Filters
Enumerators are objects and can be supplied blocks are really cool. They can generate infinite sequences like the following:

		powers_of_two = Enumerator.new do |yielder|
			number = 2
			count = 0
			loop do
				number = 2**count
				count += 1
				yielder.yield number
			end
		end
		
		5.times { puts powers_of_two.next } # 1 2 4 8 16

When the enumerator object needs to supply its next value via 'next' it begins executing at the top and then stops when it yields a value. If it needs to supply another value via a `next` call, it will resume one line after where it stopped.

Enumerator objects are naturally enumerable and enumerable methods can be invoked on them. Instead of using `5.times` we can just use the enumerable method `first`:

		puts powers_of_two.first(5) # 1,2,4,8,16 

And whenever first needs a new value, the enumerator resumes, executing its code from the next line after the line where it provided its previous value.

`powers_of_two` could go on infinitely because it does not terminate in supplying values. Calling `count` or `select` have no stopping point like `first` and will never return!

You can create infinite methods pretty easily with enumerators:

		def infinite_select(enum, &block)
			Enumerator.new do |yielder|
				enum.each do |value|
					yielder.yield value if block.call(value)
				end
			end
		end

		puts infinite_select(powers_of_two) {|b| b % 4 == 0}.first(5) #4, 8, 16, 32, 64

Here we are passing in our enumerator `powers_of_two` to `infinite_select` along with a block. We return the first 5 that satisfy the block. Calling `enum.each` makes this infinite, it will keep requesting values from `powers_of_two` and keep getting fed values. This **only** works because we turn the block into an enumerator using `first`. This makes it only request 5 values and then stop. If we bracketed this then did `first` it would not return.

We request only the first 5 that satisfy the block. These are called filters.

What have implemented above are **lazy enumerators**.

# Lazy Enumeration
In `infinite_select`, it worked because the infinite set only generates its next value if a new value is requested.

`[1,2,3].each` is of class `Enumerator` and has the methods of that class. `[1,2,3].lazy.each` is of class `Enumerator::Lazy` and has the lazy methods of that class.

*A lazy enumerator only checks its values if requested, it then goes through each of its values to see if it should be returned, if not it goes on to the next value, otherwise it returns the value and then checks if another value is requested, repeating the process*.

This is what was designed with `infinite_select`.

Instead of having to build *custom enumerators* that do not preload the collection, Ruby 2.0 comes with a `lazy` method which converts **any** enumerator into a **Lazy Enumerator**. This is really cool, now not only infinite collections would work, but if you request the first `n` of a a set, it won't preload the **entire** collection, but just return the first three before even looking at the fourth.

`(1..Float::INFINITY).select{|a| a > 10}.first(5) # never returns, because first it preloads the entire collection, which is infinite.

But, `(1..Float::INFINITY).lazy.select{|a| a > 10}.first(5) # 11, 12, 13, 14, 15

The lazy version knows that it will be working with an infinity collection, but instead of preloading it, it keeps going through values until it satisfies the condition of the `select`, and does that while it keeps having values requested, which is limited by the `first`.

## Defining Lazy Methods
For custom methods, just append `lazy` at the end of an `enumerator` to make it a lazy enumerator.

    def Integer.all
      Enumerator.new do |yielder, n: 0|
        loop { yielder.yield(n += 1) }
      end.lazy
    end

Now our enumerator is lazy and will invoke the `Enumerator::Lazy` methods.

		Integer.all.select{|a| a > 10}.first(5)

Works. Plus, since a `Lazy Enumerator` returns an object of class `Enumerator::Lazy` they can be chained.

		Integer.all.select{|a| a % 5 == 0}.select{|a| % 10 == 0}.first(5)

Since these are all objects, the code can be split up:

		divisible_by_5 = Integer.all.select{|a| a % 5 == 0}
		divisible_by_5_and_10 = divisible_by_5.select{|a| a % 10 == 0}
		divisible_by_5_and_10.first(5) # 10, 20, 30, 40, 50

While the normal Enumerator can do `Integer.all.select` and return an enumerator, it could not filter.










