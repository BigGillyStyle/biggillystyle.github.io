---
title: 'Ruby Memoization Techniques'
---

Back in 2011 the Rails team [deprecated the ActiveSupport::Memoizable module](https://github.com/rails/rails/commit/36253916b0b788d6ded56669d37c96ed05c92c5c).  If you read through the comments on that commit you'll see that they more-or-less decided to use "basic Ruby memoization techniques" rather than continue to support that functionality.  Despite some outcry from users it indeed was removed altogether.  Of course, in the beauty of open source the functionality was "rebirthed" in at least two gems: [Memoizable](https://github.com/dkubb/memoizable) and [Memoist](https://github.com/matthewrudy/memoist).  The Rails team did provide examples of [several different memoization techniques](https://github.com/rails/rails/commit/f2c0fb32c0dce7f8da0ce446e2d2f0cba5fd44b3) that were used in replacement of the Memoizable module.

On our team we had been using the Memoist gem, because I really preferred how it separated any "business logic" in a method from the need to memoize:

```ruby
def area
  length * width
end
memoize :area
```

Very clean indeed, in my opinion.  However, because the gem made it so easy to memoize any code, I believe I stopped thinking critically about the proper use cases for memoization.  As a result, our application memory usage on Heroku was starting to get alarmingly high (for reference: at the time of this writing we are using the latest Rails 4.1.8 and Ruby 2.1.5).  There are definitely many different sources of "memory growth", but I took the time to investigate this more, remove memoization in several places, and resort to "basic Ruby memoization techniques" in most areas.  There were [some posts](https://bibwild.wordpress.com/2011/07/14/beware-of-activesupportmemoizable/) back when Memoizable was removed indicating that possibly the overhead of the Memoizable module technique was quite powerful and useful on one hand but also resulted in some negative performance on the other.  With that in mind, I thought I'd share some of the memoization techniques I uncovered that I've decided are very Ruby-like (in my opinion):

1) Memoization for a method that will not return nil or a falsey value

```ruby
def area
  @area ||= length * width
end
```

This is the most basic technique and works for methods that (1) don't take arguments and (2) don't return nil or falsey values.  This covers probably at least 50% of my use cases, if not more.

2) Memoization for a multiline method

```ruby
def foo
  @foo ||= begin
    # a calculation here
    # a method call here
    # @foo assignment here
  end
end
```

3) Memoization for a method that *may* return nil or a falsey value

```ruby
def foo
  unless defined? @foo
    # a calculation here
    # a method call here
    # @foo assignment here
  end
  @foo
end
```

4) Memoization for methods that have arguments

```ruby
def foo(bar)
  @foo ||= Hash.new do |h, key|
    h[key] = # my calculated value for this argument
  end
  @foo[bar]
end
```

Justin Weiss has [an excellent article](http://www.justinweiss.com/blog/2014/07/28/4-simple-memoization-patterns-in-ruby-and-one-gem/) that explains this technique very well and shows how you can use this for any number of arguments.

## Final Thoughts ##

I still don't love that this mixes business logic in some of my methods with memoization, because in my mind those are two separate concerns.  However, this seems to be an accepted Ruby idiom and is not that much "mental overhead" once you've seen the techniques put into practice a few times.  If you have comments or suggestions on other techniques, I'd love to hear about them in the comments.

## References ##

Very little of this article is "original thought".  I'd like to recognize all of the following links as ones I used in putting this information together:

1. [https://github.com/rails/rails/commit/36253916b0b788d6ded56669d37c96ed05c92c5c](https://github.com/rails/rails/commit/36253916b0b788d6ded56669d37c96ed05c92c5c)
2. [https://github.com/rails/rails/commit/f2c0fb32c0dce7f8da0ce446e2d2f0cba5fd44b3](https://github.com/rails/rails/commit/f2c0fb32c0dce7f8da0ce446e2d2f0cba5fd44b3)
3. [http://stackoverflow.com/questions/9132197/which-ruby-memoize-pattern-does-activesupportmemoizable-refer-to](http://stackoverflow.com/questions/9132197/which-ruby-memoize-pattern-does-activesupportmemoizable-refer-to)
4. [https://bibwild.wordpress.com/2011/07/14/beware-of-activesupportmemoizable/](https://bibwild.wordpress.com/2011/07/14/beware-of-activesupportmemoizable/)
5. [http://gavinmiller.io/2013/basics-of-ruby-memoization/](http://gavinmiller.io/2013/basics-of-ruby-memoization/)
6. [http://gavinmiller.io/2013/advanced-memoization-in-ruby/](http://gavinmiller.io/2013/advanced-memoization-in-ruby/)
7. [http://www.scottlowe.eu/blog/2011/08/06/memoize-your-ruby-methods-with-a-begin-block/](http://www.scottlowe.eu/blog/2011/08/06/memoize-your-ruby-methods-with-a-begin-block/)
8. [http://www.justinweiss.com/blog/2014/07/28/4-simple-memoization-patterns-in-ruby-and-one-gem/](http://www.justinweiss.com/blog/2014/07/28/4-simple-memoization-patterns-in-ruby-and-one-gem/)