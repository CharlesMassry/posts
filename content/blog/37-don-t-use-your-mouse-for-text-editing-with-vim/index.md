---
id: 37
date: "2014-10-06"
title: "Don't Use Your Mouse for Text Editing with Vim"
---
Sublime Text is a very popular editor, but you must use your mouse at times, which takes some time away from your hands being at the keyboard, thus slowing you down. [Vim](http://www.vim.org) solves this problem and so much more. As you do not use your mouse, Vim has a steep learning curve, to get started with vim

    $ vimtutor

This introduces you to the basic movement commands, like moving around with `h`, `j`, `k`, `l`, or moving forward or backwards in words with `e` and `b` and combine these commands with numbers so you'll DRY up your keystrokes. It also shows you the basics of entering insert mode to enter characters into the buffer, visual mode, which allows you to select characters, and replace mode, where you can replace the character your cursor is over with your keypress.  

Once you have the basics down, you'll realize you can combine motions, for example `ciw`(change in word) lets you start typing and when you press `ESC` the word you typed replaces the word the cursor was over. Another cool command is `A`, which brings you into insert mode at the end of the line, and `I`, which brings you into insert mode at the beginning of the line.

Once you get a little more advanced, you might want to create macros that run shell commands for you. For example, I'd like a command that run all my tests for me in a Rails project. I use [Spring](https://github.com/rails/spring) so my Rails environment doesn't have to start up every time I run RSpec, so I would just type in Vim when I want to run my tests

    :!spring rspec

This can get annoying as that is 14 keystrokes not including `SHIFT`. Fortunately you can write macros using VimScript. While not nearly as elegant as Ruby, VimScript gets the job done easily enough. In your .vimrc file

    function RunSpringRSpec()
      execute "!spring rspec"
    endfunction
    map <Leader>s :call RunSpringRSpec()<CR>

`function` declares a new function and its name is `RunSpringRSpec` and the parentheses are necessary. It executes `!spring rspec`, the `!` is to execute terminal commands in vim and anything after that gets executed. After the function is declared, `<LEADER>s` is mapped to the function. `<LEADER>` is whatever key you designate as the `<LEADER>` key, `\\` by default. Every time I press `\\s` the function gets executed. `<CR>` or carriage return is simply the enter key. Once you get the hang of it, Vim is very intuitive, robust, text editor.