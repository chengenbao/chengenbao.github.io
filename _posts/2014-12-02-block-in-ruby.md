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
A block in ruby is actually a code block. Block is the most common way to implement closure.

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
def func(arg1, arg2, &p)
  # do something with arg1 and arg2

  if block_given?
    p.call # you can add arguments too
  end
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
Block is code block, Proc is an object, there is only an block parameters in the argument list, but
can be many Proc or Lambda. Block can be considered as an instance of Proc.

Lambda will check the parameter, but Proc not. So all you should remember is, use Block and Lambda, but
not Proc.

{% highlight ruby %}
# block Examples 
[1,2,3].each { |x| puts x*2 } # block is in between the curly braces
[1,2,3].each do |x| puts x*2 # block is everything between the do and end end

# Proc Examples
p = Proc.new { |x| puts x*2 } 
[1,2,3].each(&p) # The '&' tells ruby to turn the proc into a block 
proc = Proc.new { puts "Hello World" } 
proc.call # The body of the Proc object gets executed when called 

# Lambda Examples 
lam = lambda { |x| puts x*2 } 
[1,2,3].each(&lam) 
lam = lambda { puts "Hello World" } 
lam.call

# parameter transfer 
lam = lambda { |x| puts x } # creates a lambda that takes 1 argument
lam.call(2) # prints out 2 
lam.call # ArgumentError: wrong number of arguments (0 for 1) 
lam.call(1,2,3) # ArgumentError: wrong number of arguments (3 for 1) 

proc = Proc.new { |x| puts x } # creates a proc that takes 1 argument
proc.call(2) # prints out 2
proc.call # returns nil 
proc.call(1,2,3) # prints out 1 and forgets about the extra arguments

# Return behavor
# in lambda, return jump out lambda，continue runing the code outside of lambda

def lambda_test 
  lam = lambda { return } 
  lam.call puts "Hello world" 
end 
lambda_test # calling lambda_test prints 'Hello World'

def proc_test 
#in proc, return jump out directly
  proc = Proc.new { return } 
  proc.call 
  puts "Hello world"
end 
proc_test # calling proc_test prints nothing
{% endhighlight%}
