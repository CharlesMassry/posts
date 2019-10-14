---
id: 1
date: "2013-12-22"
title: "Welcome."
---
I built this blog to showcase problems, solutions and thoughts about web development, web design and technology. That being said I did run into a number of problems and their solutions while actually building this site. 

In my attempts to make this site as responsive as possible I ran into a few problems, such as the browser caching the webpage dependencies, the scroll effect for the blog posts not being responsive, and having a side navigation turn into a top navigation bar on smaller devices.

I also ran into some problems on the back end such as setting up my virtual private server for hosting, converting my SQLite database to a MySQL database for production, and finally running the website using Nginx.

Rails 4 implemented turbolinks.js by default, which is good if you don’t have any differences in CSS or Javascript throughout your website. I ran into this problem because my front page style is drastically different from the rest of my pages and didn’t realize it at first because Google Chrome automatically didn’t cache the dependencies. Once I tested it on multiple browsers I realized something was afoot. I then figured out that it was [turbolinks.js](https://github.com/rails/turbolinks) and read the documentation which is a simple HTML attribute, “data-no-turbolink,” in any ancestor element of that particular link. 

The scroll effect was not designed for full page usage, so I had to change the height of the list. I then discovered that the CSS property height default is set to the document height and not the window height. This means that to make the height of the document the height of the window you need to use Javascript. Using [jQuery](http://jquery.com/) makes this much easier to do, but I had to get the height of the window which was converted to both a string and a float and needed to be an integer. Once I did this everything fell into place. 

When I originally designed the site I wanted a side navigation panel that spanned the length of the window, however I realized that this practice did not lend itself to responsive web design which is necessary in this day in age. I figured the easiest way to do this was to use [Bootstrap](http://getbootstrap.com/), specifically a horizontal navigation. This didn’t work when I tried to make my own side navigation panel and then switch to the Bootstrap navigation bar for smaller screens. What I ended up doing was changing the Bootstrap navigation bar to the side navigation panel that I wanted, which worked except the links were still in the original top navigation bar position. This was fixed with a simple jQuery class change and some CSS modification. 

Once I was done with these issues I then moved on to launching this website for production. This proved more difficult than I thought. I set up a virtual private server with [Digital Ocean](https://www.digitalocean.com/), and installed all the necessary dependencies until it came time to set up a proxy server and decided to go with [Nginx](http://nginx.org/) with [Passenger](https://www.phusionpassenger.com/). My virtual private server wouldn’t let me set it up at first because there was not enough memory, and I needed to create a swapfile, which is to use hard drive space as RAM. This didn’t work as the commands I gave my swapfile to change the priority proved useless. I gave up for the day and checked back the next and it just worked without even a swapfile. 

[MySQL](http://www.mysql.com/) is a very powerful database system that I wanted to use for the database for this site. For Ruby on Rails, the gem is called mysql2, however the documentation is not ideal as it doesn’t say how to set up for production. The problem I had was within the database.yml file options. I looked around for the answer and couldn’t find one until I consulted the [Agile Web Development with Rails 4](http://pragprog.com/book/rails4/agile-web-development-with-rails-4), which gave very detailed information on how to setup MySQL for production while still using SQLite for developing and specifically how to edit the database.yml file. 

My last issue was with Nginx and how to set up the domain name, specifically how to make it run on port 80 while the Nginx was running on port 80, with a cryptic warning that further configuration was required. Once I figured out what the issue was, it was simply a matter of closing Nginx and starting passenger on port 80 as Nginx was already configured. 

In a future update I would like to rewrite the CSS in SASS as I haven’t discovered it until after this site was launched and so far, it looks very organized and intuitive. Stay tuned…