---
id: 26
date: "2014-08-08"
title: "You're so vain, you probably think this URL is about you."
---
In Ruby on Rails, when you want to make a particular route that has a parameter with spaces in it, you would typically get something like /regions/new%20york for a region named new york. This however looks terrible and your users would be very upset about typing this out. Fortunately, there is a way to remedy this by creating a URL slug.

First add a column called slug to the table of the resource you want to fix.

    rails generate migration AddSlugToRegion slug

This creates the migration that you MUST edit before running `rake db:migrate`.

The preferred way is to copy and parameterize the name column of each entry after the column is created.

    class AddSlugToRegion < ActiveRecord::Migration
      class Region < ActiveRecord::Base
      end

      def change
        add_column :regions, :slug, :string, index: true

        Region.all.each do |region|
          region.slug = region.name.parameterize
          region.save
        end
      end
    end

You don't have to open up the Region model as you are just adding to a new column and you shouldn't have any validations affecting it, but it can be a good habit to use. Then, you are adding a column called slug which is a string and is indexed. Lastly, you are iterating through each row and changing the column to the parameterized name.

You must also edit the corresponding model.

    class Region < ActiveRecord::Base
      validates :name, uniqueness: true, presence: true
      after_create :generate_slug
      before_update :assign_slug

      def to_param
        slug
      end

      private

      def assign_slug
        self.slug = to_slug
      end

      def generate_slug
        update_attributes(slug: to_slug)
      end

      def to_slug
        name.parameterize
      end
    end

This seems like a lot to add, but what is happening, during record creation, is it is updating the slug after the record is created. For updating, it must be before the record is saved because it would already have a url assigned to it.

The controller must also be edited.

    class RegionsController < ApplicationController
      ...
      def show
        @region = Region.find_by(slug: params[:id])
      end
      ...
    end

Here it is finding the region by its slug value. So the slug value `new-york` would be equal to the record that has `new york` in the name field. Now you know how to combine [this post](http://www.charliemassry.com/posts/18-to_param-the-best-method-ever) to use the `to_param` method with the current post to really generate nice routes with any number of named records. I should note, make sure you validate that the name would not equal a path you would use, like `new` or `edit`. Look for more dynamic routes in a future post.