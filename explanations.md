
# lazy arguments without P
Comma concatenation is a GCC extension.

```c
#define LEFT(P,...) P##__VA_ARGS__
#define LAZY_WITHOUT_P(...) LEFT(,##__VA_ARGS__)
#define NOTHING

#define A() ~

LAZY_WITHOUT_P(A ()) // ~
LAZY_WITHOUT_P(A NOTHING ()) // A ()
```

# `do { } while (0)`

```C
#define FOR_EACH(f,a) \
    do { \
        int i; \
        for (i = 0; i < sizeof (a) / sizeof *(a); i++) { \
            (f) ((a)[i]); \
        } \
    } while (0)
```

# macros expansion isn't recursive

```C
#define A(x) x
#define B(x) x
#define C() C ()

A (B) (0) // 0
A (A) (0) // A (0)

C() // C ()
```

# `__FILE__`/`__LINE__`
`__FILE__` expands to a string literal containing the name of the current file.
`__LINE__` expands to an integer literal of the value of the line where it is expanded.

# `__DATE__`/`__TIME__`
`__DATE__` expands to a string literal containing the date of compilation.
`__TIME__` expands to a string literal containing the time of compilation.

# `#error`
Produces a custom error recorded on the buildlog as well as halting compilation.

```C
#error "You did something wrong at line something."
```

# `#line`
Sets a new value for `__FILE__` and `__DATE__`.

```C
#line "I/am/the/capitain.now" 42
__LINE__:__FILE__ // 42:"I/am/the/capitain.now"
```

# `#2""3`
The first five characters of a dirty PPMP test file, that remove most warnings and shortens the current file name to nothing.
Under GCC, this is called a line marker. This one sets the line under it as `2` and the file name as nothing (some systems will
transform this as `"<stdin>"` on the buildlog). The `3` tells the compiler to treat the current file as a system header.

This trick is usually used in dirty PPMP tests to shorten the file names, though the system header status becomes a necessity
when using warning prone features such as GCC's `#assert`.

Syntax: `#` <*line number*> <*file name*> <*flag*>

```C
#2""3

#warning "Not sneaky"              // appears on the buildlog
#pragma GCC warning "Very sneaky" // doesn't appear on the buildlog
#error "Not a warning"           // appears on the buildlog

/*
buildlog:
:3:2: warning: #warning "Not sneaky" [-Wcpp]
:5:2: error: #error "Not a warning"
*/
```

# `a##b`

```C
#define CAT(a,b) a##b
#define FOOBAR ~
CAT(FOO,BAR) // ~
```

# `#a`

```C
#define STR(a) #a
STR(123 foo bar) // "123 foo bar"
```

# `#define CAT(a,b) CAT_(a,b)` `#define CAT_(a,b) a##b`

```C
#define CAT(a,b) CAT_(a,b)
#define CAT_(a,b) a##b

#define foo FOO
#define bar BAR
#define FOOBAR ~

CAT_(foo,bar) // foobar
CAT(foo,bar) // ~
```

# file-function table

`#include` is a file-function table.

Firstly, a file-function depends on a set of Named External Arguments (NEA), which are macros defined prior to the inclusion of the file. A good example of this are Chaos-pp slots, which take `CHAOS_PP_VALUE` as a NEA. The slot assignment file-function then defines 21 macros to produce a memorization of an integer literal.

Secondly, `#include` accepts a macro translation unit (MTU). The of values of the set of macros considered by this MTU is the domain of the function. The codomain is the set of paths the MTU produces. A good (if extreme) example of this is [`#include __DATE__`](https://github.com/JadLevesque/my-ppmptd).

Finally, we can understand a file inclusion as being an MTU indexed function table where the arguments to the function called are all global.

Example with Chaos-pp slots
```C
#define CHAOS_PP_VALUE 5 + 6       // ENA
#include CHAOS_PP_ASSIGN_SLOT (1) // File-function table indexed at slot number 1
CHAOS_PP_SLOT (1)                // 11
```


# no more than 4095

The [C99 translation limits](https://port70.net/~nsz/c/c99/n1256.html#5.2.4.1) only require an implementation to support 4095 simultaneously defined macros, and crucially only requires to support 4095 characters in a logical source line.

Macros can only be defined in a single logical source line, that sets the limit on the macro replacement list length, for portable programs, to `4085` characters (assuming you use `#define A ...`).

The same is true for C11 and C2x. In C89 the limits are 1024 simultaneously defined macros and 509 characters in a logical source line.

