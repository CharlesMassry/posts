---
id: 36
date: "2014-09-30"
title: "Using Redis for Autocompletion on Rails"
---
For a website that has search functionality, you might want to have autocompletion, so when the user starts typing it automatically pops up suggestions. This is actually pretty tricky to implement because of a couple of reasons, namely performance. The actual Javascript function of getting the text to pop up is actually really easy if you use the [JQuery-UI autocomplete](http://jqueryui.com/autocomplete/) library.

  

    $(function(){
      $("#q").autocomplete({
        source: "/search_suggestions"
      });
    });

Your application would need to have a `/search_suggestion` path and JSON data at that path. The main question is how to model this data, specifically knowing that SQL queries can take a lot of time. Enter Redis.

[Redis](http://www.redis.io) is an in memory key-value store, which is exactly what we need. You can even try using Redis for free from [your browser](http://try.redis.io/) The idea is to take all the searchable items, make an AJAX request when the user types each letter and render the results as JSON.

First you must setup Redis which is easy enough if you are on Mac OSX.

    $ brew install redis

Next you can start the Redis server in a separate terminal tab.

    $ redis-server

Once that is done you must configure Redis to work with Rails. Luckily, there's a gem for that. In your Gemfile

    gem "redis"

Then

    $ bundle install

Now in `config/initializers/redis.rb` you can have Redis work globally by assigning the instance to a global variable like

    $redis = Redis.new

Since Redis is now configured, we must get Redis to work with the search suggestions.

    class SearchSuggestion
      def self.terms_for(prefix)
        $redis.zrevrange("search-suggestion:#{prefix.downcase}", 0, 9)
      end
    end

In controller you can just render the model as JSON.

    class SearchSuggestionsController < ApplicationController
      def index
        render json: SearchSuggestion.terms_for(params[:term])
      end
    end

Nothing will be displayed however right now as `#zrevrange` is equivalent to a `SELECT` statement. So to insert the data, I recommend writing a rake task to index the terms, as you don't want to have Rails process all of the indexing and get hung up.

You can write a rake task by creating a file in the `lib/tasks` directory ending in `.rake`, we'll call it `search_suggestion.rake`.

    namespace :search_suggestions do
      desc "Generate search suggestions from location"
      task index: :environment do
        SearchSuggestion.index_location
      end
    end

Now when you type in `$ rake -T` you will see your rake task on the rake task list. You can run this task by simply typing

    $ rake search_suggestions:index

Now how do we form that `.index_location` class method. We can take all of the locations and index each of them by their combined letters up to but not including their last letter. We also want to make each individual word indexed by its letters.

    def self.index_locations
      Location.find_each do |location|
        index_term(location.name)
        location.name.split.each { |term| index_term(term) }
      end
    end

    def self.index_term(term)
      1.upto(term.length - 1) do |n|
        prefix = term[0, n]
        $redis.zincrby("search-suggestion:#{prefix.downcase}", 1, term.downcase)
      end
    end

The entire location name is being indexed in the `.index_term` method, and then the individual words are being indexed.

This will be much faster than just using `SQL` queries, but you still probably want to squeeze out that last bit of performance. Well there is a way that involves modifying the Rack Middleware. If you're not familiar with Rack Middleware, what happens when a request comes to your Rails application, the request passes through the Rack Middleware and each piece of Middleware changes the request on the way down until it finally reaches your application. We will want to change this because each piece of Middleware can add some time to the request. First, we must create the Middleware we want to add. We can make a new directory called `app/middleware`. In this directory we can make the Middleware file `search_suggestions.rb`.

    class SearchSuggestions
      def initialize(app)
        @app = app
      end

      def call(env)
        if env["PATH_INFO"] == "/search_suggestions"
          request = Rack::Request.new(env)
          terms = SearchSuggestion.terms_for(request.params["term"])
          [200, {"Content-Type" => "application/json"}, [terms.to_json]]
        else
          @app.call(env)
        end
      end
    end

Each Middleware is initialized with the Application object and each middleware uses the `#call` method on the Application. This is a form of polymorphism as all each Middleware must do is respond to `#call`. Here the request is instantiated with the Application object and the `params["term"]` is extracted to call the search suggestion. The result is then sent back to the client as JSON. To add the Middleware to the top of the stack, you want to go into `config/application.rb`, and inside of the Application class, add

    config.middleware.insert_before(0, "SearchSuggestions")

This inserts the middleware at the front, and now you are all setup with fast autocompletion. There is still some CSS issues that you will run into as the JQuery-UI autocomplete doesn't add any CSS, but I'll leave that to you.