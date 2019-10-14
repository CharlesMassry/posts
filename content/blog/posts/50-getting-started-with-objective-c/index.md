---
id: 50
date: "2015-01-08"
title: "Getting Started With Objective C"
---
If you have an iPhone, and want to be able to make apps for it, you might want to consider learning Objective C. For iPhone development, you have a few options, you can use [Swift](https://developer.apple.com/swift/), Apple's newest programming language, you can try out [Ruby Motion](http://www.rubymotion.com/) to write Ruby code instead of Objective C, or you can just start out with Objective C. I recommend starting out with Objective C especially if you understand how a statically typed language works and are pretty good at an object oriented language like Ruby. I suggest this because this is where it all began, and you'll find a lot of tutorials on how to get started with Objective C and iOS, which is definitely not the case with Ruby Motion or Swift. Also the iOS API was specifically created for Objective C, so it looks out of place in Ruby or Swift.

For those not familiar with statically typed languages, Objective C forces you to consider what type of variables you are using. Objective C is a language that is on top of C by adding classes and methods to C. Since you are so close to the machine, you must do a lot of things yourself, like manually allocating memory for objects. Fortunately this is not too difficult.

    NSString *firstName = [[NSString alloc] init];

This gives you a string object but it doesn't contain anything. To do that you must initialize it with a string.

    NSString *firstName = [[NSString alloc] initWithString:@"Charlie"];

There are a few things to talk about here. First, you must call the type of object that you are creating when initializing the variable. Second, notice the `*` that precedes the `firstName` variable; that is actually a pointer to the memory location of that object; whenever you call it, you can simply use `firstName` and leave off the `*`. Third, you are sending the message `alloc` on `NSString` which is like a class method that allocates memory for a string. Fourth, you are sending the message `initWithString` with the argument `@"Charlie"` on the result of `[NSString alloc]`.

If your familiar with Ruby you can think of this like `NSString.alloc.initWithString("Charlie")` as the square brackets are used instead of the periods on method calls. Also like Ruby, you can initialize these primitive objects literally.

    NSString *firstName = @"Charlie";

There is one caveat with Objective C; a method is defined by its arguments. In Ruby we have the much underused keyword arguments, and we have something similar in Objective C.

    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"];

        if (cell == nil) {
            cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cell"];
            cell.textLabel.text = [self.posts[indexPath.row] heading];
        }

        return cell;
    }

This is what a method definition looks like in Objective C. This is from the iOS API where, if you are familiar with how the iPhone works, you have a list of items that you want to display, you would use this method for that. If you use the default mail client that comes with the iPhone, it uses this to display the mail. In the method signature you see that first you are declaring what the method will return, a `UITableViewCell` object. This means that the variable that gets returned must be a `UITableViewCell` instance, which `cell` is. Then you see that you pass in arguments with their keywords but you use the arguments as the local variables and the keywords as part of the method definition, so you have access to `tableView` and `indexPath`. The body of the method is unimportant for now and I'll cover it later as it is very important to know how to make this particular view for iOS.

Every Objective C object inherits from `NSObject`, but you can't perform C operations on them like addition. For example, there exists a class called `NSNumber`, which is very similar to the Ruby `Integer` class. This means that you can all methods on `NSNumber` that you can't on C `int`s. If you want to add two numbers in Objective C, you have to add them in a special manner.

    NSNumber *num1 = @1;
    NSNumber *num2 = @2;
    NSNumber *num3 = @([num1 intValue] + [num2 intValue]);

Notice how you must call `intValue` on the numbers to get their literal integer value so you can add them together. This is because it uses C to add them together and not Objective C.

Objective C can be a difficult language to pick up, but once you get the hang of it, it can be very useful to you in learning the iOS API and developing for iOS.