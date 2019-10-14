---
id: 47
date: "2014-11-17"
title: "Reduce Your SQL Queries With a Counter Cache"
---

Counter caching is a technique to reduce the number or SQL queries when attempting to call count on a resources' association. For example, on my blog, I have posts which have many comments, and on the index page, I am displaying the number of comments on each post. Originally it was firing off ten SQL queries for this plus one for all the posts, but I was able to reduce it to just one because I implemented a counter cache.

This past weekend I taught at [Railsbridge](https://www.bridgetroll.org/) and towards the end the students seemed very enthusiastic about implementing this counter cache in order to add sorting functionality and performance. The app was called Suggestotron which was a simple app that lets a user add a topic such as `"fruit"` and then the user can vote on it and any other topics. While this app was simple, we were able to finish early and begin the extra suggested exercises. One was to sort the topics by the votes count. One idea that was tossed around was to implement a counter cache on the association. To add a counter cache to the Topic model we would need to generate a migration.

    $ rails generate migration AddVotesCountToTopic votes_count:integer

This generates a migration with the appropriate timestamp.

    class AddVotesCountToTopic < ActiveRecord::Migration
      def change
        add_column :topics, :votes_count, :integer
      end
    end

This will add the right column, but if we think about it, this is not exactly what we want. Two problems come to mind. First, the column will default to `nil` which is bad as we can't perform an operation like `nil + 1`. If we do, we will get something like `NoMethodError: undefined method '+' for nil:NilClass`. Ideally we'd like to start a zero. Also we would like to update the column for records that already have votes on them. To do this we will have to execute Ruby code to take the result of `topic.votes.count` and set it to `topic.votes_count`. We can do this with an iterator like `#each`

    class AddVotesCountToTopic < ActiveRecord::Migration
      def change
        add_column :topics, :votes_count, :integer, default: 0

        topics = Topic.all

        topics.each do |topic|
        topic.update!(votes_count: topic.votes.count)
        end
      end
    end

The code `default: 0` will automatically set the `votes_count` column to `0` instead of `nil`, and the iterator will make sure `votes_count` accurately reflects the number of votes. Also we want to use `#update!` instead of `#update` because it will raise an error if it fails and undo the migration instead of a half finished migration.

Now you can run that migration.

    $ rake db:migrate

Now that you have your database set up, how will Rails know to increment the counter every time? You could add it into your controller on vote creation, or you can use this [belongs\_to](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#method-i-belongs_to) option, `counter_cache: true`. In the model that is on the `belongs_to` end of the relationship, in this case Votes, we can add the counter cache association.

    class Vote < ActiveRecord::Base
      belongs_to :topic, counter_cache: true
    end

Now every time the Topic receives a vote it increments the `votes_count` column. Also if you were to use the destroy method, it would decrement the `votes_count` column. Now we are ready to sort by votes.

Adding the code for this is now easier and more efficient. In the controller to display topics by the number of votes, we can change the index method to finalize this feature.

    class TopicsController < ApplicationController
      def index
        @topics = Topic.order(votes_count: :desc)
      end
    end

Now that your topics have their votes counted and ordered, you can display them in the view using embedded Ruby.