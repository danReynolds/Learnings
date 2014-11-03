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

Notice that this was unstable, while index 2 came before 3 in the hash it is behind after the sort.










