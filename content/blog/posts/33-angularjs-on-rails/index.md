---
id: 33
date: "2014-09-16"
title: "AngularJS on Rails"
---
Recently, I have started looking into AngularJS, a popular front-end Javascript MVC, to help me write cleaner, better, front-end code. [AngularJS](https://angularjs.org/), developed by [Google](https://www.google.com/), enhances your HTML by providing useful HTML attributes, data-binding, and helpful methods to pull it all together. 

To get started on an existing Rails project, you should create a JSON feed, I like to use [jbuilder](https://github.com/rails/jbuilder). For example, if your controller looks like:

    class PlayerStatsController < ApplicationController
      ...
      def show
        team = find_team
        @player_stat_column = team.player_stat_columns
        @players = team.players
      end
      ...
    end

Your jbuilder file would look like:

    json.player_stat_column @player_stat_column.names

    json.players @players do |player|
      json.email player.email
      json.stats player.player_stat.statistics
    end

This would create a JSON page at whatever the URL is, plus `.json`. The JSON page would look then render something that looks like this:

    {
      player_stat_column: [
        "completions",
        "attempts",
        "goals"
      ],
      players: [
        {
          email: "me@charliemassry.com",
          stats: {
            completions: 77,
            attempts: 90,
            goals: 24
          }
        },
        {
          email: "something1@charliemassry.com",
          stats: []
        }
      ]
    }

Now you have your data, but how do you pass that into the view using AngularJS. Well first, you must require the AngularJS file by typing

    <%= javascript_include_tag "https://ajax.googleapis.com/ajax/libs/angularjs/1.2.24/angular.min.js" %>

in either `application.html.erb` or typing in

    <% content_for :angular do %>
      <%= javascript_include_tag "https://ajax.googleapis.com/ajax/libs/angularjs/1.2.24/angular.min.js" %>
    <% end %>

in the current file and then adding `<%= yield :angular %>` in the `application.html.erb` file. In both cases, you must require AngularJS before the standard `javascript_include_tag`. One problem with using a "Content Delivery Network" to include the AngularJS library is if you need to work on the project offline, say on a train or such, it won't work so you will have to manually include the project in your application.

Now that you have AngularJS and you JSON feed set up you can finally begin. First, you need to create the scope of the AngularJS app, so if you are not creating an entire AngularJS app and it will be local to one particular site simply make a div tag that holds the entire partial and pass in the attribute `ng-app` and create a module name for it.

    <div ng-app="playerStats"></div>

Next, you must require the AngularJS module, so in a Javascript file, let's call it `main.js` and let's put it in `app/assets/javascripts/`. Let's also make sure to require it before the rest of the Javascript files. So in your `application.js` file

    ...
    //= require main
    //= require tree .

Now that that's out of the way we can write our module file, for now it is just a one liner in the `main.js` file.

    var playerStats = angular.module('playerStats', []);

Now we can move on to our controller.  

What I'd like to happen is take our JSON feed and display it on the page and map the players' stats to the column names. This can be done fairly easily with AngularJS. First you must scope the controller to a particular part of a page. So now your page should look like

    <div ng-app="playerStats">
      <div ng-controller="playerStatsController"></div>
    </div>

Second, create a controller, we'll put it in `app/assets/javascripts/angular/controllers`, and we will call it `playerStatsController.js`.

    playerStats.controller("playerStatsController", ["$scope", "$http", function($scope, $http){
      var path = location.pathname + ".json";
      $http.get(path).success(function(data){
      $scope.columns = data.player_stat_column;
      $scope.players = data.players;
      });
    }]);

What is happening in this controller is, its name is being assigned and then it is passed in two variables that it has access to, `$scope` and `$http`. `$scope` allows you to pass variables to the view for easy data-binding, and `$http` allows you to access other websites. Within the controller, the url plus `.json` is being assigned to the `path` variable to retrieve data from the JSON feed we created earlier. Once it retrieves the data, it then assigns the Javascript objects to the `$scope` so the view will have access to them.

So we will change the view to loop through each of the `player` objects and show their stats in the view.

    <div ng-app="playerStats">
      <div ng-controller="playerStatsController">
        <h3>Player Statsheet</h3>
        player <span ng-repeat="column in columns">{{column}} </span>
        <div ng-repeat="player in players">
          {{player.email}} <span ng-repeat="column in columns">{{player.stats[column]}} </span>
        </div>
      </div> 
    </div>

First, using the `ng-repeat` attribute, it is looping over all the columns. Calling it columns is important, so it knows where to grab the variable from the scope. It will repeat each column and print it out to the document using the `{{}}` AngularJS tags. It then loops through each player and calls each column name in the stat and prints out the result using a nested loop. And that's all there is to getting started with AngularJS on Rails. Stay tuned for a future update on some more advanced functionality.