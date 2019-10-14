---
id: 52
date: "2015-01-13"
title: "Caching Rails Views"
---
"There are only two hard things in Computer Science: cache invalidation, naming things, and off-by-one errors." Fortunately for us, Rails makes [caching](http://guides.rubyonrails.org/caching_with_rails.html) quite simple.

In your log files you might notice that the views take a lot longer to process than you might hope, but you can cache them easily. To get started in `config/development.rb` add the line `config.action_controller.perform_caching = true`.

Now, you can cache any ActiveRecord partial and it automatically expires when the `updated_at` column changes.

    <% cache post do %>
      <%= div_for post do %>
        <h3><%= link_to post.title, post %></h3>
        <p><%= truncate(post.text, length: 160, separator: '.') %></p>
        <p><%= display_time(post) %> on <% post. tags.each do |tag| %>
          <%= link_to capitalize_tag(tag.name), tag_path(tag.name) %>
        <% end %>
        <%= post.comments_count %>
      <% end %>
    <% end %>

Because `#comments_count` is part of an association, we must update the comments model so the post gets updated whenever anyone comments on it.

    class Comment < ActiveRecord::Base
      belongs_to :post, counter_cache: true, touch: true
    end

By adding `touch: true` whenever anyone comments on the post, the post will get updated.

You can even add caches to static fragment views easily like say a navigation bar partial. Just wrap it in a `cache` block. You can specify which version of the cache you want as the argument, `<% cache 'v1' do %>`.

One downside of this fragment caching is that when the partial gets updated it will take longer to render the first time because it must write the file to the filesystem and then read it, but every subsequent view will be significantly faster. For something that gets updated quite frequently, consider foregoing fragment caching, as this performance hit on updating the cache can be too big.

When you are finished with this feature, go ahead and set `config.action_controller.perform_caching = false` in `config/development.rb`. It is automatically true on production, but can be a pain during development. With view caching, your performance can be greatly improved by having the views already generated, and you'll get a really nice boost in performance.