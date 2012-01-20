---
layout: post
title: "Map multiple commands in Vim"
date: 2011-12-27 19:20
comments: true
categories: [ruby, vim]
---

I've been learning a lot of new and subtle things about the Ruby language by
working through Ruby Koans.  Things get a little tedious as you work through
each step, having to run `ruby path_to_enlightenment.rb` every time you want to
assess your progress.

I use the vim editor, and to speed things up a little I mapped this little
command to write the file I am working with and run the above command.

``` vim
    :map ,r :w\|!ruby path_to_enlightenment.rb<CR>
```

Nice - while working on one of the Ruby Koans, this allows me to use two quick keystrokes ',r' to assess my progress.
