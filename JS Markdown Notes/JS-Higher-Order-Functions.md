# JavaScript - Higher Order Functions and Abstractions Over Lists

**NOTE:** In this document, the terms "array" and "list" are used
interchangeably.

Very often, when we are working with lists, certain families of operations
appear over and over again. For example, suppose you had a list and you wanted
to:
* Go through the list and log the value of every element
* Go through the list and log the type of every element

Typically, to accomplish each of the above, you would have to have two distinct
functions something the lines of:

```javascript
function logEachVal(array) {
  for( int i = 0; i < array.length; i++ ) {
    console.log(array[i]);
  }
}

function logEachType(array) {
  for( int i = 0; i < array.length; i++ ) {
    console.log(typeof(array[i]));
  }
}
```

For usage, you may do something along the lines of:
```javascript
var someArr = [1, 2, {foo: "foo"}, "cats", "dogs", function() {}];

console.log("These are the values in someArr:");
logEachVal(someArr);
console.log("These are the types in someArr:");
logEachType(someArr);
```
This gives the following output:
```
These are the values in someArr:
1
2
{foo: "foo"}
"cats"
"dogs"
function (){...}
These are the types in someArr:
number
number
object
string
string
function
```

We see that in both operations, there is a common *theme*. Namely, that we are:
* Going through the entire list and calling a function on each element

Even the code suggests that there is a lot of common ground in-between the two
functions: By inspection, we can see that the only real difference between them
is the fact that one calls `console.log(array[i])` while the other one calls
`console.log(typeof(array[i]))`.

This is cause for us to seek some sort of **abstraction** over these two
functions. In programming, **abstraction** is basically the process of finding
*generalizations* over otherwise distinct things. For the above example, an
abstraction would be the creation of a function, call it `forEach()`, that
takes in an array and a function. `forEach()` simply goes through the array it
is given, and applies its given function to each and every element:

```javascript
function forEach(array, action) {
  for( int i = 0; i < array.length ; i++ ) {
    action(array[i])
  }
}
```

We now have a function that is much more versatile and capable than our
previous two functions. Note that we can still accomplish what we wanted to do
before using `forEach()`, albeit with a little bit more work:

```javascript
var someArr = [1, 2, {foo: "foo"}, "cats", "dogs", function() {}];

console.log("These are the values in someArr:");
forEach(someArr, console.log)
console.log("These are the types in someArr:");
forEach(someArr, function(each){ console.log(typeof(each)); });
```

Taking advantage of the fact that functions in JavaScript are just values, we
are able to pass in the *action* we would like to have done on each element of
an array to the function `forEach()` along with the array itself, and
`forEach()` dutifully applies our *action* to each element.

As it turns out, `forEach()` is available to us a standard member function of
any array in JavaScript. The `forEach()` function takes in a function value as
argument and, for every element in the array that calls it, runs the function
by passing in three arguments (In the exact order) :
* The value of the element
* The position of the element
* The array itself

The behavior of this function is aptly demonstrated in this following example:
```javascript
var someArr = [2, 4, 6, 8, 10];

someArr.forEach(console.log);
```
The above gives the following output:
```
2 0 [2, 4, 6, 8, 10]
4 1 [2, 4, 6, 8, 10]
6 2 [2, 4, 6, 8, 10]
8 3 [2, 4, 6, 8, 10]
10 4 [2, 4, 6, 8, 10]
```

If you don't the last two arguments, you may do:
```javascript
someArr.forEach(function(value){ console.log(value); });
```

### Higher-Order Functions

Perhaps a bit informally, we call functions that operate on other functions,
either by taking them in as arguments or by returning them, as **higher-order
functions**, a name which draws inspiration from analogous concepts in
Mathematics.

Because JavaScript treats functions as values, it facilitates the usage of
higher-order functions quite nicely. We have already seen how functions can be
passed as arguments to other functions, so in the following examples, we will
focus instead on functions that return other functions. This, in essence,
allow us to write functions that "make" other functions:

```javascript
function identicalTo(measure) { // Function that makes other functions
  return function(test) { return test === measure; };
}

var equalsTen = identicalTo(10);
// function equalsNine = identicalTo(9); // Error, incorrect syntax
console.log(equalsTen(10)); // True
console.log(equalsTen(10.0)); // True, all Numbers in JS are 64-bit floating
console.log(equalsTen("10")); // False, type non-matching
console.log(equalsTen([10])); // False, type non-matching
```

Now suppose we want to make a function called `noisy(f)` that takes in another
function `f` and returns a new function that is exactly the same as `f` except
for the addition of some print print statements that show the arguments `f` was
called with along with the result.

In order words, we want:
```javascript
function someFunc(...) {...}

var noisySomeFunc = noisy(someFunc);

noisySomeFunc(...);
// We want the output to be:
// Calling someFunc with arguments ...
// Got result:
// ...
```

This function could be very helpful debugging. But nevertheless, we could write
`noisy(f)` in the following manner:

```javascript
function noisy(f) {
  return function(arg) {
    console.log("Calling function with argument: ", arg);
    console.log("Function's printouts (If any):");
    var result = f(arg);
    console.log("Function's result: ", result);
    return result;
  }
}
```

To show that this works:
```javascript
var noisyFirstOf = noisy( function(array) { return array[0]; } );
var first = noisyFirstOf([1, 2, 3]);
console.log(first);

var noisyIncrement = noisy( function(num) { num++; } );
var five = 5;
var result = noisyIncrement(five);
console.log(five);
console.log(result);
```
Output:
```
Calling function with argument: [1, 2, 3]
Function's printouts (If any):
Function's result: 1
1
Calling function with argument: 5
Function's printouts (If any):
Function's result: undefined
5
undefined
```
The output of the first example is fairly straightforward. To explain the
output of the second example:
* Recall that JS passes by value. For primitive data types, the value of the
  data is copied over to the variables in the parameters. Hence, `five` is
  never changed.
* The function passed to noisy, the increment function that is, does not return
  anything. In these circumstances, the "return value" of the function becomes
  `undefined`

The keen eye will recognize that as it currently stands, `noisy()` has (at
least) one major flaw: It is non-compatible with functions that accept/except
more than one argument:
```javascript
var noisyAdd = noisy( function(opA, opB) { return opA + opB; }  );
noisyAdd(5, 6);
```
Output:
```
Calling function with argument: 5
Function's printouts (If any):
Function's result: NaN
```
What happened? Due to the implementation of `noisy()`, `noisyAdd()`, even
though it looks like it received two arguments (5 and 6), only passed the first
argument off to its underlying function! In JavaScript, the default "value" of
all function parameters is `undefined`. Hence, what happened in the underlying
function is that `opA` got the value `5` while `opB` got the value `undefined`.
An addition operation is attempted on these two variables, which results in
`NaN`.

The fix, as it turns out, is fairly simple. Recall that in JS, every function
has a member called `arguments`, which is an array-like object containing all
of the arguments that the function was called with. Functions have another
member, called `apply(thisArg, [argsArray])`, that we can use to call the
function with an array acting as the arguments list. Here is a demonstration:

```javascript
function add(opA, opB) { return opA + opB; }
add(5, 6); // 11
add([5, 6]); // WEIRD behavior! JS seems to to a string concatenation of
             // "[5, 6]" and "undefined"
add.apply(null, [5, 6]); // 11, don't worry about the "null" for now
```

As such, we can now rewrite `noisy()` to work with functions that take in any
number of arguments:

```javascript
function noisy(f) {
  return function() {
    console.log("Calling function with argument list:");
    console.log(arguments);
    console.log("Function printouts (If any):");
    var result = f.apply(null, arguments);
    console.log("Function's result: ", result);
    return result;
  }
}
```

Usage:
```javascript
var noisyAdd = noisy( function(opA, opB) { return opA + opB; } );
noisyAdd(5, 6);
```
Output:
```
Calling function with argument list:
{0: 5, 1: 6}
Function printouts (If any):
Function's result: 11
```

### JSON Files

JSON, which stands for JavaScript Object Notation, is type of text formatting
that closely mirrors JavaScript's objects syntax. JSON-formatted text is either
the string representation of a single JavaScript object or, possibly, an array
of JavaScript objects. For example:
```
[{"name": "Billy", "age": 10},
 {"name": "Troy", "age": 14},
 {"name": "Sally", "age": 9},
 {"name": "Muhammed", "age": 9},
 {"name": "Lin", "age": 11}]
```

JavaScript has an object called `JSON` that contains a number of different
methods which allow us to convert JSON-formatted strings to JavaScript Objects
and back again. These methods, respectively, are `JSON.parse()` and
`JSON.stringify()`:

```JavaScript
var jsonString = '[{"name": "Billy", "age": 10}, \
                   {"name": "Troy", "age": 14}, \
                   {"name": "Sally", "age": 9}, \
                   {"name": "Muhammed", "age": 9}, \
                   {"name": "Lin", "age": 11}]'; // '\' to escape newlines
var parsedObject = JSON.parse(jsonString);
console.log(typeof parsedObject);
console.log(parsedObject);

var gtr = {name: "GTR", make: "Nissan", price: 125500};
var stringRep = JSON.stringify(gtr);
console.log(typeof stringRep);
console.log(stringRep);
```
In the above example, in order to make `jsonString` a little bit easier to
read, we have used `\` to escape the newlines so that we can split the string
up across different lines. Generally speaking, this is not a good idea and
strings should really all be on one line. Nevertheless, the result of the above
code is:
```
object
[{name: "Billy", age: 10}, {name: "Troy", age: 14}, {...}, ...]
string
{"name":"GTR","make":"Nissan","price":125500}
```

Moving on, however, we will keep using the `jsonString` from the previous
example to demonstrate the other abstract list functions that JavaScript offers
as default methods for all array.

#### filter()

First up is `filter()`. `filter()` takes in a single function value as
argument. This function value, let's call it `test()`, **MUST** return a
boolean value. `filter()` will then go through every element in its calling
array, passing the value of each as the first argument to `test()`.

Here is a sample (Probably not real) implementation of filter as a method:
```JavaScript
function myFilter(test) {
  var passed = [];
  for( var i = 0; i < this.length; i++ ) {
    if( test(this[i]) ) {
      passed.push(this[i]);
    }
  }
  return passed;
}
```
Note that the method does **not** change its calling array. Instead, it
creates a new array, adds to it the elements which "passed" `test`, and returns
the new array.

Here is an example of the method in action:
```JavaScript
var parsedArray = JSON.parse(jsonString); // jsonString defined previously
var tenAndOlder = parsedArray.filter(function(element) {
                                        if( element.age >= 10 ) {
                                          return true;
                                        }
                                        return false;
                                     });
console.log(JSON.stringify(tenAndOlder));
```
Output:
```
[{"name":"Billy","age":10},{"name":"Troy","age":14},{"name":"Lin","age":11}]
```

#### map()

Next up is the `map()` method. Like `filter()`, the `map()` method will take in
a single function value as argument, call this function `transform()`. `map()`
goes through the elements of its calling array, passing in each one as the
argument to `transform()`. It then takes the value returned by `transform()` to
build a new array.

As with before, here is a sample implementation:
```JavaScript
function myMap(transform) {
  var mapped = [];
  for( int i = 0; i < this.length; i++ ) {
    mapped.push(transform(this[i]));
  }
}
```

What happens when `transform()` doesn't return anything? You get an array of
`undefined`s:
```JavaScript
var parsedArray = JSON.parse(jsonString);
var mapped = parsedArray.map(function() {});
console.log(mapped);
console.log(JSON.stringify(mapped));
```
Output:
```
[undefined, undefined, undefined, undefined, undefined]
[null,null,null,null,null]
```
For some reason, it seems like `JSON.stringify()` will turn `undefined`s into
`null`s.

Working with the same `jsonString` as before, suppose we don't really care
about the age of all the students on the list and only want the names:

```JavaScript
var parsedArray = JSON.parse(jsonString);
var justNames = parsedArray.map(function(element) {
                                  return element.name;
                                });
console.log(JSON.stringify(justNames));
```
This gives the output:
```
["Billy","Troy","Sally","Muhammed","Lin"]
```

#### reduce()

More often than not, there is little value in having a list in its entirety,
especially if the list is large and unwieldy. In these situations, we would
like to work with some smaller, more manageable representation of the data the
list contains. For example, the *mean*, *median* and *mode* are three values
which represent a list.

For this purpose, we have the `reduce()` method. Unlike the previous two
methods, which both produced a new array from an older one, the `reduce()`
methods serves a more "condensational" purpose. Although the method *can* still
produce new lists, it is much more than that.

As previous, here is a sample implementation of the method:
```JavaScript
function reduce(combine, start) {
  var current = start;
  for( var i = 0; i < this.length; i++ ) {
    current = combine(current, this[i]);
  }
  return current;
}
```

The `reduce()` method has two parameters, a function value called `combine()`,
and a starting value of some type, called `start`. These two parameters can be
given any arguments as long as the following is satisfied:
* `combine()` must accept two arguments
* `combine()` must return a value
* `combine()` must combine start with the first value of the array to create a
  new value, which it will then use to combine with the second value of the
  array, so on and so forth.

Going back to the `jsonString` of prior examples, suppose we want a string that
contains all of the names of the students, separated by commas. How would we go
about doing that?

The key idea here is we are trying to coalesce our entire array into a single
value, some singular representation of the data it contains. Whenever this is
the case, there is a strong incentive to use the `reduce()` function:

```JavaScript
var parsedArray = JSON.parse(jsonString);
var allNames = parsedArray.reduce(function(thusFar, next) {
                                    return thusFar + next.name + ",";
                                  }, "");
console.log(allNames);
```
This gives the output:
```
Billy,Troy,Sally,Muhammed,Lin,
```

The function `combine()` provided to `reduce()` tells it *how* to combine the
previous result with the next element in the list. In the case, we are saying
the result thus far, whatever it may be, should be concatenated with the value
of the `name` field of the next element, along with an additional comma. The
second argument to `reduce()` is the empty string, which is used to combine
with the first element in the array.

When `reduce()` is called with only one argument, which is supplied as the
value to `combine()`, it assumes that the value of the first element in the
array is to be used as the value for `start`:

```JavaScript
var someNums = [2913, -2, -4912, 30131, 19, 192];

// We wish to find the greatest value in someNums
var greatest = someNums.reduce(function(thusFar, next) {
                                 if( next > thusFar ) {
                                   return next;
                                 }
                                 return thusFar;
                               });
console.log(greatest);
```
