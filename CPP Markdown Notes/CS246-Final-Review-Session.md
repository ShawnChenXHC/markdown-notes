# CS246 Final Review Document

## Section 1

1. The big 5 have been declared in this class definition above. Which is which?

   **Answer:**  
   ```cpp
   ~Node(); // Destructor
   Node(const Node& other); // Copy ctor
   Node& operator=(const Node& other); // Copy assignment operator
   Node(Node &&other); // Move ctor
   Node& operator=(Node &&other); // Move assignment operator
   ```

2. What is an r-value reference and why do move semantics require it?

   **Answer**  
   An r-value reference is variable of the form `Type &&identifier`, where
   `Type` is the data type of the r-value the variable is referring to. An
   r-value reference is required as a part of move semantics in order to
   distinguish itself from copy operations because the signatures for copy and
   move constructors/assignment operators are the same aside from the parameter
   list.

3. What is an l-value reference? When can you not use references?

   **Answer**  
   An l-value reference is a variable of the form `Type &identifier`, where
   `Type` is the data type of the l-value the variable is referring to.
   An l-value has an address, and therefore you cannot create (non-const)
   l-value references to things which do not have an address, such as literal
   values or other types of r-values.

4. What happens when the copy constructor is not defined?

   **Answer**  
   When the user does not define a copy constructor, a default is provided by
   the compiler. The default copy constructor makes the new object by calling
   the copy constructor of the base class using the base of the other object,
   and then by performing a shallow copy of the fields of the other object into
   the newly created object.

5. How are move semantics different from copy semantics?

   **Answer**  
   Copy construction/assignment presumes that the object being copied from
   persists after the operation is complete. As such, copy operations must
   preserve the integrity of data in the object being copied from. In other
   words, copy construction/assignment does not alter the data of the object
   from which the copy is made. Move construction/assignment, on the other
   hand, cares less about the data integrity of the source object. It presumes
   that the source object is obsolete after the operation, and hence has no
   problem with changing the data in said object during the operation.

6. What is a deep copy? What is a shallow copy?

   **Answer**  
   When a class has member variables which are pointers, there are two different
   types of copies which one can create from objects of said class. The first
   of these types is a *shallow* copy. In a shallow copy, the values of pointer
   variables is exactly the same as the values of the pointers from the source
   object. That is to say that if the source object has a pointer to some data
   somewhere else, then shallow copies of the source object will also have
   pointers pointing to the same data. *Deep* copies, on the other hand, do not
   have pointers to the same place as their source object. Pointers in deep
   copies of an object will instead point to other parts of memory, but the
   composition of the parts they point to will be identical to the memory
   pointed to by the pointers of the source object.

7. What is pass-by-value? What is pass-by-reference?

   **Answer**  
   When data is transferred in a program, pass-by-value is the practice of
   making copies of the thing being transferred and giving off the copies
   rather than the thing itself. Pass-by-reference, on the other hand, is the
   practice of giving off addresses, or "ways", to get to the thing that is
   being transferred. For example, suppose you had a function that takes in an
   integer. If the function that the following signature:  
   ```cpp
   void takeAnInt(int a);
   ```
   Then calling `takeAnInt(5)` will create a copy of the value 5 and pass it
   in to the function. However, if the function had the following signature:  
   ```cpp
   void takeAnInt(int &a);
   ```
   Then calling `takeAnInt(a)` will pass in the variable `a` as a C++ reference.

8. For Q1.5, which is more efficient? Why?

   **Answer**  
   Moving is usually more efficient than copying, especially if the class owns
   memory on the heap in the form of pointers. When an object owns memory on
   the heap, copy operations involving said object must create an exact replica
   of the memory the object points to. Whereas with move operations, no such
   creation occurs and instead, the pointer is just copied over by value and
   the pointer in the source object is given the value `nullptr` to prevent
   accidental deallocation.

9. Why do we declare some operators outside of the class?

   **Answer**  
   Operators declared within the class, the so-called *member operators*, all
   have an implicit first argument which is the `this` pointer. Because the
   first argument is also what goes on the left side of the

10. What is a stream?

    **Answer**  
    A stream is, in simple and unprofessional words, a sequence of data which
    can be accessed in some "more or less" ordered fashion and may sometimes be
    manipulated. In C++, input streams are usually sequences of characters which
    one may read in information from in a FIFO way. Output streams, on the other
    hand, are sequences of data which may be written to in order to produce
    some output to either the standard output, a file, or some other terminal.

11. What is separate compilation? And why do we need include guards?

    **Answer**  
    When a program gets large, it is unwieldy and impractical to write have
    everything that make up the program inside of one file. Hence, good practice
    dictates that multiple files should be used to split up the program into
    multiple, logical modules which then can be "pieced together" to make the
    program. Separate compilation is the process of compiling the separate
    components of a program individually, and then linking them together at the
    very end to make the complete program.

    C++ allows multiple declarations of an a variable, but only one definition.
    Because we often need to `#include` the definitions of classes from other
    header files, it is very likely that, with larger programs, we will
    accidentally define the same class multiple times in the same file. Without
    include guards, this will cause compilation to fail.

12. What is the difference between command-line arguments and standard input?

    **Answer**  
    Command-line arguments are passed in along with the command name when the
    command is executed within the terminal:
    ```
    $ somecmd arg1 arg2 arg3
    ```
    Standard input is provided to the program after its execution has already
    begun after a command call:
    ```
    $ somecmd
      first line of input
      second line of input
    ```

13. What are `new` and `delete`?

    **Answer**  
    `new` and `delete` are keywords used in C++ to allocate and deallocated
    memory on the heap.

14. What is overloading? What is operator overloading?

    **Answer**  
    Function overloading is the technique of writing different functions that
    have the same name, but differ in terms of their parameter lists. This
    is done when one wishes to extend the functionalities a certain, well,
    function, to multiple different types without having to come up with
    different names for each of the different variants. Function overloads
    must have parameters that are different from each other in one or more of
    the following ways:
    * Number of parameters
    * Types of parameters
    * Ordering of the types of parameters

    There are times when the signature of a function passes as an overload of
    another function, but an "ambiguous call" error occurs when one actually
    tries to call the function.

    Operator overloading is much the same as function overloading. C++ offers
    a set of operators that work things such as integers, pointers and other
    fundamental data types. These operators are implemented via functions.
    Sometimes, one may find it intuitive to create versions of these operators
    for their own user-defined types. In these situations, they may overload
    these operators to behave in whatever way they desire(Though it is highly
    recommended that they behave somewhat analogous to their original, language
    defaults).

15. How do you declare overloaded functions? When can you not overload a
    function?

    **Answer**  
    To declare a function overload, one simply declare a new function that
    shares the same name as a previously declared function, ensuring that the
    parameter list of the newly declared function is *sufficiently* different
    from that of the previous one.  
    The return type is not considered a part of the function signature when
    overload is intended. That is to say that if two functions are meant to be
    overloads of each other but only differ on their return types, a compilation
    error will result.

16. What is the difference between `class` and `struct`?

    **Answer**  
    In C++, the only difference between a `class` and a `struct` is the default
    access specifier of the members. By default, the members of a class declared
    with the `class` keyword are private. In contrast, the default access
    specifier of members of a class declared with the `struct` keyword is
    public.

17. What does the `explicit` keyword do in C++?

    **Answer**  
    The `explicit` keyword, when used

18. What does the keyword `extern` do in C++? In what situations is it useful?

    **Answer**  
    The `extern` keyword is primary used in two different capacities:
    1. To forward declare a global variable
    2. To provide external linkage to a global variable

    To deal with the first case, there may be situations where one wishes to
    utilize a single definition of a global variable across multiple files. In
    these circumstances, one would typically have the definition of the variable
    in the `.cc` file of the associated module and provide a forward declaration
    in the `.h` file, as illustrated:  
    ```cpp
    // module.cc
    int globalNum = 365;

    // module.h
    extern int globalNum;
    ```
    This way, any file which `#include`s `module.h` will have access to the
    variable `globalNum` defined in `module.cc`.

    To deal with the second case, there may also be situations where one wishes
    to allow other files to access constants defined within a file. Since
    constants have internal linkage by default, an `extern` keyword is needed
    in order to provide other files access to it, as illustrated:
    ```cpp
    // module.cc
    extern const int SOME_CONST = 2361912;
    // module.h
    extern const int SOME_CONST;
    ```
    This way, any file which `#include`s `module.h` will have access to the
    variable `SOME_CONST` defined in `module.cc`.

19. What is cascading?

    **Answer**  
    Operator cascading is the technique of chaining together a sequence of
    operator calls into a single expression. This is only achievable if
    the operator used is implemented in such a way that the type of the value
    returned is the same type as the arguments which the operator accepts.
    Examples of cascading can be found in basic arithmetic expressions:
    ```cpp
    1 + 2 + 3 * 5;
    ```
    The above expression is synonymous to `(1 + (2 + (3 * 5)))` and only works
    because the output of `operator+` and `operator*` are both integers
    themselves.
    Operator cascading is also possible with the insertion `<<` and extraction
    `>>` operators:
    ```cpp
    cout << "Hello " << "World!" << endl;
    cin >> foo >> bar; // Where foo and bar are string variables or something
    ```

20. What is Member Initialization List? Why is it useful?

    **Answer**  
    
