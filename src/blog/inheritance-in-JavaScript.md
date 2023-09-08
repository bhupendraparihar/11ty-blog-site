---
title: Inheritance in JavaScript
author: Bhupendra Parihar
date: 2023-07-30 16:15:32
tags: ['post', 'featured']
image: /assets/blog/programmer.jpg
description: Inheritance in JavaScript is very important topic to understand, but it has prototypal inheritance. This is not so easy to understand because this type of inheritance does not easily available in popular language like C++ or Java.
---

Inheritance in JavaScript is very important topic to understand, but it has prototypal inheritance. This is not so easy to understand because this type of inheritance does not easily available in popular language like C++ or Java.

For more details, <a href="http://underscorejs.org/#throttle">please follow the link</a>.

So If you are coming from a comparative approach, this idea will look clumsy. Lets try. Suppose we want to create a Vehicle constructor.

<pre>
function Vehicle(options){
    this.manufacturingYear = options.manufacturingYear;
}

Vehicle.prototype.drive = function() {
    return 'Vroom Vroom';
}
</pre>

Now, lets create a Car constructor which can inherit from Vehicle.

<pre>
function Car(options) {
    this.engineType = options.engineType;
}

Car.prototype.start = function() {
    console.log(`{this.engineType} starting`);
}

const bmw = new Car({
    manufacturingYear: 2023,
    engineType: 'Petrol'
});

console.log(bmw.manufacturingYear); // undefined- no such property inside car

</pre>

We need to make sure Car has all the features of Vehicle.

<pre>
function Car(options) {
    Vehicle.call(this, options);
    this.engineType = options.engineType;
}
</pre>

By calling the Vehicle constructor inside Car, you can get all the properties of Car which is defined inside Vehicle. for example.

<pre>
const bmw = new Car({
    manufacturingYear: 2023,
    engineType: 'Petrol'
});

console.log(bmw.manufacturingYear); // 2023 as output
</pre>

but what about if we want to access drive method of Vehicle which is the part of Vehicle prototype. To make this happens, we need to change the prototype of Car.

<pre>
Car.prototype = Object.create(Vehicle.prototype);
</pre>

Assigning constructor, change the constructor of the Car instance.

<pre>
console.log(bmw.constructor); // Vehicle
</pre>

Lets reassign the constructor back to Car.

<pre>
Car.prototype.constructor = Car;
</pre>

The combined code will look like this.

<pre>
function Vehicle(options){
    this.manufacturingYear = options.manufacturingYear;
}

Vehicle.prototype.drive = function() {
    console.log('Vroom Vroom');
}

function Car(options) {
    Vehicle.call(this, options);
    this.engineType = options.engineType;
}

Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

Car.prototype.start = function() {
    console.log(this.engineType + 'starting');
}

const bmw = new Car({
    manufacturingYear: 2023,
    engineType: 'Petrol'
});

console.log(bmw.manufacturingYear);

bmw.drive()

console.log(bmw.constructor)
</pre>

This is very confusing even to a experienced developer. So JavaScript came up with Class. and the same can be approached using the below code

<pre>
class Vehicle{
  constructor(options){
    this.manufacturingYear = options.manufacturingYear;
  }
  
  drive() {
    console.log('Vroom Vroom');
  }
}

class Car extends Vehicle{
  constructor(options){
    super(options);
    this.engineType = options.engineType;
  }
  
  start() {
    console.log(this.engineType + 'starting');
  }
}

const bmw = new Car({
    manufacturingYear: 2023,
    engineType: 'Petrol'
});

console.log(bmw.manufacturingYear);

bmw.drive()
</pre>
