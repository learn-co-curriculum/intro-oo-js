# Introduction to Object-Oriented JavaScript

## Objectives

1. Explain how prototype lookup works in JavaScript
2. Construct a pair of objects to demonstrate context and properties
3. Explain how `instanceof` can provide type-checking functionality in a dynamic JavaScript environment
4. Refactor these objects to demonstrate prototypal inheritance
5. Build a small demo to show object-oriented JavaScript in action

## Object-Oriented Programming — Review

When you think of object-oriented programming, what's the first thing that comes to mind? Maybe it's the classic animal example, where you build animals that say different things:

```ruby
class Animal
  def speak!
    puts sound
  end

  def sound
    "I don't know what to say!"
  end
end

class Dog < Animal
  def sound
    "woof!"
  end
end

class Cat < Animal
  def sound
    "meow!"
  end
end

Animal.new.speak! # I don't know what to say!
Dog.new.speak! # woof!
Cat.new.speak! # meow!
```

Or maybe you jump immediately to a higher level of abstraction, thinking about how to delineate one _type_ of code from another. Either way, you've stretched yourself thinking about classes, methods, and inheritance, and you've decided that you want to bring the powers of object-orientation to the parts of your application that live in the browser.

There's just one problem: JavaScript doesn't have classes.*

<small>* Okay, you caught me: ECMAScript6 introduces `class` support; but under the hood, we'll see that JavaScript classes still largely depend on the concept of a `prototype`, introduced below.

## Prototyping like it's 1995

In JavaScript (or, more precisely, ECMAScript 5 — for our purposes, we can use "JavaScript" and "ECMAScript" interchangeably), object-oriented programming depends on the concept of a `prototype`. A JavaScript `prototype` isn't like a prototype of an application — for example, it doesn't (necessarily) have functionality mocked out — but is instead very literally a _first_ or _preliminary_ (_proto-_) version of its kind (_-type_).

Open up Chrome and type `Option+CMD+J` to open the developer console. (You can also go to your terminal and type `node`, if you have Node.js installed — the examples in this Readme will run in both environments.) This will be our REPL for this lesson. Let's start by defining a function called `PrototypeExplorer`:

```javascript
function PrototypeExplorer() {}
```

You'll notice that this function doesn't do anything — or at least, it doesn't look like it does anything. But let's try an experiment: in the console, type

```javascript
PrototypeExplorer.prototype
```

Whoa — did you see that? Your console should have just printed:

```javascript
PrototypeExplorer {}
```

If you expand that _object_, you should see two properties:

```javascript
{
  constructor: function PrototypeExplorer(),
  __proto__: { ... }
}
```

Where did they come from? An object's `constructor` and `__proto__` are defined on a function's `prototype`. The `constructor` points to the function that _constructs_ an instance of this prototype; `__proto__` points to properties of the function and gives your code a way to _climb up_ the prototype chain. If you expand `__proto__`, you should see that it too has a `constructor`, the function `Object()`. This function lies behind every object in JavaScript, and provides the functionality that lets us set properties on objects in JavaScript. And because functions _inherit_ from objects, we can set properties on them, too:

```javascript
PrototypeExplorer.hereProperty = "Property is theft!"
```

Now, if you enter `PrototypeExplorer.hereProperty` in your REPL, you should see that message. Let's try something a little fancier. Generally, you'll only want to do the following in production code if you're sure you want to change _every_ instance of a given prototype. It's important to know that changes like this _can_ happen, even if we won't deliberately perform them often, because they can make debugging code a bit funky.

![Funky Left Shark](http://i.giphy.com/5ZoKgx4bC8yxW.gif)

## The Prototype Ch-ch-chain

```javascript
Object.prototype.everywhereProperty = "I am infinite!"
Object.prototype.hereProperty = "I'm just here."

PrototypeExplorer.everywhereProperty // "I am infinite!"
PrototypeExplorer.hereProperty // "Property is theft!"
```

What just happened? We changed the value of `everywhereProperty` on the `prototype` of `Object`. This makes `everywhereProperty` available to ever object that inherits from `Object`, so since `PrototypeExplorer` has `Object` in its prototype chain, it also has access to `everywhereProperty`.

We also changed the value of `hereProperty` on `Object.prototype`, but the value of `hereProperty` didn't change on `PrototypeExplorer` — why do you think that is? When JavaScript looks for an object's property, it starts at the object itself. So when we ask for `PrototypeExplorer.hereProperty`, JavaScript finds `hereProperty` on `PrototypeExplorer` directly and returns the result. In other words, `hereProperty` is one of `PrototypeExplorer`'s _own_ properties.

```javascript
PrototypeExplorer.hasOwnProperty('hereProperty') // true
```

Conversely, when we ask for `PrototypeExplorer.everywhereProperty`, JavaScript does not find `everywhereProperty` on `PrototypeExplorer`. So now we turn to `PrototypeExplorer`'s prototype, and after climbing up the prototype chain to `Object`, we find `everywhereProperty` and return its value. But `everywhereProperty` is not one of PrototypeExplorer's own properties:

```javascript
PrototypeExplorer.hasOwnProperty('everywhereProperty') // false
```

An object's own properties — those properties for which `hasOwnProperty()` will return `true` — are only those properties that are set on the object itself. It might be helpful to think of a prototype like a classroom. In the classroom, pencils get passed around to each of the students, but these pencils belong to the classroom itself. So when we ask a student for a pencil, she or he can provide us with one — but she or he does not _own_ the pencil unless she or he brought it in her-/himself.

## Cats and Dogs

To try our hand at object-oriented JavaScript, we're going to build a small zoo — to start, it only contains cats and dogs. Enter the following in your console:

```javascript
function Cat() {
  this.numberOfLegs = 4
  this.sound = 'meow'
}

function Dog() {
  this.numberOfLegs = 4
  this.sound = 'woof'
}

Cat.prototype.speak = function() {
  console.log(this.sound)
}

Dog.prototype.speak = function() {
  console.log(this.sound)
}

var cat = new Cat()
var dog = new Dog()
```

We've introduced two new concepts in only a few lines of code, so let's slow down and look at what's above line by line. We know from `PrototypeExplorer` that you can declare a "class" — or, more properly, a **constructor** (more on that in a second) — in JavaScript by declaring a function. By convention, we start functions-as-classes with a capital letter. (If you're curious, the reason for this convention is that JavaScript doesn't have a strong enough type system to distinguish between a function called `cat()` and a constructor called `Cat()`. Types are a bit beyond the scope of this lesson, but check out the [#resources](Resources) section below if you're interested in learning more.)

Inside every function — whether it's a constructor or a plain ol' function — is its context. In JavaScript, context is assigned to `this`: at the top level in the browser `this === window`. In `Cat()` and `Dog()` — and any function that's used as a **constructor** — we say that `this` "is bound to the new object being constructed" ([MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)). Using the examples above, you can think of the `this` of `var cat = new Cat()` belonging to `cat` and the `this` of `var dog = new Dog()` belonging to `dog`.

What's all this talk about **constructor** and **new**, though? Remember the `constructor` property on `PrototypeExplorer` above? Whenever you use the `new` keyword with a function, you are using that function as a constructor — and setting the `constructor` of the resulting instance to the function itself. What is the `constructor` of `dog`?

```javascript
dog.constructor // returns the stringified function `Dog()`!
```

Pretty cool, right?

![Cool](http://i.giphy.com/KXY5lB8yOarLy.gif)

As with `PrototypeExplorer`, any attempt to access the methods of an instance of `Cat` or `Dog` will look up the prototype chain. So you can call `cat.speak()` and print `meow`, since we have assigned the function `speak()` on the `prototype` of `Cat`.

## How do we know what's a Cat and what's a Dog?

Imagine that we actually named our cats and dogs:

```javascript
var thelma = new Cat()
var louise = new Dog()
```

And imagine passing those instances through a few layers of code. Imagine further — if you will — that you have a function `likesWalks()`:

```javascript
function likesWalks(animal) {
  // If the animal is a Dog, return `true`.
  // If the animal is not a dog, return `false`.
}
```

This function takes any input, but we want to make sure that we only take instances of `Dog` for a walk — other animals, including `Cat`, are simply too lazy. How can we make sure that we don't attempt to drag an instance of `Cat` outside and suffer a thousand scratches?

JavaScript provides a handy infix operator (meaning it appears _in between_ the things that it operates on) called `instanceof`. (It's sort of ugly in that it's one word and it isn't camelCased, but we allow JavaScript these warts.) We use `instanceof` like so:

```javascript
function likesWalks(animal) {
  return animal instanceof Dog
}
```

Now we can be sure that we're only taking instances `Dog` for a walk!

```javascript
var fifi = new Cat()
var fido = new Dog()

likesWalks(fifi) // false
likesWalks(fido) // true
```

## What do dogs and cats have in common?

You probably noticed above that the `Dog()` and `Cat()` constructor functions look pretty much the same — the only difference is what we assign to `this.sound`. Are you thinking what I'm thinking? Let's refactor!

![High five!](http://i.giphy.com/CCJnMBqEYxxEk.gif)

When refactoring object-oriented programs — or, really, many programs — it's always good to start with the question, "What do these pieces of the puzzle have in common?" In the case of `Cat()` and `Dog()` they both have `numberOfLegs`, and they both `speak()`. Let's move those properties to a new constructor called `Animal`:

```javascript
function Animal(numberOfLegs, sound) {
  this.numberOfLegs = numberOfLegs
  this.sound = sound
}

Animal.prototype.speak = function() {
  console.log(this.sound)
}
```

You might be wondering why we assign `speak()` to `Animal`'s `prototype` and not to `this` inside the constructor function. The truth is, while both approaches would work, assigning `speak()` to the prototype allows the method to be instantiated only once, whereas if we assigned `speak()` to `this`, we would be creating a new method for every instance of `Animal` that we called up.

We also fixed a bit of nastiness by parameterizing `numberOfLegs` and `sound` — now we aren't limited to animals with 4 legs and types of `Animal`s that only make one sound!

Now we need to redeclare `Dog()` and `Cat()` such that they inherit from `Animal()`. In modern/ES2015/ES6 JavaScript, inheritance looks a lot like it does in Ruby:

```javascript
class Animal {
  // ...
}

class Dog extends Animal {
  // ...
}

class Cat extends Animal {
  // ...
}
```

But since most browsers still don't support this method of inheritance natively (as of February 2016), it's important to learn the traditional way of handling JavaScript inheritance.

Luckily, modern browsers (and Node.js) have adopted `Object.create()` — this method is really handy for cloning objects in JavaScript, and we can use it to establish prototypal inheritance:

```javascript
function Cat(sound) {
  Animal.call(this, 4, sound)
}

Cat.prototype = Object.create(Animal.prototype)

// We need to re-assign `Cat`'s constructor for `instanceof` to work correctly
Cat.prototype.constructor = Cat
```

The first thing you should notice is that the `Cat` constructor now accepts an argument — our `Cat`s can make different sounds!

The second thing you'll notice is `Animal.call(this, 4)`. Recall that `this` inside the `Cat` constructor is the new instance of `Cat` being constructed. The function `call()` and its sibling `apply()` let us _change_ the `this` value of the function that they're calling. These functions, `call()` and `apply()`, are largely the same — both take the value being substituted for `this` as their first argument — differing only in how they handle subsequent arguments: `call()` expects an arbitrary number of additional arguments (zero to many), while `apply()` expects either nothing or an array as its second argument. `Animal.call(this, 4, sound)` is the same as `Animal.apply(this, [4, sound])`.

In the case of `Cat()`, we call `Animal()` not as a constructor (that requires the `new` keyword) but as a plain function, passing it the new `instance` of `Cat` for its this value and the number `4` for its `numberOfLegs` parameter. Let's try it out to see how it works:

```javascript
var tabby = new Cat('mraw')

tabby.numberOfLegs // 4
tabby.speak() // 'mraw'
```

Very cool! We can do the same with `Dog()`:

```javascript
function Dog(sound) {
  Animal.call(this, 4, sound)
}

Dog.prototype = Object.create(Animal.prototype)
Dog.prototype.constructor = Dog

var borderCollie = new Dog('Ruff!')

borderCollie.numberOfLegs // 4
borderCollie.speak() // 'Ruff!'
```

## Down on the Farm

As you can see, object-oriented JavaScript provides us with a lot of power and flexibility. You might, with all this new-found power, be imagining a herd of animals at your fingertips:

```javascript
function Chicken(sound) {
  Animal.call(this, 2, sound)

  this.isLost = false
}

Chicken.prototype = Object.create(Animal.prototype)
Chicken.prototype.constructor = Chicken
Chicken.prototype.runAway = function() {
  console.log('The chicken that goes, "' + this.sound + '", ran away.')
  this.isLost = true
}

var coolChicken = new Chicken('cluck cluck, everybody')
var scaredChicken = new Chicken('OMG!')

coolChicken.runAway = function() {
  console.log("Cool chickens don't run.")
}

coolChicken.runAway() // "Cool chickens don't run."
scaredChicken.runAway() // 'The chicken that goes, "OMG!", ran away.'

coolChicken.isLost // false
scaredChicken.isLost // true
```

Well, we blew it: our power let one of the chicken get away. Notice how the `coolChicken` doesn't run, because we have modified `runAway()` directly on that instance. The `scaredChicken`, however, looks at `Chicken.prototype` for the default `runAway()` behavior and scatters accordingly.

Let's add a method to `Dog` so that we can find the lost `scardChicken`:

```javascript
Dog.prototype.herd = function(animal) {
  this.speak()

  animal.isLost = false

  console.log(
    "The dog that goes, \"" + this.sound + "\", brought back the " +
    (animal.constructor.name).toLowerCase() + " that goes, \"" +
    animal.sound + "\"."
  )
}
```

See how we're calling `speak()` on the instance of `Dog` that's performing `herd()`? Similarly, we're modifying the value of `isLost` on the `animal` that is being herded — the `herd()` method itself is indifferent to the kind of animal that's being herded.

And by assigning `herd()` to the `prototype` of `Dog`, we have made it available to every instance of `Dog`. So we can find our `borderCollie` from the previous example and use her to bring back the lost chicken:

```javascript
borderCollie.herd(scaredChicken)

/*
 * 'Ruff!'
 * 'The dog that goes, "Ruff!", brought back the chicken that goes, "OMG!".'
 */

scaredChicken.isLost // false
```

We're well on our way to having a whole menagerie of `Animal`s. We can probably think of ways to continue refactoring our constructors and even making these pretend `Animal`s do more useful things; but for now, let's appreciate how prototypes in JavaScript give you a way of passing information down from a higher-level constructor to concrete instances, and how instances can modify each other's properties as they're passed around. Good work!

![Good work!](http://i.giphy.com/3PhrIC5stiSli.gif)

## Resources

* [Wikipedia](https://en.wikipedia.org/wiki/Type_system) - [Type system](https://en.wikipedia.org/wiki/Type_system)
* [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) - [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

<a href="https://learn.co/lessons/introduction-to-object-oriented-javascript" data-visibility='hidden'>View this lesson on Learn.co</a>
