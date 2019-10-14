---
id: 35
date: "2014-09-29"
title: "Google Maps on Rails"
---
Previously, I had blogged about how to set up your website to make requests to match string addresses to latitude and longitude coordinates. I had [wrote about](/posts/34) how to use the [Geocoder](http://www.rubygeocoder.com) gem to accomplish this task. Now, to pull it all together with [Google Maps](https://developers.google.com/maps/web/). Getting started is simple, first I recommend getting an API key from Google Maps. I also recommend installing the [dotenv-rails](https://github.com/bkeepers/dotenv) gem for easy API key management. Now simply create a `.env` file in the root of your Rails directory and enter your Google Maps API key in it like

    GOOGLE_MAPS_API_KEY=your_api_key

Now, in the file where you want to display the map, you would want the files in your `<head>` tag so to easily do that in rails you can add a `content_for` erb tag which takes a block like

    <% content_for :google_maps do %>
      ...
    <% end %>

In `layouts/application` add `<%= yield :google_maps %>` to above the standard `javascript_include_tag`.

Inside this block, you'll want to add the Google Maps logic and styling to get the map working. You'll need to have a starting point, how deep of a zoom, and optionally markers that display location of specific items. As for the actual `<body>` tag, you'll need to set a div with a particular id and reference that in your javascript to display the map at that part of the page like `<div id="map-canvas"></div>`.

You'll want to add something for the style so it displays, like

    html, body, #map-canvas {
      height: 100%; margin: 0; padding: 0;
    }

And for your Javascript

    var latLng = new google.maps.LatLng(
      <%= @current_location.latitude %>,
      <%= @current_location.latitude %>
    );
    var mapOptions = {
      center: latLng,
      zoom: 12
    };

    var map = new google.maps.Map(
      document.getElementById('map-canvas'),
      mapOptions
    );

    setMarkers(map, locations);

A few things of note, you can get the user's current location a few ways, one of which is to get the request object and call location on it like `request.location`. The request object is an instance of the `ActionDispatch::Request` class that has a lot of data about the request such as the IP address and the path, as well as a bunch of constants like what url the request was coming from. If you have the `pry-rails` gem installed, you can simple add `binding.pry` into your controller, refresh the page, go into your terminal and type in `request` to see the entire object. Additionally, Geocoder adds a location method to this object which looks up the object based on the IP address, which is an expensive lookup but is not obtrusive to the user. Also it won't work in development as `request.location` will return `[0.0, 0.0]` because it is coming from `127.0.0.1` or `localhost`. It will put you at those coordinates which is where the Equator and the Prime Meridian intersect; off the coast of Africa in the Atlantic Ocean.

Another thing to note is the `locations` variable that I show being passed into the `setMarkers` function. Ideally this locations variable is valid JSON and `setMarkers` iterates through the JSON feed.

    function setMarkers(map, locations) {
      for (var i = 0; i < locations.length; i++) {
        var location = locations[i];
        var locationLatLng = new google.maps.LatLng(location[1], location[2]);

        var marker = new google.maps.Marker({
          position: locationLatLng,
          map: map,
          title: location[0],
          zIndex: location[3]
        });

        var contentString = '<a href="/locations/' + location[4] + '">' + location[0] + '</a>'

        var infoWindow = new google.maps.InfoWindow({
          content: contentString
        });

        google.maps.event.addListener(marker, 'click', function(){
          infoWindow.open(map, marker);
        });

        marker.setMap(map);
      };
    }

You can the nearby locations using the Geocoder method `#near` which takes two arguments, one of which is where it originates from and the other one is the number of miles in the radius. So in your controller you might have

    def index
      @current_location = request.location
      @locations = Location.near(@current_location.coordinates, 15)
    end

Once you have all of this set up you can call

    google.maps.event.addDomListener(window, 'load', initialize);

and your all set.