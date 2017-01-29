# JavaScript - Objects

In programming, the concept of "objects" have been around for a while now. The
so-called "objected-oriented" principles of programming advocate the idea that
computer code can be (and should be) written to model the way real-life objects
behave and interact with each other. Simply put, object-oriented programming
says that good programs should:
* Be modular; consist of smaller, more manageable parts that are, as much as
  possible:
  * Each individually coherent and self-contained
  * Separated and independent from each other
* Hide (encapsulate) their inner workings and present a simple and intuitive
  interface to clients

In order to achieve these principles (and many more), there came about the
"object". An object is, in essence, an attempt at modeling real-life objects.
As such, when talking about "objects" in programming, we will often talk about
the object's:
* **fields**, which represent what the object is *like*
* **methods**, which represent what the object *can do*

The concepts of objected-oriented programming are language-independent, and
different object-oriented programming languages have different
*implementations* of these ideas. In this document, we will examine and discuss
the ways in which objects are implemented in JavaScript.

## Properties

### Object Properties

In JavaScript, there are a total of **SIX** primitive data types:
* Boolean
* Null
* Undefined
* Number
* String
* Symbol

Anything that is not one of the above is an **object** in JavaScript! This
includes things such as *arrays*, *functions*, and so-called "normal" objects,
user-defined or otherwise. Capturing such a large collection of data types, it
would be natural to think that JavaScript objects are overbearingly complex to
say the least. But, that is not true.

See, in JavaScript, *objects* are nothing more than a type of value. In fact,
JavaScript objects are really little more than a collection of name-value
pairs that we call **properties**. Here is an example:

```JavaScript
var bar = {name:"bar", id:123}; // bar is an object
```

In the above example, `bar` is an object which contains two properties: `name`
and `id`. We can access the value of these two properties and change them, if
we'd like, as per the following:

```JavaScript
console.log(bar.name);
console.log(bar.id);
// Mutations
bar.name = "foo";
bar.id = 2381;
// We can even add new fields
bar.friends = ["barr", "fafa"];
console.log(bar.name);
console.log(bar.id);
console.log(bar.friends);
```
Output:
```
bar
123
foo
2381
["barr", "fafa"]
```

Object properties, in most aspects, are the same as regular variables. The only
difference between the two is the fact that object properties are *tied* to an
object. For those who are familiar with terminology from other object-oriented
programming languages, one can think of the properties of an object as being
the "members" of said object. Properties with values that are not functions are
also known as the **fields** of an object.

In JavaScript, variable names must follow the traditional, C-styled rules for
variable names:
* They can only contain alpha-numerical characters, underscore, and the dollar
  sign
* They cannot begin with a number
* They cannot be a reserved JavaScript keyword

Property names, on the other hand, are less restrictive. In fact, property
names can be any valid string, even the empty string! Turns out, what we have
been using up until this point to access fields, something along the lines of
`someObj.itsProperty`, is called **dot notation**. There is another, called
**bracket notation**, that can give us access to properties unavailable through
dot notation:

```JavaScript
var test = {};
test[""] = "Empty string";
test["123"] = "numerical string";
test["hey two spaces"] = "string with spaces"
test[9381] = "number converted to string";
test[{foo: 0}] = "object converted to string";

console.log(test[""]);
console.log(test["123"]);
console.log(test[123]); // 123 is converted into string "123"
console.log(test["hey two spaces"]);
var acc = "hey two spaces";
console.log(test[acc]); // Variables work too!
console.log(test["9381"]);
console.log(test[{foo: 0}]);
acc = {foo: 0};
console.log(test[acc]);
```
Output:
```
Empty string
numerical string
numerical string
string with spaces
string with spaces
number converted to string
object converted to string
object converted to string
```

Using bracket notation, JavaScript will interpret whatever is inside of the
brackets as an expression. It will take the value yielded by said expression
and try to convert it into a string. As such, it is possible to put things like
numbers and variables inside of the brackets and have it work. None of the
above techniques are possible with dot notation. In fact, using dot notation,
it is only possible to access properties whose names also happen to be valid
variable names.

In a sense, then, the fields accessible through dot notation is a proper subset
of the fields accessible through bracket notation.

An important feature of bracket notation is the ability for us to use variables
to access object properties. This provides us with the means of *traversing*
all of the fields of an object using a (for lack of a better name)
`for ... in ...` loop (Note: This is also why objects are also referred to as
*associative arrays*):

```JavaScript
var foo = {alpha: 1, beta: "two", charlie: {three: 3}};
for( field in foo ) {
  console.log(foo[field]); // Print out string representations of all fields
}
```
Output:
```
1
two
{three: 3}
```

### Object Methods

**Methods** are properties of an object which happen to hold a function value.
For example, we can have:

```JavaScript
var foo = {doSomething: function() { console.log("Hello"); }};
foo.doSomething();
foo["doSomething"]();
```
Output:
```
Hello
Hello
```

In the above example, we say that `doSomething()` is a method of the object
`foo`. Note that in order to call the function, we could use either dot
notation or bracket notation, though the latter is rather awkward. More will be
said about methods (and functions in general, for that matter), but for now,
just note that every method also has a special variable, called `this`, that
acts as a "pointer" to the calling object:

```JavaScript
var foo = {name: "Foo",
           fun: function() { console.log(this.name); }};
foo.fun();
```
Output:
```
Foo
```
As opposed to:
```JavaScript
var foo = {name: "Foo",
           fun: function() { console.log(name); }};
foo.fun(); // For some reason, an empty string is printed.
```

Note that it is also possible to do something like this:
```JavaScript
function fun() {
  console.log(this.name);
}
var alpha = {name: "Alpha"};
var beta = {name: "Beta"};
alpha.fun = fun;
beta.fun = fun;
alpha.fun();
beta.fun();
```
Output
```
Alpha
Beta
```

## JavaScript Object Creation

### Object Initializer/Literal Notation

There are several ways you could create an object in JavaScript. So far, we
have been using what is known as an **object initializer** in order to create
our objects. This method is also known as creating objects with *literal
notation*.

```JavaScript
var obj = {name: "Foo", id: 123};
```

In the above example, `obj` is a variable that holds the value of the
expression to the right of the assignment operator. On the right side of the
assignment operator is the *object initializer*. It is an expression which
produces an object with the fields `name` and `id`. One can also consider this
an object literal as it is "literally" an object.

The object initializer does not have to be stored into a variable; it can also
be, for example, passed inline as an argument to a function:
```JavaScript
function printName(obj) { console.log(obj.name); }
printName({name: "Billy"});
```
Output:
```
Billy
```

### Constructor Functions

The next way to create objects is to use the `new` keyword and a special type
of functions called **constructor functions**. Below is an example of this
method:

```JavaScript
function Cat(name, breed, age) {
  this.m_name = name;
  this.m_breed = breed;
  this.m_age = age;
}

var samuel = new Cat("Samuel", "Tabby", 4);
```

In the above example, `Cat()` is a constructor function that, when used with
the `new` keyword, creates objects. Note the use of the `this` keyword inside
of the `Cat()` function; when `Cat()` is called with `new`, `this` will point
to a newly created object. As such, each individual `this.fieldName = value`
will create a field inside of the newly created object and assign it with the
given value. Unless a specific return statement exists inside of `Cat()`, the
expression `new Cat(...)` will produce a newly created object with fields and
values as per defined in the `Cat()` function.

We can then work with `samuel` as we would any other object:
```JavaScript
console.log(samuel.m_name + " is a " + samuel.m_age + "-year-old "
            + samuel.m_breed + " cat.");
```
Output:
```
Samuel is a 4-year-old Tabby cat.
```

Note that when not used with the `new` keyword, the `Cat()` constructor
function behaves just like any normal, top-level function. This means that the
`this` inside of the function will point to the "default" top-level object:

```JavaScript
var kaiser = Cat("Wilhelm", "German", 2);
console.log(kaiser); // undefined, because "Cat()" doesn't return anything
// Instead, certain global variables are defined:
console.log(m_name);
console.log(m_breed);
console.log(m_age);
```
Output:
```
undefined
Wilhelm
German
2
```

Usually, it doesn't make much sense to call a constructor function without the
`new` keyword. Likewise, you probably wouldn't want to call non-constructor
function with the `new` keyword, either. As such, it is a *convention* to
**capitalize** constructor functions to distinguish them from normal functions.

The advantage of constructor functions is that it allow us to repeatedly create
*similar* (Same fields, different values) objects:

```JavaScript
var abdul = new Cat("Abdul", "Persian", 5);
var james = new Cat("James", "Bengal", 1);
var lilWayne = new Cat("Lil Wayne", "Bombay", 5);
```

For this reason, it is *convention* to call `Cat()` the **constructor** for the
"Cat" **class** and to call `abdul`, `james` and `lilWayne` **instances** of
the class.

### Object.create()

There is another way to create objects, using a method of the predefined object
`Object` called `create()`. This function is the most useful when you want to
create a distinct object that is "similar" to a pre-existing object. Consider
the following:

```JavaScript
var sam = {name: "Sam", age: 4};
// I want a *distinct* object that is "similar" to sam
var bob = sam; // This doesn't work, bob points to sam
bob.name = "Bob";
console.log(bob.name);
console.log(sam.name);
```
Output:
```
Bob
Bob
```

Because `bob` is a pointer to `sam`, the above doesn't not achieve our goal of
having a separated, similar object. Hence, we have to use `Object.create()`:

```JavaScript
var sam = {name: "Sam", age: 4};
var bob = Object.create(sam);
console.log(sam.name);
console.log(bob.name);
console.log(sam.age);
console.log(bob.age);
```
Output:
```
Sam
Sam
4
4
```

Using `Object.create()`, the resulting object `bob` has all of the properties
of `sam`. In addition, we can add properties and "change" existing properties
of `bob` without affecting `sam`:

```JavaScript
bob.name = "bob";
bob.age = 10;
bob.job = "student";
// sam still stays the same
console.log(sam.name);
console.log(sam.age);
```
Output:
```
Sam
4
```

So how does `Object.create()` actually work? Well, truth be told, when I said
that `bob` is similar to `sam`, what I really meant was that `bob` is a new
object that has `sam` as its **[[Prototype]]**. For more information about
`Object.create()` and *JavaScript Object Inheritance* in general, consult the
separate document.

## Getters/Setters

In addition to regular properties, JavaScript offers the `get` and `set`
keywords to give objects pseudo-properties and (perhaps more importantly), a
degree of encapsulation over its data.

### Getters

In JavaScript, we can use the `get` keyword to create a pseudo-property for an
object. This idea is perhaps best illustrated with an example:

```JavaScript
var foo = { get val(){ return 5; } };
console.log(foo.val); // Prints "5"
```

In the example above, we have used `get` to create the pseudo-property `val`
inside of our object `foo`. Notice that the `get` keyword is followed by a
*function* definition. Additionally, notice that the identifier of said
function becomes the identifier of our pseudo-property. After defining `val`,
we can access it just as if it were any regular property (ie. by using
`foo.val`). Note that even though `val` was defined "like" a function, it
"behaves" like a property. That is to say that the following will generate an
error:

```JavaScript
foo.val(); // ERROR: foo.val is not a function
```

When I say that `val` is a pseudo-property, what I mean is that you can access
`val` as if it were any property and the expression `foo.val` will return a
value just as any other property would. However, under the hood, what you are
really doing is running the function `val()`, defined inside of `foo`. If the
function `val()` does not return anything, then the expression `foo.val` yields
`undefined`.

In addition, note that by default (And I don't know if you can actually change
this), `val` is *enumerable* (More on *enumeration* in the separate document on
object inheritance, by the way):

```JavaScript
for( propertyName in foo ) {
  console.log(foo[propertyName]);
}
```
Output:
```
5
```

JavaScript does not allow you to change getters. Any attempts to set the value
of `foo.val` will be **ignored**. This gives the object a degree of
encapsulation over the pseudo-property `val`.

```JavaScript
foo.val = 10;
foo.val = 55;
foo.val = function() { return 10; };
console.log(foo.val); // Still 5
```

If you are using a getter, make sure that the identifier of the getter is
**unique**. If you accidentally (Or intentionally) define another property, with
the same name as the pseudo-property created by the getter, the *last*
definition *shadows* all of the previous:

```JavaScript
var bar = { get myVal() { return 10; },
            myVal: 15 }; // actual property shadows getter
var foo = { myVal: 25,
            get myVal() { return 20; } }; // getter shadows actual property
var rix = { get myVal() { return 30; },
            get myVal() { return 40; } }; // getter shadows getter

console.log(bar.myVal); // 15
bar.myVal = 100; // Mutating actual property
console.log(bar.myVal); // 100

console.log(foo.myVal); // 20
foo.myVal = 100; // Nothing done
console.log(foo.myVal); // 20

console.log(rix.myVal); // 40
rix.myVal = 100; // Nothing done
console.log(rix.myVal); // 40
```

### Setters

The `set` keyword allows you to define a function that is executed whenever the
identifier of the function is used in an assignment operation; the value to the
right of the assignment operator will become the sole argument to the function.
Confused? Let's see an example:

```JavaScript
var foo = { set val(arg) { console.log("Arg: " + arg); } };

foo.val = 10;
console.log(foo.val);
```
Output:
```
Arg: 10
Undefined
```

In the example above, notice that when we tried to assign a value to `foo.val`,
what actually happens is we run the function `val()` and pass in 10 as
argument. Furthermore, notice that `foo.val` evaluates to `undefined` since we
have never actually defined a property called `val` inside of `foo`.

JavaScript prevents

## JavaScript Object Inheritance

See Separate Document.

## Et Cetra

The following are some additional information related to JavaScript objects
that I couldn't find an appropriate section to stick into. So here they are.

### JavaScript is Pass-by-Value

JavaScript is pass-by-value, forever and always. But, that may not be saying a
whole lot when JavaScript "values" are in fact references. To be more specific,
*all object variables* in JavaScript hold a reference to their object.

This is aptly demonstrated by the following:

```JavaScript
var num = 3131; // num holds the value 3131
var numCopy = num; // the value 3131 is copied to numCopy
num = 120; // Does not affect numCopy
numCopy = 3913; // Does not affect num
console.log(num); // prints 120
console.log(numCopy); // prints 3913

var bar = {val: 3131}; // bar is a reference to an object object
var barCopy = bar; // The reference is copied
bar.val = 100; // DOES affect barCopy
console.log(bar.val); // 100
console.log(barCopy.val); // 100 as well

var foo = {val: 190};

function change(arg1, arg2, arg3) {
  arg1 = 909; // Changes local
  arg2.val = 391; // Changes object referenced by arg2
  arg3 = {val: 200}; // Points arg3 to something else
}

change(num, bar, foo);
console.log(num); // 120 still
console.log(bar.val); // Changed to 391
console.log(foo.val); // 190 still
```

One last thing to mention is the idea of equivalence. The JavaScript operators
`===` and `==` are both used to test for equality between different expressions
in JavaScript. The first is what is known as *strict-equality* operator while
the latter is the *loose-equality* operator. When it comes to objects, though,
both perform exactly the same way.

So how do they actually perform, then? As it turns out, the equality operator,
when used between objects, does an *address* comparison. That is to say that
two object variables are considered equal if and only if they are referencing
the same piece of memory:

```JavaScript
var foo1 = {val: 100};
var foo2 = {val: 100};
var bar = foo1; // bar references the same section in memory as foo1

console.log(foo1 === foo2); // False
console.log(bar === foo1); // True
```

### JavaScript Functions and `this`

Let's get something straight: JavaScript functions *ARE* objects. They inherit
from other objects, have their own properties (Including methods) and are held
as references by variables. One can consider them the same as regular objects,
with the added feature that when the variable that holds them is called with
a pair of parenthesis (Which may or may not contain arguments), cool,
function-like stuff happens.

```JavaScript
function func(arg) {
  console.log(arg);
}

// func IS an object, and all functions inherit from Function.prototype
console.log(Object.getPrototypeOf(func) === Function.prototype); // True
func.call(null, 5); // Using a method of func inherited from Function.prototype

// We can even give func properties if we wished
func.makesNoSense = "Ayy lmao"; // Given func a field
func.saySomething = function() { console.log(this.makesNoSense); } // A method
func.saySomething();

var func2 = func; // func2 REFERENCES the object func references
// Changing a property of the underlying object changes both of them
func2.makesNoSense = "Jet fuel can't melt steel beams";
func2.saySomething();
func.saySomething();
```
Output:
```
True
5
Ayy lmao
Jet fuel can't melt steel beams
Jet fuel can't melt steel beams
```

Earlier, we have shown that JavaScript has a keyword called `this` that is a
reference to an object. Depending on the runtime context, the object pointed to
by `this` changes.

When used in the top-most, global context, `this` references the "global"
object. In most (If not all) web browsers today, this global object is the
`window` object:

```JavaScript
console.log(this === window);
this.alpha = "alpha";
console.log(window.alpha);
```
Output:
```
True
alpha
```

When the `this` keyword appears inside of a function, however, the object it
points to is determined at runtime, depending on the calling context. If the
function is called outside of the context of any object (ie. not as a method),
`this` will refer once again to the global object. If not, it will refer to the
calling object:

```JavaScript
function fun() {
  console.log(this.someVariable);
}

someVariable = "Global"; // Setting the global object's someVariable

var foo1 = {someVariable: "Foo1"};
var foo2 = {someVariable: "Foo2"};

foo1.m_fun = fun;
foo2.m_fun = fun;

fun();
foo1.m_fun();
foo2.m_fun();
```
Output:
```
Global
Foo1
Foo2
```

The above example deserves a little bit more explanation. First, we created a
variable called `fun` which holds a reference to a function (Which is, as
you'll recall, an object). Then, we created two objects, `foo1` and `foo2`. For
each object, we created a property called `m_fun` and gave them the value held
in `fun`. This means that `foo1.m_fun` and `foo2.m_fun` both have a reference
to the same function that `fun` has a reference too. **NOTE:** Throughout this
entire process, there is only **one** function value in memory. Following, we
call the function three times, but each time, we are calling the function in a
different context: global, foo1 and foo2. The runtime is smart enough to figure
out what the context is in each instance and change the value of `this` to
match. As a result, we get the above output.

One last thing to mention about functions themselves concerns the way in which
they are defined. You see, there are two different ways in which you can define
a function (Well, there's one more, which is by using the pre-defined function
`Function`, but that will not be discussed here. For more info, visit
[this website] [Site 1]). I like to call one *function-like* and the other one
*variable-like*:

```JavaScript
function foo() {
  console.log("This is function-like");
}

var bar = function() {
  console.log("This is variable-like");
}
```

For the most part, these two syntaxes are equivalent to one another, but there
is a small difference that should be pointed out. JavaScript is an interpreted
language. This means that when an *interpreter* is given a JavaScript file, it
will go through the file line by line, from the top to the bottom. As a result,
if the identifier (name) of a variable is used before it is defined, an error
will result:

```JavaScript
console.log(foo()); // ERROR! "foo" doesn't exist yet!

var foo = function() { return "foo"; };
```

This is *expected* behavior. But notice that in the above, the function `foo`
is defined using a variable-like definition. This was intentional; had we
defined `foo` using a function-like definition, this error would not arise:

```JavaScript
console.log(foo());

function foo() { return "foo"; }
```
Output:
```
foo
```

Wow, why does that work? Well, as it turns out, when you define a function
using a function-like definition, the identifier for the function is
**hoisted**. This is just a fancy way of saying that you can use the function
even before the interpreter has come across the definition. If you are familiar
with C++ (and similar languages), imagine that the JavaScript interpreter helps
you out by **forward-declaring** all functions defined in the form of
`function function_name(...) {...}`.


[Site 1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
