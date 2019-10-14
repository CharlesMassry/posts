---
id: 28
date: "2014-08-14"
title: "Multiple Images with PaperClip"
---
For any given web application, you probably want your users to be able to embed pictures or other types of multimedia for a richer experience. This can be difficult to do as you **NEVER** want to store files in the database (this is one of those rare absolute cases where there are no exceptions). Fortunately for you Ruby on Rails developers, there's a gem for that. [Paperclip](https://www.github.com/thoughtbot/paperclip), made by [thoughtbot](http://thoughtbot.com), makes it easy to store files in your file system (or AWS if you want) and simply have the database store the path to the file. Another benefit of Paperclip is it integrates very nicely with [imagemagick](http://www.imagemagick.org/), so you can process uploaded files with relative ease. If you aren't sold on Paperclip yet, I should probably tell you that Paperclip is really simple to get up and running thanks to the robust documentation. While the documentation doesn't go into detail about having a model that has many attachments, it can also be done with relative ease.

If the main product of your website needs multiple photos, you can create a new table that just has the foreign key of your main product. For example, if the main product is `Listing`, and you want each `Listing` to have many photos, you can create a new table called `photos` and make sure it has a `listing_id` in your migration. In the listing model, you can add `has_many :photos` and you are ready to add attachments to photos with Paperclip, so simply read Paperclip documentation and you are good to go.

One thing of note, the location that Paperclip stores files is in the `public/system/:class/:attachment/:id_partition/:style` folder, where the colons represent variables. You may want to add one of these subdirectories to your `.gitignore` file if you are using git as it will upload all your development files to your git server, such as Github.