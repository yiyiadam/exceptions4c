
# exceptions4c

{% hint %}
Bring the power of exceptions to your C applications with this tiny, portable library.
{% endhint %}


## An exception handling framework for C

This library provides you with a simple set of keywords (*macros*, actually)
which map the semantics of exception handling you're probably already used to:

- `try`
- `catch`
- `finally`
- `throw`

You can use exceptions in C by writing `try/catch/finally` blocks:

    #include "e4c.h"

    int foobar(){

        int foo;
        void * buffer = malloc(1024);

        if(buffer == NULL){
            throw(NotEnoughMemoryException, "Could not allocate buffer.");
        }

        try{
            foo = get_user_input(buffer, 1024);
        }catch(BadUserInputException){
            foo = 123;
        }finally{
            free(buffer);
        }

        return(foo);
    }

This way you will never have to deal again with boring error codes, or check
return values every time you call a function.


## Exception Hierarchies

The possible exceptions in a program are organized in a *pseudo-hierarchy* of
exceptions. `RuntimeException` is the root of the exceptions *pseudo-hierarchy*.
**Any** exception can be caught by a `catch(RuntimeException)` block, **except**
`AssertionException`.

When an exception is thrown, control is transferred to the nearest
dynamically-enclosing `catch` code block that handles the exception. Whether a
particular `catch` block handles an exception is found out by comparing the type
(and supertypes) of the actual thrown exception against the specified exception
in the `catch` clause.

A `catch` block is given an exception as a parameter. This parameter determines
the set of exceptions that can be handled by the code block. A block handles an
actual exception that was thrown if the specified parameter is either:

- the same type of that exception.
- the same type of any of the *supertypes* of that exception.

If you write a `catch` block that handles an exception with no defined
*subtype*, it will only handle that very specific exception. By grouping
exceptions in *hierarchies*, you can design generic `catch` blocks that deal
with several exceptions:

    /*                   Name             Default message   Supertype */
    E4C_DEFINE_EXCEPTION(ColorException, "Colorful error.", RuntimeException);
    E4C_DEFINE_EXCEPTION(RedException,   "Red error.",      ColorException);
    E4C_DEFINE_EXCEPTION(GreenException, "Green error.",    ColorException);
    E4C_DEFINE_EXCEPTION(BlueException,  "Blue error.",     ColorException);

    ...

    try{
        int color = chooseColor();
        if(color == 0xff0000) throw(RedException, "I don't like it.");
        if(color == 0x00ff00) throw(GreenException, NULL);
        if(color == 0x0000ff) throw(BlueException, "It's way too blue.");
        doSomething(color);
    }catch(GreenException){
        printf("You cannot use green.");
    }catch(ColorException){
        const e4c_exception * e = e4c_get_exception();
        printf("You cannot use that color: %s (%s).", e->name, e->message);
    }

When looking for a match, `catch` blocks are inspected in the order they appear
*in the code*. If you place a handler for a superclass before a subclass
handler, the second block will in fact be **unreachable**.


## Dispose Pattern

There are other keywords related to resource handling:

- `with... use`
- `using`

They allow you to express the *Dispose Pattern* in your code:

    /* syntax #1 */
    FOO f;
    with(f, e4c_dispose_FOO) f = e4c_acquire_FOO(foo, bar); use do_something(f);

    /* syntax #2 (relies on 'e4c_acquire_BAR' and 'e4c_dispose_BAR') */
    BAR bar;
    using(BAR, bar, ("BAR", 123) ){
        do_something_else(bar);
    }

    /* syntax #3 (customized to specific resource types) */
    FILE * report;
    e4c_using_file(report, "log.txt", "a"){
        fputs("hello, world!\n", report);
    }

This is a clean and terse way to handle all kinds of resources with implicit
acquisition and automatic disposal.


## Signal Handling

In addition, signals such as `SIGHUP`, `SIGFPE` and `SIGSEGV` can be handled in
an *exceptional* way. Forget about scary segmentation faults, all you need is to
catch `BadPointerException`:

    int * pointer = NULL;

    try{
        int oops = *pointer;
    }catch(BadPointerException){
        printf("No problem ;-)");
    }


## Multithreading

If you are using threads in your program, you must enable the *thread-safe*
version of the library by defining `E4C_THREADSAFE` at compiler level.

The usage of the framework does not vary between single and multithreaded
programs. The same semantics apply. The only caveat is that **the behavior of
signal handling is undefined in a multithreaded program** so use this feature
with caution.


## Integration

Whether you are developing a standalone application, or an external library that
provides services to independent programs, you can integrate `exceptions4c` in
your code very easily.

The system provides a mechanism for implicit initialization and finalization of
the exception framework, so that it is safe to use `try`, `catch`, `throw`, etc.
from any external function, even if its caller is not exception-aware
whatsoever.


## Portability

This library should compile in any ANSI C compiler. It uses nothing but standard
C functions. In order to use exceptions4c you have to drop the two files
(`e4c.h` and `e4c.c`) in your project and remember to `#include` the header file
from your code.

In case your application uses threads, `exceptions4c` relies on pthreads, the
**POSIX** application programming interface for writing multithreaded
applications. This *API* is available for most operating systems and platforms.


## Lightweight Version

If you have the feeling that the standard version of **exceptions4c** may be
*a bit overkill* for your specific needs, there exists a *lightweight version*,
targeted at small projects and embedded systems. Use it when you just want to
handle error conditions that may occur in your program through a simple yet
powerful exception handling mechanism. It provides the
**core functionality of exceptions4c in less than 200 source lines of code**.