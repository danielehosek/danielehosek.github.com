---
layout: post
title:  "InvalidOperationException: Variable 'x' of type referenced from scope, but it is not defined."
date:   2020-04-14 20:53:30 +0200
categories: [csharp]
---
Expressions Trees give us an ability to write executable code different way than used to but sometimes you can end up at the very beginning while trying out. Have a look at a given example below.

{% highlight java %}
Expression<Func<Car, bool>> isRed = x => x.Color == "Red";
Expression<Func<Car, bool>> isCheap = x => x.Price < 1000.0;
Expression<Func<Car, bool>> isRedOrCheap = Expression.Lambda<Func<Car, bool>>(Expression.Or(isRed.Body, isCheap.Body), isRedParameters.Single());
var compiled = isRedOrCheap.Compile();
{% endhighlight %}

We have two functions, `isRed` and `isCheap`, checking properties of a car but suddenly a business requirement is changed and a developer is asked to check both of the properties at once, therefore one expression `isRedOrCheap` function combining both and accepting `x` parameter of `isRed` (`isRed.Parameters.Single())`) is introduced and compiled. Looks simple and fine at first sight but results in **a runtime error**.

{% highlight java %}
InvalidOperationException: variable 'x' of type referenced from scope, but it is not defined
{% endhighlight %}

Of course, the developer could've solved the business requirement by reimplementing whole code into something like this

{% highlight java %}
Func<Car, bool> isRed = x => x.Color == "Red";
Func<Car, bool> isCheap = x => x.Price < 1000.0;
Func<Car, bool> isRedOrCheap = x => isRed(x) || isCheap(x);
{% endhighlight %}

or even simpler..

{% highlight java %}
Func<Car, bool> isRedOrCheap = x => x.Color == "Red" || x.Price < 1000.0;
{% endhighlight %}

and runtime wouldn't have thrown an exception, but let's stick with a fact that he had no choice.

But back to the exception - the problem is that `x` parameter of `isRed` *(which was actually that one we used for constructing)* is not the same as `x` parameter of `isCheap`. **Expression Trees** do not **compare parameters by** their names or types, but **references** - they are completely different in this context.

This can be fixed many ways and one of them is implementing a visitor pattern overriding a method of an abstract class named `ExpressionVisitor` of `System.Linq.Expressions` namespace.

{% highlight java %}
public class ParameterExpressionVisitor : ExpressionVisitor
{
    readonly ParameterExpression newOne;
	
    public ParameterExpressionVisitor(ParameterExpression newOne)
    {
        this.newOne = newOne;
    }

    protected override Expression VisitParameter(ParameterExpression _) // not used
    {
        return newOne;
    }
}
{% endhighlight %}

We implemented the visitor which did nothing, but **always returned a replacement given by the constructor**. Now let's update the caller and use what we've done.

{% highlight java %}
Expression<Func<Car, bool>> isRedOrCheap = Or(isRed, isCheap); // update
var compiled = isRedOrCheap.Compile();
var result = compiled(new Car("Red", 55555.0)); // true

public static Expression<Func<Car, bool>> Or(Expression<Func<Car, bool>> a, Expression<Func<Car, bool>> b)
{
    var parameter = a.Parameters.Single(); // the same as 'isRed.Parameters.Single()'
    var visitor = new ParameterExpressionVisitor(parameter);
    var updated = visitor.Visit(b.Body); // visit 'isCheap' and change its parameter
    var body = Expression.Or(a.Body, updated);
    return Expression.Lambda<Func<Car, bool>>(body, parameter);
}
{% endhighlight %}

Now we can run it without any troubles or the runtime error since both expression functions have the same reference of the `x` parameter. We take the `x` parameter and its reference of `isRed` and reuse it for `isCheap`.