# macros just replace text

Macros just replace text, they don't know anything about the surrounding C code. TODO


# make sure to parenthesize arguments

Often macros are used for code generation purposes, take for example:
```c
#define ADD(a, b) a += b
```
This will have unexpected behavior in many circumstance:
```
ADD(x, y; z);
w = (2 + ADD(x, y) + z);
w = (ADD(x, y) + z);
```
To solve these problems make sure to parenthesize arguments, and the complete expression: w
```c
#define ADD(a, b) ((a) += (b))
```


# `do { } while (0)`

When writing a more complex code generation macro that isn't a single expression, then you want it to at least fit into a single statement, so it behaves like other language elements.

The naiive implementation doesn't have such properties:

```C
#define FOR_EACH(f,a) \
        int i; \
        for (i = 0; i < sizeof (a) / sizeof *(a); i++) { \
            (f) ((a)[i]); \
        }
// what about this case?
for (int i = 0; i < 10; ++i)
    FOR_EACH(f,a[i]);
```

A somewhat better solution is to enclose the code in a compound-statement:

```C
#define FOR_EACH(f,a) \
    { \
        int i; \
        for (i = 0; i < sizeof (a) / sizeof *(a); i++) { \
            (f) ((a)[i]); \
        } \
    }
// this works now!
for (int i = 0; i < 10; ++i)
    FOR_EACH(f,a[i]);
//                  ^ 
```
This works, but some compilers warn about the highlighted useless semicolon. The canonical way to get rid of this warning to use a `do {} while (0)` block:
```C
#define FOR_EACH(f,a) \
    do { \
        int i; \
        for (i = 0; i < sizeof (a) / sizeof *(a); i++) { \
            (f) ((a)[i]); \
        } \
    } while (0)
// this still works
for (int i = 0; i < 10; ++i)
    FOR_EACH(f,a[i]);
```


# macro expansion isn't recursive

```C
#define A(x) A(x x)
A(x) // A(x x)

#define B(x) C(x x)
#define C(x) B(x x)
B(x) // B(x x x x)
```

# `#a`

```C
#define STR(a) #a
STR(123 foo bar) // "123 foo bar"
```

# `a##b`

```C
#define CAT(a,b) a##b
#define FOOBAR ~
CAT(FOO,BAR) // ~
```

# `#error`
Produces a custom error recorded on the buildlog as well as halting compilation.

```C
#error "You did something wrong at line something."
```

# `__FILE__` `__LINE__`
`__FILE__` expands to a string literal containing the name of the current file.
`__LINE__` expands to an integer literal of the value of the line where it is expanded.

# `__DATE__`  `__TIME__`
`__DATE__` expands to a string literal containing the date of compilation.
`__TIME__` expands to a string literal containing the time of compilation.

# `#line`
Sets a new value for `__FILE__` and `__DATE__`.

```C
#line "I/am/the/capitain.now" 42
__LINE__:__FILE__ // 42:"I/am/the/capitain.now"
```

# function like macros only see parentheses

Function like macros only see parentheses when it comes to splitting up the arguments, e.g. `FOO({1,3})` calls `FOO` with the arguments `{1` and `3}`.

This problem often occurs when passing a compound literal, e.g. `(struct Vec3){1,2,3}`, to a function like macro.

To circumvent this, always pass compound literal enclosed in parentheses.

# `#if static_cast<bool>(-1)`

The `#if` statement replaces, after macro expansion, every remaining identifier with the pp-number 0.
So `#if static_cast<bool>(-1)` is equivalent to `#if 0<0>(-1)`, `#if 0 > -1`, and `#if 1`.


# preprocessing directives can't be inside of macros

> If there are sequences of preprocessing tokens within the list of arguments that would otherwise act as preprocessing directives, the behavior is undefined.

(https://port70.net/~nsz/c/c11/n1570.html#6.10.3p11)

> Each # preprocessing token in the replacement list for a function-like macro shall be followed by a parameter as the next preprocessing token in the replacement list. 

(https://port70.net/~nsz/c/c11/n1570.html#6.10.3.2p1)

Hence, it's not possible to generate preprocessor directives using standard macros.

(The rules are similar in C++)

# `-P -E`

Many compilers (gcc,clang,tcc,...) have the options `-E` for only running the preprocessor and `-E` for not omitting line marks.

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

# `__COUNTER__`

clang and gcc offer the language extension `__COUNTER__`, expands to an integer value starting at `0` and incrementing the value after every expansion:
```c
__COUNTER__ // 0
__COUNTER__ // 1
__COUNTER__ // 2
```

# overloading macros based on argument count

```c
#define GET_MACRO(_1,_2,_3,x,...) x
#define FOO(...) GET_MACRO(__VA_ARGS__,FOO3,FOO2,FOO1)(__VA_ARGS__)

FOO(1)     // FOO1(1)
FOO(1,2)   // FOO2(1,2)
FOO(1,2,3) // FOO3(1,2,3)
```

# default arguments

By [overloading macros based on argument count](#overloading-macros-based-on-argument-count) it's posible to implement default arguments for functions:

```c
void foo(int a, int b, float c);
#define GET_ARGS(_1,_2,_3,...) __VA_ARGS__
#define foo(...) GET_MACRO(__VA_ARGS__, \
                           foo(__VA_ARGS__), \
                           foo(__VA_ARGS__,2), \
                           foo(__VA_ARGS__,2,3))
foo(1,2,3) // foo(1,2,3)
foo(1,2)   // foo(1,2,3)
foo(1)     // foo(1,2,3)
```

# prefix namespaces

Because macro definitions are global there is no builtin namespace facility, it's recommended for libraries to prefix all the macros they define with a characteristic prefix, e.g.: `LIBRARYNAME_`


# `SCAN`

The `SCAN` macro can be used to scan it's arguments twice:

```c
#define SCAN(...) __VA_ARGS__
#define SCAN2(...) SCAN(__VA_ARGS__)
#define STR(x) #x
#define A(x) (x+x)
#define B(x) (x+x)

STR A(1)          // STR (x+x)
SCAN(STR A(1))    // "(x+x)"
SCAN(STR B A(1))  // STR (x+x+x+x)
SCAN2(STR B A(1)) // "(x+x+x+x)"
```


```
1. STR A(1)
   ^ no arguments supplied, so it's ignored
2. STR A(1)
       ^ expands
Isolated expansion of A's arguments to (1+1)
3. STR (1+1)
Done
```

```
1. SCAN(STR A(1))
   ^ expands
Isolated expansion of SCAN's arguments
   2. STR A(1)
      ^ no arguments supplied, so it's ignored
   3. STR A(1)
          ^ expands
   Isolated expansion of A's arguments to (1+1)
Expansion of SCAN finished, resulting tokens are rescanned
5. STR (1+1)
   ^ expands
   Isolated expansion of STR's arguments to "1+1"
6. "(1+1)"
Done
```







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



# tcc's non recurisve expansion is recursive


# gcc's macro's can be recursive


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

The same is true for [C11](https://port70.net/~nsz/c/c11/n1570.html#5.2.4.1) and [C2x](https://port70.net/~nsz/c/c2x/n2434.pdf#subsubsection.5.2.4.1). In [C89](https://port70.net/~nsz/c/c89/c89-draft.html#2.2.4.1) the limits are 1024 simultaneously defined macros and 509 characters in a logical source line.

For C++ both minimal limits are defined as 65536.





# Single Instruction/Continuation Multiple Data

Some algorithms can be sped up a lot by processing multiple elements of data in a single continuation, since continuations have a comparatively large overhead.

Reversing tuple for example can be easily done 8 elements at a time:

```c
#define E4(...) E3(E3(E3(E3(E3(E3(E3(E3(E3(E3(__VA_ARGS__))))))))))
#define E3(...) E2(E2(E2(E2(E2(E2(E2(E2(E2(E2(__VA_ARGS__))))))))))
#define E2(...) E1(E1(E1(E1(E1(E1(E1(E1(E1(E1(__VA_ARGS__))))))))))
#define E1(...) __VA_ARGS__

#define EMPTY()
#define CAT(a,b) CAT_(a,b)
#define CAT_(a,b) a##b
#define FX(f,x) f(x)
#define TUPLE_TAIL(x,...) (__VA_ARGS__)
#define TUPLE_AT_2(x,y,...) y
#define CHECK(...) TUPLE_AT_2(__VA_ARGS__,0,)
#define EQ_END_END ,1

// reverse 8 arguments at a time, defer to LOOP is there are less then 8 arguments
#define SIMD_() SIMD
#define SIMD(a,b,c,d,e,f,g,h,...) CAT(SIMD, CHECK(EQ_END_##h))(a,b,c,d,e,f,g,h,__VA_ARGS__)
#define SIMD1 LOOP
#define SIMD0(a,b,c,d,e,f,g,h,...) SIMD_ EMPTY() ()(__VA_ARGS__),h,g,f,e,d,c,b,a

// reverse 1 argument at a time
#define LOOP_() LOOP
#define LOOP(x,...) CAT(LOOP, CHECK(EQ_END_##x))(x,__VA_ARGS__)
#define LOOP1(x,...) 
#define LOOP0(x,...) LOOP_ EMPTY() ()(__VA_ARGS__),x

#define REVERSE(...) FX(TUPLE_TAIL,E4(SIMD(__VA_ARGS__,END,END,END,END,END,END,END,END)))

REVERSE(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15)
//     (15,14,13,12,11,10,9,8,7,6,5,4,3,2,1)
```

# lazy evaluation

Lazy evaluation is important for both performance and functionnality. 

## lazy `P` argument

This strategy involves having the first argument of a macro (usually called `P` for historical reasons) be used for prefix concatenation. See bellow for exceptions.

Exceptions: prominent in Order-pp.

```C
#define ORDER_PP_9BIN_PR(P,b,p) (,ORDER_PP_OPEN p##P,ORDER_PP_OPEN b##P,8BIN_PR,P##p)
```

## lazy `__VA_ARGS__`

Used when `__VA_ARGS__` is needed for macro overload to avoid scanning its contents.

```C
#define GET_MACRO(_1,_2,_3,x,...) x
#define FOO(...) GET_MACRO(0##__VA_ARGS__,FOO3,FOO2,FOO1)(__VA_ARGS__)

FOO(1)     // FOO1(1,2,3)
FOO(1,2)   // FOO2(1,2,3)
FOO(1,2,3) // FOO3(1,2,3)
```

Note: in this example the first arguement must be a concatenable token.

## tuple open

GCC extension.

Used to open a tuple without rescanning the elements.

```C
#define R2(x) R(R(R(R(R(R(R(R(R(R(x))))))))))
#define R(x) x,x,x,x,x,x
#define OPEN(...) __VA_ARGS__
#define OPENq(P,...) P##__VA_ARGS__

#define EAT(...) EAT_(__VA_ARGS__)
#define EAT_(...)

EAT(OPEN(R2(a))) // 4.2 seconds
EAT(OPENq(,R2(a))) // 3.1 seconds
```

## pp-num suffix

Prominent in Order-pp

```C
ORDER_PP
(8let ((8X, 8nat (1,2,3))
       (8Y, 8nat (4,5,6))
      ,8to_lit (8mul (8X, 8Y))
      )
) // 56088
```

## defer

See: [http://saadahmad.ca/cc-preprocessor-metaprogramming-evaluating-and-defering-macro-calls/](http://saadahmad.ca/cc-preprocessor-metaprogramming-evaluating-and-defering-macro-calls/)

## no args

To stop a function-like macro from expanding, don't give it arguments.

