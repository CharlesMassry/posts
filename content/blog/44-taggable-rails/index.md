---
id: 44
date: "2014-10-30"
title: "Taggable Rails"
---
The popular social media website Twitter has made features such as tagging through their system of hashtags very intuitive and fun. You can add similar functionality in your Rails app with relative ease using a gem called [Acts-As-Taggable-On](https://github.com/mbleigh/acts-as-taggable-on). First you must add the gem to your Gemfile.

    gem "acts-as-taggable-on", "~> 3.4"

Then

    $ bundle install

Now you can run the migration generator

    rake acts_as_taggable_on_engine:install:migrations

This copies some files from the gem to your `db/migrations` folder. When you look through these migrations, you'll notice that the tags are polymorphic, which is good so you can easily just slap a `acts_as_taggable` on your taggable model. You should then go ahead and migrate your newly generated files.

    rake db:migrate

Now in the model you want taggable just add the aforementioned class method.

    class Post < ActiveRecord::Base
      acts_as_taggable
      ...
    end

Now you can tag your posts, at least in the console. To get it working in the browser requires a little configuration but it shouldn't be too difficult. In your form, just add

    <p>
      <%= form.label :tag_list %>
      <%= form.text_field :tag_list %>
    </p>

Now you must configure it for strong params

    def post_params
      params.require(:post).permit(:title, :text, :tag_list)
    end

Now you can add tags, but how do you display them?

    <% @post.tags.each do |tag| %>
      <%= link_to tag, tag_path(tag.name) %>
    <% end %>

So you are looping through your tags and making a separate `tags#show` page for each of them. this will generate paths like `/tags/ruby`. Next you need to make a path and a controller for the `tags#show` action. In `config/routes.rb` add

    resources :tags, only: [:show]

Then make the controller.

    class TagsController < ApplicationController
      def show
       @posts = Post.tagged_with(params[:id]).page(params[:page]).includes(:tags)
      end
    end

What this method chain is doing is finding the tags by their name, which is in `params[:id]`, which you'll remember we linked to the tag name to give us paths like `/tags/ruby` and not `tags/1`, `.tagged_with` finds tags by name and probably uses polymorphism internally to determine the type of tags it should find. The `#page` method is for pagination with [Kaminari](https://github.com/amatsuda/kaminari) as this feature still works because a tag collection is just an ActiveRecord Relation object, albeit a modified one. And you'll want to slap an `#includes` method to the end to get rid of any N+1 queries. The view would look the same as the posts view but you might want to add the name of the tag which you can get without even setting an instance variable of using a SQL query by just adding `params[:id]` where you want the name to be displayed. Now your Rails app can act as taggable.

But wait, there's more. This gem also includes this cool feature called a tag cloud which changes the css class based on the tags frequency. You can leverage this to create larger links for more frequent tags for a cool effect. You can see this result on my [posts page](/posts).

    <% tag_cloud(@tags, %w(sm md lg xl)) do |tag, css_class| %>
      <%= link_to tag.name, tag_path(tag.name), class: css_class %>
    <% end %>

What this does is add the different css classes to your html links and you can style them by adding to your stylesheet.

    .sm {
      font-size: 1em;
    }

    .md {
      font-size: 1.33em;
    }

    .lg {
      font-size: 1.66em;
    }

    .xl {
      font-size: 2em;
    }

Now the more frequent tags will be larger and the less frequent will be smaller. Pretty cool feature I'd say.