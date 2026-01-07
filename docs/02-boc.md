---
title: Basics of C
---

# Lab Assignment 0 Basics of C

The following is a short intro to C.

## 1. Example files

All the example files are to be found in the Git repo for this lab; see the Basics of Shell for instructions to clone the repo. The example files are located in `~/csc-374-lab0/boc`, which you can copy in your own folder:

```shell
$ cp -r ~/csc-374-lab0/boc .
$ cd boc
$ ls
arithmetic.c       flow.c              pointers7.c
array2D.c          getopt_example.c    pointers.c
arrays.c           hello.c             power.c
ascii.c            loops.c             preproc.c
blocks.c           malloc_example0.c   sort.c
cmdline0.c         malloc_example1A.c  str.c
cmdline1.c         malloc_example1.c   struct2.c
count2.c           malloc_example2.c   struct3.c
count.c            pointers2.c         struct.c
data-types.c       pointers3.c         sum.c
demo1.c            pointers4.c         temp.c
fileIO_example1.c  pointers5.c         terminal.c
fileIO_example2.c  pointers6.c         test.txt
```

## 2. Compiling a C program

“Compiling” is usually understood as the process of turning source files into an executable binary. This entails two main steps:

1. Translating each source code file into an object file, which is a file written in machine code.
2. Linking all these files together, with the external libraries used and the OS-dependent libraries.

The first step could further be divided in two steps: translating the source code into assembly, then assemble that file into binary.

All of these steps are taken care of by [`gcc(1)`](https://man7.org/linux/man-pages/man1/gcc.1.html) when you call it:

```shell
$ cp ~/csc-374-lab0/boc/hello.c .
$ gcc -o hello hello.c
$ ./hello
hello, world
```

But you could do these steps yourself, manually (this is never recommended, except for learning purposes):

```shell
$ gcc hello.c -S -masm=intel           # Produce a human-readable assembly source
$ cat hello.s
  ...
main:
        push    rbp
        mov     rbp, rsp
        mov     edi, OFFSET FLAT:.LC0
        call    puts
        pop     rbp
        ret
  ...
$ as -o hello.o hello.s                # Assemble the file into an object file
$ cat hello.o
ELF>�@@
  [...MORE GARBAGE, NOT MEANT TO BE HUMAN READABLE...]
$ ld -o hello hello.o -dynamic-linker /usr/lib64/ld-linux-x86-64.so.2 /usr/lib64/crt[1in].o -lc
$ ./hello                              # `hello' is the linked version of hello.o and some libs
hello, world
```

For simplicity in these notes, I will compile without the option `-Wall`, but you should always compile your files with that option, that tells [`gcc(1)`](https://man7.org/linux/man-pages/man1/gcc.1.html) to give us warnings for nonfatal programming errors:

```shell
$ gcc -Wall hello.c
hello.c: In function ‘main’:
hello.c:6:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
```

Please edit `hello.c` so that the last line of the main is `return 42;`.

## 3. Fundamentals of a C program

Example file: `demo1.c`

```c
/* demo1.c - demo program */

#include <stdlib.h> // scanf
#include <stdio.h>  // printf

int main() {
  int x;
  int y = 34;

  printf ("Please enter an integer: ");
  scanf ("%d", &x);  /* &x is memory address of variable x */
  printf ("x: %d  y: %d\n", x, y);

  return 0;
}
```

Output:
```
Please enter an integer: 11
x: 11  y: 34
```

The `#include` line at the top are similar to an import in Java or Python but they differ significantly. In C, the `#include` is a preprocessor directive that instructs the compiler to replace the `#include` line with the text from the indicated file. This file is called a header file and it contains definitions of functions that the program uses, without their implementation.

The `main` function has some similarity to Java also in that just like a Java program, a C program must have exactly one `main` function. (The Python equivalent to `main` is the top-level code in a module, i.e. code that is not inside a function or class definition.) The `main` function should return an `int`, and this value can be accessed from the shell as `$?`:

```shell
$ nano hello.c 
$ cat hello.c
#include <stdio.h>

int main()
{
  printf("hello, world\n");
  return 42;
}
$ gcc hello.c -o hello
$ ./hello
hello, world
$ echo $?
42
```

Variable declarations are similar to Java. However in C, the compiler makes no initializations as in Java. Newly declared variables contain garbage values. An `int` is a 32-bit signed value (between -2^31 and 2^31 - 1). An `unsigned int` is a strictly positive 32-bit value (between 0 and 2^32). C has floating point types `float` and `double`. The `char` type is an 8-bit signed quantity that stores the ascii code for a character. Thus a `char` is just an 8-bit `int` and has an `unsigned` version also. We will discuss data types some more later.

Console I/O can be done via the many print, scan and get functions (see the man pages).

Our program uses:

```c
printf ("Please enter an integer: ");
...
printf ("x: %d  y: %d\n", x, y);        
```

In the first [`printf(3)`](https://man7.org/linux/man-pages/man3/printf.3.html) we supply a string literal. In the second [`printf(3)`](https://man7.org/linux/man-pages/man3/printf.3.html) call we pass in a format string, followed by one or more arguments (the `x` and `y` variables). The format string is printed as is, except for placeholders denoted with a `%` symbols. The values of the arguments (`x` and `y`) will be printed exactly where the placeholders are. Each placeholder (`%`) is followed by a conversion instruction—`d` in our examples—that says how the value is to be printed (in decimal notation, or binary, hexadecimal, etc.) In our case, `%d` means decimal notation. At the end of the format there is an escape sequence `\n` which denotes a newline.

To read data from the keyboard our first C program uses:

```c
scanf ("%d", &x);
```

The first argument is, again, a format string. It is used to instruct the function [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html) how to interpret what the user types. In this case it instructs [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html) to interpret the user’s input as an integer in decimal notation. The second argument contains the name of the variable where the input is to be stored (`x`). Note the ampersand character (`&`) before the `x` variable. This ampersand character is the address-of operator. This operator, when placed before a variable, produces the address in memory of that variable, rather than the value of the variable. This address value is a positive memory address between 0 and however much memory you have on your computer. [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html) needs the address of `x`, not the value of `x`, because [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html) wants to know where to store the numeric conversion of the string you typed.

Recall that when we just want the value of `x` we just use the name `x` in a statement such as:

```c
x = 15;  // assignment into the value of x
y = x + 5; // lookup the value of x
printf ("value of x: %d", x);  // lookup the value of x 
```

This is the first time we have ever been concerned with the address of where a variable is stored in memory. Java intentionally shields us from any such concerns. C does not. Thus it is important to understand the distinction between the value of a variable and the address of a variable. They are not the same. If we want the address we must put the `&` operator immediately to the left of the variable name.

## 4. Documentation of functions

All the functions that we use without implementing are provided by libraries or the system. A documentation of each function is provided using the man pages:

```shell
$ man scanf
SCANF(3)                  Linux Programmer's Manual                  SCANF(3)

NAME
       scanf, fscanf, sscanf, vscanf, vsscanf, vfscanf - input format conver‐
       sion
  ...
```

The number (3) on top of the man page indicates the section number in which the function is. This is important in particular since some functions appear in different section numbers; 1 is for executables, 2 is for system calls, and 3 is for library functions:

```shell
$ man 3 printf
PRINTF(3)                 Linux Programmer's Manual                 PRINTF(3)

NAME
       printf,  fprintf,  dprintf,  sprintf, snprintf, vprintf, vfprintf, vd‐
       printf, vsprintf, vsnprintf - formatted output conversion

SYNOPSIS
       #include <stdio.h>

       int printf(const char *format, ...);
       int fprintf(FILE *stream, const char *format, ...);
       int dprintf(int fd, const char *format, ...);
       int sprintf(char *str, const char *format, ...);
       int snprintf(char *str, size_t size, const char *format, ...);
   ...
```

The functions [`printf(3)`](https://man7.org/linux/man-pages/man3/printf.3.html) and [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html) are provided by the Standard C Library. For instance, [`printf(3)`](https://man7.org/linux/man-pages/man3/printf.3.html) relies on [`write(2)`](https://man7.org/linux/man-pages/man2/write.2.html) to actually print to the terminal (this notation, with the section number in parentheses, is standard).

## 5. Basic data types and printing them

Example file: `data-types.c`

```c
#include <stdio.h>

int main() {
  int x, y;
  char a;
  float f, e;
  double d;

  x = 4;
  y = 7;
  a = 'H';
  f = -3.4;
  d = 54.123456789;
  e = 54.123456789;

  printf ("%d %c\n", x, a);
  printf ("%3d %5c\n", y, a);
  printf ("%f %e\n", e, d);
  printf ("%.9f %.9e\n", e, d);
  printf ("%20.9f %20.9e\n", e, f);

  return 0;
}
```

Output:
```
4 H
  7     H
54.123455 5.412346e+01
54.123455048 5.412345679e+01
        54.123455048     -3.400000095e+00
```

The four basic data types are `int`, `float`, `double`, and `char`.

In the following, conversion instructions other than d are used:

```c
printf ("%d %c\n", x, a);
printf ("%3d %5c\n", x, a);
printf ("%f %e\n", e, d);
printf ("%.9f %.9e\n", e, d);
printf ("%20.9f %20.9e\n", e, d);
```

Here are the conversions we will use:

| Specification | Output |
| :--- | :--- |
| `%c` | character |
| `%s` | string of characters |
| `%d` or `%i` | decimal integer |
| `%e` | floating point number in e-notation |
| `%f` | floating point number in decimal notation |
| `%p` | pointer |
| `%u` | unsigned decimal integer |
| `%o` | octal integer |
| `%x` | hexadecimal integer, using lower case |
| `%X` | hexadecimal integer, using upper case |
| `%%` | Prints a % sign |

A number right after the placeholder specifies the width (i.e., the number of characters) that the value should occupy, with blank spaces used as fillers when necessary. For float and double values, a precision can be specified using a period .:

```c
printf ("%3d %5c\n", x, a);
printf ("%20.9f %20.9e\n", e, d);
```

The following is a list of some common escape sequences:

| Escape Sequences | Meaning |
| :--- | :--- |
| `\n` | New break |
| `\b` | Backspace |
| `\f` | Form feed |
| `\r` | Carriage return |
| `\t` | Horizontal tab |
| `\\` | Prints a `\` |
| `\'` | prints a `'` |
| `\"` | prints a `"` |

To learn more about `printf(3)`, use the man page: `printf(3)`.

In C, the `char` type is really an integer type. We use the conversion instructions to illustrate this in `ascii.c`:

Example file: `ascii.c`

```c
#include <stdio.h>

int main() {
  int i;

  for (i = 0; i < 128; i++)
    printf ("%c:  %d\n", i, i);

  return 0;
}
```

Output:
```
�:  0
:  1
:  2
:  3
:  4
:  5
:  6
:  7
:  8
	:  9

:  10
:  11
:  12

:  13
:  14
:  15
:  16
:  17
:  18
:  19
:  20
:  21
:  22
:  23
:  24
:  25
:  26
:  27
:  28
:  29
:  30
:  31
 :  32
!:  33
":  34
#:  35
$:  36
%:  37
&:  38
':  39
(:  40
):  41
*:  42
+:  43
,:  44
-:  45
.:  46
/:  47
0:  48
1:  49
2:  50
3:  51
4:  52
5:  53
6:  54
7:  55
8:  56
9:  57
::  58
;:  59
<:  60
=:  61
>:  62
?:  63
@:  64
A:  65
B:  66
C:  67
D:  68
E:  69
F:  70
G:  71
H:  72
I:  73
J:  74
K:  75
L:  76
M:  77
N:  78
O:  79
P:  80
Q:  81
R:  82
S:  83
T:  84
U:  85
V:  86
W:  87
X:  88
Y:  89
Z:  90
[:  91
\:  92
]:  93
^:  94
_:  95
`:  96
a:  97
b:  98
c:  99
d:  100
e:  101
f:  102
g:  103
h:  104
i:  105
j:  106
k:  107
l:  108
m:  109
n:  110
o:  111
p:  112
q:  113
r:  114
s:  115
t:  116
u:  117
v:  118
w:  119
x:  120
y:  121
z:  122
{:  123
|:  124
}:  125
~:  126
:  127
```

## 6. Arithmetic operations

Example file: `arithmetic.c`

```c
#include <stdio.h>

int main() {
  int x, y;
  int r1, r2, r3, r4, r5;

  x = 4;
  y = 7;
  printf ("x = %d\ny = %d\n", x, y);

  r1 = x + y;
  r2 = x - y;
  r3 = x / y;
  r4 = x * y;
  printf ("r1 = x+y = %d\nr2 = x-y = %d\n" // implicit concat
          "r3 = x/y = %d\nr4 = x*y = %d\n",
          r1, r2, r3, r4);

  printf ("x/3.0 = %f\n", x / 3.0);

  r3++;
  r4--;
  r5 = r4 % r1;
  printf ("r3++ = %d\nr4-- = %d\nr4%%r1 = %d\n", r3, r4, r5);

  return 0;
}
```

Output:
```
x = 4
y = 7
r1 = x+y = 11
r2 = x-y = -3
r3 = x/y = 0
r4 = x*y = 28
x/3.0 = 1.333333
r3++ = 1
r4-- = 27
r4%r1 = 5
```

Notes:

- `/` is the integer division operator when the numerator and denominator are integers.
- `%` is the modulus (remainder) operator.

## 7. Loops

Example file: `loops.c`

```c
#include <stdio.h>

int main() {
  int i, x;

  x = 0;

  for (i = 0; i < 4; i++) {
    x = x + i;
    printf ("%d\n", x);
  }

  while (i < 7) {
    x = x + i;
    i++;
    printf ("%d\n", x);
  }

  do {
    x = x + i;
    i++;
    printf ("%d\n", x);
  } while (i < 9);

  return 0;
}
```

Output:
```
0
1
3
6
10
15
21
28
36
```

Example file: `sum.c`

```c
#include <stdio.h>

int main() {
  int n, i, sum = 0;

  printf ("\nThis program computes 1 + 2 + ... + n\n");
  printf ("Enter a positive integer value for n: ");

  scanf ("%d", &n);

  for (i = 1; i <= n; i++) {
    sum += i;
  }

  printf ("\nThe sum of 1 + ... + %d is %d\n\n", n, sum);

  return 0;
}
```

Output:
```
This program computes 1 + 2 + ... + n
Enter a positive integer value for n: 11

The sum of 1 + ... + 11 is 66
```

Example file: `temp.c`

```c
#include <stdio.h>

#define LOWER  0     /* lower limit of table */
#define UPPER  300   /* upper limit */
#define STEP   20    /* step size */
#define ftoc(fahr) (5.0/9.0)*((fahr)-32)

/* print Fahrenheit-Celsius table
   for fahr = 0, 20, ..., 300 */

int main() {
  int fahr;

  for (fahr = LOWER; fahr <= UPPER; fahr = fahr + STEP)
    printf ("%3d %6.1f\n", fahr, ftoc (fahr));

  return 0;
}
```

Output:
```
  0  -17.8
 20   -6.7
 40    4.4
 60   15.6
 80   26.7
100   37.8
120   48.9
140   60.0
160   71.1
180   82.2
200   93.3
220  104.4
240  115.6
260  126.7
280  137.8
300  148.9
```

There are three types of loops in C:

- `for` loop, with an initialization, condition, and “afterthought”;
- `while` loop, with a condition that is tested before each iteration;
- `do-while` loop, with a condition that is tested after each iteration.

## 8. More on the preprocessor

Example file: `temp.c`
```c
#include <stdio.h>

#define LOWER  0     /* lower limit of table */
#define UPPER  300   /* upper limit */
#define STEP   20    /* step size */
#define ftoc(fahr) (5.0/9.0)*((fahr)-32)

/* print Fahrenheit-Celsius table
   for fahr = 0, 20, ..., 300 */

int main() {
  int fahr;

  for (fahr = LOWER; fahr <= UPPER; fahr = fahr + STEP)
    printf ("%3d %6.1f\n", fahr, ftoc (fahr));

  return 0;
}
```

Output:
```
  0  -17.8
 20   -6.7
 40    4.4
 60   15.6
 80   26.7
100   37.8
120   48.9
140   60.0
160   71.1
180   82.2
200   93.3
220  104.4
240  115.6
260  126.7
280  137.8
300  148.9
```

Note the use the `#define` line to define symbolic constants `LOWER`, `UPPER`, and `STEP`. It is good practice to use them instead of “burying” numbers like 300 and 20 deep inside the code. Symbolic constants can refer to values of any type, they are just considered as text.

The command `#define STEP 20` causes any occurrence of the word `STEP` in the source file to be replaced by 12 by the preprocessor. Note also the use of the function-like macro `ftoc`; the code fragment `(5.0/9.0)*((fahr)-32)` will be used to substitute every occurence of `ftoc(fahr)` in the source code.

The C preprocessor ([`cpp(1)`](https://man7.org/linux/man-pages/man1/cpp.1.html)) interprets (and removes) all the lines that start with `#`. A slightly more complex example is given with `preproc.c`:

Example file: `preproc.c`
```c
#ifdef Z
# define F 32
#else
# define F 18
#endif
#define T(a) ((a) + 8)

int main() {
  return T (F);
}
```

```shell
$ cpp preproc.c 
int main() {
  return ((18) + 8);
}
$ cpp -DZ preproc.c
int main() {
  return ((32) + 8);
}
```

## 9. Conditionals

Example file: `blocks.c`
```c
#include <stdio.h>

int main() {
  int i;

  for (i = 0; i < 5; i++) {
    if (i == 0)
      printf ("%d is 0.\n", i);
    else if (i % 2 == 0) {
      printf ("%d is even.\n", i);
    } else {
      printf ("%d is odd.\n", i);
    }
  }

  return 0;
}
```

Output:
```
0 is 0.
1 is odd.
2 is even.
3 is odd.
4 is even.
```

Example file: `terminal.c`
```c
# include <stdio.h>
//
//  Program 5.7, Stephen Kochan, Programming in C.
//

int main (void) {
  char c;

  printf ("Enter a single character to be classified: ");
  scanf ("%c", &c);

  if (('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z'))
    printf ("\"%c\" is an alphabetic character.\n", c);
  else if ('0' <= c && c <= '9')
    printf ("\"%c\" is a digit.\n", c);
  else
    printf ("\"%c\" is a special character.\n", c);

  return 0;
}
```

Output:
```
Enter a single character to be classified: z
"z" is an alphabetic character.
```

Note that the conditional expression must be in parentheses. Compound conditional expressions are created using `||` (or), `&&` (and), and `!` (not).

## 10. Flow Control

Example file: `flow.c`
```c
#include <stdio.h>

int main() {
  int i, x;

  x = 0;

  for (i = 0; i < 5; i++) {
    if (i % 2 == 0)
      continue;

    x = x - i;

    if (i % 4 == 0)
      break;

    printf ("%d\n", x);
  }

  return 0;
}
```

Output:
```
-1
-4
```

C supports two statements that modify the execution of a loop:

- `continue`: skip the rest of the loop iteration and continue with the next iteration (if the loop condition still holds).
- [`break`](https://man7.org/linux/man-pages/man2/break.2.html): skip the rest of the loop iteration and exit the loop

## 11. Functions

Example file: `power.c`
```c
#include <stdio.h>

/* power: raise base to n-th power; n >= 0 */
int power (int base, int n) {
  int i, p;
  p = 1;

  for (i = 1; i <= n; ++i)
    p = p * base;

  return p;
}

/* test power function */
int main() {
  int i;

  for (i = 0; i < 10; ++i)
    printf ("%d %4d %6d\n", i, power (2, i), power (-3, i));

  return 0;
}
```

Output:
```
0    1      1
1    2     -3
2    4      9
3    8    -27
4   16     81
5   32   -243
6   64    729
7  128  -2187
8  256   6561
9  512 -19683
```

Parameter passing: In C, function arguments are passed by value. This means that the called function is given copies of the original values. For example, when the main program executes `power(2, i)` when the value of `i` is 5, copies of 2 and 5 are written to local variables `base` and `n`: they are local to the execution of function `power`.

## 12. Pointers

Example file: `pointers.c`
```c
#include <stdio.h>

int main() {
  int val = 12345;
  float a = 3.14;
  int* valp;
  float* ap;

  printf ("val = %d\n", val);
  printf ("a = %3.2f\n", a);
  printf ("&val = %p\n", &val);
  printf ("&a = %p\n", &a);

  valp = &val;
  ap = &a;
  printf ("valp = %p\n", valp);
  printf ("ap = %p\n", ap);

  return 0;
}
```

Output:
```
val = 12345
a = 3.14
&val = 0x7fff9547c610
&a = 0x7fff9547c614
valp = 0x7fff9547c610
ap = 0x7fff9547c614
```

We already saw that the `&` operator is used to obtain the (memory) address of a variable. That means that the function [`scanf(3)`](https://man7.org/linux/man-pages/man3/scanf.3.html), for instance, takes as argument a variable whose type is a pointer to an `int`.

A pointer is a variable that contains a memory address.

Declaring a pointer variable is done as follows:

```c
int* pi   // creates a 64-bit variable whose data type is pointer to int
int i;    // an int var
pi = &i;  // pi now holds i's address - i.e it points to i
```

Let’s stress this point: every variable in C has at least 2 properties, its value and its address. That means that a variable of type “pointer to `int`”, for instance, is itself a variable and its value can change. A pointer variable can in particular contain the value `NULL`, that indicates that it is not pointing to anywhere in memory.

You can declare a pointer to any data type;

```c
char* pc;   /* pc is type: pointer to char */
float* pf;  /* pf is type: pointer to float */
double* pd; /* pd is type: pointer to double */

typedef struct {
    double cost;
    int age;
} student_type;  // This is a type that bundles two variables, just like a class in Java.

student_type* ps;          /* ps can store the address of any student_type variable */

student_type one_student;
ps = &one_student;         /* ps has the address where one_student is stored in memory */
```

We can use pointer variable to manipulate the contents of its pointed variable. To do so we must dereference it:

```c
int i;
int* pi = &i;
*pi = 15;
printf (" value pointed to by pi is: %d\n", *pi); /* deref's pi and prints 15 */
```

In summary, when `pi` is defined as `int *pi`, `pi` is a pointer to an integer, `*pi` behaves exactly as an integer variable (whose address is `pi`).

## 13. Pointers as function arguments

Example file: `pointers2.c`
```c
#include <stdio.h>

void set (int p) {
  p = -123;
}

int main() {
  int val = 0;
  printf ("val = %d before calling set.\n", val);
  set (val);
  printf ("val = %d after calling set\n", val);
  return 0;
}
```

Output:
```
val = 0 before calling set.
val = 0 after calling set.
```

Example file: `pointers3.c`
```c
#include <stdio.h>

void swap (int x, int y) {
  int temp;

  printf ("x = %d, y = %d before the swap.\n", x, y);

  temp = x;
  x = y;
  y = temp;

  printf ("x = %d, y = %d after the swap.\n", x, y);
}

int main() {
  int a = 0;
  int b = -123;

  printf ("a = %d, b = %d before the swap.\n", a, b);
  swap (a, b);
  printf ("a = %d, b = %d after the swap.\n", a, b);

  return 0;
}
```

Output:
```
a = 0, b = -123 before the swap.
x = 0, y = -123 before the swap.
x = -123, y = 0 after the swap.
a = 0, b = -123 after the swap.
```

Example file: `pointers4.c`
```c
#include <stdio.h>

void set (int* p) {
  *p = -123;
}

int main() {
  int val = 0;

  printf ("val = %d before calling set.\n", val);
  set (&val);
  printf ("val = %d after calling set\n", val);
  return 0;
}
```

Output:
```
val = 0 before calling set.
val = -123 after calling set.
```

In C, arguments to functions are passed by value. So, does the below function swap work correctly?

```c
void swap (int x, int y) {
  int temp;

  temp = x;
  x = y;
  y = temp;
}

int main () {
  int a = 0;
  int b = -123;
  swap (a, b);
  return 0;
}
```

To alter the value of a variable (e.g., `val` in example `pointers2.c` and `a` and `b` in example `pointers3.c`) in the calling function (i.e., function `main`), the calling function must pass a pointer to the variable.

The correct way to swap in C is exemplified in `pointers5.c`:

Example file: `pointers5.c`
```c
#include <stdio.h>

void swap (int* px, int* py) {
  int temp;

  printf ("x = %d, y = %d before the swap.\n", *px, *py);

  temp = *px;
  *px = *py;
  *py = temp;

  printf ("x = %d, y = %d after the swap.\n", *px, *py);
}

int main() {
  int a = 0;
  int b = -123;

  printf ("a = %d, b = %d before the swap.\n", a, b);
  swap (&a, &b);
  printf ("a = %d, b = %d after the swap.\n", a, b);

  return 0;
}
```

Output:
```
a = 0, b = -123 before the swap.
x = 0, y = -123 before the swap.
x = -123, y = 0 after the swap.
a = -123, b = 0 after the swap.
```

## 14. Arrays and strings

Example file: `arrays.c`
```c
#include <stdio.h>

int main() {
  int i;

  int a[3];    // 12 bytes, split into 3 4-byte ints
  char b[5];   // 5 bytes, split into 5 1-byte chars
  double c[3]; // 24 bytes, split into 3 8-byte doubles

  printf ("&a = %p\t&a[1] = %p\t&a[2] = %p\n\n",
          &a, &a[1], &a[2]);

  for (i = 0; i < 5; i++)
    printf ("&b[%d] = %p\n", i, &b[i]);

  printf ("\n");

  for (i = 0; i < 3; i++)
    printf ("&c[%d] = %p\n", i, &c[i]);

  return 0;
}
```

Output:
```
&a = 0x7ffc69ff2b34	&a[1] = 0x7ffc69ff2b38	&a[2] = 0x7ffc69ff2b3c

&b[0] = 0x7ffc69ff2b63
&b[1] = 0x7ffc69ff2b64
&b[2] = 0x7ffc69ff2b65
&b[3] = 0x7ffc69ff2b66
&b[4] = 0x7ffc69ff2b67

&c[0] = 0x7ffc69ff2b40
&c[1] = 0x7ffc69ff2b48
&c[2] = 0x7ffc69ff2b50
```

Example file: `str.c`
```c
#include <stdio.h>

int main() {
  char s[10] = "hello";
  int i = -1;

  do {
    i += 1;
    printf ("%c\t%x\n", s[i], s[i]);
  } while (s[i] != '\0');

  return 0;
}
```

Output:
```
h	68
e	65
l	6c
l	6c
o	6f
�	0
```

Arrays store a set of “indexable” values of the same type. For an arrays with `n` entries, the valid index values are from `0` to `n-1`.

A C string is simply an array of characters; because the size of an array is not embedded in the variable, in C we use the character `\0`, the ascii-zero character, to indicate the end of the string.

An array is in fact simply a pointer to the first element of the array.

## 15. C pointer arithmetic

Example file: `pointers6.c`
```c
#include <stdio.h>

int main() {
  int* iptr;
  int val = 12;
  iptr = &val;

  printf ("val = %d\n", val);
  printf ("sizeof(int) = %ld\n", sizeof (int));
  printf ("iptr     = %p\n", iptr);
  printf ("iptr + 1 = %p\n", iptr + 1);
  printf ("iptr + 2 = %p\n", iptr + 2);

  char* cptr;
  cptr = (char*) &val;

  printf ("cptr     = %p\n", cptr);
  printf ("cptr + 1 = %p\n", cptr + 1);
  printf ("cptr + 2 = %p\n", cptr + 2);
  printf ("cptr + 3 = %p\n", cptr + 3);

  printf ("*iptr     = %x\n", *iptr);
  printf ("*cptr     = %x\n", *cptr);
  printf ("*(cptr+1) = %x\n", * (cptr + 1));
  printf ("*(cptr+2) = %x\n", * (cptr + 2));
  printf ("*(cptr+3) = %x\n", * (cptr + 3));

  return 0;
}
```

Output:
```
val = 12
sizeof(int) = 4
iptr     = 0x7ffde93a8af4
iptr + 1 = 0x7ffde93a8af8
iptr + 2 = 0x7ffde93a8afc
cptr     = 0x7ffde93a8af4
cptr + 1 = 0x7ffde93a8af5
cptr + 2 = 0x7ffde93a8af6
cptr + 3 = 0x7ffde93a8af7
*iptr     = c
*cptr     = c
*(cptr+1) = 0
*(cptr+2) = 0
*(cptr+3) = 0
```

Pointers can be incremented, and the actual increment in memory address depends on the type that is pointed.

```c
int y;
int* p = &y;
p = p + 5;
```

Here, the pointer `p` is incremented by 5 * 4 = 20 bytes. This is because `p` is an pointer to `int` (4 bytes) and what `p + 5` really means is the 5th integer after the one that `p` references.

## 16. C pointers and arrays

Example file: `pointers7.c`
```c
#include <stdio.h>

int main() {
  int a[5] = {5, 10, 15, 20, 25};
  int* p = a;

  printf ("a[0] = %d\n", a[0]);
  printf ("a[1] = %d\n", a[1]);
  printf ("a[2] = %d\n", a[2]);

  printf ("*p     = %d\n", *p);
  printf ("*(p+1) = %d\n", * (p + 1));
  printf ("*(p+2) = %d\n", * (p + 2));

  printf ("p[0] = %d\n", p[0]);
  printf ("p[1] = %d\n", p[1]);
  printf ("p[2] = %d\n", p[2]);

  printf ("*a     = %d\n", *a);
  printf ("*(a+1) = %d\n", * (a + 1));
  printf ("*(a+2) = %d\n", * (a + 2));

  return 0;
}
```

Output:
```
a[0] = 5
a[1] = 10
a[2] = 15
*p     = 5
*(p+1) = 10
*(p+2) = 15
p[0] = 5
p[1] = 10
p[2] = 15
*a     = 5
*(a+1) = 10
*(a+2) = 15
```

Arrays and pointers are interchangeable in C, since an array is simply a pointer to the first element of the array. Consider:

```c
int a[10];
int* p;
p = a;
```

Then:

- element `a[0]` can be accessed as `*pa`, or `*a`, or `p[0]`,
- element `a[1]` can be accessed as `*(pa+1)`, or `*(a+1)`, or `p[1]`,
- element `a[2]` can be accessed as `*(pa+2)`, or `*(a+1)`, or `p[2]`.

This is true regardless of the type of the elements (`int` here). In fact, C compiles `a[i]` to `*(a+i)`

## 17. Arrays as function arguments

Example file: `sort.c`
```c
#include <stdio.h>

void sort (int a [], int n) {      /* insertion sort */
  int i, j, tmp;

  for (i = 1; i < n; i++) {
    tmp = a[i];
    j = i - 1;

    while ((j >= 0) && tmp < a[j]) {
      a[j + 1] = a[j];
      j--;
    }

    a[j + 1] = tmp;
  }
}

int main() {            /* test program */
  int ar[10] = {3, 4, 7, 0, 9, 8, 1, 6, 5, 2};
  int n = 10;
  int i;

  printf ("The original array is: ");

  for (i = 0; i < n; i++)
    printf ("%d ", ar[i]);

  printf ("\n");

  sort (ar, n);

  printf ("The sorted array is: ");

  for (i = 0; i < n; i++)
    printf ("%d ", ar[i]);

  printf ("\n");
  return 0;
}
```

Output:
```
The original array is: 3 4 7 0 9 8 1 6 5 2 
The sorted array is: 0 1 2 3 4 5 6 7 8 9 
```

## 18. Structures

Example file: `struct.c`
```c
#include <stdio.h>

struct employee {
    char* first;
    char* last;
    int age;
};

int main() {

  struct employee e = {"Sam", "Smith", 55};

  printf ("%s %s, age %d\n", e.first, e.last, e.age);

  return 0;
}
```

Output:
```
Sam Smith, age 55
```

Example file: `struct2.c`
```c
#include <stdio.h>

typedef struct {
    char* first;
    char* last;
    int age;
} employee;

int main() {
  employee e = {"Sam", "Smith", 55};

  printf ("%s %s, age %d\n", e.first, e.last, e.age);

  return 0;
}
```

Output:
```
Sam Smith, age 55
```

A structure is a grouping of several variables, possibly of different type, under a single name. The `typedef` instruction, which defines a new (user-defined) type, can be used to facilitate the usage of structs.

## 19. Structures and pointers

Example file: `struct3.c`
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    char* first;
    char* last;
    int age;
} employee;

int main() {

  employee* eptr;
  eptr = (employee*) malloc (12);
  (*eptr).first = "Sam";
  (*eptr).last = "Smith";
  (*eptr).age = 55;

  // alternative notation
  eptr->first = "Sam";
  eptr->last = "Smith";
  eptr->age = 55;

  printf ("%s %s, age %d\n",
          (*eptr).first, (*eptr).last, (*eptr).age);

  // alternative notation
  printf ("%s %s, age %d\n",
          eptr->first, eptr->last, eptr->age);

  return 0;
}
```

Output:
```
Sam Smith, age 55
Sam Smith, age 55
```

If `e` is defined as a pointer to an `employee` structure:

```c
employee* eptr;
```

then the pointer must be dereferenced in order for the structure content to be accessed:

```c
(*eptr).first = "Sam";
(*eptr).last = "Smith";
(*eptr).age = 55;
printf ("%s %s, age %d\n", (*eptr).first, (*eptr).last, (*eptr).age);
```

An alternative notation is typically used:

```c
eptr->first = "Sam";
eptr->last = "Smith";
eptr->age = 55;
printf ("%s %s, age %d\n", eptr->first, eptr->last, eptr->age);
```

## 20. Memory allocation

Example file: `malloc_example0.c`
```c
#include <stdlib.h>  // to use malloc, free 
#include <stdio.h>

int main() {
  int i;
  int* array = (int*) malloc (10 * sizeof (int));

  for (i = 0; i < 10; i++)
    array[i] = 5 * i;

  for (i = 0; i < 10; i++)
    printf ("array[%d] = %d\n", i, array[i]);

  free (array);

  return 0;
}
```

Output:
```
array[0] = 0
array[1] = 5
array[2] = 10
array[3] = 15
array[4] = 20
array[5] = 25
array[6] = 30
array[7] = 35
array[8] = 40
array[9] = 45
```

The function [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) is C’s version of the `new` operator you have seen in Java. When you use `new` in Java, some space in memory is allocated to store the object’s instance variables.

C has no notion of object (or class), but it does support dynamically allocated memory using the [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) function. This function takes as input an `int n` and allocates `n` consecutive bytes in memory; it returns a pointer to the first of the `n` bytes. The signature of [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) is:

```c
void* malloc (int n)
```

The return type is a generic pointer, `void*`. To dynamically create a new integer, one uses [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) to allocate 4 bytes. However, in order to use these 4 bytes as an `int`, you will need to cast (i.e., change the type of) the `void*` pointer into a `int*` pointer. Therefore, the typical usage of [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) to allocate an integer dynamically is:

```c
int* iptr = (int*) malloc (4);
```

It would be cumbersome (and error prone) to remember or compute the amount of memory that each type requires—4 bytes in the case of `int`. A builtin of C is the function `sizeof`, that takes a type and return the amount of memory that a variable of this type needs. A typical initialization of an array of 10 integers thus look like:

```c
int* array = (int*) malloc (10 * sizeof (int));

for (int i = 0; i < 10; i++)
  array[i] = 0;
```

It is imperative to free the memory that has been [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html)’ed after use. There is no garbage collector in C. If no variable points to a [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html)’ed block, then nothing magical will happen to make this memory available again. Thus, after use, one has to actually call:

```c
free (array);
```

## 21. Two-dimensional arrays

Example file: `array2D.c`
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

int64_t A[2][3] = {
  {2, 0, 4},
  {1, 3, 0}
};

int main() {
  int i, j;
  int64_t s = A[0][2] + A[1][0];

  printf ("s = %li\n", s);
  A[0][2] = 1;

  for (i = 0; i < 2; i++) {
    for (j = 0; j < 3; j++) {
      printf ("%li ", A[i][j]);
    }

    printf ("\n");
  }

  return 0;
}
```

Output:
```
s = 5
2 0 1 
1 3 0 
```

## 22. Dynamically allocated two-dimensional arrays

Example file: `malloc_example1A.c`
```c
#include <stdlib.h>   // to use malloc, free
#include <stdio.h>    // scanf
#include <stdint.h>   // for int64_t

#define IDX(n, i, j) ((i)*(n)+(j))

int main() {
  int i, j, n;

  printf ("What size matrix do you need? ");
  scanf ("%d", &n);

  // Dynamically obtain an array of n*n integers
  int64_t* t = (int64_t*) malloc (sizeof (int64_t) * n * n);

  // Put value i+j in row i, column j of t
  for (i = 0; i < n; i++)
    for (j = 0; j < n; j++)
      t[IDX (n, i, j)] = (int64_t) i + j;

  // Print the value in table t, row by row
  for (i = 0; i < n; i++) {
    for (j = 0; j < n; j++)
      printf ("%ld ", t[IDX (n, i, j)]);

    printf ("\n");
  }

  return 0;
}
```

Output:
```
What size matrix do you need? 11
0 1 2 3 4 5 6 7 8 9 10 
1 2 3 4 5 6 7 8 9 10 11 
2 3 4 5 6 7 8 9 10 11 12 
3 4 5 6 7 8 9 10 11 12 13 
4 5 6 7 8 9 10 11 12 13 14 
5 6 7 8 9 10 11 12 13 14 15 
6 7 8 9 10 11 12 13 14 15 16 
7 8 9 10 11 12 13 14 15 16 17 
8 9 10 11 12 13 14 15 16 17 18 
9 10 11 12 13 14 15 16 17 18 19 
10 11 12 13 14 15 16 17 18 19 20 
```

Example file: `malloc_example1.c`
```c
#include <stdlib.h>   // to use malloc, free 
#include <stdio.h>

typedef int* row_t;       // define row type
typedef row_t* table_t;   // define table type

int main() {

  int num_rows = 2;
  int num_columns = 3;
  table_t t;

  int i, j;

  // allocate space for holding num_rows row references
  // t will refer to this spaces
  t = (table_t) malloc (sizeof (row_t) * num_rows);

  for (i = 0; i < num_rows; i++)
    // for every row allocate space for num_columns integers
    t[i] = (int*) malloc (sizeof (int) * num_columns);

  // Fill table t with zeros
  for (i = 0; i < num_rows; i++)
    for (j = 0; j < num_columns; j++)
      t[i][j] = 0;

  // Print the value in table t, row by row
  for (i = 0; i < num_rows; i++) {
    for (j = 0; j < num_columns; j++)
      printf ("%d ", t[i][j]);

    printf ("\n");
  }

  // free the dynammically allocated space before terminating
  for (i = 0; i < num_rows; i++) {
    free (t[i]);
  }

  free (t);

  return 0;
}
```

Output:
```
0 0 0 
0 0 0 
```

Example file: `malloc_example2.c`
```c
#include <stdlib.h>  // to use malloc, free
#include <stdio.h>

typedef unsigned int addr_t;    // memory address type

typedef struct {                // struct type for storing
  int a;                        // an integer and
  addr_t c;                     // a memory address
} entry_t;

typedef entry_t* row_t;         // define row type
typedef row_t* table_t;         // define table type

table_t t;

int main() {
  int num_rows = 2;
  int num_columns = 3;

  int i, j;
  t = (row_t*) malloc (sizeof (row_t) * num_rows);

  for (i = 0; i < num_rows; i++) {
    t[i] = (entry_t*) malloc (sizeof (entry_t) * num_columns);

    for (j = 0; j < num_columns; j++) {
      t[i][j].a = i;
      t[i][j].c = j;
    }
  }

  for (i = 0; i < num_rows; i++) {
    for (j = 0; j < num_columns; j++)
      printf ("%d%u ", t[i][j].a, t[i][j].c);

    printf ("\n");
  }

  for (i = 0; i < num_rows; i++) {
    free (t[i]);
  }

  free (t);

  return 0;
}
```

Output:
```
00 01 02 
10 11 12 
```

There are two ways to create dynamically allocated 2D arrays. The first is to simply use a 1D array. This approach requires a function that maps row index `i` and column index `j` to the appropriate index of the 1D array (e.g., `i * NUMBER_OF_COLUMNS + j`).

Another approach is to create a multi-level array. To do this, it is helpful and cleaner to use the C `typedef` instruction to define new types for a table row and for the table itself:

```c
typedef int* row_t;
typedef row_t* table_t;
```

The new type `row_t` is just a new name for the `int*` type. As we’ve seen in [Memory allocation](https://d2l.depaul.edu/content/enforced/1120653-CSC_374_501_1165/basics-of-c.html?ou=1120653&d2l_body_type=3#orgc75e422), it can be used to refer to an array of integers, i.e., a row of a 2D array of integers.

The new type `table_t` is defined as `row_t*` and can thus be used to refer to an array of `row_t` entries. In other words, a `table_t` variable will refer to an array of (integer) arrays, i.e. a 2D array of integers.

This applies naturally to 2D arrays of any type. Note that free’ing a 2D array requires to [`free(3)`](https://man7.org/linux/man-pages/man3/free.3.html) each of the rows, then [`free(3)`](https://man7.org/linux/man-pages/man3/free.3.html) the array of rows.

## 23. Command line arguments

Example file: `cmdline0.c`
```c
#include <stdio.h>

int main (int argc, char* argv[]) {
  if (argc == 2)
    printf ("Hello, %s!\n", argv[1]);
  else
    printf ("Hello, World!\n");

  return 0;
}
```

Output:
```
$ ./cmdline0 a b c
Hello, World!
```

Example file: `cmdline1.c`
```c
#include <stdio.h>

int main (int argc, char* argv[]) {
  int i;

  printf ("%d arguments\n", argc);

  // argv[0] is the name of the program, skip it.

  for (i = 1; i < argc; i++)
    printf (" %d: %s\n", i, argv[i]);

  return 0;
}
```

Output:
```
$ ./cmdline1 lorem ipsum
3 arguments
 1: lorem
 2: ipsum
```

Programs can take command line arguments:

```shell
$ ./cmdline0 you
Hello, you!
$ ./cmdline0 goodbye
Hello, goodbye!
$ ./cmdline0
Hello, World!
```

The `main` function can be prototyped to receive these arguments:

```c
int main (int argc, char** argv)
```

The argument `argc` will be assigned the number of (blank separated) command line arguments when the program is executed; this includes the name of the executable (`./cmdline0` in the above examples). The argument `argv` is an array of strings that contains all the command line arguments. For example, in

```shell
$ ./cmdline0 Opsadelie
Hello, Opsadelie!
```

the `argc` is 2, `argv[0]` is the string `"./cmdline0"` and `argv[1]` is `"Opsadelie"`.