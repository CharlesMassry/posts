---
id: 53
date: "2015-01-30"
title: "Building a Blog Reader in iOS, Part 1"
---
In this post, we will be building an app that retrieves a collection from the web and view it as a table. The data is an index of my blog posts and can be found at [http://www.charliemassry.com/posts.json](http://www.charliemassry.com/posts.json).

![finishedTableViewController](http://i.imgur.com/pjJPsnE.png)

To accomplish this we will need a way to render each cell in this table and fetch the data from the JSON feed. Luckily for use, this table view is a default in iOS development and there is a easy library to use for the fetching of the JSON feed called [AFNetworking](http://afnetworking.com/).

To begin, open up Xcode and create a new app. For this app we will simply use the single view application for learning purposes.

In the `application:didFinishLaunchingWithOptions:` method in our `AppDelegate.m` file, there is some boilerplate code we must add to get started.

    UINavigationController *navController = [[UINavigationController alloc] init];
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.rootViewController = navController;
    [self.window makeKeyAndVisible];  

This seems like a lot, but all this does is make a navigation controller the main controller of the application. When you compile and run the code, you will simply see a main screen that is black. Now how will we get the table?

We must create the table view controller. In the menu, simply click on File \> New \> File. When prompted, make the new file a subclass of the `UITableViewController` and lets call it, `PostsTableViewController`.

First we must initialize the posts in the interface.

    @property (strong, nonatomic) NSArray *posts;

In a table view, there are a few things you need to be aware of. There are properties like title, number of sections, and number of cells per section. In the `viewDidLoad` method, add the line `self.title = @"Blog";` to add the title to the view. In the `numberOfSectionsInTableView:` method, you can set how many sections there are in the table by its return value; for this, just `return 1;`. The `tableView:numberOfRowsInSection:` method behaves similarly, although this one describes the number of cells in each section; `return 20;` for now.

We now have to give the cells data, and we can do this in the `tableView:cellForRowAtIndexPath:` method.

    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
        UITableViewCell *cell = [tableView
            dequeueReusableCellWithIdentifier:@"cell"];

        if (cell == nil) {
            cell = [[UITableViewCell alloc]
                initWithStyle:UITableViewCellStyleDefault
                reuseIdentifier:@"cell"];
        }
        cell.textLabel.text = @"banana";
        return cell;
    }

This method can be very confusing. What we are doing is initializing a table cell and giving it a string to see if we can reuse the particular cell. The reason why we want to do this is to prevent a memory leak. Basically everytime you scroll up and down on a table view, it must recreate every table view cell and they won't be freed from memory. There will always be a fixed number of table view cells, but they will each have different properties such as the title. To fix this you can give it a reusable identifier to know if it has been created. Either way, with this code, it will always give the display text of `"banana"`.

That's a great start but we want to have it display a feed of data from the web, specifically a collection of posts. We can do this by creating a post model and have it loop over a collection of posts and displaying the title individually.

Let's create a new `Post` model and have it inherit from `NSObject`. Next we will want to give it the title, text, id, and comments count properties in the interface.

    @interface Post : NSObject
        @property(strong, nonatomic)NSString *title;
        @property(strong, nonatomic)NSString *text;
        @property(strong, nonatomic)NSNumber *idNo;
        @property(strong, nonatomic)NSNumber *commentsCount;
    @end

Because we will be getting data from the web in the form of a JSON feed, we should create a method to populate it with data. Objective-C has a class called NSDictionary which is almost identical to a Hash in Ruby or a JavaScript Object, but you won't have to worry about JSON parsing with AFNetworking. We should create this method called `initWithDictionary:` that takes an NSDictionary, creates the relevent fields, and returns itself.

    -(id)initWithDictionary:(NSDictionary *)json {
        self = [super init];
        self.title = json[@"title"];
        self.text = json[@"text"];
        self.idNo = json[@"id"];
        self.commentsCount = json[@"comments_count"];
        return self;
    }

This will make our lives much easier very shortly.

Now we must get this data from the web, populate it in the model, and then display it in the view. This is perfect for AFNetworking to handle. To handle dependencies easily, we can use Cocoapods. Like RubyGems, Cocoapods lets you easily keep track of your external dependencies.

    $ gem install cocoapods
    $ pod setup
    $ pod init

In the Podfile we just created, we want to list our dependencies.

    target 'Blog' do
        source 'https://github.com/CocoaPods/Specs.git'
        platform :ios, '~> 7.0'
        pod 'AFNetworking', '~> 2.4'
    end

Be sure to close Xcode and open it up through the `.xcworkspace` file for CocoaPods to work.

    $ open Blog.xcworkspace  

Now how will we fetch that data, I propose creating a separate client to better organize your code as you don't want your model to be responsible for making web requests. Let's make a new file called `PostClient` which also inherits from `NSObject`. Let's make a class method called `getPosts`, which gets posts and creates an array of posts. Also we might want to define a base `PostURL` for reuse later.

First be sure to add the two dependencies' header files that you'll need, `AFNetworking` and `Post`.

    #import "AFNetworking.h"
    #import "Post.h"

Then define the base `postURL`.

    NSString *const postURL = @"http://www.charliemassry.com/posts";

Now for the method implementation.

    +(void)getPosts {
        NSString *postIndexURL = [postURL stringByAppendingString:@".json"];
        AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
        [manager GET:postIndexURL
            parameters:nil
            success:^(AFHTTPRequestOperation *operation, id JSON) {
            NSMutableArray *tmpPosts = [[NSMutableArray alloc] init];
                for (NSDictionary *tmpDictionary in JSON) {
                    Post *tmpPost = [[Post alloc] initWithDictionary:tmpDictionary];
                    [tmpPosts addObject:tmpPost];
                }
            NSArray *posts = [[NSArray alloc] initWithArray:tmpPosts];
            [[NSNotificationCenter defaultCenter]
                postNotificationName:@"postIndexFinishedLoading"
                object:posts];
            } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                NSLog(@"Error: %@", error);
            }];
    }

I'll take this one step at a time. First we are getting the constant and adding a `.json` to the end of it. This is because when we go later to make the request for each post individually, we won't have to write the route again, and we won't leave any room for error. We then initialize the AFNetworking request and send the method `GET:parameters:success:failure:`. `GET` is the URL, the parameters is nil because it is a simple GET request, but there is that weird caret for `success:` and `failure:`. This the Objective-C syntax for a block or anonymous function. The `success:` block is what it does with a successful web response, and you can see that it has the JSON parameter, which is your JSON feed and is either an NSDictionary or NSArray, depending on the request. In this case it is an NSArray of NSDictionaries. If you are familiar with making AJAX calls using jQuery, this should click for a couple of reasons; the parameters that need to be passed in are the same and it is done asynchronously.

Inside of your success block, you have access to the JSON feed, and you'll want to create an NSArray of posts using it. Because NSArrays are immutable, meaning once created they can not be modified, like appending or poping, you must initialize the NSArray after you've already built a temporary one using an NSMutableArray. So you'll loop through the JSON feed, create a post, and add it to the temporary array. Once that's done it will create the NSArray. Then a really tricky thing happens. Because of the way iOS works, it doesn't leave you hanging on a web request, it runs the code asynchronously and the web response is not immediate, so you must give it a way to hook into your `PostTableViewController`. Fortunately this works pretty easily with `NSNotificationCenter`. You send the message `postNotificationName:object:` where the postNotificationName is a string that the controller will be listening for and the object is the object you just created. That's it for the `PostClient`, all we have left is to tell the controller to execute this code and listen for the response.

In the controller, in the `viewDidLoad` method, we will call this class method we just created to get the posts from the `PostClient` and we will have it listen for the `"postIndexFinishedLoading"` notification.

    [super viewDidLoad];
    [PostClient getPostIndex];
    [[NSNotificationCenter defaultCenter]
        addObserver:self
        selector:@selector(dataRetrieved:)
        name:@"initWithJSONFinishedLoading"
        object:nil];

In `viewDidLoad` in the `PostTableViewController`, after we call `[super viewDidLoad];` we have the `PostClient` begin fetching the data from the server. It then listens for the message and calls the `dataRetrieved:` method that we have yet to implement.

    -(void)dataRetrieved:(NSNotification *)retrievedPosts; {
        self.posts = [retrievedPosts object];
        [self.tableView reloadData];
    }

This method gets called with the object that had been passed in to it when the `success:` block finished, so the `retrievedPosts` variable is a `NSNotification` instance, but when you send it the message `object` it returns whatever object that had been passed in. Next you **MUST** tell the table to reload its data or you will never see the posts again.

After all of this you should see your posts displaying and scrollable. Stay tuned for a future post when I go over how to select a post to be displayed individually.