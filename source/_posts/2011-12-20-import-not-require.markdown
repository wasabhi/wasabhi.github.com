---
layout: post
title: "import not require"
date: 2011-12-20 15:09
comments: true
categories: [ruby, rake]
---

Stumbled across something small but useful.  Writing rake files for a Ruby gem?
Then consider arranging them using the following conventions:

* place them under lib/tasks
* ensure they all end with the .rake extension

How to ensure that all your rake files under lib/tasks are available when you run a `rake -T`?

Easy - put this into your Rakefile.

```ruby
Dir.glob('lib/tasks/*.rake').each { |r| import r }
```

Some things to note:

* the .rake extension means that you can't use require
* require works with files using the .rb extension
* import is your friend in this case
