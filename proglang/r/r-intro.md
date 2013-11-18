# R for Programmers

A programming language without scalars? Where the expressions `2`, `2[1]`,
`2[1:1]`, and `2[[1]]` all return the same thing (a vector containing the number
2)? With not one, but two object systems? Where periods are common in
identifiers and fields are accessed with `$` or `@` (depending on the object
system)?

Where function arguments are passed by value but evaluated lazily? Where a
humble assignment to an element of a vector may create a deep copy (and
logically always does)?  Where any object may have arbitrary attributes (one of
which being the list of class names in one of the object systems)?

A programming language that otherwise looks like a "normal" language of the
C-family, with curly braces, normal expression and function call syntax, and the
usual control structures? And which also happens to be the de-facto standard
for statistics and allows to create sophisticated diagrams with a few lines of
code?

What kind of language is this? R is one of these programming languages that
first look deceptively easy, but reveal more and more complex and interesting
concepts as you dig deeper. In this sense, R is similar to JavaScript and other
"scripting languages" that sit on top of specialized applications.

There are plenty of tutorials showing how to use R or accomplish a specific
task. This short introduction is aimed at programmers who already know a few
programming languages and would like to quickly understand how R works (and why
it works the way it does) before learning how to use it.

Thanks to R's dynamic nature and good reflection API we can explore the language
interactively in the R shell (as in Lisp also known as the REPL or Read Eval
Print Loop).

## Vectors

As mentioned above, R does not have scalar types such as integers. If this is
true, what happens to an integer literal?

    > 10
    [1] 10

The square brackets in the R output give a first hint that R is interpreting the
integer as a vector containing a single element (at index 1 - statisticians
start counting at 1). To confirm this suspicion, we can try and apply the
subscript operator (which should be applicable to vectors).

    > 10[1]
	[1] 10

Indeed, the subscript operator works and returns the first (and only) element
which is of course wrapped in a single-element vector.

The `c` (for "combine") function is the most basic way to construct non-trivial
vectors in R.

    > c(5, 1, 22)
    [1]  5  1 22
    > c(c(2, 4), c(1, 3))
    [1] 2 4 1 3
    > c(c(2, 4), c(1, 3), 10, 20)
    [1]  2  4  1  3 10 20

This function takes any number of vectors and concatenates them. As we will see
later, `c` can combine all kinds of objects.

Let's go back and take a closer look at the type used by R to represent the
integer literal. The `typeof` function returns the name of the type used by R
internally (basically the C structure that is used in R's implementation).

    > typeof(10)
    [1] "double"

In this case, the value is stored in a structure that keeps the values of the
vector in an array of doubles - ready to be passed as a pointer to efficient
numerical routines.

By default, number literals are interpreted as doubles, but we can explicitly
convert them to integers.

    > as.integer(10)
    [1] 10
    > typeof(as.integer(10))
    [1] "integer"

Note, that the type is returned as a string, or, more precisely (since there are
no scalars), a vector containing a single string (called "character" in R).

    > typeof(typeof(10))
    [1] "character"

Besides double, integer, and character, there are two more basic (vector) types,
namely *logical* (boolean) and *complex* (complex numbers using doubles). The
two boolean literals are `TRUE` and `FALSE`, which are also available as the
values of the variables `T` and `F`, respectively.

	> TRUE
	[1] TRUE
	> typeof(quote(TRUE))
	[1] "logical"
	> T
	[1] TRUE
	> typeof(quote(T))
	[1] "symbol"
	> get("T")
	[1] TRUE

The special `quote` function here prevents the evaluation of the expression so
that we can see that `T` is not a logical literal but indeed the name (symbol)
of a variable that happens to be bound to TRUE. The `get` function gets the
value associated with a name in an environment. Both, `quote` and the "symbol"
type will be all too familiar to Lisp programmers.

Complex numbers use the mathematical notation with an `i` suffix.

	> 1i^2
	[1] -1+0i
	> (1.5 + 2.3i)^2
	[1] -3.04+6.9i
	> typeof((1.5 + 2.3i)^2)
	[1] "complex"

R's vectors are homogeneous, that is, all elements must be of the same type.
This ensures that R can store and manipulate vectors efficiently, including
passing them to external routines implemented in FORTRAN or C. When combining
vectors with the `c` function, R automatically coerces all values to the
"biggest" type where character > complex > double > integer > logical. Logical
values can be converted to the integers 0 and 1, integers to doubles, doubles to
complex numbers, and numbers to their string representation.

	> c(10, TRUE, FALSE)
	[1] 10  1  0
	> c(10, TRUE, FALSE, "foo")
	[1] "10"    "TRUE"  "FALSE" "foo"  
	> typeof(c(as.integer(1), 1.2))
	[1] "double"
	> typeof(c(1, 1i))
	[1] "complex"
	> typeof(c(1, "foo"))
	[1] "character"
	> c(1, "foo")
	[1] "1"   "foo"
	> c(1i, "foo")
	[1] "0+1i" "foo" 

There is another notion of a type in R, the *mode*, which can be viewed as a
logical type derived from the internal type returned by `typeof`. Doubles and
integers are both of mode "numeric".

    > mode(10)
    [1] "numeric"
    > mode(1.5)
    [1] "numeric"

As in other languages, the colon operator can be used to create integer ranges.

    > 1:5
    [1] 1 2 3 4 5

This is a special case of the `seq` function that generates all kinds of
"sequence" vectors, for example, with a given step size `by` or of a given
`length`.

	> seq(0, 9)
	 [1] 0 1 2 3 4 5 6 7 8 9
	> seq(from=1, to=3, by=0.2)
	 [1] 1.0 1.2 1.4 1.6 1.8 2.0 2.2 2.4 2.6 2.8 3.0
	> seq(from=0, by=0.1, length=10)
	 [1] 0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9

Another useful function in this context is `rep` which repeats a vector a given
number of times.

	> rep(1, 10)
	 [1] 1 1 1 1 1 1 1 1 1 1
	> rep(c(1, 2), 5)
	 [1] 1 2 1 2 1 2 1 2 1 2
	> rep(c(1, 2), 5, each=2)
	 [1] 1 1 2 2 1 1 2 2 1 1 2 2 1 1 2 2 1 1 2 2

In fact, R often performs this repetition automatically when operating on
vectors of different length. Let's take the sum of two vectors of different
sizes as an example:

    > c(1, 2) + 3
    [1] 4 5

What happened here? The plus operator adds two vectors. If one of the vectors is
too short, it's repeated. If the vector contains just a single element, this is
equivalent to adding a scalar. Viewing any number as a vector turns out to
generalize well in combination with this "automatic repetition" feature. But it
also works in the more general case.

    > c(1, 2, 3, 4) + c(10, 20)
    [1] 11 22 13 24

In a language that distinguishes scalars and vectors we would need to define two
functions (or four in case of a binary operator).

Many functions and operators work element-wise when applied to vectors.

    > c(1, 2, 3) == c(1, 1, 1) + 1
    [1] FALSE  TRUE FALSE

This comes sometimes as a surprise when using boolean expressions (which R
points out as a warning message).

    > if (c(1, 1) == c(1, 2)) print("equal")
    [1] "equal"
    Warning message:
    In if (c(1, 1) == c(1, 2)) print("equal") :
      the condition has length > 1 and only the first element will be used

You can resolve these situations by using the `all` or `any` functions which
return a boolean (one element logical vector).

	> all(c(1, 1) == c(1, 2))
	[1] FALSE
	> any(c(1, 1) == c(1, 2))
	[1] TRUE

What about the single and double subscript operators mentioned above? Single
brackets return a *slice* of an object which is always of the same kind as the
original object. In other words, a slice of a vector is a vector, and a slice of
a list (which we will get to soon) is a list. R is very flexible regarding the
argument passed to the slice operator. It can be the usual single integer as we
saw above, but also a vector of integers or a vector of booleans (acting as a
filter). We can also use negative indexes to remove elements (but fortunately
not mix positive and negative indexes).

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

The double bracket operator accesses elements (rather than slices) of a
container. For vectors, an "element" means a vector of length one.

    > c(10, 20, 30)[[1]]
    [1] 10
    > c(10, 20, 30)[[1:2]]
    Error in c(1, 2, 3)[[1:2]] : attempt to select more than one element

## Functions

Before we continue investigating data structures, we should take a closer look
at the dual building block of R: functions. We have used them a lot already, not
just in the obvious calls to functions like `c` and `seq`, but also because all
operators are just syntactic sugar for function calls.

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
the `function` keyword followed by the argument list and the function's body
which in R's case is simply an expression.

    > f <- function(x) 10 * x + 4
    > f(10)
    [1] 104

You may have expected curly braces around the function body, but we only need
those for sequences of expressions. In fact, curly braces are just syntactic
sugar for the `{` function that evaluates its arguments in sequence and
returns the value of the last one (like Lisp's `progn` form).

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
expressions to the formal arguments of the function and creates function objects
("promises") for each of them. These promises are evaluated (in the calling
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

Function arguments can have default expressions, and the actual arguments can be
provided by position (as we have done so far) or name. This is used a lot by R's
base functions to provide flexible APIs that are nonetheless easy to use.

	> f <- function(a=1, b=3*a) a + b
	> f(1, 10)
	[1] 11
	> f(1)
	[1] 4
	> f(b=5)
	[1] 6
	> f(b=5, a=10)
	[1] 15

The handling of the defaults is similar to the actual arguments: If the caller
does not provide an actual argument for a formal argument and this argument has
a default expression, the default expression is wrapped in a promise and
evaluated on demand against the function's (not the caller's) environment.

An interesting case are assignments to parts of an object such as setting an
element of a vector.

    > x <- c(2, 4, 6)
    > y <- x
    > y[2] <- 8
    > y
    [1] 2 8 6
    > x
    [1] 2 4 6

What's happening here? First, assignments to parts of an object (such as the
slice identified with the subscript operator) are again just syntactic sugar for
the call to a function that handles this special kind of assignment. In this
case, it is the "slice assignment" function `[<-`. So, instead of the assignment
to the slice using the slice and assignment operators, we could have called the
slice-assignment function `[<-` with the vector, index, and new element value as
arguments and assigned the result back to the variable `y`.

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

Any R object may have "attributes" associated with it. These attributes are
name-value pairs that allow for attaching any kind of "meta data" to an
object. We could, for example, add a "description" attribute.

    > x <- 10
    > attributes(x)
    NULL
    > attr(x, "description") <- "foo"
    > attributes(x)
    $description
    [1] "foo"

The `attributes` function gets all attributes of a function (as a list which we
will introduce below), the `attr` function allows to get or set them by name.

## Matrices and Arrays

One application of attributes in R are matrices. A matrix is basically a vector
containing the data in column major order with a `dim` attribute containing the
two dimensions (numbers of rows and columns) of the matrix. This matches the way
the good old numerical FORTRAN libraries (BLAS, LAPACK) expect the data and
therefore makes for an efficient interface between R and these libraries. The
`matrix` function constructs a matrix given the data (as a vector) and the
dimension(s) `nrow` and `ncol`.

	> a = matrix(1:6, nrow = 2)
	> a
	     [,1] [,2] [,3]
	[1,]    1    3    5
	[2,]    2    4    6
	> mode(a)
	[1] "numeric"
	> typeof(a)
	[1] "integer"
	> attributes(a)
	$dim
	[1] 2 3

The subscript operator for matrices takes up to two arguments and returns the
sub-matrix associated with the specified row and column sets. With a single
argument, the subscript operator simply operates on the underlying vector.

    > a[2, 2]
    [1] 4
    > a[, 2:3]
         [,1] [,2]
    [1,]    3    5
    [2,]    4    6
	> a[2:4]
	[1] 2 3 4

As a mathematician (or matlab/octave programmer) you might be surprised to see
that R performs element-wise multiplication for the normal multiplication
operator:

	> a * a
	     [,1] [,2] [,3]
	[1,]    1    9   25
	[2,]    4   16   36

The matrix operators are enclosed in percent signs:

	> a %*% t(a)
	     [,1] [,2]
	[1,]   35   44
	[2,]   44   56
	> a %*% matrix(c(1, 1, 1))
	     [,1]
	[1,]    9
	[2,]   12

Here, `t` is the transpose function and `matrix(c(1, 1, 1))` is a 3x1 matrix
(column vector).

Similar to matrices, a multi-dimensional array combines the data with the `dim`
attribute containing the vector of dimensions.

	> array(1:24, c(4, 3, 2))
	, , 1

	     [,1] [,2] [,3]
	[1,]    1    5    9
	[2,]    2    6   10
	[3,]    3    7   11
	[4,]    4    8   12

	, , 2

	     [,1] [,2] [,3]
	[1,]   13   17   21
	[2,]   14   18   22
	[3,]   15   19   23
	[4,]   16   20   24

## Lists

As we have seen, R's vectors are homogeneous arrays of the basic types (boolean,
numeric, string), well suited for efficient storage and vector manipulation, but
not meant as general collections of R objects. If we try to create a vector of
functions with the `c` function, for example, we do not get a vector of
functions but a *list*.

	> f <- function (x) 2 * x
	> typeof(c(f, f))
	[1] "list"

Lists are R's universal collections. The can contain any other object (including
other lists) and provide flexible access to their elements using indexes or
names. Lists are explicitly created with the `list` function.

	> x = list(1, "foo", list(2, 3))
	> x
	[[1]]
	[1] 1

	[[2]]
	[1] "foo"

	[[3]]
	[[3]][[1]]
	[1] 2

	[[3]][[2]]
	[1] 3

The list `x` contains an integer, a string (character), and another list of two
integers. The single bracket "slice" operator returns the sub lists specified by
the indexes.

	> x[1]
	[[1]]
	[1] 1
	> x[2]
	[[1]]
	[1] "foo"
	> x[1:2]
	[[1]]
	[1] 1

With the double brackets we can access the elements of the list:

	> x[[1]]
	[1] 1
	> x[[2]]
	[1] "foo"
	> x[[3]]
	[[1]]
	[1] 2

	[[2]]
	[1] 3

The first two elements are (single element) vectors, and the last one the nested
list. We see that the two substrict operators (slice versus element) are really
different for lists, even when used with a single integer index.

It's also apparent that the use of indexes to access list element can quickly
become confusing. Most lists therefore use named elements. R stores the element
names in the `names` attribute of the list.

	> y <- list(a=1, b="foo", c =list(x=0, y=1)) 
	> names(y)
	[1] "a" "b" "c"
	> attributes(y)
	$names
	[1] "a" "b" "c"

We can still access the elements by index, but the names attribute now also
allows us to address elements by name using the `$` operator:

	> y[[2]]
	[1] "foo"
	> y[[3]][[2]]
	[1] 1
	> y$a
	[1] 1
	> y$c$y
	[1] 1
	> `$`(y, "a")
	[1] 1

Since `names` is just an attribute, we can change the names as well:

	> names(y) <- c("foo", "bar", "baz")
	> y$foo
	[1] 1

## S3 Objects

R's first object system gets its name from the third version of the S language
that introduced this concept. It's key feature is a dispatching mechanism based
on the "class" attribute of a normal R object using a naming scheme for the
specialized methods.

In most cases S3 objects are lists with named elements (which make them look
like "objects" with fields), but plain vectors can be S3 objects as well. We
have seen two S3 "classes" already: matrices and arrays:

	> a <- matrix(1:6, nrow=2)
	> typeof(a)
	[1] "integer"
	> class(a)
	[1] "matrix"

Let's take a simple two-dimensional "point" object as an example. We want the
point to have an `x` and a `y` coordinate, and it should print nicely such as
(4, 5). For convenience, we define a function `point` that constructs this kind
of two-element list and adds the "point" class to them.

	> point <- function(x, y) {
	+   p <- list(x=x, y=y)
	+   class(p) <- "point"
	+   p
	+ }
	> u <- point(1, 2)
	> u$x
	[1] 1
	> u$y
	[1] 2

This gives us the point structure with the x and y coordinates, but the print
function is still the default one:

	> u
	$x
	[1] 1

	$y
	[1] 2

	attr(,"class")
	[1] "point"

The REPL uses the `print` function when showing expressions (the "P" in REPL).
This `print` function is an S3 generic function that dispatches based on the
class name. If we defined a function called `print.point`, the dispatcher calls
this specialization instead of the generic one.

	> print.point <- function(p, ...) {
	+   cat("(", p$x, ", ", p$y, ")\n", sep="")
	+   invisible(x)
	+ }
	> print(u)
	(1, 2)
	> u
	(1, 2)

Here we use two new functions: `cat` (concatenate and print) for the output and
`invisible` to return the object without printing it again (which is a common
idiom for a print function).

Since printing an object is such a common operation, the print function is
specialized for a lot of classes (as a call to `methods("print")` will show
you).

How does R know that `print` should dispatch by class name? Let's take a look at
how `print` is implemented:

	> print
	function (x, ...) 
	UseMethod("print")

It simply calls the primitive built-in function `UseMethod` with the name of the
function ("print") to use for the dispatch. The `UseMethod` function looks at
the class of the function's first argument and checks if a function with the
composed name ("print." followed by the class) exists. If so, this function is
called. Otherwise, the default implementation, the function "print.default" is
called which prints exactly what we observed before defining the specialized
print function for our "point" class.

	> print.default(u)
	$x
	[1] 1

	$y
	[1] 2

	attr(,"class")
	[1] "point"

Turning to the typical "shape" example (the "hello world" of object
orientation), let's implement a generic `area` function for circles and
rectangles. First, we define "constructor" functions for circles and rectangles:

	> circle <- function(p, r) {
	+   c <- list(p=p, r=r)
	+   class(c) <- "circle"
	+   c
	+ }
	> rectangle <- function(p1, p2) {
	+   r <- list(p1=p1, p2=p2)
	+   class(r) <- "rectangle"
	+   r
	+ }

A circle is defined by its center `p` (a point) and its radius `r`, a rectangle
by two corner points `p1` and `p2`.

For the generic `area` function, we now need three pieces: The `area` function
itself that simply delegates to `UseMethod` and the two specializations
`area.circle` and `area.rectangle`:

	> area <- function(x) UseMethod("area")
	> area.circle <- function(c) c$r * pi ^ 2
	> area.rectangle <- function(r) abs((r$p2$x - r$p1$x) * (r$p2$y - r$p1$y))
	> area(circle(point(0, 0), 1))
	[1] 9.869604
	> area(rectangle(point(1, 2), point(3, 5)))
	[1] 6

## Data Frames

R's main structure for representing tabular data is a *data frame*. A data frame
is a list with the class `data.frame` whose elements are vectors of the same
length. In statistics terms, each of these vectors represents a variable or
feature of the data, and the rows, that is, the elements with the same index,
are the samples or instances of the data.

- reading data frames from csv
- list-based manipulations of data frames
- simple plots, linear regression

## ggplot2



## Appendix: Installation

I found RStudio to be a perfect companion for R. Installation (on Ubuntu in my
case) was a breeze, and the integration of console, editor, package management,
graphics, and the help system worked like a charm.

