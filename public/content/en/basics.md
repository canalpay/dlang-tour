# D's basics
# Import's & Modules

To write a simple hello world program in D you need
`import`s. The `import` statement makes all public functions
and types from the given **module** available.

The standard library, called [Phobos](https://www.dlang.org/phobos/),
is located under the **package** `std`
and those modules are referenced through `import std.MODULE`.

The import statement can also be used to selectively
import certain symbols of a module. This improves
the already short compile times of D source.

    import std.stdio: writeln, writefln;

An import statement need not appear at the top a source file.
It can also be used locally within functions.

The *package* name is induced from the parent folder's name.
So all modules in the directoy `mypkg` can be imported
using `import mypkg.module`.

## {SourceCode}

// Either: import std.stdio;
// or import std.stdio: writeln;

void main()
{
    // This works too:
    import std.stdio;
    writeln("Hello World!");
}

# Basic types

D provides a number of basic types which always have the same
size **regardless** of the platform - the only exception
is the `real` type which provides the highest possible floating point
precision. There is no difference
between the size of an integer regardless whether the application
is compiled for 32bit or 64bit systems.

    bool                 (8 bit)
    byte, ubyte, char    (8 bit)
    short, ushort, dchar (16 bit)
    int, uint, wchar     (32 bit)
    long, ulong          (64 bit)

Floating point types:

    float                (32 bit)
    double               (64 bit)
    real                 (depending on platform, 80 bit on Intel x86 32-bit)

The prefix `u` denotes *unsigned* types. `char` translates to
UTF-8 characters, `dchar` is used in UTF-16 strings and `dchar`
in UTF-32 strings.

A conversion between variables of different types is only
allowed by the compiler if no precision is lost. A conversion
from `double` to `float` is allowed though.

A conversion to another type may be forced by using the
`cast(TYPE) var` expression.

The special keyword `auto` creates a variable and infers its
type from the right hand side of the expression. `auto var = 7`
will deduce the type `int` for `var`. Note that the type is still
set at compile-time and can't be changed - just like with any other
variable with an explicitly given type.

If no other value is given in the declaration all integers
are initialized with `0` and floating points
with `NaN` (*not a number*).

## {SourceCode}

import std.stdio;

void main()
{
    int b = 7;
    short c = cast(short) b; // cast needed here.
    uint d = b; // fine
    int f; // contains 0

    auto f = 3.1415f; // postfix f denotes a float
    // typeof(VAR) returns the type of an expression
    // .name is a builtin property
    writeln("type of f is %s", typeof(f).name);
    double pi = f; // fine
    // would be an error:
    // float bad = pi;
}

# Memory

D is a system programming language and thus allows to manually
manage and mess up your memory. Nevertheless D uses a
*garbage collector* per default to free unused memory.

D provides pointer types `T*` like in C:

    int a;
    int* b = &a; // b contains address of a
    auto c = &a // c is int* and contains address of a

A new memory block on the heap is allocated using the
`new` expression which returns a pointer to the managed
memory:

    int* a = new int;

As soon as the memory referenced by `a` isn't referenced anymore
through any variable in the program, the garbage collector
will free its memory.

D also allows pointer arithmetic. This is *not* allowed in
code which is marked as `@safe`
but only in `@system` code.

    void main() @safe {
        int* p = &a;
        int* c = a + 5; // error
    }

Unless specified otherwise the default is `@system`. Using `@safe`
a subset of D functionality can be forced by design to prevent memory
bugs.

## {SourceCode}
//TODO

# Storage classes

D is a statically typed language so once a variable has been declared
its type can't be changed from that point onwards. This allows
the compiler to prevent bugs early and enforce limitations
at compile time.

Besides static types D provides storage classes that enforce additional
constraints on certain objects. For example an `immutable` object can just
be initialized once and then isn't allowed to change - never, ever.

    immutable int err = 5;
    // or: immutable arr = 5 and int is inferred.
    err = 5; // won't compile

`immutable` objects can thus safely be shared among different threads
because they never change by design. And `immutable` objects can
be cached perfectly.

`const` objects can't be changed but the restriction is not as tight
as with `immutable`. To a `const` object can't be written, but someone
holding a mutable to the same object might just well. A `const` pointer can
be created from `immutable` or mutable object.

    immutable a = 10;
    int b = 5;
    const int* pa = &a;
    const int* pb = &b;
    *pa = 7; // disallowed

`static` allows declaring an object that holds state that is
is global for the *current* thread. Every thread will get its own
`static` object (*TLS - thread local storage*). This is different to
e.g. C/C++ and Java where `static` indeed means global
for the application, entailing synchronization issues
with multi-threading.

If you want to declare a "classic" global variable that
every thread can see and modify, use the storage class `__gshared` which is equivalent
to C's `static`. The ugly name is just a friendly reminder to use it rarely.

## {SourceCode}
//TODO

# Functions, part I

You've seen one function already: `main()` - the starting point of every 
D goodness. A function may return something - or be declard with
`void` if nothing is returned - and an arbitray number of parameters.

    int add(int lhs, int rhs) {
        return lhs + rhs;
    }

If the return type is defined as `auto` the D compiler infers the return
type automatically. If the types of different `return` statements within
the function's body don't match the compiler will certainly make you
aware of that.

    auto add(int lhs, int rhs) { // still `int`
        return lhs + rhs;
    }

Functions might even be declared inside others functions where they may be
used locally and aren't visible to the outside world.
These function can even have access to objects local to
the parent's scope

    void fun() {
        int local = 10;
        int fun_secret() {
            local++; // that's legal
        }
        ...

## {SourceCode}
//TODO

# Functions, part II

A function can also be a parameter to another function:

    void doSomething(int function(int, int) doer);
    doSomething(add); // use global function `add` here
                      // add must have to int parameters

`doer` can then be called like any other normal function. Local functions
or member functions of objects are called `delegate` because they
contain a context pointer where the information about their enclosure
is stored (*closure*):

    void foo() {
        void local() {
            writeln("local");
        }
        auto f = &local; // f is of type delegate()
    }

The same function `doSomething` taking a `delegate`
would look like this:

    void doSomething(int delegate(int,int) doer);

`delegate` and `function` objects cannot be mixed. But the
standard function `std.function.toDelegate` converts a `function`
to a `delegate`.

Nameless function which are called *lambdas* can be defined in two ways:

    auto f = (int lhs, int rhs) {
        return lhs + rhs;
    };
    auto f = (lhs, rhs) => lhs + rhs;

The second form is a shorthand form for lambdas that consist
of just one line.

## {SourceCode}
//TODO

# Structs

One way to define compound or custom types in D is to
define them through a `struct`:

    struct Person {
        int age;
        int height;
        float ageXHeight;
    }

`struct`s are always constructed on the stack (unless created
with `new`) and are copied **by value** in assignments or
as parameters to function calls.

    Person p(30, 180, 3.1415);
    auto t = p; // copy

When a new object of a `struct` type is created its members can be initialized
in the order they are defined in the `struct`. A custom constructor
is defined through a `this(...)` member function:

    struct Person {
        this(int age, int height) {
            this.age = age;
            this.height = height;
            this.ageXHeight = cast(float)age * height;
        }
            ...
    
    Person p(30, 180);

A `struct` might contain any number of member functions. Those
are per default `public` and accessible from the outside. They might
as well be `private` and thus only be callable by other
member functions.

    struct Person {
        void doStuff() {
            ...
        private void privateStuff() {
            ...
    
    p.doStuff(); // call do_stuff
    p.privateStuff(); // forbidden

## {SourceCode}

//TODO

# Arrays

The are two types of Arrays in D: **static** and **dynamic**
arrays. Access to arrays of any kind are always bounds checked;
a failed range check yields a `RangeError` which aborts the application. The brave
can disable this with the compiler flag `-boundschecks=off` to squeeze
the last cycles out of their binary.

**static** arrays are stored on the stack and have a fixed,
compile-time known length. An static array's type includes
the fixed size:

    int[8] arr;

`arr`s tye is `int[8]`. Note that the size of the array is denoted
near the type and not after the variable name like in C/C++.

**dynamic** arrays are stored on the heap and can be expanded
or shrunk at runtime. A dynamic array is created using a `new` expression
and its length:

    int size = 8; // run-time variable
    auto arr = new int[size];

The type of `arr` is `int[]` which is technically a
**slice** - those are introduced in the next section.

Both static and dynamic array provide the property `.length`
which is read-only for static arrays but can be written to
in case of dynamic arrays to change its size dynamically.

When indexing an array through the `arr[idx]` syntax the special
`$` syntax denotes an array's length. `arr[$ - 1]` thus
references the last element and is a short form for `arr[arr.length - 1]`.

## {SourceCode}

import std.stdio;

void main()
{
    // TODO
}

# Slices

Slices are objects from type `T[]` for any given type `T`.
Slices provide a view on a subset of an array
of `T` values - or just point to the whole array. **Slices and
dynamic arrays are technically the same.**

A slice has a size of `2 * sizeof(T*)` so 16 bytes on 64bit platforms
and 8 bytes on 32bit. It consists of two members:

    T* ptr;
    size_t length; // unsigned 32 bit on 32bit, unsigned 64 bit on 64bit

If a new dynamic array is created we really get a slice to that freshly
allocated memory:

    auto arr = new int[5];
    assert(arr.length == 5); // memory referenced in arr.ptr

Using the `[Start .. End]` syntax a sub-slice is constructed from an existing
slice:

    auto newArr = arr[1 .. 4]; // index 4 ist NOT included
    assert(newArr.length == 3);
    newArr[0] = 10; // changes newArr[0] aka arr[1]

Slices generate a new view on existing memory. They *don't* create
a new copy. If no slice holds a reference to that memory anymore - or a *sliced*
part of it - it will be freed by the garbage collector.

Using slices it's possible to write very efficient code for e.g. parsers
that just operate on one memory block and just slice the parts they really need
to work on - no need allocating new memory blocks.

Like seen in the previous section the `[$]` expression indexes the element
one past the slice's end and thus would generate a `RangeError`
(if bounds-checking hasn't been disabled).

# Alias & Strings

Now that we know what arrays are, have gotten in touch of `immutable`
and had a quick look at the basic types, it's time to introduce two
new constructs in one line:

    alias string = immutable(char)[];

The term `string` is defined by an `alias` expression which defines it
as a slice of `immutable(char)`'s. That is, once a `string` has been constructed
its content will never change again. And actually this is the second
introduction: welcome `string`! This is how an UTF-8 string
is defined in D.

Due to its `immutabl`ility `string`'s can perfectly be shared among
different threads. Being a slice parts can be taken out of it without
allocating memory. The standard function `std.algorithm.splitter`
for example splits a string by newline without any memory allocations.

Beside the UTF-8 `string` there are two more:

    alias dstring = immutable(dchar)[]; // UTF-16
    alias wstring = immutable(wchar)[]; // UTF-32

The variants are most easily converted between each other using
the `to` method from `std.conv`:

    dstring myDstring = to!dstring(myString);
    string myString = to!string(myDstring);

## {SourceCode}

import std.stdio;

void main() {
    alias mystring = immutable(char)[];
}

# All the classic for's

D provides four loop constructs.

The classical `for` loop known from C, Java and the like
with initiliazer, loop condition and loop statement:

    for (int i = 0; i < arr.length; ++i) {
        ...

The `while` and `do .. while` loops execute the
given code block while a certain condition is met:

    while (condition) {
        foo();
    }
    
    do {
        foo();
    } while (condition);

The `foreach` loop which will be introduced in the
next section.

The special keyword `break` will immediately abort the current loop.
If we are in a nested loop a label can be used to break any outer loop:

    outer: for (int i = 0; i < 10; ++i) {
        for (int j = 0; j < 5; ++j) {
            ...
            break outer;

The keyword `continue` starts with the next loop iteration.

## {SourceCode}

//TODO

# Foreach

D features a `foreach` loop which makes iterating
through data less error-prone and easier to read.

Given an array `arr` of type `int[]` it is possible to
iterate through the elements using this `foreach` loop:

    foreach(int e; arr) {
        writeln(e);
    }

The first field in the `foreach` definition is the variable
name used in the loop iteration. Its type can be omitted
and is then induced `auto`-style:

    foreach(e; arr) {
        // typoef(e) is int
        writeln(e);
    }

The second field must be an array - or a special iterable
object called a **range** which will be introduced in the next section.

Elements will be copied from the array or range during iteration -
this is okay for basic types but might be a problem for
large types. To prevent copying or enable *in-place
*mutation use `ref`:

    foreach(ref e; arr) {
        e = 10; // overwrite value
    }

## {SourceCode}

// TODO

# Ranges

If a `foreach` is encountered by the compiler

    foreach(element; range) {

.. it's rewritten to something equivalent to the following internally:

    for (; !range.empty; range.popFront()) {
        auto element = range.front;
        ...

Any object which fulfills the above interface is called a **range**
and is thus something that can be iterated over:

    struct Range {
        @property empty() const;
        void popFront();
        T front();
    }

The functions that are in the `std.range` and `std.algorithm` modules also provide
building blocks that make use of this interface. Ranges allow
to compose complex algorithms behind an object that
can be iterated with ease.

### Exercise

Complete the source code to create the `FibonacciRange` range
that returns numbers of the
[Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number).
Don't fool yourself into deleting the `assert`ions!

## {SourceCode}

# Associative Arrays

D has builtin *associative arrays* also known as hash maps.
An associative arry with a key type of `int` and a value type
of `string` is declared as follows:

    int[string] arr;

The syntax follows the actual usage of the hashmap:

    arr["key1"] = 10;

To test whether a key is located in the associative array, use the
`in` expression:

    if ("key1" in arr)
        writeln("Yes");

The `in` expression actually returns a pointer to the value
which is not `null` when found:

    if (auto test = "key1" in arr)
        *test = 20;

Access to a key which doesn't exist yields an `RangeError`
that immediately aborts the application.

AA's have the `.length` property like arrays and provide
a `.remove(val)` member to remove entries by their key.
The special `.byKeys` and `.byValues` return ranges which
do something which is left as an exercise to the reader.

## {SourceCode}

int[int][string]

# Classes

D provides support for classes and interfaces like in Java or C++.

Any `class` type inherits from `Object` implicitely. D classes can only
inherit from one class.

    class Foo { } // inherits from Object
    class Bar: Foo { } // Bar is a Foo too

If a member function of a base class is overridden, the keyword
`override` must be used to indicate that. This prevents unintentional
overriding of functions.

    class Bar: Foo {
        override functionFromFoo() {}
    }

A function can be marked `final` in a base class to disallow overriding
it. A function can be declared as `abstract` to force base classes to override
it. A whole class can be declared as `abstract` to make sure
that it isn't instantiated.

Classes in D are generally instantiated on the heap using `new`:

    auto bar = new Bar;

Class objects are always references types and unlike `struct` aren't
copied by value.

    Bar bar = foo; // bar points to foo

The garbage collector will make sure the memory is freed
after nobody references the object anymore.

# Interfaces

D allows defining `interface`s which are technically like
`class` types but whose member functions must be implemented
by any class inheriting from the `interface`.

    interface Animal {
        void makeNoise();
    }

The `makeNoise` member function has to be implemented
by `Dog` because it inherits from the `Animal` interface.
Technically `makeNoise` behaves like an `abstract` member
function in a base class.

    class Dog: Animal {
        override makeNoise() {
            ...
        }
    }
    
    auto dog = new Animal;
    Animal animal = dog; // implicit cast to interface
    dog.makeNoise();

A `class` type can inherit from as many `interface`s it wishes
but just from *one* base class.

D easily enables the **NVI - non virtual interface** idiom by
allowing the definition of `final` functions in an `interface`
that aren't allowed to be overridden. This enforces specific
behaviours customized by overriding the other `interface`
functions.

    interface Animal {
        void makeNoise();
        final doubleNoise() /* NVI pattern */ {
            makeNoise();
            makeNoise();
        }
    }

## {SourceCode}

import std.stdio;

interface Animal {
    void makeNoise();

    final void multipleNoise(int n) {
        for(int i = 0; i < n; ++i) {
            makeNoise();
        }
    }
}

class Dog: Animal {
    override void makeNoise() {
        writeln("Bark!");
    }
}

class Cat: Animal {
    override void makeNoise() {
        writeln("Meeoauw!");
    }
}

void main() {
    auto dog = new Dog;
    auto cat = new Cat;
    auto animals = [ cast(Animal)dog, cast(Animal)cat ];
    foreach(animal; animals) {
        animal.multipleNoise(5);
    }
}

# Templates

Like in C++ **D** allows defining templated functions which is a means
to define **generic** functions which work for any type
that compiles with the statements within the function's body:

    auto add(T)(T lhs, T rhs) {
        return lhs + rhs;
    }

The template parameter `T` is defined in a set of parentheses
in front of the actual function parameters. `T` is a placeholder
which is replaced by the compiler when actually *instantiating*
the function using the `!` operator:

    add!int(5, 10);
    add!float(5.0f, 10.0f);
    add!Animal(dog, cat); // won't compile; Animal doesn't implement +

If no template parameter is given for a templated function the compiler
tries to deduce the type using the parameters the function is fed:

    int a = 5; int b = 10;
    add(a, b); // T is to deduced to `int`
    float c = 5.0f;
    add(a, c); // ERROR: conflict because compiler
               // doesn't know what T should be

A function can have any number of template parameters which
are specified during instantiation using the `func!(T1, T2 ..)`
syntax. Template parameters can be of any basic type
including `string`s and floating point numbers.

Unlike generics in Java, templates in D are compile-time only, and yield
highly optimized code tailored to the specific set of types
used when actually calling the function

Of course, `struct`, `class` and `interface` types can be defined as template
types too.

    struct S(T) {
        // ...
    }

## {SourceCode}

import std.stdio;

class Animal(string noise) {
    void makeNoise() {
        writeln(noise ~ "!");
    }
}

class Dog: Animal!("Bark") {
}

class Cat: Animal!("Meeoauw") {
}

void multipleNoise(T)(T animal, int n) {
    for (int i = 0; i < n; ++i) {
        animal.makeNoise();
    }
}

void main() {
    auto dog = new Dog;
    auto cat = new Cat;
    multipleNoise(dog, 5);
    multipleNoise(cat, 5);
}
