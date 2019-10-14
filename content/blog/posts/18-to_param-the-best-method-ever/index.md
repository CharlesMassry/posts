---
id: 18
date: "2014-07-15"
title: "to_param, the Best Method Ever"
---
If you want to create vanity URLs, and who doesn't, then to\_param is the method for you. In fact it is my personal favorite method in Ruby on Rails, narrowly edging out time\_ago\_in\_words.

On Twitter, the link for a users profile  would look something like https://twitter.com/charlesmassry which would call  the to\_param method on the user model, if in fact Twitter used Rails. There is  one difficulty in this, users can't have their usernames as certain reserved words,  like sign-out, or search. This can be remedied in two ways, one, you can check validation on the  usernames, or two, you can make it a part of the user resources,  /users/charlesmassry. The second way is much easier because you would  have to check validation on every reserved route if you chose the first one, however, I think the first way looks a lot cleaner.

    app/models/user.rb
    
    class User < ActiveRecord::Base
      ...
      validates_exclusion_of :username, in: %w{ sign-out search }
      ...
    end

As for the to\_param method itself, you would just redefine it in your model.

    app/models/user.rb
 
    class User < ActiveRecord::Base
      ...
      def to_param
        username
      end
      ...
    end

By default, it is just an instance method that essentially returns a string  with the id of that resource to give you routes like https://twitter.com/1 as  opposed to the username at the end.