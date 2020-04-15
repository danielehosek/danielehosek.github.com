---
layout: post
title:  "Parametrized Table Valued Function using Dapper avoiding SQL Injection"
date:   2020-04-15 19:00:00 +0200
categories: [csharp]
author: Daniele Hosek
---
Using SQL Parameters does not only help readability, it also **helps to prevent SQL Injection**. What you typically do is binding key-value pairs of input parameters using `DynamicParameters` object of `Dapper` namespace in case of classic SQL SELECT which results in `SELECT * FROM [Table] WHERE [Column] = @Value`. Actually, this does not apply for Table Valued Function since it does not have the already mentioned key-value pairs, **but a value**. There is not an article or a tutorial on the internet explaining how to execute Table Valued Function with an input parameter without openning a door for the devil called SQL Injection.  

{% highlight csharp %}
var searchParameter = "value"; // a value obtained by a user via UI
connection.Query($"SELECT * FROM [dbo].[TableValuedFunction]('{searchParameter}')");
{% endhighlight %}

This is what I have already seen. No matter where and when, but I have. Not only does it loose prevention of SQL Injection, **but also transparency of an input data type because of quotation marks**. 

{% highlight csharp %}
var searchParameter = "value"; // a value obtained by a user via UI

var command = connection.CreateCommand();
command.CommandText = $"SELECT * FROM [dbo].[TableValuedFunction](@Value)";

var parameter = command.CreateParameter();
parameter.ParameterName = "@Value";
parameter.Value = searchParameter;
parameter.DbType = DbType.String;

command.Parameters.Add(parameter);
command.ExecuteReader();
{% endhighlight %}

This can be solved by creating `IDbDataParameter` bound to `IDbCommand` guaranteeing type transparency and SQL Prevention.
