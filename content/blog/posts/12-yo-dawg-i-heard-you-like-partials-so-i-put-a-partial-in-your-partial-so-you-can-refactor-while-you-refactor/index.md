---
id: 12
date: "2014-07-03"
title: "Yo Dawg, I Heard You Like Partials So I Put a Partial In Your Partial So You Can Refactor While You Refactor."
---
One of the founding principles of Ruby on Rails is "Don't Repeat Yourself", or DRY. There are many reasons to follow this principle, not the least of which is if you change one instance of repetition, you change the other. One way to insure this in your views is to use partials.

    <%= render form %>

This would render a file in the current directory called "\_form.html.erb". This is useful because you can use the same view for your new form as well as your edit form. But wait, there's more. Partials aren't just limited to forms, you can refactor anything, just have it's name start with an underscore and call it with the render method in your view. You can then render another partial in your partial so you can have a partial inside of your partial.

    <%= render "errors", object: form.object %>
    <div>
      <%= form.label :name %>
      <%= form.text_field :name %>
    </div>

This would first look in the current directory for a "\_errors.html.erb" file, and then it would look in the "app/views/application" directory. Once found, it will render that form's errors with the failed validations. So that's three layers deep for those of you keeping score at home.

    <% if object.errors.any? %>
      <p>Could not save:</p>
      <% object.errors.full_messages.each do |error| %>
        <li><%= error %></li>
      <% end %>
    <% end %>

This would be the third layer of the form and would only display if there was a failed validation. The source code can be found on [Github](https://github.com/CharlesMassry/pixtr_on_rails).