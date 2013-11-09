# R For Programmers

A programming language without scalars? Where the expressions 2, 2[1], 2[1:1],
and 2[[1]] all return the same thing (a vector containing the number 2)? With
not one, but two object systems? Where identifiers may (and often do) contain
periods and fields are accessed either with $ or @ (depending on the object
system)? Where function arguments are passed by value but evaluated lazily?
Where a humble assignment to an element of a vector may create a deep copy (and
logically always does)?  Where any object may have arbitrary attributes (one of
which being the list of class names in one of the object systems)?

That otherwise looks like "normal" language of the C-family, with curly braces
and the usual control structures? And which also happens to be the de-facto
standard for statistics and allows to create sophisticated diagrams with a few
lines of code?

What kind of language is this? R is one of these programming languages that
first look deceptively easy, but reveal more and more complex and interesting
concepts as you dig deeper. In this sense, R is similar to JavaScript and other
"scripting languages" that sit on top of specialized applications.

## Vectors

Let's start with the "no scalars" concept and the subscript operators by
evaluating an integer literal in the R shell (R's REPL - Read Eval Print Loop).

    > 10
    [1] 10

As the echo indicates, the integer literal 10 is interpreted as a vector with a
single element (index 1 - statististians start counting at 1). A closer look at
the type reveals that the integer was actually turned into a double.

    > typeof(10)
    [1] "double"

The ``typeof`` function returns the name of the type used by R internally. In
this case, the value is stored in a structure that keeps the values of the
vector in a double array - ready to be passed as a pointer to efficient
numerical routines.

By default, number literals are interpreted as doubles, but you can explicitly
convert to integers.

    > as.integer(10)
    [1] 10
    > typeof(as.integer(10))
    [1] "integer"

Note, that the type is returned as a string, or, more precisely (since there are
no scalars), a vector containing a single string (call "character" in R).

    > typeof(typeof(10))
    [1] "character"

There is another notion of a type in R, the *mode*, which can be viewed as a
logical type derived from the internal type returned by ``typeof``. Doubles and
integers are both of mode "numeric".

    > mode(10)
    [1] "numeric"

There is no special syntax for creating a vector with more than one
element. Instead, the "c" (for "combine" or "concatenate") function appends all
its arguments.

    > c(1, 2)
    [1] 1 2
    > c(c(1, 2), c(3, 4), 5)
    [1] 1 2 3 4 5

As in other languages, the colon operator can be used to create integer ranges.

    > 1:5
    [1] 1 2 3 4 5

While the "no scalars" concept may seem a little odd at first, it actually turns
out to be convenient in combination with other R properties such as the
automatic repetition of vectors in operations that expect vectors of the same
length.

    > c(1, 2) + 3
    [1] 4 5

What happened here? The plus operator adds two vectors. If one of the vectors is
too short, it's repeated. If the vector contains just a single element, this
equivalent to adding a scalar. But it also works in the more general case.

    > c(1, 2, 3, 4) + c(10, 20)
    [1] 11 22 13 24

In a language that distinguishes scalars and vectors we would need to define two
functions (or four in case of a binary operator).

Many functions and operators worked element-wise when applied to vectors.

    > c(1, 2, 3) == c(1, 1, 1) + 1
    [1] FALSE  TRUE FALSE

This comes sometimes as a surprise when using boolean expressions (which R
points out as a warning message).

    > if (c(1, 1) == c(1, 2)) print("equal")
    [1] "equal"
    Warning message:
    In if (c(1, 1) == c(1, 2)) print("equal") :
      the condition has length > 1 and only the first element will be used

So, what about the subscript operators? Single brackets return a slice of an
object which is always of the same kind as the original object. In other words,
a slice of a vector is a vector, and a slice of a list (which we'll get to soon)
is a list. R is very flexible regarding the argument passed to the slice
operator. It can be the usual single integer, but also a vector of integers or a
fector of booleans (acting as a filter). Negative indexes remove elements (and
cannot be mixed with positive indexes).

    > x <- c(1, 2, 3, 4)
    > x[1]
    [1] 1
    > x[c(1, 3)]
    [1] 1 3
    > x[2:3]
    [1] 2 3
    > x[c(-1, -3)]
    [1] 2 4
    > x[x > 2]
    [1] 3 4

This example also introduced the assignment operator `<-` (you can also use the
equal sign, but the arrow is more common). This operator stores the expression
on the right hand side under the name on the left hand side in the current
environment (which is typically the global environment in the REPL or the local
environment of a function - we will take a closer look at environments later).

The double bracket subscript operator accesses elements (rather than slices) of
a container. For vectors, an "element" mean a vector of length one.

    > c(10, 20, 30)[[1]]
    [1] 10
    > c(10, 20, 30)[[1:2]]
    Error in c(1, 2, 3)[[1:2]] : attempt to select more than one element

## Functions

Before we continue investigating data structures, we should take a first look at
the dual building block of R: functions. We have used them a lot already,
because all operators are just syntactic sugar for function calls.

    > 2 + 3
    [1] 5
    > `+`(2, 3)
    [1] 5

Here, the plus operator is enclosed in back quotes so that R does not attempt to
parse it using the operator syntax. Even the subscript operators are functions.

    > `[`(c(10, 20, 30), 2:3)
    [1] 20 30
    > `[[`(c(10, 20, 30), 2)
    [1] 20

As a functional language, R treats functions as normal objects that can be
created on the fly using function literals and assigned to variables or passed
around as arguments just like any other object. A function literal consists of
the ``function`` keyword followed by the argument list and the functions body
which in R's case is simply an expression.

    > f <- function(x) 10 * x + 4
    > f(10)
    [1] 104

You may have expected curly braces around the function body, but we only need
those for sequences of expressions. In fact, curly braces are just syntactic
sugar for the ``{`` function that evaluates its arguments in sequence and
returns the value of the last one (like Lisp's ``progn`` form).

    > { x <- c(2, 4, 6); x[2] }
    [1] 4
    > x
    [1] 2 4 6
    > `{`(x <- c(2, 4, 6), x[2])
    [1] 4

R treats function arguments in an interesting "functional" way: arguments are
passed by value but evaluated lazily. To see this, let's define a simple
function that returns its second argument.

    > h <- function(x, y) { x <- 100; y }
    > h(1, 3)
    [1] 3

It also changes its first argument, but because arguments are always passed by
value, this does not change the value of a variable passed as an argument.

    > x <- 10
    > h(x, x)
    [1] 10
    > x
    [1] 10

The more interesting question is what happens if the arguments are more complex
expressions including other function calls.

    > f <- function(x) { cat("evaluating f\n"); 2 * x }
    > g <- function(x) { cat("evaluating g\n"); 3 * x }
    > h(f(1), g(1))
    evaluating g
    [1] 3
    > h(g(1), f(1))
    evaluating f
    [1] 2

R only evaluates the expressions it needs. Internally, R matches the argument
expressions to the formal arguments of the function and creates functions
objects ("promises") for each of them that are evaluated (in the calling
context) when the function uses the argument for the first time.

This means that control statements and "short circuit" operators that need
special treatment in other languages can be viewed as normal functions (even
though they may be implemented differently in R).

    > if (x > 10) f(x) else g(x)
    evaluating g
    [1] 30
    > `if`(x > 10, f(x), g(x))
    evaluating g
    [1] 30

An interesting case are assignments to parts of an object such as setting an
element of a vector.

    > x <- c(2, 4, 6)
    > y <- x
    > y[2] <- 8
    > y
    [1] 2 8 6
    > x
    [1] 2 4 6

What's happening here? First, assignements to parts of an object (such as the
slice identified with the subscript operator) are again just syntactic sugar for
the call to a function that handles this special kind of assignment. In this
case, it is the "slice assignment" function
`[<-`. So, instead of the assignment to the slice using the slice and assignment operators, we could have called the slice-assigment function `[<-` with the vector, index, and new element value as arguments and assigned the result back to the variable `y`.

    > y <- x
    > `[<-`(y, 2, 8)
    [1] 2 8 6
    > y
    [1] 2 4 6
    > y <- `[<-`(y, 2, 8)
    > y
    [1] 2 8 6

The call to the assignment version of the function is a normal function
call. Arguments are passed by value and evaluated lazily as before. This implies
in particular that the original object is not modified. The variable `y` only
gets a new value after the final assigment and the value of `x` does not change
at all.

In general, R converts an assignment with a function call (including operators)
on the left hand side to a call of the assignment version of the function, that
is, the function with the `<-` suffix followed by a normal assignment to the
first argument.

Here is an (admittedly useless, because we could do the same thing with the
subscript operator) example demonstrating how to define such a pair of access
and assignment functions. The function `f` extracts and sets the two elements
wth the indexes `i` and `j`.

    > f <- function(x, i, j) x[c(i, j)]
    > `f<-` <- function(x, i, j, value) { x[i] <- value; x[j] <- value; x }
    > y <- x
    > f(y, 1, 3)
    [1] 2 6
    > f(y, 1, 3) <- 10
    > y
    [1] 10  4 10

## Attributes

All R attributes may have "attributes". Those are just name-value pairs
associated with an object. The can be used to attach "meta data" to an
object. R's attributes may remind Lisp programmers of (symbol) properties, but
in contrast to those, R attributes are attached to the objects rather than the
names (symbols).

## Appendix: Installation

I found RStudio to be a perfect companion for R. Installation (on Ubuntu in my case) was a breeze,
and the integration of console, editor, package management, graphics, and in the help system worked
like a charm. A very impressive IDE!

