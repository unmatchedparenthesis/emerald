---
layout: post
title: Input validation with clojure.spec
---

I have written recently some simple REST-based application. With ring and compojure it's really nice and easy. However, most tutorials I found completely ignores the need of input validation.

In my research on "The Clojure way of data validation" I stumbled upon an [answer on StackOverflow](http://stackoverflow.com/questions/14542707/is-there-a-smart-way-to-validate-function-input-in-clojure/14543185#14543185) in which author claims that _the Clojure attitude is "just assume you got the right thing"_. It seems to be valid for most cases when we are not leaving Clojure code. Yet I feel that it's better to check what kind of input is sent by potentially unknown REST client.

It looks that the new feature announced recently can help me with my struggle. Of course I'm talking about `clojure.spec`.

## Adding `clojure.spec` to project

The new module will be availiable in Clojure 1.9 which is currently in alpha. Yet, if you prefer to use stable version (1.8) you can still profit on this terrific tool. Just add `spec-backport` dependency to your project:


{% highlight clojure %}
[clojure-future-spec "1.9.0-alpha13"]
{% endhighlight %}

(or some [more recent version](https://clojars.org/search?q=clojure-future-spec) if it exists)

## Writting specification

Let's write some specs for validation user record. It should have some login, password and email. Login and password should be of restricted lenght and email should match generic email regex.

First of all we need to add `require` statement into namespace definition:

{% highlight clojure %}
(ns user
  (:require [clojure.spec :as s]))
{% endhighlight %}

Now we can define specs for login and password using `s/def` macro:

{% highlight clojure %}
(s/def ::login (s/and string? #(< 5 (count %) 15)))
(s/def ::password (s/and string? #(< 8 (count %) 50)))
{% endhighlight %}

Double collons in specs names means that those are namespace-scoped keywords. `s/def` registers specs in global registry therefore those names should be scoped (at least that's how I understand it).

`s/and` is a simple macro that combines multiple predicates and creates a spec that is requires all of this predicates to be fullfiled.

Email is not that much complicated. We will just use a regex match as one of conditions:

{% highlight clojure %}
(def email-regex
  #"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,63}$")
(s/def ::email (s/and string? #(re-matches email-regex %)))
{% endhighlight %}

Now we can combine these specs to create a bigger one for user object:

{% highlight clojure %}
(s/def ::user (s/keys :req-un [::login ::password ::email]))
{% endhighlight %}

I'd prefere user object to have keys as unscoped (single-colon) keywords therefore I used `:req-un` instead of `:req`.

## Validating object

Now that we have our specs ready we can run them against actual objects:

{% highlight clojure %}
(s/valid? ::user
  {:login "hello" :password "foobarbaz" :email "hello@world.com"}) ; true
(s/valid? ::user
  {:login "hello" :password "foobarbaz"}) ; false - missing email
(s/valid? ::user
  {:login "hello" :password "pass" :email "hello@world.com"}) ; false - password too short
{% endhighlight %}

In actual program we can use `assert` function to throw exception in case of invalid object or write our own validation error handler (maybe even using information provided by `s/conform`).

## Further reading

This was of course simple example for more details please read [official spec guide](http://clojure.org/guides/spec) which gives more examples on validating different data.
For general overview and rationale behind this module refer to [this article](http://clojure.org/about/spec).
