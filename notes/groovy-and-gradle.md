# Groovy and Gradle

## Groovy

The [grails][grails] hype has long been surpassed by the JavaScript libraries
(node, Angular, React), and [Scala][scala] offers similar DSL capabilities in a
type-safe package, but [Groovy][groovy] seems to have a future nonetheless if
only because of [Gradle][gradle], the build system adopted by many Java
projects including Android.

The installation of both, Groovy and Gradle, follows the typical procedure for
Java-based tools (unzip, add bin directory to path), and with the Groovy shell
`groovysh` and its UI cousin `groovyConsole` we are off to a quick start
(ignoring the one second delay in comparison to native programs such as node or
python).

Groovy turns Java into a modern scripting language.  Starting with plain Java
syntax (and only slightly modified semantics), Groovy adds all the features we
expect these days from a scripting language: expressive syntax, optional types,
dynamic typing and type coercions (in particular to boolean), full object
orientation ("everything is an object"), closures, collection literals, string
interpolation, operator overloading, well-supported regular expressions, and
some kind of meta object protocol allowing for the interception of method calls
and property access.

Groovy delivers in all these areas.  In other words, Groovy turns the verbose
but fairly small, straight-forward, and more or less statically typed Java into
a dynamic language whose pleasing outside hides a lot of magic on the inside.

While all JVM languages benefit from Java's rich library eco system, Groovy
takes this one step further and also adopts Java's syntax as the starting
point.  Most valid Java code is also valid Groovy code, and the interpretation
is only slightly different:

* Groovy's default visibility is public, and there is no package visibility
  (Java's default).
* Checked exceptions don't have to be declared.
* Decimal literals are interpreted as BigDecimals by default.
* Often used standard packages are implicitly imported.
* `==` actually means `equals` for all types.
* Java's generics (introduced after Groovy) are supported only at the syntax
  level.

So, how does Groovy implement the "modern scripting language" features?

### Optional Types and Compact Syntax

Groovy allows to omit unnecessary semicolons, return keywords, and even
parentheses (which is normally only used for closure arguments).  It doesn't go
as far as Scala, though, and requires the dot for member access.

Type declarations are optional.  For arguments, they can simply be dropped, and
in (variable or function) definitions they can be replaced with the generic
`def`.  In contrast to Scala or Python, `def` is used for fields and methods,
that is, is replaces the field type or return type including `void`.

Non-private fields are automatically turned into Java properties, and Java
properties can be accessed using field syntax.  Groovy also adds a constructor
taking named arguments (a Map with syntactic sugar, see below) that sets the
fields.

Taken together, we can define a Java bean with a fraction of the Java code:

```
class Person {
  def firstName
  def lastName
  def getFullName() { firstName + " " + lastName }
}
def p = new Person(firstName: "Homer", lastName: "Simpson")
assert p.firstName == "Homer"
assert p.getLastName() == "Simpson"
assert p.fullName == "Homer Simpson"
```

Under the hood, the constructor calls the default constructor and sets the
fields which requires them to be mutable.  Making them final will lead to a
run-time error.

Groovy automatically coerces any value to a boolean if necessary.  Null, zero
numbers, and empty strings and collections are considered "falsy". Everything
else is "truthy".

As another code reduction feature, Groovy supports default values for method
arguments.

### Object Orientation

Unlike Java, Groovy does not distinguish between primitive types and objects.
All objects have methods and all the other features of objects. Java's
primitive types are automatically boxed. This means that one can call the
methods of the Integer wrapper class and some methods added by Groovy directly
on an integer literal:

```
assert 55.toString() == "55"
```

We can also ask the number for its methods (using the collection methods and
closures covered below):

```
1.metaClass.methods.collect { it.name }
===> [equals, getClass, hashCode, notify, notifyAll, toString, ...]
```

### Closures

Syntactically, Groovy's closures are Java blocks, optionally with a list of
arguments (without the parentheses) followed by `->`.  Semantically, they are
true closures that can access (read and write) any variable that is in scope
where the closure is defined.  In particular, a closure will see the value that
a referenced variable has at the time when the closure is executed.

```
z = 55
f = {x, y -> x + y + z}
assert 60 == f(2, 3)
z = 10
assert 15 == f(2, 3)
```

Groovy's closures also support an implicit `it` variable denoting the first
argument and JavaScript-like overriding of the `delegate` providing the context
for a closure call:

```
c = { foo() }
class Foo {
  def foo() { "hello foo" }
}
c.delegate = new Foo()
assert "hello foo" == c.call()
```

In combination with the optional parentheses, closures make higher order
functions look like built-in features (and are therefore used heavily in DSLs):

```
def f(x, g, h) { g(x) + h(x) }
assert 15 == f(3, {x->x*x}, {x->2*x})
assert 15 == f(3, {x->x*x}) {x->2*x}
assert 15 == f(3) {x->x*x} {x->2*x}
```

### Collection Literals and Operations

Groovy adds syntax for list (ArrayList) and map (HashMap) literals:

```
l = [1, "foo", new Date()]
m = ["a": 55, b: "foo"]
assert 55 == m.a
assert "foo" == m["b"]
```

Similar to JavaScript objects, one can drop the quotes for identifier-like
string keys and use field syntax to access items in maps.  Groovy uses this
syntax for named arguments (as in the automatic bean constructor shown above).
The named arguments are passed to a function in a map as the first argument:

```
def f(kw, x) { kw.a + x }
assert 3 == f(1, a: 2)
assert 3 == f(a: 2, 1)
```

Groovy has a `for (a in b)` loop syntax (preceding Java's `for (a : b)` syntax)
and a syntax for ranges, `1..10` (inclusive) and `1..<10` (exclusive).

Groovy adds the collections methods that one missed in Java before Java 8 such
as `collect` (map), `each`, `find`, `findAll`, and `sort`.  

### Operators and Operator Overloading

Operators are syntactic sugar for methods with corresponding names. The `==`
operator, for example, calls the `equals` method, the other comparison
operators (`<`, `>`, etc.) interpret the result of `compareTo`, and the
arithmetic operators (`+`, `-`, etc.) delegate to the appropriately called
methods `plus`, `minus`, and so forth.  The indexing operator `[]` can also be
defined using the `getAt` (read) and `putAt` (write) methods, and overloading
the shift operators with `leftShift` or `rightShift` allows for C++ like stream
APIs.

If we want a class to support any of these operators, we only have to add the
corresponding method.

Groovy adds some new operators such as `=~` and `==~` for regular expression
matching, `?.` for null-safe member access, and the spaceship operator `<=>`
which returns the result of `compareTo` directly.  

### Strings

There are four kinds of strings literals: single quotes (normal string with
escape characters such as `\n'), double quotes (GString with interpolation, see
below), "slashy" string with the slash as only special character (for regular
expressions), and multiline "dollar slashy" string enclosed in `$/` and `/$`.
Single and double quoted strings have Python-like multiline versions.

The GString interpolation includes lazy evaluation via closures:

```
assert "1 + 1 = 2 (yes, 2)" == "1 + 1 = ${1 + 1} (yes, ${w -> w << (1 + 1)})"
```

### Meta Programming

And then, there is the "fun" part: "meta" programming either at run time or at
compile time (AST manipulation). At run time, we can use the dynamic features
to call methods by name

```
class Foo {
  def foo() { "hello foo" }
  def bar() { "hello bar" }
}
def f(foo, method) { foo."${method}"() }
assert "hello bar" == f(new Foo(), "bar")
```

or intercept method calls:

```
class Foo {
  def foo() { "hello foo" }
  def invokeMethod(String name, args) {
    "${name} ${args[0]}"
  }
}
assert "bar 55" == new Foo().bar(55)
assert "hello foo" == new Foo().foo()
```

The latter is often combined with closure arguments (setting the delegate to
`this`) to implement DSLs.  The simple XmlBuilder in the Groovy documentation
demonstrates this combination of `invokeMethod` with closures nicely:

```
class XmlBuilder {
  def out
  XmlBuilder(out) { this.out = out }
  def invokeMethod(String name, args) {
    out << "<$name>"
    if (args[0] instanceof Closure) {
      args[0].delegate = this
      args[0].call()
    } else {
      out << args[0].toString()
    }
    out << "</$name>"
  }
}
def xml = new XmlBuilder(new StringBuffer())
xml.html {
  head {
    title "Hello World"
  }
  body {
    p "Welcome!"
  }
}
```

Similar to `invokeMethod`, one can intercept property access with the
`getProperty`, `setProperty`, or `propertyMissing` methods.  More dangerously,
we can add methods to an existing class (or change its methods) at run-time by
modifying its meta class:

```
Integer.metaClass.multiply = {s -> ((1..delegate).collect {s}).join()}
assert "aaaa" == 4 * "a"
```

If this is not enough magic (and for Gradle, it isn't), Groovy allows to modify
its AST (abstract syntax tree) during different phases of the compile.  One can
associate annotations with AST transformations (so-called local AST
transformations) as well as register global AST transformations (by registering
them using a file in `META-INF/services`).

## Gradle

When using ant, I always thought that the same could be accomplished with plain
make (as least on Unix systems) in a less verbose way.  Maven's project
conventions and repository structure are of tremendous value and one of the
main foundations of Java's vast library eco system, but its verbose
configuration and complex plugin system cause headaches as soon as one has to
deviate from the standard build lifecycle.

Gradle claims to be solution to these problems (at least in the Java space).
Based on a (Java) [domain model](http://www.gradle.org/docs/current/javadoc/)
capturing the entities of a build (project, repository, dependency, task,
action, etc.) Gradle uses Groovy to implement a
[DSL](http://www.gradle.org/docs/current/dsl/index.html) that allows for
defining the build process in a declarative manner while allowing for easy
customization and extensions.

### Hello World

Gradle's hello world script looks like this:

```
task hello {
  doLast { println 'Hello World' }
}
```

Assuming that this script is stored in `hello.gradle`, we can run the task using

```
gradle -b hello.gradle -q hello
```

Without the explicit build script name (`-b` option), Gradle is going to look
for `build.gradle`.  The quiet (`-q`) option restricts the output to error
messages and explicit output such as our 'Hello World'.

With a bit more DSL sugar, we can simplify this task to

```
task hello << { println 'Hello World' }
```

This looks almost like Groovy code using the closure and overloaded operator
syntax, but how does `task hello` map to Groovy?  Indeed, Gradle applies AST
transformations to the build scripts to allow for this DSL.  The task syntax,
for example, is handled by the
[TaskDefinitionScriptTransformer][TaskDefinitionScriptTransformer] which is one
of the AbstractScriptTransformers a build script is piped through.

### Build Scripts

The hello world build script looks like a custom DSL, but, apart from the AST
transformation, it is plain Groovy.  This means that a build script can employ
any Groovy feature and has access to the full Gradle API.  In other words, the
build script is really a script rather than a declarative build configuration
(in contrast to, for example, Maven's XML project definitions).

We can, for example, create multiple tasks using iterations:

```
for (name in ['foo', 'bar', 'baz']) {
  def tmp = name
  task "$name"  << { println "Hello $tmp" }
}
```

Note that we have to assign the loop variable `name` to a local variable in
order to get the correct value in the closure.

### Dependencies

The build graph consists of tasks as nodes and dependencies as edges. Each task
has a `dependsOn` property which can be defined when constructing the task or
after the fact by calling the `dependsOn` method.

```
task init << { println 'init' }
task compile(dependsOn: init) << { println 'compile' }
task deploy << { println 'deploy' }
deploy.dependsOn compile
```

One can see the tasks and their dependencies using Gradle's `tasks` task with
the `--all` option.  

### Build Time

Coming from other environments such as node.js, you might be wondering why it
takes several seconds (three seconds on my laptop) to run this trivial build
script.  Starting the JVM and all the initialization simply takes that "long".
Gradle's daemon mode (`--daemon` option) reduce this overhead to about a second
by delegating the main work to a continously running process.

### Java

See [Java Quickstart][gradle-java-quickstart] in the Gradle documentation.

#### Hello World

All specific build logic is handled by plugins.  In Gradle's build script DSL,
a plugin is "applied" using the `apply` command, e.g.:

```
apply plugin: 'java'
```

For a plain Java project following the maven layout, this is the only line
required.  As the "hello world" of Java projects, consider a single `SampleApp`
class in the `com.example.sample` package:

```
.
├── build.gradle
└── src
    └── main
        └── java
            └── com
                └── example
                    └── sample
                        └── SampleApp.java
```

Running `gradle build` in this project results in the following structure:

```
.
├── build
│   ├── classes
│   │   └── main
│   │       └── com
│   │           └── example
│   │               └── sample
│   │                   └── SampleApp.class
│   ├── dependency-cache
│   ├── libs
│   │   └── hello.jar
│   └── tmp
│       ├── compileJava
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
└── src
    └── main
        └── java
            └── com
                └── example
                    └── sample
                        └── SampleApp.java
```

To create a runnable jar file, we can add the main class attribute to the
`MANIFEST.MF`:

```
jar {
  manifest {
    attributes 'Main-Class': 'com.example.sample.SampleApp'
  }
}
```

#### Dependencies and Unit Tests

To add unit tests, we need the JUnit4 libraries as a dependency.  As an
additional common library, we add guava.  This is also a good time to generate
the eclipse project so that we can use code completion when writing the code.
Finally, we should set the description and version of the project.  The
resulting build script looks like this:

```
description = 'This is a gradle sample project'
version = '0.1'

apply plugin: 'java'
apply plugin: 'eclipse'

jar {
  manifest {
    attributes 'Main-Class': 'com.example.sample.SampleApp'
  }
}

repositories {
  mavenCentral()
}

dependencies {
  compile group: 'com.google.guava', name: 'guava', version: '18.0'
  testCompile group: 'junit', name: 'junit', version: '4.+'
  testCompile group: 'com.google.guava', name: 'guava-testlib', version: '18.0'
}
```

Now the eclipse project (`.classpath` and `.project` files) can be generated
with `gradle eclipse`, and the tests can be run with `gradle test`.  Gradle
shows the dependency graph (including any conflict resolution decision it has
made) with `gradle dependencies`.  The downloaded libraries are stored in the
`~/.gradle/caches/modules-2` directory by default.

Now, all that's left is to learn the (well-documented) Gradle API.

[groovy]: http://groovy.codehaus.org/ "Groovy"
[grails]: http://grails.org/ "Grails"
[gradle]: http://www.gradle.org/ "Gradle"
[scala]: http://www.scala-lang.org/ "Scala"
[gradle-task-dsl-ast-transformation]: http://groovy.329449.n5.nabble.com/Explain-Gradle-DSL-for-declaring-tasks-td5715160.html
[TaskDefinitionScriptTransformer]: https://github.com/gradle/gradle/blob/master/subprojects/core/src/main/groovy/org/gradle/groovy/scripts/internal/TaskDefinitionScriptTransformer.java
[gradle-java-quickstart]: http://www.gradle.org/docs/current/userguide/tutorial_java_projects.html

