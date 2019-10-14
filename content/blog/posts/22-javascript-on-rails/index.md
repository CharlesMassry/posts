---
id: 22
date: "2014-07-24"
title: "Javascript on Rails"
---
The Javascript syntax is not nearly as fun as Ruby's but you must use it if you want to do anything dynamic on the client-side. Fortunately, Ruby on Rails comes with some built in Javascript helpers, such as easy AJAX support.

AJAX, Asynchronous Javascript and XML, is a way to send and receive data from the server without refreshing the page. This can be very useful depending on what the client wants to do, such as post a comment to the bottom of the page which would normally require a page refresh, sending you back to the top. Programming this can be quite difficult but as luck would have it, Ruby on Rails comes prepackaged with jQuery, the premier Javascript library.

To easily AJAXify your web app with Rails, first you must add `remote:
true` to your form tag, then you must make your controller respond to JSON and render a Javascript/ERB view. It sounds like a few steps but there is only 4 lines of code that need to be added to get it working.

If you want to write more complex web apps like one that dynamically renders an edit form based on user click position, you must write Javascript and use the jQuery library yourself as opposed to just having Rails use it in the background. You need to create a `PATCH` request using the `$.ajax(...);` method. You would also have to render the form using jQuery and you can get rid of the form with the AJAX success callback.