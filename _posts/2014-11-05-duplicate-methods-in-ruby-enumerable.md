---
title: One Reason for Duplicate Methods in Ruby's Enumerable
---

I've often wondered if there was a *good* reason for all the duplicate methods in Ruby core classes, especially in the [Enumerable](http://ruby-doc.org/core-2.1.4/Enumerable.html) module.  I'm talking about `map`/`collect` and `reduce`/`inject`...just to name a couple.  I've come across some people who really have a preference of one over the other, while others don't really care (I'd put myself in the latter camp).  Coming from a Java background and being somewhat familiar with Hadoop and map-reduce I tend to use those two variants.  Well, for the first time today it turned out quite handy that there is a duplication between `find` and `detect`.

On my current Rails project we're using MongoDB with the [Mongoid](http://mongoid.org/en/mongoid/index.html) gem.  One of the nice features of Mongoid is that when you create a query (which are Mongoid Criteria objects), they are lazily evaluated (no DB hit) until records are actually needed.  I was doing something conceptually similar to this...

{% highlight ruby %}
User.where(first_name: 'John').find {|user| user.plays_football?}
{% endhighlight %}

Now the more experienced among you may already see my folly, but alas the problem actually cost me about an hour of hair-pulling time.  The problem?  While Criteria includes the Enumerable module, it **also** defines its own `find` method.  Of course...duh...I use `User.find '123456789'` and the like in Pry all the time, but that fact just escaped me at the time.  In this particular instance I was needing to get all the records back from the query and then 'find' the first record that met some criteria that did not fit into a database query.  The solution?  `detect`

That was one of those "feather in the cap" moments that'll probably stick in my brain for quite awhile.  Hopefully if Google serves up this page for you, I've saved you a little frustration and/or time.