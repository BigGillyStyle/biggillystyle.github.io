There are plenty of cases in my Rails applications when I'm sorting through data that I can't sort with just a good database query (which is almost always the best option when possible).  As such, I've become more interested in "algorithmic efficiency".  In my recent work on [exercism.io](http://exercism.io) I came across a programming challenge to do a simple string substituion.  My solution at its core looked like this (`DNA_TO_RNA` is a hash that maps one char to another):

{% highlight ruby %}
  dna_str.gsub(/[#{DNA_TO_RNA.keys.join}]/, DNA_TO_RNA)
{% endhighlight %}

...however, as I perused other students' solutions, I noticed that a more common approach was this...

{% highlight ruby %}
  dna_str.each_char.map {|char| DNA_TO_RNA[char]}.join
{% endhighlight %}

I was curious about either of the two was a better performing algorithm.  Truthfully I liked the readability of the latter solution, as my solution would require many Rubyists to look up [String#gsub](http://www.ruby-doc.org/core-2.1.4/String.html#method-i-gsub) to see why/how this worked.  In fact, I only discovered this technique while reading through the Ruby docs as part of this exercise.

Here is the benchmark that I ran:

{% highlight ruby linenos %}
Benchmark.ips do |x|
  x.report('dna_convert_with_gsub') do |times|
    i = 0
    while i < times
      Complement.of_dna('ACGTGGTCTTAA')
      i += 1
    end
  end

  x.report('dna_convert_with_map_join') do |times|
    i = 0
    while i < times
      Complement.of_dna2('ACGTGGTCTTAA')
      i += 1
    end
  end

  x.compare!
end
{% endhighlight %}

...and here are the benchmark results:

    Calculating -------------------------------------
    dna_convert_with_gsub
                              6199 i/100ms
    dna_convert_with_map_join
                             14988 i/100ms
    -------------------------------------------------
    dna_convert_with_gsub
                            70749.2 (±4.8%) i/s -     353343 in   5.006677s
    dna_convert_with_map_join
                           197282.1 (±3.2%) i/s -     989208 in   5.019425s

    Comparison:
    dna_convert_with_map_join:   197282.1 i/s
    dna_convert_with_gsub:    70749.2 i/s - 2.79x slower

I was somewhat expecting that gsub might be better than iterating over each character of the string, but alas...the map/join technique is noticeably faster.  I didn't try out various string lengths, so I imagine that could be a factor.  I did run multiple benchmarks, and all produced nearly identical results.  Of note...this was run using Ruby 2.1.4.