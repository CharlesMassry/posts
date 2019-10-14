---
id: 42
date: "2014-10-28"
title: "Don't use Metaprogramming When You Can Just Throw Classes at Things"
---
Ruby has very powerful support for metaprogramming, which is also known as code that writes code, but it can be difficult to understand what is going on. One of the more obscure features of Object-Oriented Programming is the [Singleton Pattern](http://en.wikipedia.org/wiki/Singleton_pattern), where you can add behavior to specific instances of classes and not all instances.

    x = "wombat"
    y = "wobbegong"

    def x.foo
      "foo"
    end

    x.foo # => "foo"
    y.foo # => NoMethodError: undefined method `foo' for "wobbegong":String

What's going on here is you are defining `#foo` on the instance of String "wombat" but nothing else. When you define this method, internally Ruby is inserting a Singleton class with that method into the objects inheritance structure. When you call this method, Ruby doesn't find it in the String class and it walks up the tree of inheritance till it either finds the method you called, or it calls `#method_missing` which defaults to raising that `NoMethodError` we saw earlier. This is not the only way to define Singleton methods.

    x = "wombat"

    x.define_singleton_method(:foo) do
      "foo"
    end

    x.foo # => "wombat"

You can even do this to a bunch of instances all at once.

    module Animals
      X = "wombat"
      Y = "wobbegong"
      Z = "giant salamander"

      constants.each do |constant|
        eval(constant.to_s).define_singleton_method(:foo) do
          "foo"
        end
      end
    end

`#each` is looking through every constant defined in the module and adding a singleton class `#foo` to them. The way the `.constants` method works is it grabs all of the constants as symbols, which won't work on `#define_singleton_class`, so they must be evaluating with `eval`, which executes any piece of code. If you use eval to parse something a client can enter, you open yourself up to code injection, which is marginally worse than SQL injection.

You could probably think of a few different ways to do this, such as opening up classes, which is also derogatorily called [Monkey Patching](http://en.wikipedia.org/wiki/Monkey_patch).

    class String
      def foo
        "foo"
      end
    end

Now every time you call `#foo` on any String instance, you get `"foo"`.

While these features seem really cool, there really isn't a great use for the Singleton Pattern here because we can just throw more classes at it.

    class Foo
      attr_reader :string

      def initialize(string)
        @string = string
      end

      def foo
        "foo"
      end

      def to_s
        string
      end
    end

    x = Foo.new("wombat")
    x # => wombat
    x.foo # => "foo"

The best parts about this is you can clearly see what is going on, and you can see that you didn't break anything in any other classes.

This is not to say don't use metaprogramming, just don't use it to take away clarity from what is going on.