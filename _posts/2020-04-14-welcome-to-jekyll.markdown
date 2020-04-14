---
layout: post
title:  "Variable 'x' of type referenced from scope, but it is not defined."
date:   2020-04-14 20:53:30 +0200
categories: dotnet csharp
---
Expressions Trees give us an ability to write executable code different way than used to but sometimes you can end up at the very beggining while trying out. Have a look at a given example below.

{% highlight java %}
Expression<Func<Car, bool>> isRed = x => x.Color == "Red";
Expression<Func<Car, bool>> isCheap = x => x.Price < 10.0;
Expression<Func<Car, bool>> isRedOrCheap = Expression.Lambda<Func<Car, bool>>(
    Expression.Or(isRed.Body, isCheap.Body), isRed.Parameters.Single());
var compiled = isRedOrCheap.Compile();
{% endhighlight %}

We have two functions checking its single property of a car but suddenly a business requirement changed and a developer is asked to check both of the properties at once. Looks simple and fine at first sight but results in a runtime error.

{% highlight java %}
variable 'x' of type referenced from scope, but it is not defined
{% endhighlight %}

Of course the developer could've solved that business requirement by reimplementing whole code to

{% highlight java %}
Func<Car, bool> isRed = x => x.Color == "Red";
Func<Car, bool> isCheap = x => x.Price < 10.0;
Func<Car, bool> isRedOrCheap = x => isRed(x) || isCheap(x);
{% endhighlight %}

and runtime would not have complained, but let's stick with a fact that he had no choice and I also have a good reason due to upcoming blog posts.

The problem is that 'x' parameter of 'isRed' expression function is not the same as 'x' parameter of 'isCheap' one. Therefore, Expression Trees do not compare parameters by their names, but references. Another problem is that we are not able to fix it without introducing more then 10 lines of code.
