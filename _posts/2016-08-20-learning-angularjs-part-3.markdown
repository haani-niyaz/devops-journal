---
layout: post
title:  "Learning AngularJS: Part III"
date:   2016-08-20
author: "Haani Niyaz"
tags: angularjs javascript
---


## Prerequisites

This article is a continuation of [Learning AngularJS: Part II](https://haani-niyaz.github.io/devops-journal/learning-angularjs-part-2).	


### Topics Covered

* Scope
* Data Binding
* Directives


## Scope

Now that we've had a taste of AngularJS, its worth introducing the concept of *scope*. Angular scope holds your data and liases with the *controllers* and gives the *views* (what you see) everything it needs. We saw this in [Part 1](https://haani-niyaz.github.io/devops-journal/learning-angularjs-part-2) when we set  the `ng-app` directive. This was the *application* scope - the html page it is declared in.

Another scope we came across was the `ng-controller` directive. Each controller has its own scope. When we set a controller witin a HTML element, the *Controller* is also accessible to all nested child elements.


Here's an example to help the idea sink in.


{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html ng-app="store">
<head>

</head>
<body>
	<div ng-controller="SimpleController">
		<textarea ng-model="text"></textarea>
	</div>	 

	<script type="text/javascript" src="js/angular.min.js"></script>	
	<script type="text/javascript" src="js/store.js"></script>

	<script>
		(function(){
			function SimpleController($scope){
				$scope.text = 'This is not confusing at all!'
			}
		})();

	</script>
</body>
</html>
{% endraw %}
{% endhighlight %}


In the above snippet, Angular will look for a function with the name of the *controller*. In this instance `SimpleController`. The `ng-model` directive set in the `textarea` will display the value of `$scope.text`.

Notice how we didn't do:

{% highlight html %}
{% raw %}
<div ng-controller="SimpleController as simple">
{% endraw %}
{% endhighlight %}

If we had, we would have resorted to using the following syntax:

{% highlight javascript %}
{% raw %}
(function(){
	function SimpleController($scope){
		this.text = 'This is not confusing at all!'
	}
})();
{% endraw %}
{% endhighlight %}


In a nutshell, `$scope.var` is the old syntax but its worth knowing as it seems to be ubiquitous.

See a detailed answer [here](http://stackoverflow.com/a/19940503/2180697). 


### What is the ngModel directive?

The `ng-model` directive stores/updates the value of the input field into/from a variable.


{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html ng-app="store">
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title></title>
	<link rel="stylesheet" href="">
</head>
<body >
	<form name="reviewForm">
	<p>{{review}}</p>
	<div class="form-group">
	  <textarea class="form-control" placeholder="Leave a Review!" ng-model="review"></textarea>
	</div>
	</form>
	<script type="text/javascript" src="js/angular.min.js"></script>	
	<script type="text/javascript" src="js/store.js"></script>
</body>
</html>
{% endraw %}
{% endhighlight %}

The example code above illustrates how the binding takes place between the `textarea` and the `ng-model` directive. You will find that you when type into the `textarea` a live preview will appear within the `p` tags.


With the above code we can only show the text as the user types it in - nothing more than a parlour trick, really. What we need to do is store the data. We won't technically store the data in this article but lay the ground work to do. 

In order to capture the data we need to pass it to a *Controller*. Let set that up.


{% highlight javascript %}
{% raw %}
// store.js

(function(){ 
	var app = angular.module('store',[]);

	var books = [
		{
			name: 'Deep Work',
			price: 9,
			desc: 'Rules for focued success in a distracted world',
			buy: false,
			soldOut: false,
			reviews: []
		},
		{
			name: 'The One Thing',
			price: 9.99,
			desc: 'The Surprisingly Simple Truth Behind Extraordinary Results',
			buy: false,
			soldOut: false,
			reviews: []
		}		
	]

	app.controller('StoreController',function(){
		this.products = books;	
	});


...

	app.controller('ReviewController', function(){
		this.review = {};

		this.addReview = function(product){
			product.reviews.push(this.review);
			// Clear form when submit button is hit
			this.review = {};
		}
	});

})();
{% endraw %}
{% endhighlight %}


The `addReview` method expects the `product` object as an argument. We previously added the `books` array of objects to the instance `products`. In `index.html` we iterated through the `products` as follows:


{% highlight html %}
{% raw %}
<div ng-repeat="product in store.products">	
{% endraw %}
{% endhighlight %}

Each product has a `reviews` array which we will need to update. More on the this method later.

Now if we modify our html, it will look like the following:


{% highlight html %}
{% raw %}

...

<div ng-repeat="product in store.products">	
 ...
<form name="reviewForm" ng-controller="ReviewController as reviewCtrl" ng-submit="reviewCtrl.addReview(product)">
		<!-- Live preview -->
		<blockquote>
			<b>{{reviewCtrl.review.stars}} Star(s)</b>
			<p>{{reviewCtrl.review.body}}</p>
			<p>
				<cite>- {{reviewCtrl.review.author}}</cite>
			</p>
		</blockquote>

		<div class="form-group">
			<select ng-model="reviewCtrl.review.stars" ng-options="stars for stars in [5,4,3,2,1]">
			</select>	
		</div>
		<div class="form-group">
			<textarea class="form-control" placeholder="Leave a Review!" ng-model="reviewCtrl.review.body"></textarea>
		</div>
		<div class="form-group">
			<input class="form-control" type="email" placeholder="email address" ng-model="reviewCtrl.review.author" />
		</div>
		<input type="submit" value="Submit">
	</form>
 ...
</div>

{% endraw %}
{% endhighlight %}


Now the things to watch out for: 

{% highlight html %}
{% raw %}
<form name="reviewForm" ng-controller="ReviewController as reviewCtrl" ng-submit="reviewCtrl.addReview(product)">
{% endraw %}
{% endhighlight %}

The above code initializes the controller as `reviewCtrl`. We also see there's an `ng-submit` directive to trigger the `addReview` method on form submit.

You will also see the `ng-model` directive for all the inputs; *stars*, *body* and *author*. 

More importantly to note here is the [scope](#scope) we discussed before.  Angular will look for a function with the name of the *controller*. In this instance `ReviewController`. The `ng-model` directive set in all the user input fields will display the value of `this.review.<property>`. 

In pure js fashion, when you do somethng like `reviewCtrl.review.body` you are dynamically adding the `body` property to `this.review` object. This is also the case for `stars` and `author`.

If we look back at the `addReview` method:

{% highlight javascript %}
{% raw %}
this.addReview = function(product){
		product.reviews.push(this.review);
		// Clear form when submit button is hit
		this.review = {};
	}
{% endraw %}
{% endhighlight %}

You will notice that we're appending the review `stars`, `body` and `author` to the the product's `reviews` array.


## Summary

In this article we looked at using the `ng-model` directive with an emphasis on how scoping works.


#### Learning Resources:

* [Conceptual Overview with an Example](https://docs.angularjs.org/guide/concepts)
* [Codeschool Shaping Up with Angular](http://campus.codeschool.com/courses/shaping-up-with-angular-js)
* [API Reference](https://docs.angularjs.org/api)
* [Should I use this or $scope?](http://stackoverflow.com/a/19940503/2180697)


#### Repo:
* [GitHub Learning AngularJS Part 3](https://github.com/haani-niyaz/learning-angularjs/tree/part-3)
