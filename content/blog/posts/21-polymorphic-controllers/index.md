---
id: 21
date: "2014-07-21"
title: "Polymorphic Controllers???"
---
That's right, polymorphic controllers in Ruby on Rails. You made your model polymorphic, but now you would have to make a controller for each use case of the association. But what if I told you that you can simply make your one controller function for each of the different associations. I say simply, but it can actually become quite complex as you have to get the correct model depending on how the URL looks and turn that string into a constant and then find the correct record.

    class HatesController < ApplicationController
      def create
        this_route = request.fullpath
        this_route.sub!(/\\/comments.*|\\/hate/, '')
        Hate.create(user: current_user, hateable: hateable)
        redirect_to this_route
      end
      ...
      private

      def hateable
        route_to_hate = request.fullpath
        klass = ["comments", "images", "galleries", "groups", "users"].detect do |k|
          route_to_hate.include?(k)
        end
        klass.singularize.classify.constantize.find(params[:id])
      end
    end

There is a lot going on in this controller. First, in the hateable method, it gets the URL and detects which model the user is hating, and returns just the first one it finds. For nested resources you must put the deepest one at the front. Once it gets the correct string, it then must turn it into the model by calling classify on the string, which makes the string into a camelcased string. Next up is constantize, which turns that camelcased string into an actual model call. Then it just finds by the id in the params hash.

The next issue is the create method, which gets the path and scrubs it to something that exists as an actual `GET` route, using a fairly complex regular expression. What this regular expression `/\\/comments.*|\\/hate/` is doing is it is getting the comments path if the word comments is in the path, then it gets the slash and the digits, but it will always get the word hate for the purposes of this controller action. This is necessary to know where to redirect the user to. It then creates the hate by using `@hateable` from earlier, and redirects to the afformentioned route.

Depending on your type of path and resource you may have to change the way you get the model from the route but it can be done with much less code then making a seperate controller for each polymorphic relationship.