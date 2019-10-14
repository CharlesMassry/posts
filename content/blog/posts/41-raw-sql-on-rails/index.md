---
id: 41
date: "2014-10-15"
title: "Raw SQL on Rails"
---
ActiveRecord is fantastic, but sometimes if you are doing complex joins and such, you may need to move away from ActiveRecord and write raw SQL. Consider an application where you need to join three tables together with one SQL query. How would you write this in SQL?

    SELECT locations.name AS location_name,
      locations.street_address,
      locations.city,
      locations.state,
      users.title,
      users.first_name,
      users.last_name,
      rooms.id,
      rooms.name AS room_name,
      rooms.room_number,
      rooms.location_id
      FROM locations
      INNER JOIN users
      ON users.location_id = locations.id
      INNER JOIN rooms
      ON rooms.location_id = locations.id
      WHERE locations.name ILIKE '%Empire%'
      AND users.last_name ILIKE '%Empire%';

This is a mouthful but what is going on is pretty straightforward; still a few questions come to mind: how do you turn it into an active record object, how to you make the search dynamic, and how do you prevent SQL injection.

The answer to the first question is to wrap the SQL statement string so ActiveRecord will know how to understand it.

    ActiveRecord::Base.
      connection.
      execute(
      <<-SQL
        SELECT locations.name AS location_name,
        locations.street_address,
        locations.city,
        locations.state,
        users.title,
        users.first_name,
        users.last_name,
        rooms.id,
        rooms.name AS room_name,
        rooms.room_number,
        rooms.location_id
        FROM locations
        INNER JOIN users
        ON users.location_id = locations.id
        INNER JOIN rooms
        ON rooms.location_id = locations.id
        WHERE locations.name ILIKE '%Empire%'
        AND users.last_name ILIKE '%Empire%';
      SQL
      )

This will execute it and return an ActiveRecord Object, but it will not be dynamic. You can just make it dynamic with `?`, but it will be vulnerable to SQL injection, which is not something you [want to do](http://xkcd.com/327/), so you will have to escape it.

    def sql_query
      ActiveRecord::Base.
        connection.
        execute(
          sanitized_sql_statement
        )
    end

    def sanitized_sql_statement
      ActiveRecord::Base.send(
        :sanitize_sql_array,
        [
          sql_string,
          "%#{query}%",
          "%#{query}%"
        ]
      )
    end

    def sql_statement
      <<-SQL
        SELECT locations.name AS location_name,
          locations.street_address,
          locations.city,
          locations.state,
          users.title,
          users.first_name,
          users.last_name,
          rooms.id,
          rooms.name AS room_name,
          rooms.room_number,
          rooms.location_id
          FROM locations
          INNER JOIN users
          ON users.location_id = locations.id
          INNER JOIN rooms
          ON rooms.location_id = locations.id
          WHERE locations.name ILIKE ?
          AND users.last_name ILIKE ?;
      SQL
    end

Of course you can rewrite this in ActiveRecord, but it might not seem as intuitive as the SQL statement.

    Location.joins(:users, :rooms).select(
      "locations.name AS location_name",
      "locations.street_address",
      "locations.city",
      "locations.state",
      "users.title",
      "users.first_name",
      "users.last_name",
      "rooms.id",
      "rooms.name AS room_name",
      "rooms.room_number",
      "rooms.location_id"
    ).where(
      "locations.name ILIKE ? users.last_name ILIKE ?",
      "%#{query}%",
      "%#{query}%
    )

Here you can see that locations is the main table that you are joining with the other tables and you are only returning certain columns. If you did `select("*")` you would get everything, including the users encrypted password; you are then performing the search query. One of the main problems I had with this was to know exactly when each particular method was supposed to be called. When you call `SELECT` in SQL it has to be the first statement but in ActiveRecord you are chaining the methods together (and doing what looks like a violation of the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)), and the ordering of the methods matter so ActiveRecord can properly construct the SQL query. 
