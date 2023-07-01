# Interlude: Thread API

This chapter briefly covers the main portions of the thread API. Each part
will be explained further in the subsequent chapters, as we show how
to use the API. More details can be found in various books and online
sources [B89, B97, B+96, K+96]. We should note that the subsequent chapters introduce the concepts of locks and condition variables more slowly,
with many examples; this chapter is thus better used as a reference.

>   CRUX: HOW TO CREATE AND CONTROL THREADS
> 
> What interfaces should the OS present for thread creation and control?
> How should these interfaces be designed to enable ease of use as well as
> utility?
>

##  Thread Creation

The first thing you have to be able to do to write a multi-threaded
program is to create new threads, and thus some kind of thread creation
interface must exist. In POSIX, it is easy:

``` c linenums="1" 
#include <pthread.h>
int
pthread_create(pthread_t *thread,
    const pthread_attr_t *attr,
        void *(*start_routine)(void*),
        void *arg);

```

This declaration might look a little complex (particularly if you haven’t
used function pointers in C), but actually it’s not too bad. There are
four arguments: `thread, attr, start routine`, and `arg`. The first,
`thread`, is a pointer to a structure of type `pthread_t`; we’ll use this
structure to interact with this thread, and thus we need to pass it to
`pthread_create()` in order to initialize it.

The second argument, `attr`, is used to specify any attributes this thread
might have. Some examples include setting the stack size or perhaps information about the scheduling priority of the thread. An attribute is
initialized with a separate call to `pthread_attr_init()`; see the manual page for details. However, in most cases, the defaults will be fine; in
this case, we will simply pass the value NULL in.

The third argument is the most complex, but is really just asking: which
function should this thread start running in? In C, we call this a function
pointer, and this one tells us the following is expected: a function name
`(start_routine)`, which is passed a single argument of type void * (as
indicated in the parentheses after `start_routine`), and which returns a
value of type void * (i.e., a void pointer).

If this routine instead required an integer argument, instead of a void
pointer, the declaration would look like this:

``` c linenums="1" 
int pthread_create(..., // first two args are the same
                    void *(*start_routine)(int),
                    int arg);
```

Finally, the fourth argument, arg, is exactly the argument to be passed
to the function where the thread begins execution. You might ask: why
do we need these void pointers? Well, the answer is quite simple: having
a void pointer as an argument to the function start routine allows us
to pass in any type of argument; having it as a return value allows the
thread to return any type of result.

Let’s look at an example in Figure 27.1. Here we just create a thread
that is passed two arguments, packaged into a single type we define ourselves `(myarg_t)`. The thread, once created, can simply cast its argument
to the type it expects and thus unpack the arguments as desired.

And there it is! Once you create a thread, you really have another
live executing entity, complete with its own call stack, running within the
same address space as all the currently existing threads in the program.
The fun thus begins!

## Thread Completion

The example above shows how to create a thread. However, what
happens if you want to wait for a thread to complete? You need to do
something special in order to wait for completion; in particular, you must
call the routine `pthread_join()`.

``` c
int pthread_join(pthread_t thread, void **value_ptr);
```

``` c linenums="1"
#include <stdio.h>
#include <pthread.h>

typedef struct {
    int a;
    int b;
} myarg_t;

void *mythread(void *arg) {
    myarg_t *args = (myarg_t *) arg;
    printf("%d %d\n", args->a, args->b);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    myarg_t args = { 10, 20 };

    int rc = pthread_create(&p, NULL, mythread, &args);
    ...
 }

```