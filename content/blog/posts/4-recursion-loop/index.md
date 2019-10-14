---
id: 4
date: "2014-06-23"
title: "Recursion != Loop"
---

I used to think that this statement would evaluate to true, until very recently, which is the reason for this blog post, and hopefully this will help those who don’t quite get it yet. Last week I started going to [Metis](http://www.thisismetis.com), a Ruby On Rails Bootcamp in New York City, it’s run by the popular web development company [thoughtbot](http://www.thoughtbot.com). So I was a little ahead of the other students as I have a little programming experience with Ruby and was able to complete the pre-work without too much trouble. One of the instructors suggested I try and implement “quicksort” in Ruby.

Quicksort is a sorting algorithm that takes a random number, called a pivot, in an array and makes sure that every other number in an array is either in a higher or lower position than the pivot. The way to do this is recursion. The problem was I kept getting an error message “Stack Level Too Deep” which is the error message if there is no exit condition on recursion, similar to an infinite loop. This was the first clue that they were different.

Having trouble, the instructor told me to think of recursion as a series of nested method calls.

    quicksort(quicksort(quicksort(quicksort(array))))

If it hits a return statement, it then starts breaking out of the nest till it’s gone. It nests itself, unlike a loop, which doesn’t add any layers dynamically.