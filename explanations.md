# lazy arguments without P

```c
#define LAZY_WITHOUT_P(v...) LEFT(,##v)
```