# CPP - The Explicit Keyword

C++ is a strongly typed language. That means that if you are working with
objects of a certain type, then the language typically does not allow you to
objects of another type in the same place.

There are many exceptions to this rule. When an object of one specific type is
used somewhere another type is expected, C++ can accommodate for this via *type
conversions*.

Type conversions occur frequently. A typical example is as follows:
```cpp
int foo = 5 + 10.0;
```

The example, although seemingly simple, contains two implicit conversions:
* Firstly, the operator `+` is overloaded by the language to either accept,
  among other things, two `int`s or two `double`s. The literal `5` is of type
  `int`, while the literal `10.0` is of type `double`. The `+` operator is not
  overloaded to accept different types of operands. Hence, an implicit
  conversion takes place (The rules of this conversion sequence is beyond the
  scope of this discussion) and the literal `5` is converted to its value
  equivalent in type `double`. Following this conversion, the overload of `+`
  that accepts two `double`s is called. This operator returns an object of type
  `double`.
* Secondly, because we are trying to initialize an object of type `int` with an
  object of type `double`, yet another conversion is needed to convert the
  result of the the expression `5 + 10.0` to a value of type `int`.

C++ comes with a set of rules dictating how various different fundamental data
types in C++ should be converted between each other. These conversions are
collectively known as *standard conversions* and often occur implicitly.
Some examples of *standard conversions*:
* Conversions in-between all of the numerical data types: int to double, float
  to double, char to short, double to long, so on and so forth
* Conversions between a pointer of any type to an from a pointer of type
  `void*`
* Conversions of a pointer of a certain derived class to one of its base
  classes (So-called *pointer upcasting*)

However, when it comes to user-defined types, the user may can also take
advantage of the C++ conversion engine by providing what is known as
**user-defined conversion sequences**, which is just a fancy name used to refer
to both **conversion constructors** and **conversion functions**.

In the following discussions, we will focus solely on **conversion
constructors**.

In C++ (Well, up until C++11 at the very least), a **conversion constructor**
is a constructor that accepts a single parameter. If defined, C++ will use
**conversion constructors** to complete certain operations that would otherwise
be impossible due to type incompatibility.

Like so many other things in C++, the best way to illustrate this is via
examples. Consider the following class definition:

```cpp
class Money {
public :
  double amount;

  Money()
  : amount{0.0} {
    cout << "Default CTOR called." << endl;
  }

  Money(double amt)
  : amount{amt} {
    cout << "Double CTOR called." << endl;
  }

  Money(const Money &other)
  : amount{other.amount} {
    cout << "Copy CTOR called." << endl;
  }
};
```

A very simple class called `Money` has been defined. All it does is wrap a
single `double` value. We have defined three constructors for the class:
* `Money()` - This is the default constructor, so-called because it takes no
  arguments
* `Money(double amt)` - This is a constructor that accepts a argument of type
  `double`. Incidentally, this is a **converting constructor**.
* `Money(const Money &other)` - This is the copy constructor. While its
  existence may not seem relevant to our discussion yet, it will be useful down
  the road.

For each of our constructors, we are also going to be using a few print-outs to
know exactly when they are called.


### Example 1 : Simple Conversions

Suppose now that we have the following function:
```cpp
void display_balance(const Money balance) {
  cout << "The balance is: " << balance.amount << endl;
}
```

This function takes in an object of type `Money` and simply prints out its
`amount` field.

Now, suppose we went:
```cpp
Money payable{79.99};
display_balance(payable);
display_balance(49.95);
display_balance(10);
```
Our output will be:
```
Double CTOR called.
Copy CTOR called.
The balance is: 79.99
Double CTOR called.
The balance is: 49.95
Double CTOR called.
The balance is: 10
```

Isn't that interesting? Clearly, in our definition of `display_balance()`, we
have said that we would be taking in an object of type `Money`, so why did
the second and third calls to `display_balance()`, which did not passing in
a `Money` object, work? To investigate, we will go through each and every line
along with their associated outputs:

1. `Money payable{79.99}`  
   We create a `Money` object using the input `79.99`, which is a `double`
   literal. This calls the constructor `Money(double amt)`. Hence we have the
   output:  
   ```
   Double CTOR called.
   ```
2. `display_balance(payable)`  
   Here we passing in `payable`, an object of type `Money`, to
   `display_balance()`. This is the exact type that `display_balance()` needs,
   hence, everything works as expected we get the output:
   ```
   Copy CTOR called.
   The balance is: 79.99
   ```
3. `display_balance(49.95)`  
   Here we pass in `49.95`, a `double` literal, to `display_balance()`. C++
   recognizes the incompatibility in the types, and tries to convert `49.95` to
   something compatible. It sees that `Money` has exactly what it needs: A
   **conversion constructor** that it can use to make a `Money` object from
   a `double`. It uses this constructor, and then passes in the subsequent
   r-value `Money` as input to `display_balance()`. Hence we have the output:
   ```
   Double CTOR called.
   The balance is: 49.95
   ```
   **Note:** The copy constructor is not called because of copy/move elision.
4. `display_balance(10)`  
   Here we pass in `10`, an `int` literal, to `display_balance()`. C++ sees the
   incompatibility in the types. It sees that `Money` has a conversion
   constructor, but it only takes in a `double`! But wait, C++ knows how to
   convert an `int` to a `double`. Hence, it does just that. After the
   conversion, it calls the conversion constructor and hence we get the output:
   ```
   Double CTOR called.
   The balance is: 10
   ```
   **NOTE:** Even though the output `10` looks like an `int`, it is really a
   `double`. The stream insertion operator `<<` is the one that cut off the
   decimals.
