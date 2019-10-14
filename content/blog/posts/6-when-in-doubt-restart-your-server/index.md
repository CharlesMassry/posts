---
id: 6
date: "2014-06-25"
title: "When in Doubt, Restart Your Server"
---
This line of advice should always come in handy when working with Sinatra. We learned about the popular Domain Specific Language, Sinatra, yesterday at [Metis](http://www.thisismetis.com). Often overshadowed by the behemoth that is Ruby on Rails, [Sinatra](http://www.sinatrarb.com/) is a RubyGem to start a web server quickly with syntax that looks like http requests.

There are a few issues however, to running a Sinatra application. One of which that I mentioned earlier, is if you change the main Ruby file that you start to start the server, you must restart your server, which can lead to a lot of confusing error messages. Fortunately, you can pull out a bit of logic out of your server file and edit other files without issue, but always leave restarting your server in your back pocket.

Another thing that was a little confusing was the URL parameter.

    require “sinatra”
    
    get "/galleries/:name" do
      ...
      @name = params[:name]
      ...
    end

By putting the colon in front of name, you can turn that into whatever the user inputs, which I like to think of as the URL equivalent to `gets.chomp`. You then insert that as a key to the “params hash” and set an instance variable equal to its value, and now you can extract that into other files such as embedded Ruby, or erb.

If the user were to type into the URL `localhost:4567/galleries/cats`, “cats” would be set to `@name`. This can enable you to do complex stuff, or you could just create a cat gallery.