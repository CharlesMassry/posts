---
id: 45
date: "2014-11-03"
title: "When Is It Okay to Monkey Patch?"
---
Monkey Patching is a term in Object-Oriented Programming when you open up an existing class and add new methods to it. A good way to find cool Ruby tricks is to look into the implementation of the [ActiveSupport](https://github.com/rails/rails/tree/master/activesupport) gem included in Rails. They do have some Monkey Patching, although it was definitely done after careful consideration.


    class String
      def remove(pattern)
        gsub pattern, ''
      end
    end

A lot of times you may want to remove characters from strings using `#gsub`, and it was decided that it may be a lot nicer to have `#remove`.

    "foo".remove(/o/) # => "f"
    "foo".gsub(/o/, "") # => "f"

You won't find any bad examples of Monkey Patching in ActiveSupport, but if you were to try and add your own, it might not end well. The standard library in Ruby is so robust, you might find that it is not even useful to Monkey Patch classes as you can do more harm than good. With other languages, you might not be wrong to Monkey Patch. Take for example Javascript. There is no method on the string prototype `#capitalize` to turn the first letter in the string into a capital letter. If you need to use this functionality in your Javascript, you have two choices, Monkey Patch the string prototype, or add a method globally. First the former,

    String.prototype.capitalize = function () {
      return this.charAt(0).toUpperCase() + this.slice(1);
    };

Next the latter,

    function capitalize(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    }

Neither of these options are great. The former is Monkey Patching, but it can be called in a very object-oriented manner.

    "foo".capitalize(); // => "Foo"

The latter can be called with any type of argument and will give you a runtime error.

    capitalize(42)
    TypeError: Object 42 has no method 'charAt'

Sure you can do a bunch of type checks, but that's a code smell when dealing with a duck type language such as Javascript. In any event, it is really up to you to decide what to write, and either way, you will have conflicting viewpoints, but whatever you do, don't redefine methods on classes unless that is the behavior you seek.