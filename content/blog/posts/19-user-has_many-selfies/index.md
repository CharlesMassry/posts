---
id: 19
date: "2014-07-16"
title: "User has_many selfies"
---
When creating a self referential join through another table, you must think of each entry as a new row even if two of the columns have the same number, but in reverse position. For example, if you are creating an app where a user is a follower of other users that may or may not follow him, you must have two rows, one where the follower is person A and the following is person B, and the second where the follower is person B and the following is person A.

This can be problematic to think of in terms of what each column should be called, but you can find this out through trial and error in Ruby on Rails by listing the methods that your model responds to, and sorting by that alphabetically on each line through the Rails console.

    class User < ActiveRecord::Base
      ...
      has_many :followed_user_relationships,
        class_name: "FollowingRelationship",
        foreign_key: :follower_id
        has_many :followed_users,
        through: :followed_user_relationships
      ...
    end

This code would give you the ability to look at the users that are following you by calling the method `followed_users`. You can then manipulate that data by linking to their page or displaying some type of information about them.