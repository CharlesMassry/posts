---
id: 54
date: "2015-02-06"
title: "Building a Blog Reader in iOS, Part 2"
---
In iOS, you may want to create a bottom tab bar navigation as seen in any app where the type of interface varies differently, for example in the default Clock app, it has a bottom bar with the World Clock, Timer and others. The World Clock clearly uses a Table View Controller, while the Timer uses what looks like a regular View Controller. To demonstrate this functionality, I will walk you through the process of building a tab bar controller by building on the [last part](http://www.charliemassry.com/posts/53) of this walkthrough, which was just a table view for my blog reader app.

![image](http://i.imgur.com/yoj7pSr.jpg)

As you can see the bottom tab bar has been added since last time and it helps you navigate to different parts of the app. To begin, in the `AppDelegate.m` file, you must instantiate the additional view controllers that you will use as the controllers of the tab bar.

    ArtViewController *artViewController = [[ArtViewController alloc] init];
    artViewController.title = @"Art";
    UINavigationController *artNavController = [[UINavigationController alloc] initWithRootViewController:artViewController];

    LinkViewController *linkViewController = [[LinkViewController alloc] init];
    linkViewController.title = @"Links";
    UINavigationController *linksNavController = [[UINavigationController alloc] initWithRootViewController:linkViewController];

We are creating a view controller for each tab bar item and then we are creating a navigation controller where it is the root controller. We do this because when we want to view sub controllers, the navigation controller will simply push the next view onto the stack.

Next we take all of our navigation controllers, and put them in an array as the list of tab bar items. The order we put them in is the order they will display, from left to right.

    UITabBarController *tabBarController = [[UITabBarController alloc] init];
    [tabBarController setViewControllers:@[postNavController, artNavController, linksNavController]];

Now we will change what the `rootViewController` of the entire app is.

    self.window.rootViewController = tabBarController;

If you think about this as an Entity Relationship Diagram, a tab bar controller sits at the top and it has many navigation controllers, each navigation controller has one sub controller. The navigation controller is really just a controller that controls other controllers with no view associated with it so the view controller it controls can push other views onto the navigation stack.

Now when we launch our app, we will see the changes.