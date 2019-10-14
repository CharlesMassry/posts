---
id: 24
date: "2014-08-01"
title: "Keep Your Queries DRY with Eager Loading"
---
The N + 1 query problem can be very tricky to deal with or even find because you won't be typically dealing with a lot of data in development mode. Luckily, there's a gem for that. The [Bullet](https://github.com/flyerhzm/bullet) gem solves that problem. It is easy to install and can alert you in any number of ways if you have an N + 1 query problem and gives you hints on how to fix them. The problem mostly occurs with Active Record Associations. For example, if you want to select all of the shouts that belong to their respective users, it would have to select all of the shouts and then all of the users, instead of just every association.

![Not eagerly loading](http://i.imgur.com/6WHcHdo.jpg)

In this request, every shout is being loaded independently because it must match up with it's respective user. The way of getting around this is to use the eager loading technique. Luckily, Ruby on Rails makes this easy with the [includes](http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-includes) method

    Shout.where(user_id: [id] + followed_user_ids).order(created_at: :desc).includes(:content, :user)

This can be used to create a query for a page that displayed the current user's shouts and the shouts of the users she is following, in chronological order and store the content of the shout and who shouted it in memory so it doesn't have to keep making queries. This changes the above to this.

![Eagerly loading](http://i.imgur.com/b4GO8Wr.jpg)

This is so much quicker, your queries won't repeat themselves, and it just looks a lot cleaner, so use eager loading whenever you can.