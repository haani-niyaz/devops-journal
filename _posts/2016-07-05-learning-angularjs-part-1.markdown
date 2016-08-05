---
layout: post
title:  "Learning AngularJS: Part I"
date:   2016-07-05
author: "Haani Niyaz"
tags: 
 - angularjs 
 - javascript
---


## Objective

Lets learn AngularJS! 

I am looking to build an interactive dashboard and my research tells me that Angular would be a good fit. I aim to run this as a series of articles as I skill up from scratch (I have decided to 'blog as I learn' as I'm interested to know if it increases my capacity to grasp concepts quickly and improve my retention rate. So these articles will be posted hot off the press!)

### Topics Covered

* Modules
* Directives
* Expressions
* Controllers
* Filters



## Why choose AngularJS?
* A framework to keep your javascript code organized
* Create responsive web application (No more page refresh!)

### What does a 'Responsive Application' really mean?

Traditionally on every single page load, a request is sent to the web server via the browser and the server responds with a web page and assets. When a new request (e.g: click) is initiated, the browser sends a request to the web server and yet again receives the entire web page and assets.

If we contrast this to a responsive application, on the inital request the entire web page and assets are loaded. However when a new request is initated, the request will be to retreive a specific set of data. The web server subsequently responds with the data (JSON object) and the browser will load this data into the existing web page.  



## Prerequisites

I came from the following background:

* Moderate understanding Javascript, HTML & CSS
* An undertanding of MVC 
* Experience with using a web framework

### Scaffolding

The template for the `index.html` is shown below. All HTML code snippets to come will be within the `body` tag.

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
	<head>
		<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous">
		<meta charset="UTF-8">
		<title>AngularJS Basics</title>
	</head>
	<body>	
		<script type="text/javascript" src="js/angular.min.js"></script>	
		<script type="text/javascript" src="js/store.js"></script>
	</body>
</html>
{% endhighlight %}


Similarly the javascript files are located in a `js` folder and all code snippets within the `store.js` file. You will need to make the `angular.min.js` file available in the same directory or use a CDN link instead.



## Concepts


### Modules

* Makes angularjs apps more maintanable 
* Allows for better testing
* Enables modules to be reused (depedencies are defined in Modules)

#### How to Write a Module

{% highlight javascript %}
var app = angular.module('store',[]);
{% endhighlight %}

The module name `store` is the Angular application name. The empty array `[]` supplied as an argument will include the dependencies (if any). 



### Directives

A *directive* is like an attribute on a DOM element which attaches a special behaviour to it. 

For example, the following directive needs to be added so that the defined Angular module `store` is run when the document loads:


{% highlight html %}
<html ng-app="store">
{% endhighlight %}

This will treat the HTML on the page as part of the angular app.



### Expressions

Now that we've created the directive to bind the HTML to the angular app, we can insert dynamic values into the HTML via *expressions* as shown below:

{% highlight html %}
{% raw %}
<p>My age is {{ 20 + 12 }}</p>
{% endraw %}
{% endhighlight %}



### Controllers

Controllers allow javascript objects to be transported to the HTML page. They are also where we define our applications behaviour. This might all sounds like an over simplification but I do think it drives the point home.

{% highlight javascript %}

// Initialize angular module
var app = angular.module('store',[]);

// Javascript object
var book = {
		name: 'Deep Work',
		price: 9.99,
		Decription: 'Rules for focued success in a distracted world'
	}

// Define controller
app.controller('StoreController',function(){
	this.product = book
});

{% endhighlight %}

Here I created a javascript object called `book` which we will access in our HTML page in a moment.

I then proceed to define our controller `StoreController` which is attached to our Angular `app` object defined earlier.


#### How do we access this object in our HTML?

By defining the `ng-controller` directive we can attach the controller to the HTML page as shown below:

{% highlight html %}
{% raw %}
<!-- store is an alias to StoreController -->
<div ng-controller="StoreController as store">
	<h1>{{store.product.name}}</h1>
	<h2>${{store.product.price}}</h2>
	<h3>{{store.product.desc}}</h3>	
</div>
{% endraw %}
{% endhighlight %}


**Note:** the the controller object is only accessible inside the `div` element.



### Filters

Filters are used to format *expressions*. An expression is piped `'|'` into filters. In the example below we pipe the expression into the `uppercase` filter (pretty self explantory):

{% highlight html %}
{% raw %}
<div ng-controller="StoreController as store">
	<h1>{{store.product.name | uppercase }}</h1>
</div>
{% endraw %}
{% endhighlight %}

### Examples

Now that we have a foundational understanding of *modules*, *directives* and *expressions*. Lets take a look at some functional examples.


#### Using Expressions inside a Directive

{% highlight html %}
{% raw %}
<!-- store is an alias to StoreController -->
<div ng-controller="StoreController as store" ng-hide="store.product.soldOut">
	<h1>{{store.product.name}}</h1>
	<h2>{{store.product.price | currency}}</h2>
	<h3>{{store.product.desc}}</h3>	
</div>
{% endraw %}
{% endhighlight %}

the `ng-hide` directive will hide the book details if the `soldOut` attribute is `true`. Conversely, there is a `ng-show` directive which does the opposite. Its important that these directives are either set to `truthy` or `falsy` values.

#### Iterating through multiple objects

Lets add another book and also make the `StoreController` instance variable plural.

{% highlight javascript %}
var books = [
		{
			name: 'Deep Work',
			price: 9.99,
			desc: 'Rules for focued success in a distracted world',
			buy: false,
			soldOut: false
		},
		{
			name: 'The One Thing',
			price: 9.99,
			desc: 'The Surprisingly Simple Truth Behind Extraordinary Results',
			buy: false,
			soldOut: false
		}		
	]

app.controller('StoreController',function(){
	this.products = books;	
});
{% endhighlight %}


Now inside our HTML we can use **directives** to iterate over multiple objects.

{% highlight html %}
{% raw %}
<body ng-controller="StoreController as store">
	<div ng-repeat="product in store.products">
		<h1>{{product.name}}</h1>
		<h2>{{product.price | currency}}</h2>
		<h3>{{product.desc}}</h3>	
	<button ng-show="store.product.buy">Add to Bookshelf</button>
	</div>
{% endraw %}
{% endhighlight %}


## Summary

In this introductory aritcle we looked at some Angular concepts and specific examples to illustrate how they all work together.

#### Learning Resources:

* [Codeschool Shaping Up with Angular](http://campus.codeschool.com/courses/shaping-up-with-angular-js)
* [API Reference](https://docs.angularjs.org/api)

#### Repo:
* [GitHub Learning AngularJS Part 1](https://github.com/haani-niyaz/learning-angularjs/tree/part-1)

