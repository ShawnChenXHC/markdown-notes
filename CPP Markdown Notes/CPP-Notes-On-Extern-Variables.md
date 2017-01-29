# CPP - Notes on Extern Variables

In C++, every variable three characteristics which define it:
* Duration
* Scope
* Linkage

The **duration** of a variable determines how long that variable stays in
existence for. All global variables (By default), have what is called *static*
duration, meaning that they come into existence as soon as the program begins
and go out of existence once the program is over.

The **scope** of a variable determines where the variable is known to exist.
All global variables, `const` and non-`const` alike, have what is called file
scope. This is to say that their existence is known only within the file in
which they have been *declared*. This, incidentally, is also the highest level
of scope possible.

The **linkage** of a variable is only relevant once multiple files are involved
in the compilation process. The linkage of a variable determines whether or not
if the variable can be *exported* to other files. By default, non-`const`
global variables have *external* linkage, meaning that if the identifier of the
variable is found declared in other files, they will refer to the same variable.

**Duration** is fairly simple to understand, but **scope** and **linkage** can
cause a fair deal of confusion. Additionally, `extern`, the keyword used to
give a variable external linkage, is also used to *forward-declare* variables
as well (And why would it not be? Isn't C++ fucking great?)

By default, all (non-const) global variables have **file scope** and **external
linkage**. This means that the identifier of the variable is only known within
the file in which is *defined*. However, as a result of **external linkage**, if
the same variable is *declared* elsewhere, it **MUST** refer to said global
variable. At the same time, the variable cannot be re-*defined* in another
file, else the linker will complain about multiple definitions of the same
variable.

### Example 1 : External Linkage, multiple definitions

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

int global1 = 10;
```

This will not compile. `extern1.cc` and `extern2.cc` both provide a *definition*
of the global variable `global1`. Because global variables have external linkage
by default, there is a conflict.

### Example 2 : Extern Keyword as An Unnecessary Qualifier

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1 = 10;
```

This will not compile. The keen eye will notice that this example is largely
the same as the previous with the exception of the `extern` keyword which has
been added to the front of the definition for `global1` found in `extern2.cc`.
The line:
```cpp
extern int global1 = 10;
```
*defines* a integer variable named `global1` and initializes it with a value
of 10. The `extern` keyword is unnecessary as non-const global variables have
**external linkage** by default (In fact, the addition of the `extern` will
actually generate a warning).

### Example 3 : Extern Keyword as a Forward Declaration

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

int global1 = 10;
```

The above code will compile, and the output will be:
```
10
```

The difference between this example and example 1 is in `extern1.cc` and on the
line:
```cpp
extern int global1;
```
Where an addition of an `extern` keyword has changed the statement from a
*definition* of a global variable to a *forward-declaration* of a global
variable. In this instance, all that the line does is bring the variable
`global1` into the scope (Read: Declare its existence) of the file `extern1.cc`
for usage.  
The variable `global1` is actually defined in `extern2.cc`, but because the
variable has **external linkage**, if the same variable is *declared*
elsewhere, it will refer to this *definition* of the variable.

As it turns out, the presence of `=` makes a big difference in what `extern`
does:
```cpp
int global1; // Definition of variable "global1"
extern int global1; // Forward-Declaration of variable "global1"

int global2 = 0; // Definition of variable "global2"
extern int global2 = 0; // STILL a definition of variable "global2"
```

### Example 4 : No Forward Declaration

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

//extern int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

int global1 = 10;
```

The above will not compile. This is because the variable `global1` is not within
the scope (Read: Is not known to exist) within the file `extern1.cc`. Sure, a
global variable with external linkage called `global1` is defined in
`extern2.cc`, but outside of this file, unless there is a forward-declaration,
no other file knows about the existence of said global variable.

### Example 5 : No Forward Declaration

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

//extern int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1 = 10;
```

The above will not compile. An `extern` keyword has been added to the definition
of `global1` in `extern2.cc`, but because of the equals sign, this actually
makes no difference. This example is exactly the same as Example 4.

### Example 6 : No Definition

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

//extern int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1;
```

The above code will not compile. Although the same issue from examples 4 and 5
is also present in this one, namely, that `extern1.cc` does not know about the
variable `global1`, there is also another problem with this example. Inside of
`extern2.cc`, the line concerning `global1` is no longer a definition: It is
just a forward declaration! Hence, `global1` is never defined. This issue is
more acutely demonstrated below.

### Example 7 : No Definition

```cpp
// extern1.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1;

int main() {
  cout << global1 << endl;
}

// extern2.cc
#include <iostream>
#include <string>
using namespace std;

extern int global1;
```

The above will not compile. Here, the previous issue of `extern1.cc` not knowing
about `global1` is solved. The addition of a forward-declaration within the file
has brought the variable into scope. However, neither `extern1.cc` nor
`extern2.cc` bothers to actually define the variable. Hence, we will get an
error of an undefined variable.
