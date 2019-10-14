---
id: 23
date: "2014-07-31"
title: "Ruby Enumerable, Part 1"
---
The Enumerable module in Ruby can be used for arrays and hashes and can save you from writing complex, nested if statements and loops.

### all?

`#all?` takes a collection and returns true if all objects in the collection return true in the block. The example that the RubyDocs give is:

    %w(ant bear cat).all? { |word| word.length >= 3 } # => true
    %w(ant bear cat).all? { |word| word.length >= 4 } # => false

This can be really useful if you are looking into returning a boolean based on all the elements matching a conditional.

### any?

`#any?`, similar to `#all?`, also takes a collection and returns a boolean. This time it just returns true if one of the elements matches a conditional.

    %w(ant bear cat).any? { |word| word.length >= 3 } # => true
    %w(ant bear cat).any? { |word| word.length >= 4 } # => true

### chunk

`#chunk` method returns an Enumerable object that creates sub-arrays and injects the result of the chunk block into each sub-array.

    [1, 2, 4, 3, 5, 1, 6].chunk { |number| number + 1 } # => #<Enumerator: #<Enumerator::Generator:0x007ffac426adb0>:each>


This does not seem super helpful nor did it return what I said it would but it is still an Enumerable object so you can call any Enumerable method on it, like:

    [1, 2, 4, 3, 5, 1, 6].chunk { |number| number + 1 }.map { |n| n } # => [[2, [1]], [3, [2]], [5, [4]], [4, [3]], [6, [5]], [2, [1]], [7, [6]]]

As you can see, this creates a new array with each value that was created in chunk inserted into the new array before the original array value which was turned into a sub-array. In this example, the original element has 1 added to it and that value gets inserted into a new sub-array with the original element.
  
### collect

Just use map

### count

`#count` method is very useful and very simple. All it does is count the number of elements in the array.

    [1, 2, 4, 6].count # => 4
  
### cycle

`#cycle` method is like each but it never stops and continuously loops through the array executes the block and repeats after a full go around, use with caution.

    [1, 2, 4, 6].cycle { |x| print x } # => 124612461246124612461246...
  
There is one caveat. You can optionally pass an argument to tell it how many times to loop through which can prove to be very useful.  

    [1, 2, 4, 6].cycle(2) { |x| print x } # => 12461246

### detect

`#detect` method is similar to `#any?` except instead of returning a boolean, it returns the value where `#any?` would return true. Very handy.

    %w(ant bear cat).detect { |word| word.length >= 4 } # => "bear"

If it doesn't detect any of the elements, it simply outputs `nil`.

### drop
  
`#drop` method returns the array after getting rid of the number of elements in the beginning passed into it as an argument.

    [1, 2, 3, 4, 5, 6].drop(2) # => [3, 4, 5, 6]

This can be useful if you know you don't need the first couple or so of elements in the array.

### drop\_while
  
`#drop_while` method is similar to the detect method, except it returns an array of the objects that were not dropped, and ends when it reaches the first nil or false value of the block.

    %w(ant bear cat).drop_while { |word| word.length >= 4 } # => ["bear", "cat"]