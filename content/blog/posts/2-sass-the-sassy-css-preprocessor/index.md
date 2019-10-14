---
id: 2
date: "2014-04-23"
title: "SASS, the Sassy CSS Preprocessor"
---
I have updated the site with SASS which has greatly cleaned up my CSS code and you should use it too. When I first started designing the colors of the site, I was troubled by the number of steps necessary to change the main colors of the site for each particular use. For example, every time I wanted to change a single color, I would have to change the hexadecimal value in each spot that it is in, as opposed to either changing the value variable with [SASS](http://sass-lang.com/) or changing the spot of the variable.

    $primary-color: #E3E7F2;
    body{
     background-color: $primary-color;
     font: 14px Helvetica, Arial, sans-serif;
    }

Another, slightly less problematic issue, was using nested CSS statements. The problem was always repeating the CSS Selector. When I just simply wanted to change how something is supposed to behave when hovering the whole CSS Selector needs to be repeated.

These problems stem from repeating myself, which is against the rules of SASS, as the founders subscribe to the “Don’t Repeat Yourself” principle of programming, which I am a big proponent of myself, and am trying to implement it as much as I can.