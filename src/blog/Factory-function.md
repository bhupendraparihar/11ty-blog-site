---
title: Factory function
date: 2020-05-24 03:31:57
author: Bhupendra Parihar
tags: ["post", "featured"]
image: /assets/blog/article-2.jpg
description: A factory function is any function which is not a class or constructor that returns a new object. In JavaScript, any function can return an object. When it does so without the new keyword, it’s a factory function.
---

What is factory function? And before this question, whatever it is, why I am going to use it. Hah!

Let us consider this example

I am creating two persons objects like
<pre>
var johnDoe = {
	firstName : ”John”,
	lastName : ”Doe”,
	sayHi  : function(){
		return “Hi there”;
	}
};

var janeDoe = {
	firstName : ”Jane”,
	lastName : ”Doe”,
	sayHi  : function(){
		return “Hi there”;
	}
}
</pre>
We created two objects so what if I want to create 10 more person object. Should I write these lines 10 more time? Opps! Sorry!

This is the situation where we are going to use factory function. In Javascript there are two function by which we can create object, one is <b>constructor function</b> and another is <b>factory function</b>.
<pre>
var createPerson = function(fName , lName){
	return {
		firstName : fName,
		lastName : lName,
		sayHi : function(){
			return “Hi there”;
		}
	};
};

var johnDoe = createPerson(“John”,”Doe”);

var janeDoe = createPerson(“Jane”,”Doe”);
</pre>
If we want to create multiple objects with same interface, this is the better way. Remember, Interface is a signature. Hope you understood what I am trying to say.