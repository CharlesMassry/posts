---
id: 46
date: "2014-11-12"
title: "Metaprogramming with Method Missing"
---
In Ruby, you can extend the language significantly to suit your preferences, and even overide methods already used by the language itself. This is typically derogatorily called Monkey Patching as I had covered [previously](http://www.charliemassry.com/posts/45-when-is-it-okay-to-monkey-patch). One significant feature that I think all Ruby developers should know about is how the error `NoMethodError` gets generated. Because Ruby uses classical inheritance, all classes have ancestors. And as you may already know, everything in Ruby is an object, even classes are objects. Because of this, classes you create come with methods already defined on them. One method is called `#method_missing`. This very important method is defined on `BasicObject`, which is the first Ruby object, and all of its children get access to this method. The way Ruby uses this method is if a method is called on a Ruby object that doesn't have that method defined, it calls `#method_missing` which walks up the chain of inheritance to `BasicObject` to see where `#method_missing` is defined. The way it is defined on `BasicObject` would look something like this if Ruby was written in Ruby.

    class BasicObject
      def method_missing
      raise NoMethodError
      end
    end

The reason why the language works like this is so you can override it and even dynamically create an API so you can override `#method_missing` Let's say you wanted to write an API where the user can type in a camelcased number to access its index in an array.

    class Array
      def method_missing(name)
        index = EnglishNumber.word_to_number(name.to_s) - 1
        self[index] || raise
        rescue
        super
      end
    end

array = [1]
array.one # => 1
array.two # => NoMethodError undefined method 'two' for [1]:Array

Let's suppose that `EnglishNumber.word_to_number` takes a camelcased string, and converts it to a number. If you call a number on the array it will return the index or give a `NoMethodError`.

When `#one` is called on an Array instance, the interpreter checks for `#one` and if it doesn't find it, it then checks if `#method_missing` is defined, and if it is, it is called. This method, like all Ruby methods, can be overridden. In case there is no index on the array, you might want to throw an error, `self[index]` will return `nil` if the array doesn't have that index. You can call `raise` to throw a generic error, but `rescue` will handle that error by passing it off up the chain of inheritance. This will make sure that you will get what you are expecting, as opposed to just getting nil, as that is very nondescript, and `NoMethodError` is very descriptive. While this can seem like a very contrived example, there are some very powerful libraries that take control of this feature such as ActiveRecord, specifically how `.find_by` worked in Rails 3.

In Rails 3, there was no `.find_by` method but a `.find_by_#{attribute}` where attribute was defined by `#method_missing`. This was a very cool feature, but it can lead to a slowdown in performance as every time the Ruby interpreter has to call `#method_missing` and walk up the chain of inheritance. This code was removed for Rails 4 in favor of `.find_by(attribute: attribute)` for what I assume is slight performance gains.

You can use `#method_missing` for some really cool programs, just be sure to stay out of trouble by using `super` to get its default behavior if the method doesn't get what it wants.