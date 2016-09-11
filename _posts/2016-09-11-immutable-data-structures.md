---
layout: post
title: Immutable data structures
---

If you have some expirience with functional languages the concept of immutable data is not new to you. However, if you come from object-oriented world this approach might not be that familiar.

Let's start with an example in Java. We can create a list and add some values to it:

{% highlight java %}
List<Integer> lst = new ArrayList<>();
lst.add(1);
lst.add(2);
lst.add(9000);
{% endhighlight %}

In the result we have variable `lst` which is a list containing three values: 1, 2 and 9000. What will happen if we ingenuously rewrite this example in Clojure repl? Let's try it, just use vector instead of list.

{% highlight clojure %}
(def lst [])
(conj lst 1)
(conj lst 2)
(conj lst 9000)
{% endhighlight %}

Guess what is now under `lst` symbol. It's empty vector. _But why, what happend to all my precious data?_, you'll ask. Just lookup documentation of `conj` function (btw. you can do it from repl: `(doc conj)`). It says that `conj returns a new collection`. Yep, the original one stored under `lst` symbol stays intact. This is the way it works most of the time in Clojure, e.g. when you try to add something to hash-map:

{% highlight clojure %}
(def some-map {:key "value"})
(assoc some-map :new-key "brand new value")
{% endhighlight %}

And the same happens when you remove from collection with `filter` or transform the values with `map` function.

_Ok, great. Functions returns new structures with all old data and all new values on it without changing original object. But isn't it memory inefficient?_

It turns out it is not. Those structures has only new data and the reference to the old structure. It is safe because this structure cannot be changed by any means.
