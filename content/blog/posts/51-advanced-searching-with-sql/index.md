---
id: 51
date: "2015-01-09"
title: "Advanced Searching With SQL"
---
Over on the left, you see that there is a nice little search box to search through my posts. When you search for something you might notice the way that the posts are ordered are very helpful. This is because when you search, the results are sorted by the title of the post and then by the text. Implementing this was rather difficult as I had bounced around between a couple of solutions. My original solution was this.

  

    def self.search(query)
      text = search_text(query).select(:id)
      title = search_title(query).select(:id)
      results = title + text
      results.map do |result|
        find(result)
      end
    end

    def self.search_title(query)
      where("title LIKE ?", "%#{query}%")
    end

    def self.search_text(query)
      where("text LIKE ? AND title NOT LIKE ?", "%#{query}%", "%#{query}%")
    end

This code is not great for a couple of reasons. First, you must use class methods for everything because of Ruby scoping, and making class methods private in Ruby is a [hassle](http://blog.roberteshleman.com/2014/08/22/private-class-methods-in-ruby/). Second, It is a very inefficient search because it makes 2 original queries and then one additional query for every result that the first 2 queries found. This can be bad for performance if you have a lot of blog posts. Configuring this search for pagination is a little confusing because [kaminari](https://github.com/amatsuda/kaminari) isn't setup for arrays as default but an ActiveRecord Relation. Also, the way ActiveRecord works is when you are chaining the queries it builds up the query and then executes as opposed to executing sequentially.

One solution I came up with is to use a searching gem like Solr or ElasticSearch. This can be a difficult to set up and requires additional resources. Also, if you have anything against Java or the JVM, these solutions are not for you. Solr has it's problems of configuration because it treats different data types differently even if they both contain characters. ElasticSearch has it's problems of the default gem to integrate it into a Rails app has a terrible API. Both of these solutions however do allow you to boost the results depending on the column.

The SQL solution to this problem is to use weighted searches. While not built into SQL you can add if statements to SQL and create a separate column in SQL on the fly.

    SELECT *,
      IF(title LIKE '%ruby%', 20, 0) AS weight
      FROM `posts`
      WHERE (title LIKE '%ruby%' OR text LIKE '%ruby%')
      ORDER BY weight DESC;

This is how you would write a query in SQL that does this. Notice the syntax in the if statement, it's saying if the title matches `%ruby%`, then put a 20 in the weight column, otherwise put a 0. Once it does this it then filters by the `WHERE` clause and orders it by weight. Now that that's out of the way, how do you integrate this solution into a Rails app using ActiveRecord. [Previously](http://www.charliemassry.com/posts/41-raw-sql-on-rails), I had talked about how to write raw SQL using ActiveRecord but there must be a more elegant solution for this issue, right? Well there is, but there is still the issue of figuring out how to write it.

    Post.select("*, IF(title LIKE ? , 20, 0) AS weight", "%#{query}%").
      where("title LIKE ? OR text LIKE ?", "%#{query}%", "%#{query}%").
      order("weight DESC")

This was my initial idea because it's using the `SELECT` statement to create the additional column. This won't work because the `?` the query afterwards to properly escape it only works for the `where` method and not just any SQL. The way around this is to just drop it in instead of the `?`. You must escape it because it'd be a shame if anyone was to drop your database using SQL injection. Luckily there is a class method on ActiveRecord::Base that sanitizes inputs for SQL.

    Post.select("*, IF(title LIKE #{sanitize("%#{query}%")}, 20, 0) AS weight").
      where("title LIKE ? OR text LIKE ?", "%#{query}%", "%#{query}%").
      order("weight DESC")

This is the final ActiveRecord query that is used and while it doesn't read too elegantly, it works, and is much more maintainable and better performant than my original solution when you don't want to add additional dependencies.