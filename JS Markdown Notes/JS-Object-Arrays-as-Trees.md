# JavaScript - Object Arrays as Trees

An array of JavaScript objects can be used to create representations of
**trees**, as long as all of the the following conditions are satisfied:
* Every object has a field which contains a unique value that may be used to
  identify the object within the array
* Every object has two fields, each either:
  * contains the identification value of another object within the array
  * contains a value representing *nothing*
* There are no cyclical links

For a concrete example, consider the following array:
```JavaScript
var foo = [{name: "A", left: "B", right: "C"},
           {name: "B", left: "E", right: "F"},
           {name: "C", left: "G", right: "H"},
           {name: "E", left: null, right: null},
           {name: "F", left: null, right: null},
           {name: "G", left: null, right: null},
           {name: "H", left: null, right: null}];
```

As simple and meaningless as the above example seems, it really is a tree. But
to actually treat it and traverse it like a tree, we need another function,
call it `byName()`, which can accept an identifying value and return the object
associated with it:
```JavaScript
function byName(id) {
  for( var i = 0; i < foo.length; i++ ) {
    if( foo[i].name == id ) {
      return foo[i];
    }
  }
  return null;
}
```

This seems fine at first glance, but a major issue with it is the fact that it
operates in O(n) time. This means that as soon as our tree becomes a little
larger, or if the operations we wish to do are a little beyond the most basic
of complexities, this function will make our programs a lot slower.

As such, a better solution would be to perform a one-time operation at the
start of the program, in which we create a "map" of identifications to their
objects:
```JavaScript
function makeMap(array) {
  var map = {}; // Empty object
  for( var i = 0; i < array.length; i++ ) {
    map[array[i].name] = array[i];
  }
  return map;
}
```
As we can see, our "map" really is just an object that whose name-value pairs
corresponds with the identification values and actual objects within our array.

Used with the array from previously, we can make an object that will provide us
with much faster access to the elements we want:
```JavaScript
var foomap = makeMap(foo);
console.log(foomap.A); // Constant time!
console.log(foomap.L); // Gives undefined
console.log(foomap.A.left); // Gives the name of A's left
console.log(foomap[foomap.A.left]); // Gives A's left
```
Output:
```
{name: "A", left: "B", right: "C"}
undefined
B
{name: "B", left: "E", right: "F"}
```
**NOTE:** Using this technique, trying to access something which does not exist
will give `undefined`, but since `==` equates `undefined` and `null`, in most
situations, this will work out. Nevertheless, this is something to be cautious
of.

### Functions On Trees

For further discussions on trees, we will formalize the structure followed by
`foo` from the previous example. From here on out, we will make the following
definitions:

Every *node* is an object with:
* A field called 'name', which contains a string
* A left sub-*tree*
* A right sub-*tree*

Every *tree* is either:
* `null`
* A *node*

Suppose now that we want to create a string representation of our `foo` tree
that is as follows:
```
[A [B [E [] []] [F [] []]] [C [G [] []] [H [] []]]]
```

To go about doing this, we will create two **mutually-recursive** functions,
called `stringifyNode()` and `stringifyTree()`:
* `stringifyNode()` has two parameters, `map` and `node`. The value of `map`
  is an mappings object that contains all of the nodes in our map, an example
  of this the previous `foomap` variable. The value of `node` is the name of
  the node we wish to make a string representation for.
* `stringifyTree()` has two parameters as well, `map` and `tree`. The value of
  `map` is the same as the previous function. The value of `tree` is either a
  node or `null`.

Here is a concrete implementation of both functions:
```JavaScript
function stringifyNode(map, node) {
  return "[" + node.name + " " + stringifyTree(map, map[node.left])
             + " " + stringifyTree(map, map[node.right])  + "]";
}

function stringifyTree(map, tree) {
  if( tree == null ) { // Also true if tree is `undefined`
    return "[]"; // Return empty string
  }
  else {
    return stringifyNode(map, tree);
  }
}
```

The Usage:
```JavaScript
console.log(stringifyTree(foomap, foomap.L));
console.log(stringifyTree(foomap, foomap.A));
console.log(stringifyTree(foomap, foomap.B));
console.log(stringifyTree(foomap, foomap.C));
```
Output:
```
[]
[A [B [E [] []] [F [] []]] [C [G [] []] [H [] []]]]
[B [E [] []] [F [] []]]
[C [G [] []] [H [] []]]
```

So what exactly is happening? Well, the two functions, as previously mentioned,
are mutually recursive. This means that the result of one is dependent on the
result of the other and vice versa. Of these two functions, `stringifyTree()`
is the *driver* function: It is the one that is called to actually initiate the
"stringification" process on a certain node.

Once it is called, `stringifyTree()` will attempt to create a string for the
tree it was given. We know that a tree is either `null` or a node, and we know
how to create a string representation in both cases:
* If the tree is `null`, just return `[]`. This is the base case.
* If the tree is a node, call and return the result of another function called
  `stringifyNode()` which will be responsible for creating strings for nodes.

`stringfyNode()`, on the other hand, only has one case to worry about. Its sole
purpose is to create a string for a valid node, and hence should not have to
worry about invalid input. We know how to create a string representation of a
node; it is described in the body of the function.

As it turns out, when working with and operating on binary trees, this pattern
of having two mutually recursive functions appear frequently. Consider another
example: Suppose we wish to know how many nodes there are in a tree.

For this, once again, we must have one function which strictly deals with
*trees*, and another which strictly deals with *nodes*:

```JavaScript
function numNodesTree(map, tree) {
  if( tree == null ) {
    return 0;
  }
  else {
    return numNodesNode(map, tree);
  }
}

function numNodesNode(map, node) {
  return 1 + numNodesTree(map, map[node.left])
           + numNodesTree(map, map[node.right]);
}
```

Since this pattern appears so often, we would be amiss to not try and come up
with an **abstraction** for it. Before we start though, we should also
generalize definition of *trees* and *nodes* a little bit more:

A *tree* is either:
* Nothing (For our purposes, we will strictly use `null` and `undefined`)
* A *node*

A *node* can be any data structure as long as:
* It has a field named `left` which is a *tree*
* It has a field named `right` which is also a *tree*

We will also, for simplicity, assume that all of the trees we are dealing with
are indeed *valid*, which is to say they they satisfy all of the conditions
outlined at the top of this document.

Here's the abstraction that we will make. We will create a new method called
`reduceTree()` which will have the following parameters:
* `map`, whose argument will be an object that contains a mapping of node names
  to the actual nodes themselves
* `treeName`, whose argument will a string that is the name of the tree we
  would like to reduce
* `dVal`, whose argument is the default value if the tree is *nothing*
* `combine`, whose argument is a function that:
  * Must have three parameters
  * Know how to take a node and combine it with the result of calling
    `reduceTree()` on the two subtrees of said node.

Here is the concrete implementation of `reduceTree()`:
```JavaScript
function reduceTree(map, treeName, dVal, combine) {
  if( map[treeName] == null ) {
    return dVal;
  } else {
    return combine(map[treeName],
                   reduceTree(map, map[treeName].left, dVal, combine),
                   reduceTree(map, map[treeName].right, dVal, combine));
  }
}
```

With this function written, in order to accomplish our previous tasks, here's
what we have to do:
```JavaScript
// Printing out a string rep
var stringrep = reduceTree(foomap, "A", "[]",
                           function(node, lResult, rResult) {
                             return "[" + node.name + " " + lResult + " "
                                        + rResult + "]";
                           });
// Count number of nodes
var count = reduceTree(foomap, "A", 0,
                       function(node, lResult, rResult) {
                         return 1 + lResult + rResult;
                       });
```

It it important to note that this works for most basic tree operations, but
more complicated tasks may be beyond the abilities of this function. For
example, this function works well when the `combine` function has no conditions
based on the value of `lResult` and `rResult`. But if how we combine the
results of the left child, right child and the current node were a little bit
more complicated, things could get messy with this function.
