---
layout: post
title:  "Hot Cup of Javascript"
date:   2016-08-22
author: "Haani Niyaz"
tags:  javascript
---


* TOC
{:toc}


## Objective

In this post we'll look at bootstrapping our Javascript knowledge by covering essential concepts. 


## Functions

In Javascript functions can be defined in different ways and sometimes it can drive you mad. So this is an attempt to make some sense out of it.

Functions are [first class objects](http://helephant.com/2008/08/19/functions-are-first-class-objects-in-javascript/) in Javascript.

> This means that javascript functions are just a special type of object that can do all the things that regular objects can do.


### Function as a Declaration

{% highlight javascript %}
function add(a,b){
  return a+b;
}
num = add(1,2);
console.log(num);  // 3
{% endhighlight %}



### Function as an Expression

Here we use an anonymous function and assign it to the `add` variable. However, you cannot recursively call an anonymous function but you can with a *named* function (We'll look at this next).


{% highlight javascript %}
var add = function(a,b){
  return a+b;
}
num = add(1,2);
console.log(num);  //  3
{% endhighlight %}

### Function as a Named Function

In this instance we use a *named* function which is assigned to the `add` variable. 

{% highlight javascript %}
var add = function total(a,b){
  return a+b;
}
var num = add(1,2);
console.log(num);  //  3
{% endhighlight %}


Its worth noting here that when a function is assigned to a variable the function can be invoked by appending `()` to the variable.


This is probably best illustrated with step-by-step example as shown below:

{% highlight javascript %}
var foo = function(){
   return 'bar';
};


// Without parenthesis in function call
output = foo;
console.log(output) 

// Output
function (){
 return 'bar';
}

// With parenthesis in function call
output = foo();
console.log(output) 

// Output
bar
{% endhighlight %}

As you can see `foo` is referring to our *function expression* and invokes it with `()`.



### Function that Self Invokes

In the prevous section we stated that `foo` is a *function expression* which can be executed with paranethesis. So we can actually trigger the execution of the anonymous function like this:


{% highlight javascript %}
var output = function(){
   return 'bar'
}(); // <-- note the paraenthesis to invoke


// Executes function instead of returning the function
console.log(output) // bar
{% endhighlight %}



### Function Passed as an Argument

Below is a contrived example of the `add` function passed an argument to the `displayTotal` function.

{% highlight javascript %}
function add(a,b){
  return a+b;
}
  
function displayTotal(num1,num2,total){
	console.log(total(num1,num2))
}

displayTotal(num1,num2,add);
{% endhighlight %}



## A Full Example


{% highlight javascript %}
var validateDataForAge = function(data){
  person = data();
  console.log(person);
  if (person.age <1 || person.age > 99){
    return true;
  }else{
    return false;
  }
};


var errorHandlerForAge = function(error){
  console.log('Error with processing age');
};


function parseRequest(data, validateData, errorHandler){
 var error = validateData(data);
  if (!error){
    console.log("no errors");
  }else{
    errorHandler();
  }
}


var generateDataForScientist = function() {
  return {
    name: "Albert Einstein",
    age : Math.floor(Math.random() * (100 - 1)) + 1,
  };
};


parseRequest(generateDataForScientist, validateDataForAge, errorHandlerForAge);


{% endhighlight %}


## Scoping

Any variable defined is placed in the global scope. This is beyond bad. 


### Global Sope

**Global variables are created in 2 ways:**
1. Omitting the `var` keyword when declaring a variable inside a function. Functions are the only way put a variable inside a `local` scope. 
2. Omitting the `var` keyword. Apprently this is how you *imply* that a variable is global. Sounds like a terrible idea.


### Block Level Scope

Javacript like PHP doesn't have block level scoping. Variable doesn't exist inside an `if` condition for example. Instead, it has function level scoping.

### Function Level Scope

Each function defined has its local scope and any nested functions has access to its parent functions local scope. In the example below function `outer` is defined in the global scope and it has its own local scope. Function `inner` has its own local scope but has access to the parent function `outer` scope (and the global scope), in this case variables `a` and `c`. You can access the `outer` function from the global scope but you cannot access the `inner` function unless you are in the `outer` function's local scope. 

{% highlight javascript %}

function outer(){
  var a = 1;
  function inner(){
    b = 2;
  }
  var c = 3;
}
  
{% endhighlight %}

#### Named Functions and Local Scope

Althought it seems like a good idea to create named functions to apply local scope, it is not ideal as we have to define the function and then declare it.

This is solved by immediately exeucting the function by doing the following:


{% highlight javascript %}

// Treat the function as an expression and execute it with '()'
(function foo(){
  // Do something
})();
  
{% endhighlight %}

This pattern is know as *Imediately Invoked Function Expression*. 

we can also express this via an annonymous function like so:

{% highlight javascript %}

(function(){
  // Do something
})();
  
{% endhighlight %}

This version still has drawbacks. For one thing, you cannot use recursion in annonymous functions. You also cannot see the name in the stack trace which makes debugging difficult. 


### Hoisting

[This](https://www.sitepoint.com/demystifying-javascript-variable-scope-hoisting/) article does a good job of explaining this concept with simple examples.


### The 'this' Keyword in Use

Unlike ruby or python for example, the `this` keyword not only refers to a javascript object but functions when invoked has an implicit paramter named `this` which is passed to the function. It refers to the object that invokes the function which is termed as *function context*


Some examples:


When calling a function on an object the binding and the context is set to the object.

{% highlight javascript %}

var obj = {
    someData: "a string"
};

function myFun() {
    // Do something
}

obj.staticFunction = myFun;

obj.staticFunction(); 

{% endhighlight %}


When not calling a function on an object, the binding and context is set to the default `window` object.

{% highlight javascript %}

var obj = {
    myMethod : function () {
        // Do something
    }
};
var myFun = obj.myMethod;
myFun();

{% endhighlight %}



The following do a good job of demystifying the `this` keyword:

* [How does the this keyword work](http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)
* [Understanding javascript's this and mastering it](http://javascriptissexy.com/understand-javascripts-this-with-clarity-and-master-it/)


### Closures

When you define an anonymous function within another function you create a *closure*. Within this scope, you can access and manipulate variables that are outside this function. Let's break this down with an example.

{% highlight javascript %}

function sayHello(name) {
    var text = 'Hello ' + name;
    return function() { console.log(text); }
}

hello = sayHello('bob');
hello(); // Hello bob

{% endhighlight %}


In javascript when function executes it creates a *scope* object to hold the variables inside that function. So when the `sayHello` function is called, a scope object is created with the properties `name` and `text`. Then it returns an inner function. What's interesting to note here is that you'd think when you then execute the `hello()` method it will fail because how would it have access to the `text` variable after the function exited right? I would, but i'd be wrong. When the `sayHello` function returns it also returns a *reference* to the scope object created for the `sayHello` function. This enables the anonymous function to access the scope object via the reference. 


## Summary


#### Learning Resources

* [Different ways of defining functions in Javascript - This is madness!](http://davidbcalhoun.com/2011/different-ways-of-defining-functions-in-javascript-this-is-madness/)
* [Masting Javascript](https://www.packtpub.com/web-development/mastering-javascript)
* [Javascript, the right way](http://jstherightway.org/)
* [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript#Closures)