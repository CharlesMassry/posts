---
id: 57
date: "2015-02-26"
title: "Pass Data Around iOS With NSUserDefaults"
---
When dealing with the Model-View-Controller pattern, one of the problems you will face is that it is stateless, while the application itself is a fluid object, and controllers are independent objects that have no knowledge of what called it. This can be a problem if you need data to be passed around. In iOS, you can pass data around by using `-prepareForSegue:sender:`. This however can get complicated if you need to pass data through a `UITabBarController` and a `UINavigationController`.

In Rails, we might do something like pass around the params hash in the URL. This however can get really messy really quickly. A good approach in Rails would probably be to store this information to the database. In iOS, for primitive data types, there is NSUserDefaults, which is a key value store that is good for storing user preferences for your app. There is also Core Data, iOS's ORM, which I will cover in a later post.

When making an app where the user can match cards similar to concentration, one of the problems faced is how to allow the user to choose what the card back looks like. If you want to give the user that option, you need to make another View and another Controller. When a specific image is touched, set the value of the given key to a primitive data type that can be used to access the image.

    -(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
        NSUserDefaults *schoolChoice = [NSUserDefaults standardUserDefaults];
        self.school = [self schoolForButton:sender];
        [schoolChoice setObject:self.school forKey:@"school"];
        [schoolChoice synchronize];
    }

In this method, `NSUserDefaults` is first instantiated with a factory method, `+standardUserDefaults`. Next, the `-schoolForButton:` method turns the button pressed into a string to retrieve the school later and just make sure that this image name is in the images folder. That string is then stored in the `NSUserDefaults` instance by sending it the `-setObject:forKey:`. Finally you must send it `-synchronize` so it saves it for use later.


Now in the controller that will be responsible to play the game, we have to decide where we want to initialize this image name. A perfect place is in `-viewDidLoad` as the controller is already initialized, so you will have access to all its instance variables, and the user won't even see the difference.

    -(void)viewDidLoad {
        NSUserDefaults *schoolChoice = [NSUserDefaults standardUserDefaults];
        self.school = [schoolChoice stringForKey:@"school"];
        [super viewDidLoad];
        [self updateUI];
    }

Again, we are instantiating `NSUserDefaults`, but this time we are retrieving the school and passing it to the instance variable. Then we just call `super` to have the default behavior. We do however need to tell it to update the card backs on all of the cards, and `-updateUI` will provide us with this exact behavior which calls a method that sets the card back to the appropriate school image.

You can find the repository for this source code on [Github](http://github.com/CharlesMassry/Machismo).