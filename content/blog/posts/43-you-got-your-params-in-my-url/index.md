---
id: 43
date: "2014-10-29"
title: "You Got Your Params in My URL"
---
Oftentimes as Rails developers, we will run into a situation where we are making something polymorphic, and we won't want to make any more controllers than models. I had described in an [earlier post](/posts/21) to get around this by parsing the URL with a Regular Expression. This can get messy and confusing, so I want to show you an alternative. The popular URL helper method `#button_to` as well as `#link_to` can take a bunch of options, and when specified with a hash, passes that into the params hash, which is stored in the URL.

    button_to "Like", like_path(likeable_type: "image", likeable_id: 1)

This would produce a path like `/likes?likeable_type=image&likeable_id=1`. While this will look messy for the client, there is no more URL parsing.

    params[:likeable_type] # => "image"
    params[:likeable_id] # => 1

All you have left to do is setup strong params with it and you're good to go.