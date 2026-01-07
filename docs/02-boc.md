---
title: Basics of C
---

# Lab Assignment 0 Basics of C

The following is a short intro to C.

## 1. Example files
All the example files are to be found on our server; see the Basics of Shell for instructions to connect to it. The example files are located in `~prof/00-boc`, which you can copy in your own folder:

```shell
$ cp -r ~prof/00-boc .
$ cd 00-boc
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

All of these steps are taken care of by `gcc(1)` when you call it:

```shell
$ cp ~prof/00-boc/hello.c .
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

For simplicity in these notes, I will compile without the option `-Wall`, but you should always compile your files with that option, that tells `gcc(1)` to give us warnings for nonfatal programming errors:

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

In the first `printf(3)` we supply a string literal. In the second `printf(3)` call we pass in a format string, followed by one or more arguments (the `x` and `y` variables). The format string is printed as is, except for placeholders denoted with a `%` symbols. The values of the arguments (`x` and `y`) will be printed exactly where the placeholders are. Each placeholder (`%`) is followed by a conversion instruction—`d` in our examples—that says how the value is to be printed (in decimal notation, or binary, hexadecimal, etc.) In our case, `%d` means decimal notation. At the end of the format there is an escape sequence `\n` which denotes a newline.

To read data from the keyboard our first C program uses:

```c
scanf ("%d", &x);
```

The first argument is, again, a format string. It is used to instruct the function `scanf(3)` how to interpret what the user types. In this case it instructs `scanf(3)` to interpret the user’s input as an integer in decimal notation. The second argument contains the name of the variable where the input is to be stored (`x`). Note the ampersand character (`&`) before the `x` variable. This ampersand character is the address-of operator. This operator, when placed before a variable, produces the address in memory of that variable, rather than the value of the variable. This address value is a positive memory address between 0 and however much memory you have on your computer. `scanf(3)` needs the address of `x`, not the value of `x`, because `scanf(3)` wants to know where to store the numeric conversion of the string you typed.

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

The functions `printf(3)` and `scanf(3)` are provided by the Standard C Library. For instance, `printf(3)` relies on `write(2)` to actually print to the terminal (this notation, with the section number in parentheses, is standard).

## 5. Basic data types and printing them
Example file: `data-types.c`

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

## 6. Arithmetic operations
Example file: `arithmetic.c`

Notes:

- `/` is the integer division operator when the numerator and denominator are integers.
- `%` is the modulus (remainder) operator.

## 7. Loops
Example file: `loops.c`

Example file: `sum.c`

Example file: `temp.c`

There are three types of loops in C:

- `for` loop, with an initialization, condition, and “afterthought”;
- `while` loop, with a condition that is tested before each iteration;
- `do-while` loop, with a condition that is tested after each iteration.

## 8. More on the preprocessor
Example file: `temp.c`

Note the use the `#define` line to define symbolic constants `LOWER`, `UPPER`, and `STEP`. It is good practice to use them instead of “burying” numbers like 300 and 20 deep inside the code. Symbolic constants can refer to values of any type, they are just considered as text.

The command `#define STEP 20` causes any occurrence of the word `STEP` in the source file to be replaced by 12 by the preprocessor. Note also the use of the function-like macro `ftoc`; the code fragment `(5.0/9.0)*((fahr)-32)` will be used to substitute every occurence of `ftoc(fahr)` in the source code.

The C preprocessor (`cpp(1)`) interprets (and removes) all the lines that start with `#`. A slightly more complex example is given with `preproc.c`:

Example file: `preproc.c`

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

Example file: `terminal.c`

Note that the conditional expression must be in parentheses. Compound conditional expressions are created using `||` (or), `&&` (and), and `!` (not).

## 10. Flow Control
Example file: `flow.c`

C supports two statements that modify the execution of a loop:

- `continue`: skip the rest of the loop iteration and continue with the next iteration (if the loop condition still holds).
- `break`: skip the rest of the loop iteration and exit the loop

## 11. Functions
Example file: `power.c`

Parameter passing: In C, function arguments are passed by value. This means that the called function is given copies of the original values. For example, when the main program executes `power(2, i)` when the value of `i` is 5, copies of 2 and 5 are written to local variables `base` and `n`: they are local to the execution of function `power`.

## 12. Pointers
Example file: `pointers.c`

We already saw that the `&` operator is used to obtain the (memory) address of a variable. That means that the function `scanf(3)`, for instance, takes as argument a variable whose type is a pointer to an `int`.

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

Example file: `pointers3.c`

Example file: `pointers4.c`

In C, arguments to functions are passed by value. So, does the below function `swap` work correctly?

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

## 14. Arrays and strings
Example file: `arrays.c`

Example file: `str.c`

Arrays store a set of “indexable” values of the same type. For an arrays with `n` entries, the valid index values are from `0` to `n-1`.

A C string is simply an array of characters; because the size of an array is not embedded in the variable, in C we use the character `\0`, the ascii-zero character, to indicate the end of the string.

An array is in fact simply a pointer to the first element of the array.

## 15. C pointer arithmetic
Example file: `pointers6.c`

Pointers can be incremented, and the actual increment in memory address depends on the type that is pointed.

```c
int y;
int* p = &y;
p = p + 5;
```

Here, the pointer `p` is incremented by 5 * 4 = 20 bytes. This is because `p` is an pointer to `int` (4 bytes) and what `p + 5` really means is the 5th integer after the one that `p` references.

## 16. C pointers and arrays
Example file: `pointers7.c`

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

## 18. Structures
Example file: `struct.c`

Example file: `struct2.c`

A structure is a grouping of several variables, possibly of different type, under a single name. The `typedef` instruction, which defines a new (user-defined) type, can be used to facilitate the usage of structs.

## 19. Structures and pointers
Example file: `struct3.c`

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

The function `malloc(3)` is C’s version of the `new` operator you have seen in Java. When you use `new` in Java, some space in memory is allocated to store the object’s instance variables.

C has no notion of object (or class), but it does support dynamically allocated memory using the `malloc(3)` function. This function takes as input an `int n` and allocates `n` consecutive bytes in memory; it returns a pointer to the first of the `n` bytes. The signature of `malloc(3)` is:

```c
void* malloc (int n)
```

The return type is a generic pointer, `void*`. To dynamically create a new integer, one uses `malloc(3)` to allocate 4 bytes. However, in order to use these 4 bytes as an `int`, you will need to cast (i.e., change the type of) the `void*` pointer into a `int*` pointer. Therefore, the typical usage of `malloc(3)` to allocate an integer dynamically is:

```c
int* iptr = (int*) malloc (4);
```

It would be cumbersome (and error prone) to remember or compute the amount of memory that each type requires—4 bytes in the case of `int`. A builtin of C is the function `sizeof`, that takes a type and return the amount of memory that a variable of this type needs. A typical initialization of an array of 10 integers thus look like:

```c
int* array = (int*) malloc (10 * sizeof (int));

for (int i = 0; i < 10; i++)
  array[i] = 0;
```

It is imperative to free the memory that has been `malloc(3)`’ed after use. There is no garbage collector in C. If no variable points to a `malloc(3)`’ed block, then nothing magical will happen to make this memory available again. Thus, after use, one has to actually call:

```c
free (array);
```

## 21. Two-dimensional arrays
Example file: `array2D.c`

## 22. Dynamically allocated two-dimensional arrays
Example file: `malloc_example1A.c`

Example file: `malloc_example1.c`

Example file: `malloc_example2.c`

There are two ways to create dynamically allocated 2D arrays. The first is to simply use a 1D array. This approach requires a function that maps row index `i` and column index `j` to the appropriate index of the 1D array (e.g., `i * NUMBER_OF_COLUMNS + j`).

Another approach is to create a multi-level array. To do this, it is helpful and cleaner to use the C `typedef` instruction to define new types for a table row and for the table itself:

```c
typedef int* row_t;
typedef row_t* table_t;
```

The new type `row_t` is just a new name for the `int*` type. As we’ve seen in Memory allocation, it can be used to refer to an array of integers, i.e., a row of a 2D array of integers.

The new type `table_t` is defined as `row_t*` and can thus be used to refer to an array of `row_t` entries. In other words, a `table_t` variable will refer to an array of (integer) arrays, i.e. a 2D array of integers.

This applies naturally to 2D arrays of any type. Note that free’ing a 2D array requires to `free(3)` each of the rows, then `free(3)` the array of rows.

## 23. Command line arguments
Example file: `cmdline0.c`

Example file: `cmdline1.c`

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
