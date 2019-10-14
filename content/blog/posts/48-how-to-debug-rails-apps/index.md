---
id: 48
date: "2014-11-20"
title: "How to Debug Rails Apps"
---
At any point in time, you can run into an error in your Rails App, but don't worry as debugging can be easy with the help of multiple gems. When I start a Rails App from scratch there are certain gems I like to include to help me debug, four come to mind immediately. These are [better\_errors](https://github.com/charliesome/better_errors), [binding\_of\_caller](https://github.com/banister/binding_of_caller), [did\_you\_mean](https://github.com/yuki24/did_you_mean), and [pry-rails](https://github.com/rweng/pry-rails). Each of these gems have different uses but `better_errors` and `binding_of_caller` work in tandem.

`better_errors` is useful as it gives you a different error page in your web browser, this can be helpful as it lets you inspect files in the stack trace right in your browser. If you install `binding_of_caller`, then you can actually have a live shell to interact with to execute any arbitrary Ruby code to help you debug. This can help determining what each variable you set is equal to and what methods you have available.

`did_you_mean` is another awesome gem that gives autocompletion suggestions, so if you misspell a method or constant, it asks you if you meant something different. This is very useful if you are using obscure methods that you can't remember the name of or if you are just bad at spelling.

`pry-rails` is the last great gem I'd like to cover in this post. It really does three very useful things. First, when you open up the console with `rails c` it automatically uses pry instead of irb, which means you get cool features like syntax highlighting, and better displayed results. Second, you have access to special helper methods like `show-models` to list all of your models, or `show-middleware` to print out all your middleware. Third, anywhere in your Rails stack you can add a line `binding.pry` and if your application hits that line, a shell gets created in your terminal so you can execute any Ruby code to help you debug with access to all the local variables at that breakpoint.

While `binding_of_caller` and `pry-rails` sound similar, there is a important distinction. If you get an error message you can use `binding_of_caller` but if there is a logic error and the result is different than what you want, you would probably need something like pry. Also, `binding_of_caller` only works in the browser, but if you are doing TDD then `pry-rails` can help you in your tests as well, or even if you are experimenting with code in the console.

There are many more tools to help you debug and they will be applicable to different environments, but these are good tools to help you get the job done.