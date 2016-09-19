---
layout: post
title: Conditional list updating
---

I was writting a program in Clojure today. And in this program I store data in memory (it's to early to integrate it with any database). I have an `atom` containing a list of objects and I need to update single object on that list.

I thought it should be quite easy and there must be some function for it in core library. Yet, I haven't found it. It would be easy if I have map instead of list because I wanted to update object with given id. Then I'd have simply to use `update` or `update-in`.

However, I decided to write list-based implementation just to test myself and have a solution for more complicated situations in future.

# Updating every object on the list

That is an easy task. We have `map` for it. So if we need to increment every number we can write:

{% highlight clojure %}
(map inc [10 5 23 665])
{% endhighlight %}

And if we want to add `:bar` to list under key `:foo` in every objec on a list it's as easy as:

{% highlight clojure %}
(map 
  #(update % :foo conj :bar) 
  [{:foo []} {:foo [:baz]}])
{% endhighlight %}

`map` does all we need, so we don't have to write any specialized function.

# Update under condition

But what if we want to change only elements satisfying certain condition? We can still use `map` and wrap transformation function in little `if`.

Then incrementation of even numbers will look like this:

{% highlight clojure %}
(map #(if (even? %) (inc %) %) [1 24 55 68])
{% endhighlight %}

It is simple example, yet we can clearly see some problems with this. Our condition and actual transformation is hidden. Let's make it more flexible and reusable by defining function:

{% highlight clojure %}
(defn update-if [pred f coll]
  (map #(if (pred %) (f %) %) coll))
{% endhighlight %}

It looks similar to previous snippet, but now we can use it many times, e.g. previous example can be rewritten to:

{% highlight cloure %}
(update-if even? inc [1 24 55 68])
{% endhighlight %}

And some more elaborate example:

{% highlight cloure %}
(update-if #(empty? (:vals %))
           #(update % :vals conj 0)
           [{:vals []} {:vals [1 2 3]}])
{% endhighlight %}

This will add `0` to every object that has empty collection under `:vals` key.

# Update only first element satisfying condition

After writing above implementations I thought about optimization. In case of my program there was at most one object satisfying the condition. Why checking every element then?

So I started to implement some fancy reflective function that will iterate over the collection. Then I recalled that it should be done with `loop` instead of recursive calls. It was ugly and has several `if`s and anonymous functions. Everything just to track whether the condition was satisfied or not yet. Anyways it didn't look like a Clojure. It was the effect of my experience with imperative languages and strange drive for optimization.

After a few tries I gave up on efficiency and decided to use `take-while` and `drop-while`:

{% highlight cloure %}
(defn update-once-if [pred f coll]
  (let [!pred (comp not pred)
        before (take-while !pred coll)
        [x & after] (drop-while !pred coll)]
  (concat before [(f x)] after)))
{% endhighlight %}

And that's it: it takes all elements not satisfying condition, then one element from the rest and the tail. All is left is apply transformation function to the single element and concat all the parts.

Best of all - it worked. It iterated twice over collection (once with `take-while` and once with `drop-while`) but I think is not a problem at the moment.

## Revelation

Do you have those 'aha' moments when you walk away of your desk? I walked away and the thought struck me: "there must be a function in core library for splitting collection". I charged back to the computer and checked the docs. In fact there is a function tailored for my needs: `split-with`. Now I could write more efficient version of my function:

{% highlight cloure %}
(defn update-once-if [pred f coll]
  (let [!pred (comp not pred)
        [before [x & after]] (split-with !pred coll)]
    (concat before [(f x)] after)))
{% endhighlight %}


# Lesson learned

It's another situation when the solution turned out to be easier than it looked at first. I hava a feeling that when something is hard to write in Clojure, I'm using wrong approach or wrong data structure.

Oh, and it's one more prove that premature optimization is the root of all evil.

