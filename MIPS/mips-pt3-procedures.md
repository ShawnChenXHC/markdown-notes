# MIPS Pt3 - Procedures and Functions

The idea of **procedures**, or, interchangeably (At least for the extent of
this document) **functions**, is a fundamental concept of programming. It is
the idea that if there is a sub-task that is repeated over and over again in
the process of accomplishing a greater task, then there should be some sort of
way to "save" that sub-task and to "call" it repeatedly, rather than having to
write it over and over again.

In *Computer Organization and Design*, Patterson and Hennessy presents a
wonderful description for procedures, using spies as an analogy. A spy is
someone who carries out a task for a client:
* The client gives the spy just enough information so that the spy can complete
  their task.
* The spy goes out to do their thing, making sure to cover their tracks along
  the way so that their cover won't be blown.
* The client does not (Nor necessarily want to) know the methods of the spy.
* At some point, the spy will come back with the results of its mission.

Similarly, a procedure is something that can carry out a task for its caller:
* The caller gives the procedure just enough information so that the procedure
  can complete its task. This information is often called *parameters*
* The procedure goes to do its thing. It will acquire whatever resources it
  needs to complete its job. It will make sure that it "covers its tracks" by
  returning borrowed resources to their original states before it is done.
* The caller does not necessarily know how the procedure accomplishes its task.
* At some point, the procedure returns with its results.

Every high-level language implements this idea in one way or another. In C++,
a function looks something like this:

```cpp
int add(int a, int b) {
  return a + b;
}
```

And "calling," or, if you prefer, "using," the function looks something like
this:

```cpp
int a = 10;
int b = 20;
int c = add(a, b); // c will be 30
```

In MIPS, the concept of procedures is implemented using a combination of
special registers, control flow instructions and a data structure called the
stack.

We will begin with the introduction of a few special registers:
* `$a0` - `$a3` (Which are registers 4 - 7) are the so called *argument
  registers*
* `$v0` - `$v1` (Registers 2 and 3) are the *return value registers*
* `$ra` (Register 31) is the *return address register*
* `$sp` (Register 29) is the *stack pointer register*

The first of these we will look at at is `$ra`. This register is special as it
is used in direct conjunction with the `jal` instruction. The `jal` (Jump and
Link) instruction is a J-format instruction which updates the PC to the
constant
