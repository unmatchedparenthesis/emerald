---
layout: post
title: Reading Clojure code
---

Clojure syntax might seem bizzare at first glance if you are not LISP programmer. The amount of parentheses is overwhelming. One can think that the code has more brackets than actual code. And what's the fuzz with this prefix notation.

Let's start with simple function call in some C-like language:

{% highlight c %}
movePointerTo(100, 150);
{% endhighlight %}

It's as simple as can be, right? But do we really need that semicolon at the end. It's purpose is to separate statements, but which sane programmer writes the code like below?

{% highlight c %}
int answer = 42; println("The answer to the Ultimate Question of Life, the Universe and Everthing is:"); println(answer);
{% endhighlight %}

In fact many languages got rid of semicolons (e.g. Scala or Groovy). In Clojure you don't use semicolons to separate statements but as a start of single line comment. However, we all know that if there is comment in the code that there is something fishy. So let's write better code and less comments and forget about semicolons for most of the time.

It turns out Clojure does not care about commas too. They are just another whitespace characters for compliler so most of the time we skip them.

So after removing useless punctation marks our code is still readable and has the very same meaning:

{% highlight c %}
movePointerTo(100 150)
{% endhighlight %}

Now just move opening bracket to the very beginning of the statement and you got valid Clojure code

{% highlight c %}
(movePointerTo 100 150)
{% endhighlight %}

Everything put in parentheses forms a list in Clojure. And list can be evaluated. It means that Cloure interpreter takes the first element of list and executes it as a function. All the other list elements are passed to this function as an argument. Easy, right?

Whenever you read Clojure code and you see `(` you can assume that the next thing after it is a function.

Let's take a look on the example young Clojure programmers are threthened with:

{% highlight c %}
(+ 2 3)
{% endhighlight %}

I hope it's not that scarry anymore. You remember that the first element of the list is a function and it is executed with arguments `2` and `3`. Nothing crazy here. Just syntax consistency. The C-like languages are the odd ones since they once use prefix notation (for functions) and the other time infix notation (for mathematical operations). Clojure (as all Lisps) does it in one way every time.

