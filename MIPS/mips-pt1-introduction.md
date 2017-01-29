# MIPS Pt1 - An Introduction

This is going to be my notes on the assembly language MIPS. I am writing these
notes as I go through the book "Computer Organization and Design, 5th Ed" by
Patterson and Hennessy.

## What is MIPS

To talk about MIPS, we must first talk about **instruction sets**. An
instruction set, also known as an instruction set architecture, is a collection
of a statements which a processor can directly understand. Different processors
follow different instruction sets. In other words, the instruction set of a
processor is the "language spoken" by that processor. For this reason, we have
also come to call instruction sets *"machine languages"* (Source?)

There are lots of instruction sets in existence, but the most popular ones
today are Intel's *x86* and ARM's *ARMv7* (Along with its various siblings).
You will find, however, that instruction sets often have more similarities than
differences. Why? Well, over the years, as hardware design developed, good
ideas slowly became best practices, which themselves slowly became fundamental
principles. Because machine languages work directly with the hardware, as
hardware design consolidated over a few key fundamental principles, so too did
the design of machine languages.

**MIPS**, short for *"Microprocessor without Interlocked Pipeline Stages"*, is
an instruction set architecture. It is similar to ARMv7.

## Arithmetic in MIPS

We will begin our exposure to MIPS with a look at how we can do arithmetic with
it.

To add numbers in MIPS, we use `add`. `add` requires exactly three operands,
each of which is a "variable". For now, just assume that a "variable" is the
name of a place which can hold a numerical value. More rigorous definitions are
on the way.

Here is a simple example. Assume that `a`, `b` and `c` are variables:

```
add a, b, c
```

The above code will add the values stored in `b` and `c` and store the result
inside of `a`.

## MIPS Register

The above example is actually not valid MIPS code. As in, if you were to
actually "run" (whatever that means) it, you would encounter an error. This is
because, compared to high-level languages, the operands to `add` permitted by
MIPS is *very* restrictive. MIPS does not know what `a`, `b` or `c` is, and
this is not because we haven't "defined/declared" the variables to MIPS. This
is because the concept of "variables" (As we know them in high-level languages)
*does not exist* in MIPS.

Okay, are you ready? Repeat after me: The operands to `add` can **ONLY** be
from the *register*.

What does that mean? Well, first, we need to talk about what a **register** is.
Think of a register as being a little cubby hole in the *processor* (Not
memory) which can be used to store a Binary value. In the MIPS architecture (As
in, an environment "running" MIPS), a register is **32 bits**, or 4 bytes, in
size. As a "side" note, the size "32 bits" shows up so often in MIPS that we
have come to call it a **word**.

Anyhow, for reasons which may or may not be discussed later, there are a total
of **32 registers**. We can access them using the keywords: `$0` to `$31`. A
quick example to see them in action:

```
add $1, $2, $3
```

The above code will add the contents of register `$2` and `$3` and store the
results inside of register `$1`.

Besides the keywords `$0` to `$31`, there are also other names that we can use
to access and modify the 32 registers of MIPS. These are:

| Register Number(s) | Alternative name(s) |
| ------------------ | ------------------- |
| 0                  | $zero               |
| 1                  | $at                 |
| 2-3                | $v0-$v1             |
| 4-7                | $a0-$a3             |
| 8-15               | $t0-$t7             |
| 16-23              | $s0-$s7             |
| 24-25              | $t8-$t9             |
| 26-27              | $k0-$k1             |
| 28                 | $gp                 |
| 29                 | $sp                 |
| 30                 | $s8/$fp             |
| 31                 | $ra                 |

In practice, the alternatives names are often more favorable than the numerical
names (`$0` - `$31`) as they tell give you information about the *purpose* of
the register. This highlights another important point: Not all registers are
created equal! In fact, each register/range of registers in the table above
serve a different purpose. For example, `$0` is a register which only holds the
value 0; any attempts to change the contents of this register will be ignored!

For the purposes of this document, though, we will continue using the numerical
names, starting at `$1` and working our way up should we need more spaces. It
is outside the scope of this document to discuss in detail the specific
purposes of the different registers.

In C, it is the job of the compiler to associate variable names with registers.
Suppose that during the compilation of a certain C file, the compiler has
associated the variables `f`, `g`, `h`, `i`, `j` with the registers `$1` to
`$5`. Suppose now that the compiler comes across the code:

```C
f = (g + h) - (i + j);
```

How would the compiler translate this into MIPS code? Well, we are going to
need to use some extra registers to store the intermediate values because, as
you will remember, MIPS can only perform one operation per line. Furthermore,
we should also introduce the `sub` operation as we need to perform a
subtraction operation.

Fortunately the `sub` operation is very similar to the `add` operation:
```
sub $1, $2, $3
```
The above will subtract the contents of `$2` by the contents of `$3` and store
the result inside of register `$1`.

With that said, we can now compile the above C code:

```
add $6, $2, $3 # Use register $6 to store the intermediate value (g + h)
add $7, $4, $5 # Use register $7 to store the intermediate value (i + j)
sub $1, $6, $7 # Store the result of the subtraction in $1
```

## A Little More Arithmetic

There are, of course, more arithmetic instructions in addition to `add`. The
most obvious one is probably `sub`, which, as one might expect, is used to
perform a subtraction:

```
sub $1, $2, $3
```

The above will subtract the contents of `$2` by the contents of `$3` and store
the results inside of `$1`.

Like `add`, the operands to `sub` must be from the registers as well. There is,
however, a variation of the add instruction that allows you to give a constant
as an operand.

The `addi` instruction looks like the following:

```
addi $1, $2, 100
```

The above will take the contents of `$2`, add the literal constant 100 to it,
and store the result inside of `$1`. There is no `subi` instruction, however.
Why? Because `addi` can perform a pseudo-subtraction:

```
addi $1, $2, -100
```

Therefore, according to good design principles, there is no need to have a
dedicated `subi` instruction when `addi` is perfectly capable of achieving the
purposes of both.

## Data Transfer Operations

As you have probably realized, if the registers are MIPS' only way of storing
values, then the things we can do with MIPS would be very limited indeed. It is
not hard to imagine a data structure in C whose size exceeds that of the 32
4-Byte registers we are provided with.

As such, in the MIPS architecture, in addition to the 32 registers found in the
processor, there is also the **memory**. More will be said about memory in the
future, but right now, just think of it as being this abstract place where bits
can be stored to and read from. The memory is a long array of bits. However,
since bits are a little bit too small to be of any practical use, the memory is
actually addressed by individual *bytes*. This means that accessing the first
place in memory would be to access the first byte (first eight bits) in memory.

In order to *read* from the memory, we have the `lw` (load word) operation. Its
syntax is as follows:

```
lw $2, 8($1)
```

It takes in two operands. The first operand is the register which the bytes
read will be stored. The second is a constant followed immediately by a
register in parenthesis. The constant is called an **offset** while the
register in the parenthesis is known as the **base register**. The load word
operation goes to the address specified jointly by the base register and the
offset, copies the contents of the next *word*, and stores the copy inside of
the first register.

In the above example, the operation goes to the address found inside of `$1`,
advances by 8 bytes, copies the *word* stored at that address (The contents of
the next four bytes), and stores the word inside of register `$2`.

As you can see, the MIPS architecture does not actually provide us with a
native way of accessing individual bytes; `lw` reads a *word* at a time. This
is because MIPS uses something called an **alignment restriction**. In the MIPS
architecture, words in memory must start at addresses which are multiples of 4.
This means that when we are accessing data in memory, we are always accessing
addresses which are multiples of 4, and we are always reading in 4 bytes at a
time. Why this design is beneficial is beyond the scope of this document.

This means that in the above example `lw $2, 8($1)`, assuming that the address
stored in `$1` is a multiple of four, we are accessing the *second* word after
the word at the address stored in `$1`.

Forming a pair with the `lw` is the `sw` (store word) operation. Its syntax and
workings parallel that of `lw`:

```
sw $1, 32($2)
```

The above copies the word stored inside of register `$1` and stores it inside
the word whose address is 32 bytes after the address stored inside of register
`$2`.
