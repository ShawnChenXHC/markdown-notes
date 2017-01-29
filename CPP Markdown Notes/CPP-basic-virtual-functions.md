# C++ Basic Virtual Functions Notes

C++ virtual functions differ from standard functions in one way. When a virtual
function is invoked, the program will attempt to resolve to the most derived
definition of the function it can find.

This concept is best illustrated with several examples.

### Example 1: No Virtual

Consider the following code

```cpp
class A {
public :
  int m_val;

  A(int a)
  : m_val{a} {}

  void print() const {
    cout << "A: " << m_val << endl;
  }
};

class B : public A {
public :
  int m_val;

  B(int b)
  : A{10}, m_val{b} {}

  void print() const {
    cout << "B: " << m_val << endl;
  }
};

int main() {
  A foo = A{10};
  B bar = B{10};

  foo.print();
  bar.print();
}
```

The output of the above example is:
```
A: 10
B: 10
```
This is a fairly straight-forward result. B inherits from A, but it has its
own definitions `m_val` and `print()`, which in turn shadows that of the ones
found in A.  
As a result, `foo.print()` invoked the definition of `print()` found inside
of the A class. `foo` is an A object with the value of 10 for its `m_val`,
hence, `A: 10` is printed. Likewise, `bar.print()` invoked the definition of
`print()` found inside of the B class. `bar` is a B object with the value of 10
for its `m_val`, hence, `B: 10` is printed.

Had B not defined its own `print()`, then calling `bar.print()` will have
resolved to calling on the definition of `print()` found in the A class. In
this case, the function will be invoked with the A part of `bar`, and thus
something along the lines of `A: 10` will be printed (Since every B object
initializes its base class with the value 10).

### Example 2: Object Splicing

Using the same example as above, suppose now that we changed the `main()`
definition as follows:

```cpp
int main() {
  A foo = A{5};
  A bar = B{20};

  foo.print();
  bar.print();
}
```

The results of the above is:
```
A: 5
A: 10
```
The behavior of `foo` is as expected, but the behavior of `bar` can be a little
unsettling. As it turns out, what we have just done is an example of
**object slicing**.

The thing is, `bar` identifies an A object. As such, `bar`
occupies enough memory to store an A object, nothing more, nothing less.
But because B inherits from A, we can make an A object with a B object. When
this happens, the program constructs a B object, throws away everything in it
that is not a part of the "A part" of the object, and sticks it inside of the
memory allocated. Hence, `bar` ends up being nothing more than an A object with
the value 10 for its `m_val`.

It is important to know that even if we made `print()` virtual in both classes,
this will not have an effect if object slicing occurs. Object slicing
literally takes a constructed object and "slices" away any parts that cannot
fit within the type of the identifier. Hence, in the above example, `bar` is,
in every practical aspect, an A object. It is NOT a B object.

**NOTE:** Object splicing is *not* an implicit conversion!

To demonstrate this point, we will use the `explicit` keyword to create an
explicit copy constructor for the A class:

```cpp
class A {
public :
  // ... Same class implementation as before
  explicit A(int a)
  : m_val{a} {}

  explicit A(const A &other) // Copy constructor has been made explicit
  : m_val{other.m_val} {}
};

int main() {
  // The below are all okay if both the copy constructor and
  // one-int constructor are non-explicit
  A foo1 = 5; // Not okay if ctor/copy made explicit
  A foo2{5};
  A foo3 = A{5}; // Not okay if copy made explicit
  A foo4{A{5}};
  A bar = B{20}; // Not okay if copy made explicit
  A bar2{B{20}};
}
```

### Example 3: Pointers to Objects

When you have an object of some derived type, that object *is an* instance of
of its superclasses. As such, we may create pointers and references to the
superclass of said object using the object itself.

Example:
```cpp
A myA{4}; // Create an A object
B myB{5}; // Create a B object

A *foo = &myB; // This is fine
A &bar = myB; // This is fine as well
```

We have already seen from earlier what will happen if you tried to invoke
the `print()` function on myA and myB. But what will happen when you go:
```cpp
foo->print();
bar.print();
```
Well, the output, as it turns out, is:
```
A: 10
A: 10
```

The object that `foo` and `bar` are pointing too are the same, and it is true
that the underlying object is indeed that of type B. But, the pointer
themselves are of type A. As such, they only know about the parts of `myB` that
are a part of the A portion. In other words, when `foo->print()` and
`bar.print()` are called, they are invoking the definition of `print()` found
within the A class. Within that definition, `m_val` also resolves to the `m_val`
found within the A class. As such, we get the above output.

This, in most cases, is not the desired behavior. Hence, we have
**virtual functions**.

### Example 4: Basic Virtual Functions

Our desired behavior is to be able to refer to a collection of A and B objects
by pointers and references of type A. But at the same time, when we call upon
`print()`, we would like to have the B objects execute `B::print()`.

For this cause, we will make the `print()` function inside of the A class
**virtual**.

```cpp
class A {
  //... Same as before
  virtual void print() const {
    cout << "A: " << m_val << endl;
  }
};

class B : public {
  //... Same as before
};
```

By doing so, notice what happens now when we go:
```cpp
A foo{4}; // Create an A object
B bar{5}; // Create a B object

A *fooptr = &foo;
A &fooref = foo;
A *barptr = &bar; // This is fine
A &barref = bar; // This is fine as well

fooptr->print();
fooref.print();
barptr->print();
barref.print();
```
The output is:
```
A: 4
A: 4
B: 5
B: 5
```
What happened? Well, when you call upon a virtual function, the program will
resolve the call to the *most derived* version of the function possible.

To be more specific, as with before, `barptr` is a pointer of type A. So when
`barptr->print()` is called, the compiler will still resolve that as a call to
`A::print()`. At runtime, however, it is recognized that `barptr` is actually
pointing to a B object. Since `A::print()` is a virtual function, the most
derived version of `print()` is called, which, in this case, is `B::print()`.

**More will be said about how virtual functions actually accomplish this**

In the above example, we say that `B::print()` **overrides** `A::print()`.
Traditionally, whenever a function call is resolved by the compiler to
`A::print()`, the definition provided in the body of `A::print()` is ran. But
because `A::print()` is *virtual*, depending on the actual runtime situations,
there is a chance that `B::print()` is ran instead.=

### Example 5: Overriding Functions are Virtual

Consider the following class hierarchy:
```cpp
class A {
  //... Same as previous
  virtual void print() const {
    cout << "A: " << m_val << endl;
  }
};
class B : public A {
  //.. Same as previous
  void print() const override {
    cout << "B: " << m_val << endl;
  }
};
class C : public B {
public :
  int m_val;

  C(int c)
  : B{10}, m_val{c} {}

  void print() const override {
    cout << "C: " << m_val << endl;
  }
};
```
We have added another class, C, which inherits directly from B. Note that
neither `B::print()` nor `C::print()` has the `virtual` keyword.

Suppose now we went:
```cpp
C cat{6};

A *aptr = &cat;
A &aref = cat;
B *bptr = &cat;
B &bref = cat;

aptr->print();
aref.print();
bptr->print();
bref.print();
```
This produces the following output:
```
C: 6
C: 6
C: 6
C: 6
```

Note two things:
1. Even though `B::print()` is not marked virtual, referring to `cat` through
   an A pointer/reference and then calling `print()` will still execute
   `C::print()`
2. Even though `B::print()` is not marked virtual, referring to `cat` through
   a B pointer/reference and then calling `print()` will still execute
   `C::print()`

This should be plenty evidence to support the fact that **any function that is
an override is also itself virtual**. For this reason, in the above example,
because `B::print()` is an override of `A::print()`, it itself is also a
virtual function. Thus when `bptr->print()` or `bref.print()` is called,
`C::print()` is still executed, the lack of an explicit `virtual` in front of
`B::print()` be damned.

For this reason, it is **good practice** to put `virtual` in front of all
overriding functions. Though it is not strictly necessary, it does go a long
way in prevent accidental errors in the future.

### Example 6: A Break in the Chain

Consider the following class hierarchy:
```cpp
class A {
  //... Same as previous
  virtual void print() const {
    cout << "A: " << m_val << endl;
  }
};
class B : public A {
  // Same as previous, with the exception that there is no longer a
  // definition for B::print()
};
class C : public B {
  //... Same as previous
  virtual void print() const override {
    cout << "C: " << m_val << endl;
  }
};
```

In this example, we have removed the definition of the `print()` function
found inside of the B class. Everything else remains exactly the same as the
previous example.

Once again, suppose we tried:
```cpp
C cat{6};

A *aptr = &cat;
B *bptr = &cat;

aptr->print();
bptr->print();
```
We still get the output:
```
C: 6
C: 6
```
When we call `aptr->print()`, the compiler resolves this as a call to
`A::print()`. As per usual, because it is virtual, during runtime, the call
executes the "most derived" function of matching signature between the A class
and C class. This, ultimately, runs `C::print()`. Here, we see that the fact
that the B class lacks any definition for the `print()` function has no effect
on runtime resolution of virtual functions.  
When we call `bptr->print()`, the compiler sees that the B class lacks a
definition for `print()`. Hence it goes up one step in the class hierarchy and
resolves to a call to `A::print()`. The rest is the same as above.

Furthermore, suppose we went:
```cpp
B bar{4};
A *aptr = &bar;
bar.print();
aptr->print();
```
We would get the output:
```
A: 10
A: 10
```
The B class does not have a definition for `print()`, hence identifier
resolution (At compile time), goes up one step in the class hierarchy, and
resolves to a call to `A::print()`. Then, because `A::print()` is virtual, at
runtime, the program finds the "most derived" function of matching signature
between the A class and B class. This, ultimately, runs `A::print()`. Hence the
output.

## Extra Tips and Notes:

* Do not call virtual functions inside of constructors or destructors, at least,
  do not call virtual functions belonging to the same class hierarchy.
  * Any function called within the constructors and destructor of any class do
    not participate in runtime virtual resolution. They still participate in
    compilation identifier resolution, but if this process resolves to a
    virtual function either in the class itself or in one of its superclass,
    the body of said virtual function will be ran, no exceptions(probably).
