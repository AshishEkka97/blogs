---
title: Demystifying Prototypal Inheritance in JS
cover-image: suspension_bridge.jpg

---

JavaScript objects supports inheritance, but is very different from other languages like Java, Kotlin or C++.

While the latter ones use the `Class`ical inheritance, JavaScript uses something known as Prototypal Inheritance.

## What's Prototypal Inheritance? Never heard of it.

Well, in prototypal inheritance, an object can inherit properties from other objects which act as a prototype for it. This is done via an internal and hidden property - `[[Prototype]]`, which is a reference to either another object or null. This `[[Prototype]]` property can be manipulated using another property - `__proto__`. This `__proto__` is attached every time a new object is created (there's a catch though, which we will find later).

Let's clear that with an example:

```javascript
const fruits = ["🍎", "🍌", "🍇", "🍒", "🍍"];
console.log(fruits.length);
```

Now if we run this code, we get:

```bash
5
```

But how? We never wrote the `length` property in the fruits `Array`. Not only that, but like `length`, we somehow also have access to other properties like `concat`, `map`, `find`, etc. 

![JavaScript Array Properties]({{ site.baseurl }}/img/2021-07-04-demystifying-prototypal-inheritance-in-js/arrays-properties.png)

It's because `fruits` is a JavaScript `Array`, which is acting as a prototype for it. The `Array.prototype` is attached to `fruits` via the `__proto__` property.

This is evident from this below snippet.

```javascript
const fruits = ["🍎", "🍌", "🍇", "🍒", "🍍"];
console.log(fruits.__proto__);
console.log(Array.prototype);
console.log(fruits.__proto__ === Array.prototype);
```

This outputs:

```bash
[constructor: ƒ, concat: ƒ, copyWithin: ƒ, fill: ƒ, find: ƒ, …]
[constructor: ƒ, concat: ƒ, copyWithin: ƒ, fill: ƒ, find: ƒ, …]
true
```

## Own Properties vs. Inherited Properties

Above we saw that, when we create an `Array` in JS, it automatically inherits all the properties from the `Array` object. Let's check what if we create an object instead with some properties:

```javascript
const user = {
    name: "Ashish",
    age: 24,
    job: "🧑‍💻"
}

console.log(user.job);
console.log(user.toString())
```

Output:

```bash
🧑‍💻
[object Object]
```

We can see that `user.job` the `user`'s own property while `user.toString()` is not a property of `user`.

In this case, the JS engine first tries to find the `toString()` method in the `user` object. And since it can't find any `toString()` method in  `user` object, the JS engine then tries to find this in the `user`' prototype - `Object.prototype`.

That is why when we type `user.`, the property/method recommendation list contains `user`'s *own properties* like `name`, `age`, and `job` as well as *inherited properties* like `toString()`, `valueOf`.

JavaScript provides an inherited method `hasOwnProperty(key)` to determine if a particular property is an *inherited property* or is it's *own property*.

## The prototype chain

JavaScript uses this mechanism to create a inheritance chain till the `Object.prototype` is reached. This chain is called the *Prototype Chain*. This is the reason why we hear every one say:

> Everything is an `Object` in JavaScript.

Just have a look at this snippet below:

```javascript
function fun() {
    console.log("I'm a function. 🤖");
}

console.log(fun.__proto__);
console.log(Function.prototype);
console.log(fun.__proto__ === Function.prototype);
console.log(fun.__proto__.__proto__);
console.log(Object.prototype);
console.log(fun.__proto__.__proto__ === Object.prototype);

console.log(fun.__proto__.__proto__.__proto__);
console.log(Object.prototype.__proto__);
```



Output:

```bash
ƒ () { [native code] }
ƒ () { [native code] }
true
{constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
{constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
true
null
null
```

As we can see here, `fun.__proto__.__proto__` is having a reference to `Object.prototype` and the `__proto__` inside `Object.prototype` is having a reference to `null`, stating that this is the last link in the prototype chain.

## F.prototype

You must be wondering what is `Array.prototype` and `Object.prototype`? Well, `F.prototype` is nothing but a regular property named "`prototype`" on `F`. We know that we can create a new object in JavaScript using the constructor, like `new F()`.

If `F.protoype` is a property, then the `new` operator uses this property to set the `[[Prototype]]` for this new object.

The following snippet explains that pretty well:

```javascript
let animal = {
    legs: 4
}

function Dog(name) {
    this.name = name;
}

Dog.prototype = animal;

let dog = new Dog("Bruno");
console.log(dog.legs); //4
console.log(dog.__proto__ === Dog.prototype); //true
console.log(dog.__proto__ === animal); //true
```

```bash
4
true
true
```

Every function is JS, have a "`prototype`" property. If we don't supply a prototype manually, the default "`prototype`" is set to an object with only property - `constructor`, which referenced back to function itself. 

The thing is - since the inception of JavaScript, prototypal inheritance was it's core feature. There was no other way to directly access it. Developers only had the option to use the `constructor` function, which directly manipulates the  "`prototype`" property.

## What about JavaScript classes?

With ES6, `class`es were officially introduced to the JavaScript language. These were introduced to create blueprint for creating objects just like other languages like Java and C++. Let's see how it's generally used:

```javascript
class Animal {
    legs = 4;
	
	constructor(name) {
        this.name = name;
    }
}

const dog = new Animal("Bruno");
console.log(dog.legs); //4
console.log(dog instanceof Animal);
```

This will create a dog object, with properties like `legs` being inherited from `Animal`. Well the thing is that `class` syntax is just a syntactic sugar on top of prototypal inheritance. The underlying code is pretty similar to the prototypal one written above with same minor changes. Let's have a look at that:

```javascript
const animal = {
    legs = 4
};

function CreateAnimal(name) {
    return { name, __proto__: pet };
}
CreateAnimal.prototype = animal;

const dog = CreateAnimal("Bruno");

console.log(dog.legs); //4
console.log(dog instanceof Animal);
```

## In a nutshell

Unlike other languages like Java, Kotlin or C++, JavaScript supports prototypal inheritance rather than classical inheritance. The objects in JavaScript, inherit properties from other objects - known as prototypes.

This lookup for inherited properties by a JS engine is not only restricted to the prototype of an object, but also to the prototype of the prototype, till it reaches `null`, forming a chain known as the *prototype chain*.

While this may look a bit different from the notion of inheritance is in other languages, it is a very powerful feature and open limitless possibilities, once understood properly.

That's all for this blog post. Hope you learned something new today. See you in the next one. *Ciao!*