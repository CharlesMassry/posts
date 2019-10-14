---
id: 49
date: "2015-01-06"
title: "Javascript Gotchas"
---
Coming from Ruby, Javascript can be quite intimidating for a couple of reasons, mainly focused on the object system that it implements. While Javascript is an Object Oriented Language, it does not use classical inheritance, but instead uses [Prototypical Inheritance](http://en.wikipedia.org/wiki/Prototype-based_programming), this means that all objects are actually clones of other objects and there is no abstract representation of classes. There are also a few different ways to define an object in Javascript. You can define it in the same way that you would a Ruby hash.

    var person = {};

This is now an object and you can add properties to it.

    person.firstName = "Charlie";

This is interesting but this doesn't seem very Object Oriented as you can imagine this can lead to code duplication as you'll have to add each property as a separate statement. Another way to create an Object is to define it as a function and make instances of that prototype.

    function Person (firstName) {
      this.firstName = firstName;
    };

    var person = new Person("Charlie");
    person.firstName; //=> "Charlie"

This looks much better as you are able to define properties on this prototype.

What if we wanted to add functions to this object? We can do that too.

    function Person (firstName) {
      this.firstName = firstName;
      this.sayHello = function() {
        return "Hello"
      };
    };

    var person = new Person("Charlie");
    person.firstName; //=> "Charlie"
    person.sayHello(); //=> "Hello"

Note the sayHello is a function and unlike Ruby, there is a differentiation between properties and functions so for `person.firstName` we didn't have to add parenthesis to the end but we had to for the function `person.sayHello()`.

What if we had a bunch of functions that shouldn't really be part of the public API for the class. We'd want to create private methods like we would in Ruby, right? Not so fast, Javascript doesn't have the notion of private functions, but we can still acheive this through some trickery.

    function Person (firstName) {
      this.firstName = firstName;
      this.sayHello = function() {
        return hello();
      };
      var hello = function(){ return "Hello" };
    };

    var person = new Person("Charlie");
    person.firstName; //=> "Charlie"
    person.sayHello(); //=> "Hello"
    person.hello(); //=> "Uncaught TypeError: undefined is not a function"

By not binding `hello()` to the `this` keyword, we can leverage Javascript's scope to make it not accessible outside the object. This is different from a language like Ruby where you must declare methods that are private explicitly or Objective C where you must declare methods publicly.

Now that that is settled, how do we call these properties? Properties and functions can be called with either dot notation or bracket notation.

    person.firstName; // dot notation
    person["firstName"]; // bracket notation

    person.sayHello(); // dot notation
    person["sayHello"](); // bracket notation

The bracket notation looks like Ruby's hash calling syntax but when you use it to call a function, I think we can all agree that it just looks ugly. If you want to make a dynamic call however, you must use the bracket notation.

    var numbers = {
      "1": "One",
      "2": "Two",
      "3": "Three",
      "4": "Four",
      "5": "Five",
      "6": "Six",
      "7": "Seven",
      "8": "Eight",
      "9": "Nine"
    };

    var num = parseInt(Math.random() * (9 - 1) + 1);
    num //=> 3
    numbers.num; //=> undefined
    numbers[num]; //=> "Three"

A good way to demonstrate what is happening is to test it's equality.

    numbers.num === numbers["num"]; //=> true

If you actually think about this example, it shouldn't even work because the key is supposed to be the string 3 and not the integer 3, but Javascript is not one for complaining about typing.

Testing equality in Javascript can also be a task that'll leave you scratching your head. Most programming languages use one equal sign for assignment and two for equality, but Javascript uses two and three for equality. Unfortunately, it isn't always consistent knowing what is going to return true.

    1 == 1; //=> true
    1 == "1"; //=> true
    1 === "1"; //=> false
    true == "true"; //=> false
    [] == {}; //=> false
    {} == []; //=> Uncaught SyntaxError: Unexpected token ==
    {} === {}; //=> Uncaught SyntaxError: Unexpected token ===

As a rule, you always want to use the triple equals sign for equality checking as the double equal sign sometimes casts the values to strings. True is not cast to a string, and while clearly an array is not the same thing as a literal object, you can't check the equality of a literal object. You can however test equality after assignment for a literal object.

    var number = {};
    var something = {};
    number == something; //=> false
    number === something; //=> false

Another issue with Javascript's object system is it's Numeric prototype. In a lot of languages you have a number system where numbers can be either floats or integers. In static languages such as C++, the type must be declared when it is initialized so you must call it either a float or an integer. In dynamic languages such as Ruby, you can declare it literally, `x = 3` is an integer, but `x = 3.0` is a float. It does get converted in Ruby to the proper type if you need it.

    3 * 3.0 # => 9.0
    3.0 + 3 # => 6.0

In Javascript, there are no floats and integers, only numbers, which means when I called `parseInt()` up there to get a random number, it essentially just choped off the decimal place. This can lead to a gotcha if you need to match a number as a string from Ruby to Javascript. If you send a float in as JSON in Ruby that looks like `9.0`, Javascript would convert that to just a `9` because it chops of the decimal place when it sees that. To solve this issue you can make it a string on Ruby's end so you don't have to worry about it with Javascript. If you remember these features of Javascript, they can save you a lot of issues down the road.