---
layout: post
title:  "Learning AngularJS: Part II"
date:   2016-07-07
author: "Haani Niyaz"
tags: angularjs javascript
---

## Objective

So its time we look at a functional example of all of the concepts we covered in the previous article.


## Prerequisites

This article is a continuation of [Learning AngularJS: Part I]({% post_url site.baseurl/2016-07-05-learning-angularjs-part-1 %})


## Prevously..

In [Part I]({% post_url site.baseurl/2016-07-05-learning-angularjs-part-1 %}), the output looked like the following:

{% highlight html %}
Deep Work
$9.00
Rules for focued success in a distracted world
The One Thing
$9.99
The Surprisingly Simple Truth Behind Extraordinary Results
{% endhighlight %}


Lets look at presenting the `price` and `description` in tabs per Book.


### Scaffolding

I have modified the `body` section from the `index.html` page to give us another template:

{% highlight html %}
{% raw %}
<body ng-controller="StoreController as store">
	<div ng-repeat="product in store.products">	
		<ul class="nav nav-pills">
			<div class="panel"> `
				<h4>Name</h4>
				<p>{{product.name}}</p>
			</div>

	  		<li role="presentation">
	  			<a href>Price</a>
	  		</li>
	  		<li role="presentation">
	  			<a href>Description</a>
	  		</li>
		</ul>

		<div class="panel">
			<h4>Price</h4>
			<p>{{product.price | currency}}</p>
		</div>

		<div class="panel">
			<h4>Description</h4>
			<p>{{product.desc}}</p>
		</div>

	</div>
</body>
{% endraw %}
{% endhighlight %}



## Requirements

Lets create a checklist of what we want done:

1. The page should show the price per book (default tab)
2. The **price** tab should be actively selected
3. The **description** panel should be hidden until **description** tab is clicked.
4. When we click on any tab it should be active and the the respective panel should be displayed.



## Approach

As per the bootstrap documentation the class `active` will allow us make the *price* tab active by default.

{% highlight html %}
<li role="presentation" class="active"><a href="#">Price</a></li>
{% endhighlight %}

Each *tab* and *panel* will share a value. When a tab is clicked, a value will be assigned. The assigned value will determine the corresponding panel to display.

* The `ng-click` directive will allow us to set the tab value i.e: *price* will be 1, *description* will be 2.

* We are already familiar with the `ng-show` directive to show/hide the panels.



### Tab Controller

We have been talking about *tabs* and its behaviour for quite a bit now - This seems like a good candidate for a *Controller*.


{% highlight javascript %}
// File: store.js

...

app.controller('TabController',function(){
		
		// Assign default value 
		this.tab = 1;

		// Set the tab value
		this.selectTab = function (setTab){
			this.tab = setTab;
		};

		// Check if value set and selected matches
		this.isSelected = function(checkTab){
			return this.tab == checkTab;
		}

	});
{% endhighlight %}


* By default the `tab` value is 1. So the *price* tab is selected by default. 

* We will pass in a value when we click on a tab. The `selectTab` method basically sets the `tab` value. 

* The `isSelected` method will be used to determine which panel to show. As discussed earlier, the `ng-show` directive expects a `true` or `false` (return type of the `isSelected` method). This method is also used to set the active tab via the `ng-class` directive which we haven't discussed yet (to come!).

### Piecing It All Together 

Firstly, we need to attach the `TabController` to the HTML. We will insert a `section` to do this as shown below:


{% highlight html %}
...	
<div ng-repeat="product in store.products">	
	<section ng-controller="TabController as tab">
	...
	</section>
</div>
...
{% endhighlight %}


Now, Lets set the tab value when a tab is clicked by using `ng-click`.


{% highlight html %}
{% raw %}

<div ng-repeat="product in store.products">	
	<section ng-controller="TabController as tab">
		<ul class="nav nav-pills">
			
			<div class="panel">
				<h4>Name</h4>
				<p>{{product.name}}</p>
			</div>

	  		<li role="presentation">
	  			<a href ng-click="tab.selectTab(1)">Price</a>
	  		</li>
	  		<li role="presentation">
	  			<a href ng-click="tab.selectTab(2)">Description</a>
	  		</li>

		</ul>
	</section>
</div>

{% endraw %}
{% endhighlight %}


Next, lets make sure the panel is displayed when the tab is clicked by using the `ng-show` directive: 

{% highlight html %}
{% raw %}

<div ng-repeat="product in store.products">	
	<section ng-controller="TabController as tab">
		...
		<!-- ng-show expects 'true' or 'false' -->
		<div class="panel" ng-show="tab.isSelected(1)">
			<h4>Price</h4>
			<p>{{product.price | currency}}</p>
		</div>

		<!-- ng-show expects 'true' or 'false' -->
		<div class="panel" ng-show="tab.isSelected(2)">
			<h4>Description</h4>
			<p>{{product.desc}}</p>
		</div>		
	</section>
</div>

{% endraw %}
{% endhighlight %}


Lastly, we need to make sure the tab is active. As previously mentioned we need to add the `active` class when the tab is clicked. For this we use the `ng-class` directive as an *expression*.


{% highlight html %}
{% raw %}

<div ng-repeat="product in store.products">	
	<section ng-controller="TabController as tab">
		<ul class="nav nav-pills">
			
				<div class="panel">
					<h4>Name</h4>
					<p>{{product.name}}</p>
				</div>

		  		<li role="presentation">
		  		    <!-- class='active' is added if the expression evaluates to true -->
		  			<li ng-class="{active:tab.isSelected(1)}">
		  				<a href ng-click="tab.selectTab(1)">Price</a>
		  			</li>
		  		</li>
		  		<li role="presentation">
		  		 	<!-- class='active' is aded if the expression evaluates to true -->
		  			<li ng-class="{active:tab.isSelected(2)}">
		  				<a href ng-click="tab.selectTab(2)">Description</a>
		  			</li>
		  		</li>
			</ul>
	</section>
</div>

{% endraw %}
{% endhighlight %}


For those of you who didn't quite get it, here's the break down:

{% highlight html %}
{% raw %}
<li ng-class="{active:tab.isSelected(1)}">
{% endraw %}
{% endhighlight %}

If `tab.isSelected(1)` returns `true`, add the class `active` to the list element. This logic is repeated for all `ng-class` directives.


### Double vs. Single vs. No Curly Braces

This is explained quite well [here](http://stackoverflow.com/questions/17878560/difference-between-double-and-single-curly-brace-in-angular-js)



## Summary



#### Learning Resources:

* [Codeschool Shaping Up with Angular](http://campus.codeschool.com/courses/shaping-up-with-angular-js)
* [API Reference](https://docs.angularjs.org/api)
* [The many ways to use ngclass](https://scotch.io/tutorials/the-many-ways-to-use-ngclass)

#### Repo

* [GitHub Learning AngularJS Part 2](https://github.com/haani-niyaz/learning-angularjs/tree/part-2)
