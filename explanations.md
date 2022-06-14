# lazy arguments without P
Comma concatenation is a GCC extension. 

```c
#define LEFT(P,v...) P##v
#define LAZY_WITHOUT_P(v...) LEFT(,##v)
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
Produces an custom error recorded on the buildlog as well as halting compilation.

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
The first five characters of a dirty PPMP test file that remove most warnings and shortens the current file name to nothing.
Under GCC, this is called a linemarker. This one sets the line under it as `2` and the file name as nothing (some systems will
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
#define CAT_(a,b) a##b
#define FOOBAR ~

CAT_(FOO,BAR) // ~
```

# `#b`

```C
#define STR_(b...) #b

STR_ (123, foo, bar) // "123, foo, bar"
```

# `#define CAT(a,b) CAT_(a,b)`/`#define CAT_(a,b) a##b`

```C
#define CAT(a,b) CAT_(a,b)
#define CAT_(a,b) a##b

#define foo FOO
#define bar BAR
#define FOOBAR ~

CAT_(foo,bar) // foobar
CAT(foo,bar) // ~
```
