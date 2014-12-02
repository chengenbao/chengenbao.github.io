---
layout: post
title: Block in ruby language
---

One of the most useful functions in ruby programming language I think is block. With block
you can simply and beautifully write coding as following.

{% highlight ruby %}
items = ["hello", "world"]
items.each do |item|
  puts item
end

# you can easily define custom sort aglorithm as following
# sort by the length of the string

items = "In delay there lies no plenty Then come kiss me sweet and" + 
  " twenty Youths a stuff that will not endure William Shakespeare" +
  " British dramatist"

items = items.split ' '
items.sort do |a, b|
  a.length <=> b.length
end
{% endhighlight %}

As you see above, it really makes life easy with the ruby block. If you have dove into ruby,
you will find most of the Core and STD library function can support block injection, you can always
call the library function exactly as above, whitch means that you can always extend the ability of
the original library function, so now you can imagine how powerful of this can be.

what is block
-------------
A block in ruby is actually a code block, which is a object internally. Block is the most common
way to implement closure.

{% highlight ruby %}
class Array
  def reverse!(&code)
    i = 0

    while i < self.length / 2
      tmp = self[i]
      self[i] = code.call(self[self.length - 1 - i])
      self[self.length - 1 - i] = code.call(tmp)
      i += 1
    end
  end
end

array = [3, 7, 12, 15]
array.reverse!

array .each do |item|
  puts item
end
{% endhighlight %}

how to use block
----------------
When you define your function you can alway add block injection in your function, you can do this as 
following.

{% highlight ruby %}
def func(arg1, arg2 &p)
  # do something with arg1 and arg2
  p.call # you can add arguments too
end

# you can call this func with no block attached, as
func(x, y)

# or, you can also call this func as
func(x, y) do |x, y|
  x * y
end
{% endhighlight %}

deferences with proc and lambda
-------------------------------
