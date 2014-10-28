---
layout: post
title: 'Ruby Meta: Module, Class, and Method in One'
date: '2014-10-18 08:42:43'
---

I'm not a huge fan of using "excessive" Ruby metaprogramming on a development team.  First, I'm just not as competent with metaprogramming as I'd like to be.  More importantly, though, I find it to be "easily readable" (an important criteria for team-based coding, in my opinion) only for the original author.  Anyone else has to slowly read the code and think to themselves, "Now what is that doing?".  Anyway, I came up a scenario in which I needed to write 10 classes that varied in a very small way.  Here's the non-meta class:

    module Pricing
      module Fixed

        class Hour1
          def hours
            1
          end
        end
      
      end
    end

Yep, that's it.  And there needed to be Hour2, Hour3, Hour4, ... classes all the way up to Hour10.  To this point in my Ruby career I had "defined methods" but never classes (and classes within module).  After a bunch of Googling here's my solution:

    module Pricing
      module Fixed

        (1..10).each do |num|
          klass_name = "Hour#{num}"
          klass = Class.new(Pricing::ProjectEstimate) do
            def hours
              self.class.name.sub('Pricing::Fixed::Hour','').to_i
            end
          end
          const_set(klass_name, klass)
        end

      end
    end

In my Rails app I just put that in a config/initializers file, so that the classes would be defined right upon startup.  I'll examine some of the details...

`Class.new(Pricing::ProjectEstimate)` creates a new class and makes it a subclass of `Pricing::ProjectEstimate`.  The `hours` method is fairly self-explanatory.  I think I could've just as easily used `define_method` instead and then been able to use the `num` iterator, but I didn't think of that at the time.  Finally, the `const_set(klass_name, klass)` sets the generated class to be within the module I've defined at the top of the file (`Pricing::Fixed`).  It's a method on `Object`, and that was actually one of the 'trickier' parts for me, because all the examples I found on the internet showed using it with `Object`, but I didn't want my generated classes to be namespaced to the 'root' namespace.

There you have it.  Comments welcome.