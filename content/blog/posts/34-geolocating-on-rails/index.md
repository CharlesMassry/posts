---
id: 34
date: "2014-09-23"
title: "Geolocating on Rails"
---
In this day in age, 2014, websites using location services are becoming more and more common to better serve customers as they can provide a quicker, cleaner user interface. But how is geolocation integrated into a Rails app? Well there are a couple of steps, first you must get the latitude and longitude coordinates as computers need these to be able to plot on a map, almost like the [Cartesian Coordinate System](http://en.wikipedia.org/wiki/Cartesian_coordinate_system) which you may or may not remember as the x y graph from Algebra in grade school. To turn an address into longitude and latitude coordinates, there's a gem for that; I like the [Geocoder](http://www.rubygeocoder.com) gem. The Geocoder gem is simple to set up and has good documentation. To begin, in your Gemfile

    gem "geocoder"

Then run

    $ bundle install

The next part might be a little tricky as Geocoder depends on the database to store latitude and longitude coordinates, so you must create columns in your database as such. You can do this fairly easily with a migration

  

    $ rails generate migration AddLocationDataToSchools latitude:float longitude:float

When you check the migration that is generated, you will get exactly what you want, just replace `Schools` in `AddLocationDataToSchools` to whatever your model name is; isn't Rails awesome! Next, you must add it to your database, so run

    $ rake db:migrate

Now that you have the database set up you can set it to work automatically, so in your model

    geocoded_by :address
    after_validation :geocode, if: :address_changed?

Now, your address will be converted to latitude and longitude coordinates after the record is created. A couple of things to note, you are limited in your number of requests per day for geolocating, also you must be connected to the Internet, which can be a pain during development. Stay tuned for tips on how to display a map with these coordinates. 