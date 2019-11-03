---
id: 55
date: "2015-02-23"
title: "Metaprogramming with Objective C"
---
If you have fallen in love with Ruby's support for metaprogramming like I did, but want to get into iOS development, fear not as there is some features of metaprogramming you can use in Objective C.

In Ruby, if you want to dynamically send a message based on the value of a string, you can use `send`, which takes an argument and any parameters passed in. While this method might not be too useful for Ruby Standard Library Classes, you can use something like this to help out with similarly named methods on the same object. For example, when programming for iOS, you may need to manipulate the color of some view object, like a button. The traditional way to do this is to use `+colorWithRed:green:blue:alpha:` on `UIColor`. There are however convenient methods to get specific colors like `+redColor` and `+blackColor`. If you give an option to the user, say when a button is clicked it selects a color and sets it to an `NSString` instance variable.

In Ruby this would not look like anything we haven't seen before.

    @color # => "red"
    color_method = @color + "Color"
    ui_color = UIColor.send(color_method)

Here you can see the instance variable `@color` is set to `"red"`, and what we want to do is append the string `"Color"` to it and send that method to `UIColor`. This solution in Objective C is not going to be nearly as elegant.

    _color; // => @"red";
    NSString *colorNamed = [_color stringByAppendingString:@"Color"];
    SEL colorSelector = NSSelectorFromString(colorNamed);
    UIColor *uiColor = [UIColor performSelector:colorSelector];

Here you must use `-stringByAppendingString:` instead of `+`. Next, you must create a method object to be able to send it to the UIColor class by using the `NSSelectorFromString(colorNamed)` function. While rarely used, in both Objective C and Ruby, even methods themselves are objects. In Objective C, you can see this because you are declaring the type as a selector, which is what the Objective C runtime calls methods. Finally, you must perform that selector on the `UIColor` class by using `performSelector:` which takes a `selector` type and returns an `id` type and now you have the right UIColor. This method returns an `id` type because it doesn't care what it returns as long as it inherits from `NSObject`, which takes advantage of Objective C's optional dynamic typing. You must however, make sure that it only receives defined messages, or your app will crash.