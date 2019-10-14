---
id: 30
date: "2014-08-29"
title: "Nested Comments with Active Record"
---
There are a couple of ways to let your users nest comments with Active Record. The main issue is the comments must start from somewhere, such as the first comment on a post. What would that database look like?

    Column     | Type    | Modifiers
    -----------------+-----------------------------+---------------------
    id         | integer | not null
    body       | text    | not null
    user_id    | integer | not null
    comment_id | integer | not null

But wait, `comment_id` can't be null because the comment tree won't be able to start. Lets try that again.

    Column     | Type    | Modifiers
    -----------------+-----------------------------+---------------------
    id         | integer | not null
    body       | text    | not null
    user_id    | integer | not null
    comment_id | integer | 

There. We'll also need a post\_id to reference the post it falls under, so lets do that right quick.

    Column     | Type    | Modifiers
    -----------------+-----------------------------+---------------------
    id         | integer | not null
    body       | text    | not null
    user_id    | integer | not null
    comment_id | integer |
    post_id    | integer | not null

This'll work, but there is a column that is guaranteed to be null for quite a few records as it will apply to all top-level comments. Surely Active Record has a better way to model this data.

![Polymorphism](http://i.imgur.com/vZqytd1.png?1)

A new challenger approaches.

Polymorphism can allow the users to comment on multiple types of records and are very easily extendable, so you can start off with users being able to comment on posts or other comments, and then you can let them comment on any other database table by just changing a few lines of code around (just don't forget to update your [ERD](http://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)).

    Column           | Type                   | Modifiers
    -----------------+-----------------------------+---------------------
    id               | integer                | not null 
    body             | text                   | not null
    user_id          | integer                | not null
    commentable_id   | integer                | not null
    commentable_type | character varying(255) | not null

Now doesn't that look a lot better. We don't have any records that can be null, and the `not null` lines up a lot better. There is one downside to this method; when `comment.comments` is called, it fires off an extra query:

    SELECT "comments".* FROM "comments"
      WHERE "comments"."commentable_id" = $1
      AND "comments"."commentable_type" = $2
      [["commentable_id", 1], ["commentable_type", "Comment"]].

This can be taxing on a server, but there are ways around it, such as adding a column to identify how far deep the nest is that defaults to 0 and only display a certain number of comments initially, but force the user to request more comments.