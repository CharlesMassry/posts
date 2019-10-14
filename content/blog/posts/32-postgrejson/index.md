---
id: 32
date: "2014-09-15"
title: "PostgreJSON"
---
In PostgreSQL version 9.2, JSON storage functionality was introduced, so now it has the capability to be schema-less. From the outside looking in, JSON is just another type of column: text, integer, and now JSON. This creates one big advantage over using a database like MongoDB, you can have benefits of both databases. For example, if you want to allow the user to create his own spreadsheet with any number of columns, you might want to think about using a JSON column. To have this joined to each user though, you might want to use a foreign key to link it to the user, whereas using a fully schema-less database you would have to tack it on to each user, bloating the amount of data for each entry, or doing extra work to keep your database normalized. 

This is great, but how do you actually make use of this JSON feature? Well, in Rails 4, JSON support has been added. So you can simply generate a migration with a JSON type.

    class CreatePlayerStats < ActiveRecord::Migration
      def change
        create_table :player_stats do |t|
          t.belongs_to :team, index: true
          t.belongs_to :player, index: true
          t.json :stats

          t.timestamps
        end
      end
    end

If you wanted to allow users to create a statsheet, you can easily create it and link it to your existing database without having to switch between ORMs and still keeping a somewhat normalized database.  

While taking this approach won't lend itself to some of the other advantages of using a fully schema-less database, such as sharding, you can still use JSON storage to solve some crucial needs.

PostgreSQL version 9.4 should be released any day now, which introduces JSONB, PostgreSQL's answer to BSON, which should significantly improve JSON retrieval times but for now it's still the go to solution for adding schema-less functionality to your schema-full database.