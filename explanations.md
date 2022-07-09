# Above the iceberg

## [`#include`](https://en.cppreference.com/w/c/preprocessor/include)

## macros just replace text

Macros just replace text, they don't know anything about the surrounding C code.

## [header guards](https://en.wikipedia.org/wiki/Include_guard)

## [`#if #elif`](https://en.wikipedia.org/wiki/C_preprocessor#Conditional_compilation)


## make sure to parenthesize arguments

Often macros are used for code generation purposes, take for example:
```c
#define ADD(a, b) a += b
```
This will have unexpected behavior in many circumstances:
```c
ADD(x, y; z);
w = (2 + ADD(x, y) + z);
w = (ADD(x, y) + z);
```
To solve these problems make sure to parenthesize arguments, and the complete expression: w
```c
#define ADD(a, b) ((a) += (b))
```

## [`#pragma once`](https://en.wikipedia.org/wiki/Pragma_once) (extension)

## `do { } while (0)`

When writing a more complex code generation macro that isn't a single expression, then you want it to at least fit into a single statement, so it behaves like other language elements.

The naïve implementation doesn't have such properties:

```c
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

```c
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
```c
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

## [`ARRAY_LEN()`](https://www.ashn.dev/blog/2020-01-06-c-array-length.html)



# On the iceberg

## macro expansion isn't recursive

```c
#define A(x) A(x x)
A(x) // A(x x)

#define B(x) C(x x)
#define C(x) B(x x)
B(x) // B(x x x x)
```

## `#a`

```c
#define STR(a) #a
STR(123 foo bar) // "123 foo bar"
```

## `a##b`

```c
#define CAT(a,b) a##b
#define FOOBAR ~
CAT(FOO,BAR) // ~
```

## [`__VA_ARGS__`](https://stackoverflow.com/questions/26053959/what-does-va-args-in-a-macro-mean)

## [`#undef`](https://en.cppreference.com/w/c/preprocessor/replace#mw-headline)

## `#error`
Produces a custom error recorded on the buildlog as well as halting compilation.

```c
#error "You did something wrong at line something."
```

## [`defined`](https://en.cppreference.com/w/c/preprocessor/conditional#Combined_directives)

## [physical vs logical source lines](https://en.cppreference.com/w/c/language/translation_phases#Phase_2)

## [X macros](https://en.wikibooks.org/wiki/C_Programming/Preprocessor_directives_and_macros#X-Macros=)



# Below the water

## `__FILE__` `__LINE__`
`__FILE__` expands to a string literal containing the name of the current file.
`__LINE__` expands to an integer literal of the value of the line where it is expanded.

## `__DATE__` `__TIME__`
`__DATE__` expands to a string literal containing the date of compilation.
`__TIME__` expands to a string literal containing the time of compilation.

## [Trigraphs](https://en.cppreference.com/w/c/language/operator_alternative#Trigraphs)

## `#line`
Sets a new value for `__FILE__` and `__DATE__`.

```c
#line "I/am/the/capitain.now" 42
__LINE__:__FILE__ // 42:"I/am/the/capitain.now"
```

## function like macros only see parentheses

Function like macros only see parentheses when it comes to splitting up the arguments, e.g. `FOO({1,3})` calls `FOO` with the arguments `{1` and `3}`.

This problem often occurs when passing a compound literal, e.g. `(struct Vec3){1,2,3}`, to a function like macro.

To circumvent this, always pass compound literal enclosed in parentheses.

## [`__VA_OPT__`](https://en.cppreference.com/w/cpp/preprocessor/replace#Function-like_macros) (C++20 and extension)

## [`#define A(x...)`](https://gcc.gnu.org/onlinedocs/gcc/Variadic-Macros.html) (extension)

## [`,##__VA_ARGS__`](https://gcc.gnu.org/onlinedocs/gcc/Variadic-Macros.html) (extension)

## [`#pragma _Pragma()`](https://port70.net/~nsz/c/c11/n1570.html#6.10.9)

## `-P -E` (not standardized)

Many compilers (gcc,clang,tcc,...) have the options `-E` for only running the preprocessor and `-E` for not omitting line marks.

## `#if static_cast<bool>(-1)`

The `#if` statement replaces, after macro expansion, every remaining identifier with the pp-number 0.
So `#if static_cast<bool>(-1)` is equivalent to `#if 0<0>(-1)`, `#if 0 > -1`, and `#if 1`.

## preprocessing directives can't be inside of macros

> If there are sequences of preprocessing tokens within the list of arguments that would otherwise act as preprocessing directives, the behavior is undefined.

(https://port70.net/~nsz/c/c11/n1570.html#6.10.3p11)

> Each # preprocessing token in the replacement list for a function-like macro shall be followed by a parameter as the next preprocessing token in the replacement list. 

(https://port70.net/~nsz/c/c11/n1570.html#6.10.3.2p1)

Hence, it's not possible to generate preprocessor directives using standard macros.

(The rules are similar in C++)



# Middle of the iceberg

## `__COUNTER__` (extension)

clang and gcc offer the language extension `__COUNTER__`, expands to an integer value starting at `0` and incrementing the value after every expansion:
```c
__COUNTER__ // 0
__COUNTER__ // 1
__COUNTER__ // 2
```


## prefix namespaces

Because macro definitions are global there is no builtin namespace facility, it's recommended for libraries to prefix all the macros they define with a characteristic prefix, e.g.: `LIBRARYNAME_`

## stringify macros

```c
#define STR(a,b) STR_(a,b)
#define STR_(a,b) a##b

#define AWOO ~

STR_(AWOO) // "AWOO"
STR(AWOO)  // "~"
```

## `#define CAT(a,b) CAT_(a,b) #define CAT_(a,b) a##b`

```c
#define CAT(a,b) CAT_(a,b)
#define CAT_(a,b) a##b

#define foo FOO
#define bar BAR
#define FOOBAR ~

CAT_(foo,bar) // foobar
CAT(foo,bar) // ~
```

## [`__has_include`](https://en.cppreference.com/w/cpp/preprocessor/include) (C++17)

## [`#elifdef #elifndef`](http://www2.open-std.org/JTC1/SC22/WG14/www/docs/n2645.pdf) (C2x)

## [`#embed`](http://www2.open-std.org/JTC1/SC22/WG14/www/docs/n2967.htm) (C2x proposal)

## `SCAN()`

The `SCAN` macro can be used to scan its arguments twice:

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
Isolated expansion of SCAN's arguments:
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

## no more than 4095

The [C99 translation limits](https://port70.net/~nsz/c/c99/n1256.html#5.2.4.1) only require an implementation to support 4095 simultaneously defined macros, and crucially only requires to support 4095 characters in a logical source line.

Macros can only be defined in a single logical source line, that sets the limit on the macro replacement list length, for portable programs, to `4085` characters (assuming you use `#define A ...`).

The same is true for [C11](https://port70.net/~nsz/c/c11/n1570.html#5.2.4.1) and [C2x](https://port70.net/~nsz/c/c2x/n2434.pdf#subsubsection.5.2.4.1). In [C89](https://port70.net/~nsz/c/c89/c89-draft.html#2.2.4.1) the limits are 1024 simultaneously defined macros and 509 characters in a logical source line.

For C++ both minimal limits are defined as 65536.

## [mcpp](https://netcologne.dl.sourceforge.net/project/mcpp/mcpp/V.2.7.2/mcpp-summary-272.pdf)

## overloading macros based on argument count

```c
#define GET_MACRO(_1,_2,_3,x,...) x
#define FOO(...) GET_MACRO(__VA_ARGS__,FOO3,FOO2,FOO1)(__VA_ARGS__)

FOO(1)     // FOO1(1)
FOO(1,2)   // FOO2(1,2)
FOO(1,2,3) // FOO3(1,2,3)
```

## default arguments

By [overloading macros based on argument count](#overloading-macros-based-on-argument-count) it's possible to implement default arguments for functions:

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



# Bottom of the iceberg

## [Microsoft took 30 years to implement a standard complaint preprocessor](https://docs.microsoft.com/en-us/cpp/preprocessor/preprocessor-experimental-overview?view=msvc-170)

## no argument means one argument

```c
#define NO_ARGUMENT()
#define ONE_ARGUMENT(x) x

NO_ARGUMENT()
// NO_ARGUMENT(1) // error
ONE_ARGUMENT(1) 
ONE_ARGUMENT(x) 
```

## [blue paint](https://en.wikipedia.org/wiki/Painted_blue)

## `CHECK()`

The `CHECK` macro can be used to detect the existence or non-existence of a probe:

```c
#define TUPLE_AT_1(b,a,...) a
#define CHECK(...) TUPLE_AT_1(__VA_ARGS__,)
#define PROBE ,found

CHECK(PROBE,not found)     // found
CHECK(NOT_PROBE,not found) // not found
```

Crucially, this can be used to convert a token to a boolean (0 -> 0, not 0 -> 1):

```c
#define BOOL_0_0 ,0
#define BOOL(x) CHECK(BOOL_0_##x,1)
BOOL(0)   // 0
BOOL(1)   // 0
BOOL(abc) // 1
```

## saturated overloading

```c
#define TUPLE_AT_1(x0,x1,...) x1

#define COMMA_N(x) ,x
#define FOO(...) FOO_I(__VA_ARGS__,COMMA_N(FOO3),COMMA_N(FOO2),COMMA_N(FOO1),)(__VA_ARGS__)
#define FOO_I(x0,x1,x2,o,...) TUPLE_AT_1(o,FOOn,)

FOO(1)         // FOO1(1,)
FOO(1,2)       // FOO2(1,2)
FOO(1,2,3)     // FOO3(1,2,3)
FOO(1,2,3,4)   // FOOn(1,2,3,4)
FOO(1,2,3,4,5) // FOOn(1,2,3,4,5)
```

## `INC()/DEC()`

Many libraries use a structure similar to the following to implement counting in the preprocessor:

```c
#define AT_0(a,b,c,d,e,f,g,h) a
#define AT_1(a,b,c,d,e,f,g,h) b
#define AT_2(a,b,c,d,e,f,g,h) c
#define AT_3(a,b,c,d,e,f,g,h) d
#define AT_4(a,b,c,d,e,f,g,h) e
#define AT_5(a,b,c,d,e,f,g,h) f
#define AT_6(a,b,c,d,e,f,g,h) g
#define AT_7(a,b,c,d,e,f,g,h) h

#define INC_(n) AT_##n(1,2,3,4,5,6,7,0)
#define DEC_(n) AT_##n(7,0,1,2,3,4,5,6)
#define INC(n) INC_(n)
#define DEC(n) DEC_(n)

INC(6) // 7
INC(DEC(INC(INC(1)))) // 3
```

This approach is quite limited, see [#integer-arithmetics](#integer-arithmetics) for more advanced math.


## `EVAL()/DEFER()`

It is possible to defer a otherwise recursive macro expansion to avoid it getting [painted blue](#painted-blue). Thus, rescanning a defered macro causes it to expand:

```c
#define SCAN(...) __VA_ARGS__
#define EMPTY()
#define LOOP_INDIRECTION() LOOP
#define LOOP(x) x LOOP_INDIRECTION EMPTY()() (x)
SCAN(LOOP(1))
```

```
1. SCAN(LOOP(1))
Isolated expansion of SCAN's arguments:
    2. LOOP(1)
       ^ expands
    Isolated expansion of LOOP's argument to 1
    Expansion of LOOP finished, resulting tokens are rescanned
    3. 1 LOOP_INDIRECTION EMPTY()() (1)
      ^ pp-number ignored
    4. 1 LOOP_INDIRECTION EMPTY()() (1)
         ^ function like macro without arguments, ignored (crucially, not painted blue)
    5. 1 LOOP_INDIRECTION EMPTY()() (1)
                          ^ expands
    6. 1 LOOP_INDIRECTION () (1)
                          ^^^^^^ punctuators, ignored
Expansion of SCAN finished, resulting tokens are rescanned
7. 1 LOOP_INDIRECTION () (1)
   ^ pp-num ignored
8. 1 LOOP_INDIRECTION () (1)
     ^ expands
9. 1 LOOP (1)
     ^ expands
... (see 2. to 6.)
14.: 1 1 LOOP_INDIRECTION () (1)
```

The insertion of the `EMPTY` macro is sometimes abbreviated to `DEFER(LOOP_INDIRECTION)(x)` using `#define DEFER(id) id EMPTY()`, although it isn't much shorter, and just adds an unnecessary macro expansion.

There is no reason to stop at a single rescan. Using nested `SCAN` macros, in this context often called `EVAL`, one can exponentially increase the rescan count by adding another line of code.:

```c
#define EVAL3(...) EVAL2(EVAL2(EVAL2(EVAL2(__VA_ARGS__))))
#define EVAL2(...) EVAL1(EVAL1(EVAL1(EVAL1(__VA_ARGS__))))
#define EVAL1(...) __VA_ARGS__
EVAL3(LOOP(1)) // 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 LOOP_INDIRECTION () (1)
```
By stopping the fake recursion once your algorithm is complete, this technique can be used in very powerful ways: 

```c
#define E3(...) E2(E2(E2(E2(E2(E2(E2(E2(E2(E2(E2(E2(E2(__VA_ARGS__)))))))))))))
#define E2(...) E1(E1(E1(E1(E1(E1(E1(E1(E1(E1(E1(E1(E1(__VA_ARGS__)))))))))))))
#define E1(...) __VA_ARGS__

#define EMPTY()
#define TUPLE_AT_1(x,y,...) y
#define CHECK(x,...) TUPLE_AT_1(__VA_ARGS__,x,)

#define LOOP_END_END ,LOOP1
#define LOOP(f,x,...) CHECK(LOOP0,LOOP_END_##x)(f,x,__VA_ARGS__)
#define LOOP_INDIRECTION() LOOP
#define LOOP0(f,x,...) f(x) LOOP_INDIRECTION EMPTY()() (f,__VA_ARGS__)
#define LOOP1(...)

E3(LOOP(f,1,2,3,4,5,6,7,8,9,END))
```

Note that rescanning even once no more macros are defered  still takes some time, so you can't just create a macro that rescans e.g. 2^64 times without there being a rather large constant overhead for every use of that function. For a method of circumventing this, see [the continuation machine
](#continuation-machine)

## [Boost preprocessor](https://www.boost.org/doc/libs/1_75_0/libs/preprocessor/doc/index.html) 



# Below the iceberg

## list datastructure

```c
#define LIST_HEAD(a,b) a
#define LIST_TAIL(a,b) b

LIST_HEAD(1,(2,(3,))) // 1
LIST_TAIL(1,(2,(3,))) // (2,(3,))

#define TUPLE_AT_1(x,y,...) y
#define CHECK(...) TUPLE_AT_1(__VA_ARGS__,)
#define LIST_END(...) ,0
#define LIST_IS_END(x) CHECK(LIST_END x,1)

LIST_IS_END((9,))          // 0
LIST_IS_END(LIST_TAIL(9,)) // 1
```

## `#2""3`

The first five characters of a preprocessor prototyping test file, that remove most warnings and shortens the current file name to nothing.
Under GCC, this is called a line marker. This one sets the line under it as `2` and the file name as nothing (some systems will
transform this as `"<stdin>"` on the buildlog). The `3` tells the compiler to treat the current file as a system header.

> Syntax: `#` <*line number*> <*file name*> <*flag*>

```c
#2""3

#warning "Not sneaky"             // appears on the buildlog
#pragma GCC warning "Very sneaky" // doesn't appear on the buildlog
#error "Not a warning"            // appears on the buildlog
```

gcc buildlog with `#2""3`:
```
:3:2: warning: #warning "Not sneaky" [-Wcpp]
:5:2: error: #error "Not a warning"
```

gcc buildlog without `#2""3`:
```
<source>:3:2: warning: #warning "Not sneaky" [-Wcpp]
    3 | #warning "Not sneaky"             // appears on the buildlog
      |  ^~~~~~~
<source>:4:21: warning: Very sneaky
    4 | #pragma GCC warning "Very sneaky" // doesn't appear on the buildlog
      |                     ^~~~~~~~~~~~~
<source>:5:2: error: #error "Not a warning"
    5 | #error "Not a warning"            // appears on the buildlog
      |  ^~~~~
```

### lazy `P` argument

Passing an empty argument to a macro (usually the first argument called `P` for historical reasons) allows it to stop the expansion of arguments, before the tokes are rescanned:


```c
#define OPEN(...) __VA_ARGS__
#define OPENq(P,...) P##__VA_ARGS__

#define A() ~
#define NOTHING

OPEN(A NOTHING ())   // ~
OPENq(,A NOTHING ()) // A ()
```

This can also be used to increase performance, by delaying a macro expansion, when the expansion results in more tokens than the macro call.

```c
#define R2(x) R(R(R(R(R(R(R(R(R(R(x))))))))))
#define R(x) x,x,x,x,x,x

#define EAT(...) EAT_(__VA_ARGS__)
#define EAT_(...)

EAT(OPEN(R2(a))) // 4.2 seconds
EAT(OPENq(,R2(a))) // 3.1 seconds
```

## C89 `CHECK()`

```c
#define CHECK_EAT(x,y) 
#define CHECK_RESULT(x) CHECK_RESULT_ x
#define CHECK_RESULT_(x,y) y
#define CHECK(P,x,y) CHECK_RESULT((P##x,y))

#define PROBE ,found)CHECK_EAT(

CHECK(,PROBE,not found)     // found
CHECK(,NOT_PROBE,not found) // not found
```

## pp-num prefix

When a library uses identifiers that will later be concatenated with a macro, it's advisory for them to prefix these with a pp-numer.
E.g. [order-pp](#order-pp) uses this extensively:

```c
ORDER_PP
(8let ((8X, 8nat (1,2,3))
       (8Y, 8nat (4,5,6))
      ,8to_lit (8mul (8X, 8Y))
      )
)
```

If the above code wouldn't use pp-num prefixes, it would break when surrounding code, e.g. uses `#define mul ...`.


## lazy arguments without `P` (extension)

Comma concatenation is a GCC extension.

```c
#define LEFT(P,...) P##__VA_ARGS__
#define LAZY_WITHOUT_P(...) LEFT(,##__VA_ARGS__)
#define NOTHING

#define A() ~

LAZY_WITHOUT_P(A ()) // ~
LAZY_WITHOUT_P(A NOTHING ()) // A ()
```

## [`#assert`](https://gcc.gnu.org/onlinedocs/gcc-4.3.1/cpp/Assertions.html) (extension)

## constants are variables
TODO

## [`IS_EMPTY()`](https://gustedt.wordpress.com/2010/06/08/detect-empty-macro-arguments/)

## [ppstep](https://github.com/notfoundry/ppstep)

## [OBJECT disabling context](https://marc.info/?l=boost&m=118835769257658)



# Deep water

## sequence datastructure

A sequence is a group of adjacent parenthesized elements, e.g. `(1)(2)()((),w,())(awoo())(())`:

* insert front: fast (`(x)seq`)
* insert back: fast (`seq(x)`)
* pop front: fast (`EAT seq`)
* back front: slow
* iteration: fast ([see `#A(1)(2)(3)(4)(5)`](#a12345))

## `A(1)(2)(3)(4)(5)`

Iterating over a sequence is quite easy:

```c
#define A(x) f(x) B
#define B(x) f(x) A

A(1)(2)(3)(4)(5) // f(1) f(2) f(3) f(4) f(5) B
```

Sadly it's implementation defined behavior whether this works in C.

The relevant standard passage is [Annex J.1](https://port70.net/~nsz/c/c11/n1570.html#J.1):

> When a fully expanded macro replacement list contains a function-like macro name as its last preprocessing token and the next preprocessing token from the source file is a (, and the fully expanded replacement of that macro ends with the name of the first macro and the next preprocessing token from the source file is again a (, whether that is considered a nested replacement

As well as the example from [6.10.3.4p4](https://port70.net/~nsz/c/c11/n1570.html#6.10.3.4p4):

> EXAMPLE There are cases where it is not clear whether a replacement is nested or not. For example, given the following macro definitions:
> ```c
> #define f(a) a*g
> #define g(a) f(a)
> ```
> the invocation `f(2)(9)` may expand to either `2*f(9)` or `2*9*g`.
> Strictly conforming programs are not permitted to depend on such unspecified behavior.

Fortunately for us, almost all preprocessor allow for sequence iteration, and crucially it is possible to detect whether your preprocessor supports:

```c
#define CAT(a,b) CAT_(a,b)
#define CAT_(a,b) a##b

#define HAS_SEQ_ITER CAT(X_,HAS_SEQ_ITERa()()())
#define HAS_SEQ_ITERa() HAS_SEQ_ITERb
#define HAS_SEQ_ITERb() HAS_SEQ_ITERa

#define X_HAS_SEQ_ITERa() 0
#define X_HAS_SEQ_ITERb 1

#if HAS_SEQ_ITER
// use sequence iteration
#else
// don't use sequence iteration
#endif
```

### Terminating sequence iteration / removing the trailing

After the sequence iteration, you still need to get rid of the leftover function like macro without arguments.
The easiest method probably is the following:

```c
#define SEQ_TERM(...) SEQ_TERM_(__VA_ARGS__)
#define SEQ_TERM_(...) __VA_ARGS__##_RM

#define A(x) f(x),B
#define B(x) f(x),A

#define A_RM
#define B_RM

SEQ_TERM(A(1)(2)(3)(4)(5)) // f(1),f(2),f(3),f(4),f(5),
```



## Order-pp

```c
#include <stdio.h>
#include <string.h>

#include <order/interpreter.h>


#ifndef ORDER_PP_DEF_8singleton
#define ORDER_PP_DEF_8singleton ORDER_PP_FN_CM(1,8SINGLETON,0IS_ANY)
#define ORDER_PP_8SINGLETON(P,x,...) (,(P##x),P##__VA_ARGS__)
#endif

#define TOTAL_STRLEN(...) \
ORDER_PP \
(8lets ((8S, 8tuple_to_seq (8quote ((__VA_ARGS__)))) \
        (8M, 8seq_map (8compose (8adjacent (8quote (+strlen)) \
                                ,8singleton \
                                ) \
                      ,8S \
                      ) \
        ) \
       ,8seq_fold (8adjacent \
                  ,0 \
                  ,8M \
                  ) \
       ) \
)


int main () {
    printf ("%i\n"
           ,TOTAL_STRLEN ("123", "34634523", ((char[]){'5', '2', '1', '\0'}))
            // 0 +strlen("123")+strlen("34634523")+strlen(((char[]){'5', '2', '1', '\0'}))
           );
    // output: 14

    return 0;
}
```

[order-pp](https://github.com/rofl0r/order-pp)

## [chaos-pp](https://github.com/rofl0r/chaos-pp)

## [Metalang99](https://github.com/Hirrolot/metalang99)

## [Continuation machine](https://github.com/camel-cdr/bfcpp/blob/main/TUTORIAL.md#user-content-141-the-continuation-machine=)

## macro stack
TODO

## interpreters/compilers

* [bfcpp](https://github.com/camel-cdr/bfcpp) (Optimizing Brainfuck interpreter)
* [bfi](http://www.kotha.net/bfi/) (Optimizing Brainfuck interpreter)
* [CPP_COMPLETE](https://github.com/orangeduck/CPP_COMPLETE) (Brainfuck interpreter)
* [ppasm](https://github.com/notfoundry/ppasm) (x86_64 macro assembler)

## language extensions

* [datatype99](https://github.com/Hirrolot/datatype99) (Algebraic data types)
* [interface99](https://github.com/Hirrolot/interface99) (Interfaces)

## random access memory

## file slots

## file iteration

```C
// file.h
#if !CHAOS_PP_IS_ITERATING

#ifndef FILE_H
#define FILE_H

#include <chaos/preprocessor.h>
#include <order/interpreter.h>


#define ITERATION_FILE "file.h"

#define RADIUS 10


#define ORDER_PP_DEF_8radius ORDER_PP_CONST (RADIUS)
#define ORDER_PP_DEF_8height ORDER_PP_CONST (CHAOS_PP_ITERATION())


#define ORDER_PP_DEF_8output ORDER_PP_FN \
(8fn (8R, 8Y, 8X \
     ,8print (8if (8equation, 8quote (%), 8quote (~)) \
              8space \
             ) \
     ) \
)

/**
circle
x*x + y*y <= r*r
*/
#define ORDER_PP_DEF_8equation ORDER_PP_MACRO \
(8less_eq (8add (8pow (8X, 2) \
                ,8pow (8Y, 2) \
                ) \
       ,8pow (8R, 2) \
       ) \
)


#define CHAOS_PP_ITERATION_PARAMS (RADIUS)(0)(ITERATION_FILE)
%:include CHAOS_PP_ITERATE()

#define CHAOS_PP_ITERATION_PARAMS (0)(RADIUS)(ITERATION_FILE)
%:include CHAOS_PP_ITERATE()


#endif // FILE_H

#else

ORDER_PP
(8do (8seq_for_each (8output (8radius, 8height)
                    ,8seq_iota(8inc (8radius), 0)
                    )
     ,8seq_for_each (8output (8radius, 8height)
                    ,8seq_iota(0, 8inc (8radius))
                    )
     )
)

#endif
```
Output:
```C
~ ~ ~ ~ ~ ~ ~ ~ ~ ~ % % ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
~ ~ ~ ~ ~ ~ % % % % % % % % % % ~ ~ ~ ~ ~ ~
~ ~ ~ ~ % % % % % % % % % % % % % % ~ ~ ~ ~
~ ~ ~ % % % % % % % % % % % % % % % % ~ ~ ~
~ ~ % % % % % % % % % % % % % % % % % % ~ ~
~ ~ % % % % % % % % % % % % % % % % % % ~ ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
% % % % % % % % % % % % % % % % % % % % % %
% % % % % % % % % % % % % % % % % % % % % %
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ % % % % % % % % % % % % % % % % % % % % ~
~ ~ % % % % % % % % % % % % % % % % % % ~ ~
~ ~ % % % % % % % % % % % % % % % % % % ~ ~
~ ~ ~ % % % % % % % % % % % % % % % % ~ ~ ~
~ ~ ~ ~ % % % % % % % % % % % % % % ~ ~ ~ ~
~ ~ ~ ~ ~ ~ % % % % % % % % % % ~ ~ ~ ~ ~ ~
~ ~ ~ ~ ~ ~ ~ ~ ~ ~ % % ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
```

# The abyss


## Single Instruction/Continuation Multiple Data

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
#define TUPLE_AT_1(x,y,...) y
#define CHECK(...) TUPLE_AT_1(__VA_ARGS__,)

// reverse 8 arguments at a time, defer to LOOP is there are less then 8 arguments
#define SIMD_() SIMD
#define SIMD_END_END ,SIMD1
#define SIMD(a,b,c,d,e,f,g,h,...) CHECK(SIMD_END_##h,SIMD0)(a,b,c,d,e,f,g,h,__VA_ARGS__)
#define SIMD1 LOOP
#define SIMD0(a,b,c,d,e,f,g,h,...) SIMD_ EMPTY() ()(__VA_ARGS__),h,g,f,e,d,c,b,a

// reverse 1 argument at a time
#define LOOP_() LOOP
#define LOOP_END_END ,LOOP1
#define LOOP(x,...) CHECK(LOOP_END_##x,LOOP0)(x,__VA_ARGS__)
#define LOOP1(x,...) 
#define LOOP0(x,...) LOOP_ EMPTY() ()(__VA_ARGS__),x

#define REVERSE(...) FX(TUPLE_TAIL,E4(SIMD(__VA_ARGS__,END,END,END,END,END,END,END,END)))

REVERSE(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15)
//     (15,14,13,12,11,10,9,8,7,6,5,4,3,2,1)
```

## `ICE_P()`

You can overload a function depending on one of the argument being a constant expression.

It works by abusing the fact that the return type of the ternary expression `1 ? (void*)1 : (int*)1` has the type `int*`, but `1 ? (void*)0 : (int*)1` has the type `void*`:

> [...] if one operand is a null pointer constant, the result has the type of the other operand; otherwise, one operand is a pointer to void or a qualified version of void, in which case the result type is a pointer to an appropriately qualified version of void.

(https://port70.net/~nsz/c/c11/n1570.html#6.5.15p6)

Since any constant expression that evaluates to zero is a null pointer constant, you can e.g. do the following:

```c
#define ICE_P(x) _Generic((1 ? ((void*)!(x)) : &(int){1}), int*: 1, void*: 0)
#define pow(x,p) (ICE_P(p==1 || p==2) ? (p==1 ? x : (p==2 ? x*x : 0)) : pow(x, p))

pow(x,1); // (x)
pow(x,2); // (x*x)
pow(x,y); // pow(x,y)
```

(https://godbolt.org/z/bjo5TcnbT)


## macro expansion can be recursive (extension)
GCC exploit up to version 11.3

Macros can be recursive through the macro revival mechanism (term coined by Foundry). This mechanism clears the recursion inhibiting paint. This can be pushed much further like demonstrated in the now deprecated [Grail-pp](https://github.com/JadLevesque/Grail-pp)
```C
#2""3

#define PRAGMA(...) _Pragma(#__VA_ARGS__)
#define REVIVE(m) PRAGMA(push_macro(#m))PRAGMA(pop_macro(#m))
#define DEC(n,ns...) (ns)
#define FX(f,x) REVIVE(FX) f x
#define HOW_MANY_ARGS(ns...) \
REVIVE(HOW_MANY_ARGS) \
__VA_OPT__(+1 FX(HOW_MANY_ARGS, DEC (ns)))

int main () {
    printf ("%i", HOW_MANY_ARGS(1,2,3,4,5)); // 5
}
```

## [`#include` custom filesystem](https://github.com/camel-cdr/execfs)

## `F(x,y)y)y)y)y)`

A guide is a group of adjacent elements separated by closing parentheses, e.g. `1)2)3))(),w,())awoo()),())`.

It isn't a portable data structure, can't be passed to arguments directly, so it's usually constructed in place or from a [preprocessor sequence](#sequence-datastructure):

```c
#define SEQ_TERM(...) SEQ_TERM_(__VA_ARGS__)
#define SEQ_TERM_(...) __VA_ARGS__##_RM

#define EMPTY()
#define RPAREN() )

#define TO_GUIDE_A(...) __VA_ARGS__ RPAREN EMPTY()()TO_GUIDE_B
#define TO_GUIDE_B(...) __VA_ARGS__ RPAREN EMPTY()()TO_GUIDE_A
#define TO_GUIDE_A_RM
#define TO_GUIDE_B_RM
#define TO_GUIDE(seq) SEQ_TERM(TO_GUIDE_A seq)

TO_GUIDE((1)(2)(3)(4)(5)) // 1 )2 )3 )4 )5 )
```

The main advantage of the using guides is that they allow for fast iteration, very similar to the sequence iteration, but they also support passing a context between iterations, e.g.:

```c
#define TUPLE_AT_1(x,y,...) y
#define CHECK(...) TUPLE_AT_1(__VA_ARGS__,)

#define CAT_GUIDE_END_END ,CAT_GUIDE_END
#define CAT_GUIDE_A(ctx,x) CHECK(CAT_GUIDE_END_##x,CAT_GUIDE_NEXT)(ctx,x,B)
#define CAT_GUIDE_B(ctx,x) CHECK(CAT_GUIDE_END_##x,CAT_GUIDE_NEXT)(ctx,x,A)
#define CAT_GUIDE_NEXT(ctx,x,next) CAT_GUIDE_##next(ctx##x,
#define CAT_GUIDE_END(ctx,x,next) ctx

#define CAT_GUIDE(guide) CAT_GUIDE_A(,guide
#define CAT_SEQ(seq)  CAT_GUIDE(TO_GUIDE(seq(END)))

CAT_SEQ((1)(2)(3)(4)(5)) // 12345
```


The above has the same portability problems as [preprocessor sequence](#sequence-datastructure), as in it may be interpreted as a nested expansion and hence may not work on all preprocessors (although all major ones support it).

But guides can also be used, probably, by using a fixed length chain of iteration macros. The following code showcases this by implementing 8-bit integer addition:

```c
#define ADD_000(f) 0 f(0,
#define ADD_001(f) 1 f(0,
#define ADD_010(f) 1 f(0,
#define ADD_011(f) 0 f(1,
#define ADD_100(f) 1 f(0,
#define ADD_101(f) 0 f(1,
#define ADD_110(f) 0 f(1,
#define ADD_111(f) 1 f(1,
#define ADD_(c,x,y,n) ADD_##c##x##y(ADD_##n)

#define ADD_8(c,x,y) ADD_(c,x,y,7)
#define ADD_7(c,x,y) ,ADD_(c,x,y,6)
#define ADD_6(c,x,y) ,ADD_(c,x,y,5)
#define ADD_5(c,x,y) ,ADD_(c,x,y,4)
#define ADD_4(c,x,y) ,ADD_(c,x,y,3)
#define ADD_3(c,x,y) ,ADD_(c,x,y,2)
#define ADD_2(c,x,y) ,ADD_(c,x,y,1)
#define ADD_1(c,x,y) ,ADD_(c,x,y,0)
#define ADD_0(c,x,y) 

#define FX(f,...) f(__VA_ARGS__)
#define SCAN(...) __VA_ARGS__
#define ADD_8BIT(x,y) (FX(ADD_8BIT_,SCAN x, SCAN y))
#define ADD_8BIT_(x0,x1,x2,x3,x4,x5,x6,x7,y0,y1,y2,y3,y4,y5,y6,y7) \
    ADD_8(0,x0,y0)x1,y1)x2,y2)x3,y3)x4,y4)x5,y5)x6,y6)x7,y7),)

// bits are reversed (LSB,...,MSB):
ADD_8BIT((0,1,0,1,0,1,0,0),(0,0,1,0,0,1,1,0)) // (0,1,1,1,0,0,0,1)
//        0 0 1 0 1 0 1 0 + 0 1 1 0 0 1 0 0     = 1 0 0 0 1 1 1 0
//        42              + 100                 = 142
```

## [integer arithmetics](TODO)


## [`#for #endfor`](http://www2.open-std.org/JTC1/SC22/WG14/www/docs/n1410.pdf) rejected C proposal


## file-function table

`#include` is a file-function table.

Firstly, a file-function depends on a set of Named External Arguments (NEA), which are macros defined prior to the inclusion of the file. A good example of this are Chaos-pp slots, which take `CHAOS_PP_VALUE` as a NEA. The slot assignment file-function then defines 21 macros to produce a memorization of an integer literal.

Secondly, `#include` accepts a macro translation unit (MTU). The of values of the set of macros considered by this MTU is the domain of the function. The codomain is the set of paths the MTU produces. A good (if extreme) example of this is [`#include __DATE__`](https://github.com/JadLevesque/my-ppmptd).

Finally, we can understand a file inclusion as being an MTU indexed function table where the arguments to the function called are all global.

Example with Chaos-pp slots
```c
#define CHAOS_PP_VALUE 5 + 6       // ENA
#include CHAOS_PP_ASSIGN_SLOT (1) // File-function table indexed at slot number 1
CHAOS_PP_SLOT (1)                // 11
```

## tcc's non-recursive expansion is recursive

```c
#define A(x) x B
#define B(x) x A
A(1)(1)(1)(1)
```

The standard say that it's implementation defined if in the above code the macro expansions are nested or not. (see https://port70.net/~nsz/c/c11/n1570.html#6.10.3.4p4 and "When a fully expanded..." in Annex J)
So an implementation could expand the above either to "1 1 A(1)(1)..." or "1 1 1 1 A".

gcc, clang, tcc and all otherwise valid preprocessor implementation I know of expand it to "1 1 1 1 A", which is great for preprocessor meta programming.

But the problem is, whiles tcc expands the macros as though the expansion isn't nested, this isn't reflected in the tcc code.
Meaning, if instead of 4 iterations you have e.g. 20000 of them tcc segfaults, and the backtrace indicates that it's a stack overflow because of too many recursive calls.

So tcc implements non-recursive expansion recursively.


## file stack

Note: GCC only.

GCC stores file names in a 200 high stack. Information stored on this stack includes file name and line number. Using linemarkers, it is possible to push, pop and modify this stack.

There are a few subtilities to take into consideration (which happen to vary with GCC version), but the essential is:
- Push: `#` <*line number*> <*file name*> `1`
- Pop: `#` <*line number*> <*file name*> `2`


There are small but important details to keep into consideration for the proper manipulation of the file stack. Henceforth, variables will be used to succinctly refer to file names.

Note: the line number must be a literal. The linemarker does not accept an MTU

### Push
4.1.2 - 12.1
`# any-file-name 1`

Example
```c
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
#line 1 "foo.c"
INFO
#42 "bar.h" 1
INFO
#123 "baz.h" 1
INFO
```
Output:
```
[0, 1, "foo.c"]
[1, 42, "bar.h"]
[2, 123, "baz.h"]
```

Shape of the stack:
```c
  +-------+
2 | baz.h |
  +-------+
1 | bar.h |
  +-------+
0 | foo.c |
  +-------+
```


### Pop
Behaviour in function of version:

| Version range | With correct file name | With wrong file name | With empty file name |
| :---          | :---                   | :---                 | :---                 |
| 4.1.2 - 5.4   | Without carry          | With carry           | With carry           |
| 6.1 - 9.5     | Without carry          | Ignored              | Ignored              |
| 10.1 - 12.1   | Without carry          | Ignored              | Without carry        |


### Pop with carry
4.1.2 - 5.4
Example:
```c
// in "/app/example.c"
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
#line 1
#1 "bar.h" 1 // push "bar.h"
INFO
#42 "123" 2 // pop with a file name different than "/app/example.c"
INFO
```
Output:
```c
[1, 1, "bar.h"]
[0, 2, "/app/example.c"]
```

### Pop with without carry
4.1.2 - 12.1
```c
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
// in "/app/example.c"
INFO
#2"foo.h"1
INFO
#42"/app/example.c"2 // correct file name
INFO
```
Output:
```c
[0, 3, "/app/example.c"]
[1, 2, "foo.h"]
[0, 42, "/app/example.c"]
```

10.1 - 12.1
```c
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
// in "/app/example.c"
INFO
#2"foo.h"1
INFO
#42""2 // empty file name
INFO
```
Output:
```c
[0, 3, "/app/example.c"]
[1, 2, "foo.h"]
[0, 42, "/app/example.c"]
```

### Pop ignored
6.1 - 12.1
```c
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
// in "/app/example.c"
INFO
#2"foo.h"1
INFO
#42"bar.h"2 // wrong file name
INFO
```
Output:
```c
[0, 3, "/app/example.c"]
[1, 2, "foo.h"]
[1, 4, "foo.h"]
```

6.1 - 9.5
```c
#define INFO [__INCLUDE_LEVEL__, __LINE__, __FILE__]
// in "/app/example.c"
INFO
#2"foo.h"1
INFO
#42""2 // empty file name
INFO
```
Output:
```c
[0, 3, "/app/example.c"]
[1, 2, "foo.h"]
[1, 4, "foo.h"]
```



## file stack line accumulator
TODO

## operator overloading

<table>
<tr><td><b>"main.c"</b></td><td><b>"calc.c"</b></td></tr>
<tr><td>

```c
#define SLOT (2,==,2)
#include "calc.c"
VAL // 1

#define SLOT (1,<=>,12)
#include "calc.c"
VAL // -1
#define SLOT (12,<=>,12)
#include "calc.c"
VAL // 0
#define SLOT (12,<=>,1)
#include "calc.c"
VAL // 1
```

</td>
<td>

```c
#undef VAL
#define SLOT_1(a,b,c) a
#define SLOT_2(a,b,c) #b
#define SLOT_3(a,b,c) c

#define SCAN(...) __VA_ARGS__
#define LHS SCAN(SLOT_1 SLOT)
#define RHS SCAN(SLOT_3 SLOT)
#include SCAN(SLOT_2 SLOT)

#undef SLOT
```

</td></tr>
<tr><td><b>"&lt;=&gt;"</b></td><td><b>"=="</b></td></tr>
<tr><td>

```c
#if LHS < RHS
#define VAL -1
#elif LHS == RHS
#define VAL 0
#else
#define VAL 1
#endif
```

</td>
<td>

```c
#if LHS == RHS
#define VAL 1
#else
#define VAL 0
#endif
```

</td></tr>
</table>



## cross-MTU memory


```C
#define EAT(...)
#define DUMP(...) EAT(__VA_ARGS__)

#define SEQ_TERMINATE(...) __VA_ARGS__##0
#define INC_A() DUMP(__COUNTER__) INC_B
#define INC_B() DUMP(__COUNTER__) INC_A
#define INC_A0
#define INC_B0

#define FX(f,x) f x

#define MEMORISE(n) FX(SEQ_TERMINATE, (INC_A n))

#define LIT_0
#define LIT_1 ()
#define LIT_2 ()()
#define LIT_3 ()()()
#define LIT_4 ()()()()
#define LIT_5 ()()()()()
#define LIT_6 ()()()()()()
#define LIT_7 ()()()()()()()
#define LIT_8 ()()()()()()()()
#define LIT_9 ()()()()()()()()()
#define LIT_10 ()()()()()()()()()()

#define MUL_0(x)
#define MUL_1(x) x
#define MUL_2(x) x x
#define MUL_3(x) x x x
#define MUL_4(x) x x x x
#define MUL_5(x) x x x x x
#define MUL_6(x) x x x x x x
#define MUL_7(x) x x x x x x x
#define MUL_8(x) x x x x x x x x
#define MUL_9(x) x x x x x x x x x
#define MUL_10(x) x x x x x x x x x x

#define MUL(x,y) MUL_##x(LIT_##y)

int main() {

    MEMORISE(MUL(2,3))
    printf ("%i\n", __COUNTER__);

    // safe line setting
    #0""1
    #line __COUNTER__
    MEMORISE(MUL(9,5))
    printf ("%i\n", __COUNTER__ - __LINE__);
    #0""2
}
```
