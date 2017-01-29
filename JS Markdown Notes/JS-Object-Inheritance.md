# JavaScript - Object Inheritance

**NOTE:** This document is an extension of "JS-Objects.md". Read that document
before coming to this one. Or don't. I don't know lmao.

JavaScript follows what is known as a **prototypal** model of inheritance. As
you have seen previously, the "class" construct found in languages such as C++
and Java does not exist in JavaScript. JavaScript objects are nothing more than
a collection of name-value pairs, some of which are fields and others are
methods. Adding onto this, every object in JavaScript also has a
**[[Prototype]]** (Yes, the brackets come with the name).

We can think of an object's [[Prototype]] as being a special property whose
value is an object. All objects have one and only one [[Prototype]]. In the
past, the [[Prototype]] of an object was accessed through the field
`__proto__`, but this has since then been deprecated in favor of two
pre-defined methods: `Object.getPrototypeOf()` and `Object.setPrototypeOf()`.

Here is a very simple example of this in action:
```JavaScript
var animal = {country: "Africa"};
var lemur = {name: "Lemur"};
Object.setPrototypeOf(lemur, animal);
console.log(Object.getPrototypeOf(lemur));
```
Output:
```
{country: "Africa"}
```

In the above example, I have used `Object.setPrototypeOf()` to set `lemur`'s
[[Prototype]] to `animal`. And to confirm that the change has indeed happened,
I used `Object.getPrototypeOf()` to print out the string representation of
`lemur`'s [[Prototype]]. In practice, `Object.getPrototype()` is rarely used
because there is a great cost associated with changing an object's
[[Prototype]]. Instead, the prototype of an object is usually determined and
set during its construction.

As previously mentioned, *every* object in JavaScript has one [[Prototype]],
which means that the [[Prototype]] of an object has its own [[Prototype]],
which has its own [[Prototype]], and so on. This is what is known as the
**prototype chain**.

There are three pre-defined objects in JavaScript that serve as the "ancestral"
prototypes for all objects. These are `Object.prototype`, `Function.prototype`,
and `Array.prototype`. Of this holy, prototypal trinity, `Object.prototype`
is the alpha and the omega: It is the prototype of both `Function.prototype`
and `Array.prototype`. All objects in JavaScript can trace their prototype type
chains back to one of these three, and, ultimately, to `Object.prototype`
itself. `Object.prototype` is where the prototype chains of all objects end: It
is the only object whose [[Prototype]] is not an object, but rather `null`:

```JavaScript
// The following all yield "True"
console.log(Object.getPrototypeOf(Object.prototype) === null);
console.log(Object.getPrototypeOf(Function.prototype) === Object.prototype);
console.log(Object.getPrototypeOf(Array.prototype) === Object.prototype));

// Interestingly enough, the prototype of "Object" itself is Function.prototype
console.log(Object.getPrototypeOf(Object) === Function.prototype);
// This is because we can do stuff like this:
var someObj = new Object(); // This will be explained later
```

## Object Construction and Prototype

JavaScript offers several different ways of constructing objects, each way has
different rules governing the [[Prototype]] of the resulting object.

### Object Initializer/Literal Notation

Object creation using an **object initializer** looks something along the lines
of:

```JavaScript
var foo = {name: "Foo"};
```

In the above example, `foo` is a variable which contains the object yielded by
the expression `{name: "Foo"}`. Three different "types" of objects can be
created using literal notation: regular objects, functions, and arrays.

**Recall:** In JavaScript, anything that is not a Boolean, Number, String,
`null`, `undefined`, or symbol is an *object*.

Using literal notation, a regular object inherits from `Object.prototype`, a
function inherits from `Function.prototype` and an array inherits from
`Array.prototype`.

```JavaScript
var regObj = {};
function foo() {}; // function-like definition
var bar = function() {}; // variable-like definition
var arr = [];

// The following are all true
console.log(Object.getPrototypeOf(regObj) === Object.prototype);
console.log(Object.getPrototypeOf(foo) === Function.prototype);
console.log(Object.getPrototypeOf(bar) === Function.prototype);
console.log(Object.getPrototypeOf(arr) === Array.prototype);
```

### Object.create()

JavaScript offers a predefined method called `Object.create()`. When provided
with an object argument, this method will create a new object with the given
argument set as the [[Prototype]].

```JavaScript
var fooProto = {name: "FooProto"};
var foo = Object.create(fooProto); // foo's [[Prototype]] is fooProto
console.log(foo.name); // foo inherits "name" from fooProto
foo.name = "Foo"; // Adds "name" field to foo, which now shadows
                  // fooProto's "name"
console.log(foo.name);
console.log(fooProto.name); // fooProto's "name" remains unchanged
```
Output:
```
FooProto
Foo
FooProto
```

If you would like, you can think of `var foo = Object.create(fooProto)` as
being a short-hand for:
```JavaScript
var foo = new Object();
Object.setPrototypeOf(foo, fooProto);
```
This is *NOT* what actually happens, but it may be helpful for visualization.

### Object Constructors/Constructor Functions

JavaScript simulates object constructors using functions and the `new` keyword.
JavaScript constructors functions are really just normal functions. When
functions are called with the `new` keyword, however, they behave in ways that
make it look as if they are the constructors of objects of a certain class. We
typically capitalize constructor functions to distinguish them from regular
functions.

#### Sweet, Lovely Preamble

As a matter of fact, without even knowing about it, we have actually used three
constructor functions already: `Object`, `Function` and `Array`! These three
pre-defined *functions* (We know they are functions because they all have
`Function.prototype` as their prototypes) are implicitly called when we use
*object initializers* to create objects!

```JavaScript
var obj = {}; // Calls "new Object()"
function fun() {}; // Calls "new Function()"
var arr = []; // Calls "new Array()"
```

The specific workings of these three functions are beyond the scope of our
discussions. Especially if the objects we create are a little bit more
complicated than the trivial objects we have created above. But there is one
thing that I would like to point out with these functions.

You already know about the three ancestral [[Prototypes]] of JavaScript:
`Object.prototype`, `Function.prototype` and `Array.prototype`. All objects in
JavaScript can trace their prototype chains back to at least one of these three
fundamental [[Prototypes]]. Why is this the case? Well, as it turns out, this
is thanks to the constructor functions `Object`, `Function` and `Array`! You
see, when you create an object using one of these three functions, it will go
and set the [[Prototype]] of the newly created object to reference the same
object referenced by its `prototype` field. Therefore, we get:

```JavaScript
// The following are all true
console.log(Object.getPrototypeOf(obj) === Object.prototype);
console.log(Object.getPrototypeOf(fun) === Function.prototype);
console.log(Object.getPrototypeOf(arr) === Array.prototype);
```

This implies that **all** objects created via literal notation are **direct**
descendants of one of the three ancestral [[Prototypes]]. Later, you will see
that, because of the way user-defined constructor functions work, this ensures
that all prototype chains can trace themselves back to the ancestral
[[Prototypes]].

Okay, big deal, so what? Well, turns out, there is actually an operator in
JavaScript called `instanceof` that takes two operands, the first being an
object and the latter being a function, and *simulates* the operation of
testing whether or not if the object is an *instance* of the given *class*.

```JavaScript
// The following are all true
console.log(obj instanceof Object);
console.log(func instanceof Function);
console.log(func instanceof Object);
console.log(arr instanceof Array);
console.log(arr instanceof Object);
// WARNING: If the second operand is not a function, an error is generated
```

Pretty cool, right? Unfortunately, the detailed workings of `instanceof` are
once again beyond the scope of our discussions. But we can imagine it being the
following algorithm:
1. Start with the given object's [[Prototype]], if it's `null`, return false.
2. If it's not `null`, check to see if its [[Prototype]] is equal to the value
   of the second operand's `prototype` field.
3. If yes, return true. Else, move up one step in the prototype chain and start
   again.
4. Repeat the above process until the prototype chain reaches `null`, in which
   case, return false.

So, to recap, when we create an object using literal notation, we are actually
implicitly calling the pre-defined constructor functions `Object`, `Function`,
and `Array`. These three constructor functions, among other things, will set
the [[Prototypes]] of our newly constructed objects to `Object.prototype`,
`Function.prototype` and `Array.prototype` respectively.

#### Here Comes the Real Deal

Okay, here comes the kicker: **Every** function in JavaScript has a field
called `prototype`. No, I'm not talking about the [[Prototype]] of a JavaScript
function (which, as you'll recall, is always `Function.prototype`). When a
function is defined, a new object is created somewhere in memory and the
`property` field of the function will be set to reference that object.

```JavaScript
function foo() {}

// The following are all true
console.log(Object.getPrototypeOf(foo) === Function.prototype);
console.log(foo.prototype !== Function.prototype);
console.log(Object.getPrototypeOf(foo.prototype) === Object.prototype);
```

As you can see, the [[Prototype]] and the `prototype` of a function are two
different things. Also, the [[Prototype]] of the `prototype` of a function is
`Object.prototype` (Try saying that ten times fast). This last line proves that
`foo.prototype` is an object created with a call to the fundamental constructor
function `Object`. In fact, content-wise, the object referenced by
`foo.prototype` is *almost* exactly the same as an objected created by the
expression `new Object()`. (For an example of the differences, see the section
at the end of this document discussion the `constructor` field).

**NOTE:** I have avoided `console.log(foo.prototype)` because the results of
such a print-out is implementation-specific.

What is the purpose of the `prototype` field? Well, you already know that
constructor functions in JavaScript are just any function that happened to be
called together with the `new` keyword. It's time that I elaborated on that a
little bit more.

Suppose that we had a function called `Cat()`:

```JavaScript
function Cat(name, breed, age) {
  this.m_name = name;
  this.m_breed = breed;
  this.m_age = age;
}
```

We have capitalized this function to express our intention to use it as a
constructor function. Now, suppose we wanted to create a new Cat. You already
know how to do this:

```JavaScript
var darius = new Cat("Darius", "Persian", 6);
```

Well, turns out, this is really a short-hand for something *in the vein of* (As
in, this is not that actually happens, but it is close enough to allow us to
continue our discussion of constructor functions):

```JavaScript
var darius = new Object(); // This is "line 1"
Object.setPrototypeOf(darius, Cat.prototype);
Cat.call(darius, "Darius", "Persian", 6);
```

First, the variable `darius` is created and we call `Object` to prepare an
*empty* object for us. Obviously, there is a lot more happening under the hood
in this line, but for now, all me need to know is that it allow us to work with
an empty object.

Next, we set the [[Prototype]] of `darius` to reference the same object
referenced by `Cat.prototype`. Notice how closely this parallels the behavior
of the fundamental constructor functions `Object`, `Function`, `Array`. And by
"closely", I mean they are exactly the same. This is intentional, of course:
`Cat`, just like the fundamental constructor functions, will set the
[[Prototypes]] of all objects created through it to reference `Cat.prototype`.
Soon, we will begin changing the `prototype` field of constructor functions,
but right now, note that because JavaScript gave `Cat` a default object for its
`prototype` field, and because it automatically sets [[Prototypes]] of all
instances of `Cat`, we have:

```JavaScript
// The following are all true
console.log(darius instanceof Cat);
console.log(darius instanceof Object);
```

Lastly, we use `Cat.call()` to *initialize* `darius` with the desired values.
How does this work? Well, `call()` is a method that `Cat` inherits from
`Function.prototype`. Calling `Cat.call(darius, "Darius", "Persian", 6)` is
exactly the same as calling `Cat("Darius", "Persian", 6)` but with the addition
that, inside of the body of the function, the `this` keyword will given the
value of `darius`, which is a reference to the object created on line 1. As
such, inside the body of the function, whenever we say something along the
lines of  `this.property_name = some_expression`, we are creating a new
property in `darius` and giving it the value of the expression to the right of
the assignment operator.

## JavaScript Classes and Inheritance

And there you have it. With a bit of ingenuity, JavaScript is able to simulate
the concepts of *classes* and *instances* using objects and functions. Of
course, calling it a mere "simulation" is really doing a disservice to the
prototypal model of object-oriented programming. The prototypal model provides
many advantages over the classical model, and we will begin to see some of them
in this section.

### Basic Class Creation and Instantiation

As you know, when we define a constructor function, it is convention to refer
the name of said function as a **class**. Then, whenever we use `new` with the
constructor function, it is convention to say that we are creating an
**instance** of the class. To that end, let us continue working with the Cat
*class* we have created earlier:

```JavaScript
function Cat(name, breed, age) {
  this.m_name = name;
  this.m_breed = breed;
  this.m_age = age;
}
```

Now, let us create some *instances* of Cat:

```JavaScript
var abdul = new Cat("Abdul", "Persian", 5);
var james = new Cat("James", "Bengal", 1);
var lilWayne = new Cat("Lil Wayne", "Bombay", 5);
```

These three objects are all instances of Cat. Their [[Prototypes]] all refer to
the same object: `Cat.prototype`.

### Property Inheritance

In JavaScript, when we try to access an object's property, if said property
cannot be found within the object itself, the runtime will go the the object's
[[Prototype]] and look for it there. This occurs recursively until the runtime
reaches the end of the [[Prototype]] chain. If the property we try to access is
found in one of the [[Prototypes]], we say that the object has **inherited**
that property.

You may not realize it yet, but the cats we have created above, `abdul`,
`james` and `lilWayne`, have already inherited several properties from their
prototypes. For example, there is a method defined in `Object.prototype` called
`toString()` which is inherited by all three cats:

```JavaScript
console.log(abdul.toString());
console.log(james.toString());
console.log(lilWayne.toString());
```
Output:
```
[object Object]
[object Object]
[object Object]
```

It may not look like much, but the mere fact that anything is printed at all is
all the proof that we need of inheritance. Furthermore, the immediate prototype
of all three cats, `Cat.prototype`, has some properties of its own which are
inherited. An example of this is the `constructor` property, which, at this
moment, is a reference to the `Cat` constructor function:

```JavaScript
// All true
console.log(abdul.constructor === Cat);
console.log(james.constructor === Cat);
console.log(lilWayne.constructor === Cat);
```

As you can see, in JavaScript, property inheritance is fairly straight-forward.
But, as per usual, there is something that I would like to mention: Inherited
methods, when invoked on an instance (`instance.method(...)` or something
similar), will have their `this` keyword refer directly to the invoking object.

For the most part, this is the desired behavior and assists in achieving
*polymorphism*. But for those who are familiar with languages such as C++, this
behavior is worth pointing out as it is (yet again) a departure from older
object-oriented programming practices:

```JavaScript
function Foo(name) { // We will define a class called "Foo"
  this.name = name;
}

// We will define a method inside of `Foo.prototype` so that it is inherited by
// all instances of Foo
Foo.prototype.getName = function() {
  return this.name;
}

Foo.prototype.name = "Foo.prototype"; // Setting Foo.prototype's "name"

var globalGetName = Foo.prototype.getName; // Refers to the same function

var name = "Global"; // Setting global object's "name"

// Making two instances of Foo
var foo1 = new Foo("Foo1");
var foo2 = new Foo("Foo2");

console.log(globalGetName()); // Invoking function on global object
console.log(Foo.prototype.getName()); // Invoking the function on Foo.prototype
console.log(foo1.getName()); // Invoking function on the instance foo1
console.log(foo2.getName()); // Invoking function on the instance foo2
```
Output:
```
Global
Foo.prototype
Foo1
Foo2
```

Throughout the entirety of the code above, we are working with just one
function value. We have two references to this function: `globalGetName` and
`Foo.prototype.getName`. Depending on the context in which we invoke the
function, the value of the `this` keyword inside the body of the function
changes, hence we get the displayed output.

### Dynamically Adding and Changing Properties

How do you like our definition for the Cat class? You think it's nice? Oh wait,
whoops, it seems like I forgot to tell you that all Cats should also be able to
`speak()`. What do we do? Do we need to go through each instance and add the
method manually? Nope. JavaScript provides a much more elegant solution: We can
just add a method to `Cat.prototype` and have *inheritance* take care of the
rest:

```JavaScript
Cat.prototype.speak = function() {
  console.log("Meow");
}

abdul.speak();
james.speak();
lilWayne.speak();
```
Output:
```
Meow
Meow
Meow
```

Notice that none of our instances have a method called `speak()`, the runtime
will go their [[Prototypes]] to search for the identifier. In `Cat.prototype`,
it finds a matching method and proceeds to execute it. Great! But wait, our
cats are actually a little bit special: The first time they `speak()`, they
will actually tell you about themselves. From then on, though, they will just
meow. This type of behavior is a *nightmare* to implement using a classical
model of inheritance. However, once again, thanks to JavaScript's prototypal
model, we have a very elegant solution:

```JavaScript
Cat.prototype.speak = function() {
  console.log(this.m_name + " is a(n) " + this.m_age + "-year-old "
              + this.m_breed + " cat.");
}

abdul.speak();
james.speak();
lilWayne.speak();

Cat.prototype.speak = function() {
  console.log("Meow");
}

abdul.speak();
james.speak();
lilWayne.speak();
```
Output:
```
Abdul is a(n) 5-year-old Persian cat.
James is a(n) 1-year-old Bengal cat.
Lil Wayne is a(n) 5-year-old Bombay cat.
Meow
Meow
Meow
```

Would you look at that, because methods are really just references to objects,
we are able to *dynamically* change what they reference and thus use a
completely different definition. Actually, it's probably accurate to say that
JavaScript implements the *bridge-pattern* at the very heart of its definition
of objects.

Alright, one last thing: `lilWayne` is actually a special cat: No one
understands what he's saying. Therefore, its `speak()` method should always
just print "???". No problem, here's how we can implement this behavior:

```JavaScript
Cat.prototype.speak = function() {
  console.log(this.m_name + " is a(n) " + this.m_age + "-year-old "
              + this.m_breed + " cat.");
}

lilWayne.speak = function() {
  console.log("???");
}

abdul.speak();
james.speak();
lilWayne.speak();

Cat.prototype.speak = function() {
  console.log("Meow");
}

abdul.speak();
james.speak();
lilWayne.speak();
```
Output:
```
Abdul is a(n) 5-year-old Persian cat.
James is a(n) 1-year-old Bengal cat.
???
Meow
Meow
???
```

In the above example, we added a property to `lilWayne` itself. Therefore,
whenever we call `lilWayne.speak()`, the call resolves to an execution of the
`speak()` method inside of `lilWayne`, the definition in its [[Prototype]] be
damned. This is what we call **property shadowing**. Property shadowing is what
allows us to simulate the concept of *member overrides*, which is a central
part of object-oriented programming.

### Creating Inheritance Trees

Before we begin, this will be the definition of the Cat class that we will be
working with:

```JavaScript
function Cat(name, breed, age) {
  this.m_name = name;
  this.m_breed = breed;
  this.m_age = age;
}

Cat.prototype.speak = function() {
  console.log(this.m_name + " is a(n) " + this.m_age + "-year-old "
              + this.m_breed + " cat.");
}
```

We have finally come to (perhaps) the crux of object-oriented programming,
whose true power lies in the elaborate networks of inheritances that can be
created. By extending subclasses of subclasses of well, subclasses, you get the
idea, we can create some truly powerful data structures that will help make a
lot of programming tasks easier.

To this end, let us create some more specific Cat classes. In fact, why don't
we try to make the different species of cats we already have into their own
subclasses. Specifically, let's try creating three subclasses: PersianCat,
BengalCat and BombayCat.

Before we go and write the code, however, we are going to lay out some
specifications common to all three of our subclasses:
1. The constructors of all three functions should have two parameters, `name`
   and `age`, which it passes to the "superclass" constructor
2. Each class will also pass the appropriate value for `breed` to the
   "superclass" constructor

We will begin with the BengalCat as it is the simplest to write. Instances of
the BengalCat are just like normal Cat instances, except they will all have
"Bengal" as the value for their `breed` field. This is how we achieve this in
JavaScript:

```JavaScript
function BengalCat(name, age) {
  Cat.call(this, name, "Bengal", age);
}

BengalCat.prototype = Object.create(Cat.prototype);
```

Pay close to attention to what we are doing in the above example. First, we
defined a constructor function called `BengalCat()` in order to create our
BengalCat class. Notice that inside the body of the function, we will actually
use the `Cat()` constructor function. If you are familiar with other
object-oriented languages, this is how JavaScript simulates a call to the
superclass' constructor. When I say simulation, I really do mean it this time.
Notice that instead of calling `new Cat(...)`, we say `Cat.call(this, ...)`.
There is no magic here; we are just reusing code. In fact, had we just copied
over the body of `Cat()` into `BengalCat()`, the function would remain
*exactly* the same:

```JavaScript
function BengalCat(name, age) {
  this.m_name = name;
  this.m_breed = "Bengal";
  this.m_age = age;
}
```

Next comes the line where we create an object to use as `BengalCat.prototype`;
this is where all of the inheritance magic happens!
`Object.create(Cat.prototype)` will create for us a new, empty object whose
[[Prototype]] references `Cat.prototype`. Since all instances of BengalCat will
have `Bengal.prototype` as their [[Prototype]], this also means that
`Cat.prototype` will be the [[Prototype]] of the [[Prototype]] of all BengalCat
instances. As a result, we can do things like:

```JavaScript
var james = new BengalCat("James", 1);
console.log(james instanceof BengalCat); // True
console.log(james instanceof Cat); // True
james.speak(); // "James is a(n) 1-year-old Bengal cat"
```

Notice how we were able to create `james` with just two arguments, the name and
age. Since all BengalCats should have "Bengal" as their `breed`, we should not
have to tell the constructor about it. Additionally, all BengalCats *ARE* Cats,
therefore, `james` should also be an instance of Cat. Because BengalCat inherit
Cat, all instances of BengalCat also have `speak()` as a method.

```JavaScript
function PersianCat(name, age, color) {
  Cat.call(this, name, "Persian", age);
  this.m_color = color;
}

PersianCat.prototype = Object.create(Cat.prototype);

var abdul = new PersianCat("Abdul", 5, "White");
console.log(abdul instanceof PersianCat);
console.log(abdul instanceof Cat);
abdul.speak();
console.log(abdul.m_color);
```

```JavaScript
function BombayCat(name, age) {
  Cat.call(this, name, "Bombay", age);
}

BombayCat.prototype = Object.create(Cat.prototype);
BombayCat.prototype.speak = function() {
  console.log("???");
}

var lilWayne = new BombayCat("Lil Wayne", 5);
```












## Et Cetra

### [[Prototype]]-less Objects

### Non-Enumerable Properties and Own Properties

### Define the Prototype Prior to Instantiation

### The `constructor` field

```JavaScript
var obj = {}; // Calls "new Object()"
function fun() {}; // Calls "new Function()"
var arr = []; // Calls "new Array()"
```

```JavaScript
console.log(Object.prototype.constructor === Object); // True
console.log(Function.prototype.constructor === Function); // True
console.log(Array.prototype.constructor === Array); // True
```

As you can see, the value of the `constructor` of each one of the ancestral
[[Prototypes]] is their corresponding constructor function. This is important
because, as previously mentioned, objects created using literal notation will
directly inherit one of these three objects. As such, they will all also have
this field:

```JavaScript
// The following are all true
console.log(obj.constructor === Object);
console.log(fun.constructor === Function);
console.log(arr.constructor === Array);
// fun does not directly inherit Object.prototype
console.log(fun.constructor !== Object);
// arr does not directly inherit Object.prototype
console.log(arr.constructor !== Object);
```
