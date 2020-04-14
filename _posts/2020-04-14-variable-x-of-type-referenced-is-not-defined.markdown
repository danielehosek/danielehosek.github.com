---
layout: post
title:  "InvalidOperationException: Variable 'x' of type referenced from scope, but it is not defined."
date:   2020-04-14 20:53:30 +0200
categories: dotnet csharp
---
Expressions Trees give us an ability to write executable code different way than used to but sometimes you can end up at the very beggining while trying out. Have a look at a given example below.

{% highlight java %}
Expression<Func<Car, bool>> isRed = x => x.Color == "Red";
Expression<Func<Car, bool>> isCheap = x => x.Price < 1000.0;
Expression<Func<Car, bool>> isRedOrCheap = Expression.Lambda<Func<Car, bool>>(
    Expression.Or(isRed.Body, isCheap.Body), isRed.Parameters.Single());
var compiled = isRedOrCheap.Compile();
{% endhighlight %}

We have two functions checking properties of a car but suddenly a business requirement is changed and a developer is asked to check both of the properties at once. Looks simple and fine at first sight but results in a runtime error.

{% highlight java %}
InvalidOperationException: variable 'x' of type referenced from scope, but it is not defined
{% endhighlight %}

Of course the developer could've solved that business requirement by reimplementing whole code into something like this

{% highlight java %}
Func<Car, bool> isRed = x => x.Color == "Red";
Func<Car, bool> isCheap = x => x.Price < 1000.0;
Func<Car, bool> isRedOrCheap = x => isRed(x) || isCheap(x);
{% endhighlight %}

or even simpler..

{% highlight java %}
Func<Car, bool> isRedOrCheap = x => x.Color == "Red" || x.Price < 1000.0;
{% endhighlight %}

and runtime wouldn't have thrown an exception, but let's stick with a fact that he has no choice and I also have good reasons due to **upcoming blog posts** using this example.

The problem is that `x` parameter of `Expression<Func<Car, bool>> isRed` is not the same as `x` parameter of `Expression<Func<Car, bool>> isCheap`. **Expression Trees** do not **compare parameters by** their names or types, but **references**.

We can fix it by implementing a visitor pattern overriding a method of an abstract class named `ExpressionVisitor` of `System.Linq.Expressions` namespace.
