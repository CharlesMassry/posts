---
id: 20
date: "2014-07-18"
title: "Polymorphism, Very Difficult? Or Suprisingly Simple?"
---
I have heard of a polymorphic relationship before, and it was difficult to wrap my head around but it is actually very simple, as it has similarities with inheritance in object oriented programming and is an instance of don't repeat yourself. The main problem that polymorphism solves is don't repeat yourself.

Suppose an Entity Relationship Diagram has many different models, but have one common model relationship, an image, a gallery and a group all have likes. The easiest way to express this would be to make three new models of likes that all correspond to their respective likable content. However, this would only be easiest if you don't know how to express a polymorphic relationship. The true easiest way would be to have one like model that corresponds to many different likable models.  

Alright, so how do you do this in Ruby on Rails? Simple, you make a like table that has a user\_id that corresponds to the user who did the liking, the content\_type that is the same name as the corresponding model, and the content\_id that corresponds to the primary key of the corresponding table. All you need to do next is add the appropriate ActiveRecord methods to your models and you are good to go.