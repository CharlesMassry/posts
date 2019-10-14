---
id: 5
date: "2014-06-24"
title: "The SQL is Never as Good as the Original"
---
Writing in SQL can be daunting at first, but the more you practice the more it seems like it’s its own programming language. For example, the `SELECT` statement can be confusing as to what it actually does but when you realize it is the equivalent of the return statement in many languages it started to make sense. Another thing to worry about is the columns of the table I like to think of it as a hash or dictionary, where the key is the column name and the value is what is in each row’s corresponding column.

One main problem I kept running into was using `JOIN` tables. I finally realized that the table with the foreign key is the table that gets joined to the main table.

    SELECT galleries.name, images.name
    FROM galleries
    JOIN images ON galleries.id = images.gallery_id;

This would return a seemingly new table with the gallery name linked to its corresponding image name. You can further refine the results using `WHERE` to enter a conditional and `LIKE` or `ILIKE` to search text similar to a regular expression.

Another powerful SQL is a collection of methods called “aggregate functions”. These so called functions are very similar to the functions or methods of conventional programming languages like Javascript or Ruby. These can be functions such as `SUM` which can return the sum of the column, or `COUNT` which can return how many of a particular query would be returned. One problem with these functions is if you want to return at least an additional column along with the aggregate function you must use the `GROUP BY` clause. Also, if you are returning multiple columns you must specify it in the group clause when using aggregate functions.

The most difficult problem I ran into was finding the first unique name in a join table. To do this you must use the ` SELECT DISTINCT ON(x)` statement, where x is the distinct element that you want. One note with this is your `ORDER BY` argument must match the distinct element that you passed in.