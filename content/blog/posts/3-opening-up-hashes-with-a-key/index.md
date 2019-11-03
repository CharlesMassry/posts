---
id: 3
date: "2014-06-20"
title: "Opening up Hashes with a Key"
---
In Ruby, Hashes are a very powerful class that can store a key value pair. This is very useful if you need to create a database as you make one object point to a different type of object and every time you enter the first object, otherwise known as the key, you get the object it points to, otherwise known as the value.

  `db["Taylor Swift"] = ["You Belong With Me"]` 

To create a Hash.

 `db["Taylor Swift"] << "I Knew You Were Trouble"` 

This adds the song "I Knew You Were Trouble" to the value which holds an array.

    db["Taylor Swift"]

This returns an array with the string "You Belong With Me", and "I Knew You Were Trouble".

Then you can easily print out an array of Taylor Swift songs, or really whatever you want to store. Pretty powerful stuff. Also, if you're using a CSV file, make sure to put `require 'csv'` at the top.