---
title: Malloc Lab
---

## 1. Lab Assignment 4 Malloc Lab

In this lab you will be replacing the GNU libc memory allocator with your own implementation. You will be implementing:

- [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html)
- [`free(3)`](https://man7.org/linux/man-pages/man3/free.3.html)
- [`realloc(3)`](https://man7.org/linux/man-pages/man3/realloc.3.html)
- `malloc_aligned` (Bonus Part)

Except for debugging, the only external function you are allowed to use is [`sbrk(2)`](https://man7.org/linux/man-pages/man2/sbrk.2.html) (and [`memcpy(3)`](https://man7.org/linux/man-pages/man3/memcpy.3.html) in `realloc`). The system call `sbrk` increments the heap break and returns the former value of the break. As a special case `sbrk(0)` returns the current value of the break.

> **Note:** The GNU libc info manual has a section on [Replacing `malloc`](https://www.gnu.org/software/libc/manual/html_node/Replacing-malloc.html). In there, they mention that a full reimplementation of `malloc` should provide:
>
> [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html), [`free(3)`](https://man7.org/linux/man-pages/man3/free.3.html), [`calloc(3)`](https://man7.org/linux/man-pages/man3/calloc.3.html), [`realloc(3)`](https://man7.org/linux/man-pages/man3/realloc.3.html), [`aligned_alloc(3)`](https://man7.org/linux/man-pages/man3/aligned_alloc.3.html), [`malloc_usable_size(3)`](https://man7.org/linux/man-pages/man3/malloc_usable_size.3.html), [`memalign(3)`](https://man7.org/linux/man-pages/man3/memalign.3.html), [`posix_memalign(3)`](https://man7.org/linux/man-pages/man3/posix_memalign.3.html), [`pvalloc(3)`](https://man7.org/linux/man-pages/man3/pvalloc.3.html), [`valloc(3)`](https://man7.org/linux/man-pages/man3/valloc.3.html)
>
> Most of them can be implemented using a single line using just the functions you are asked to implement in this lab. These single-line implementations are *provided* and *automatically added* to your library through the `mallocless.c` file.

## 2. Downloading the assignment

Start by accepting the GitHub Assignment by clicking on the invitation link from the assignment description on D2L. Then clone your repository in your home directory on the **matrix.cdm.depaul.edu** machine:

```shell
$ cd ~/
$ git clone git@github.com:transcendental-software/csc-374-lab4-USER.git
```

This will cause a number of files to be unpacked into the directory. *You should only modify the file `src/malloc.c` and this is the only file that is submitted.* The `tests` folder contains the different tests and grading examples for your program. Use the `make(1)` command to compile your code and the command `make test` to run the test driver. This would run all the tests for all parts. This is more useful during development than in our previous labs, but short tests are still provided: see Section Driver for more finely grained testing options.

## 3. Part 0: Correctness (0pts)

In Part 0, your implementation is run in a simulated environment. This environment crashes as soon as it detects an error, printing an error message.

Part 0 is mandatory: no point is awarded to the lab if your implementation crashes.

> **Note:** There are multiple approaches to running a `malloc` implementation in a simulated environment. The approach taken here is to intercept your calls to `sbrk` using interpositioning (see Ch. 7, Lec. 7). At the beginning of the execution, some limited space is allocated using [`mmap(2)`](https://man7.org/linux/man-pages/man2/mmap.2.html) (see Ch. 9, Lec. 10) and each time your implementation calls `sbrk`, it returns a portion of that buffer. In addition, a page is allocated before that buffer, and its rights changed to `PROT_NONE` (no read nor write accesses, see Cache Lab, Sec. Watching Memory for more on page protection). Any access to that page would crash the program.

## 4. Part 1: Performance on traces (60+ points)

In this part, each test executes a series of `malloc`, [`free(1)`](https://man7.org/linux/man-pages/man1/free.1.html), and `realloc`. For each test, you implementation is run twice:

- Once measuring the time taken to complete all the operations, resulting in a *throughput* measure: the number of operations per second. This is measured in thousands of operations per second (Kops/s).
- Once measuring the *utilization* of your implementation: this is the ratio between the maximum amount `malloc`’ed at any point and the maximum heap size.

With $T$ and $U$ your average throughput and utilization over all tests, your final grade for this part is (roughly) computed as:

$$ 60 \times \left( \frac{T}{1500} \times 0.6 + \frac{U}{65} \times 0.4 \right). $$

This means that a throughput of 1500Kops/s and a utilization of 65% will give you all the points, but also that anything higher would give you *extra points* (capped at 54 points for throughput and 36 points for utilization). As a reference point, implicit list should give you 0 points, explicit list 45, segregated list 60, and any tweaking would push you above that. The GNU libc implementation scores 84 points.

> **Note:** Most of the traces executed come from actual (although partial) executions of some programs. For instance, `tests/progs/grading-p1-firefox.c` contains the successive calls to `malloc` given by Firefox. These accesses were gathered by using a library that was intercepting the calls to `malloc` using `LD_PRELOAD` (see Ch. 7, Lec. 7). A few lines of that file read:
>
> ```c
> void* p848045194363482441 = malloc (31);
> free (p848045194363482386);
> free (p848045194363482387);
> void* p848045194363482442 = malloc (11);
> ```

## 5. Part 2: Performance running Python (50+ points)

In Part 1, we compiled long C files that contain a trace of the memory allocations made by a program. Even for simple programs, these traces grow in the several [mebibytes](https://simple.wikipedia.org/wiki/Mebibyte), making them impracticable to compile. In this part, we simply run Python scripts, replacing the GNU libc `malloc` with your implementation using `LD_PRELOAD`.

In this part, only *speed* is important. The target speed is such that a normal implementation of segregated list would give all points, although you can score up to 75 points by providing a better implementation.

## 6. Bonus Part (40 points)

In this part, your implementation will be used to start Emacs (using `LD_PRELOAD`). The specificity here is that, in addition to requesting memory with `malloc`, Emacs also asks for *aligned* memory with diverse alignments. You will have to implement:

```c
void* malloc_aligned (size_t alignment, size_t size);
```

This function should return a memory address that is a multiple of `alignment`. You have a guarantee that `alignment` is a power of 2 larger than `128`.

You earn all the points if Emacs does not crash.

## 7. Evaluation

### 7.1. Methodology

The tests are mostly programs found in `tests/progs`. These are *dynamically* linked with your implementation. This means that you do not have to recompile the test programs when you update your implementation.

Your code is compiled into two libraries: one with all optimization flags (used for the tests) and one with no optimization and with all debug symbols (for your debugging). These are compiled as `src/libmalloc.opt.so` and `src/libmalloc.so`, respectively.

Each C file is compiled in 5 different versions. For instance, the file `grading-p0-sed.c` is compiled into:

- `grading-p0-sed`: This is linked with the debug version of your library, it is used for your debugging. Note that calls to `malloc` and [`free(1)`](https://man7.org/linux/man-pages/man1/free.1.html) are not “direct”; they go through some functions that I implemented to help you debug, and malloc is run in a “virtual environment”. *Use this with GDB if you want to trace your program.*
- `grading-p0-sed.stu.tim`: This variant runs the program and outputs the throughput of your implementation. (Used in Part 1.)
- `grading-p0-sed.stu.utl`: This variant runs the program and outputs the utilization of your implementation. (Used in Part 1.)
- `grading-p0-sed.glc.{tim,utl}`: These are the throughput/utilization tests, but running the GNU libc implementation.

For Part 2 and the Bonus Part, the programs, Python or Emacs, are run with your implementation using `LD_PRELOAD`.

### 7.2. Driver

This project is evaluated by a test driver. You can run the full driver by typing, at the root of your project:

```shell
$ make test
```

This makes sure that your program is compiled with the latest sources, moves to the directory `tests`, and runs the driver `./driver.sh`.

As you work on each successive parts, you may want to run the driver on each part separately and on smaller test files—this is less useful in this lab than in previous labs, except for Part 0:

```shell
$ cd tests/
$ ./driver.sh -h
usage: ./driver.sh [-sg] [-P PART]...
Grade the malloclab.
  -P PART   Grade only part PART (0, 1, 2, B).  Default is all.
  -s        Use short tests (not for grading).
  -g        Run the tests on the GNU libc implementation.
$ ./driver.sh -s -P 0
* PART 0: Correctness
[OK] progs/short-p0-10.stu
[OK] progs/short-p0-11.stu
...
```

For each test, the driver prints what is the command executed. The result, for each test, is readily printed; in bracket, there’s a shorthand notation for the result:

- `OK`: Test passed.
- `KO`: Test failed, your program printed something unexpected.
- `TO`: Time out, your program took too long to answer, it’s usually caused by an infinite loop.

## 8. Tools & Libraries

### 8.1. Size to bins

In the implementation of segregated lists, you will want to map the size of free blocks to their lists. These lists are generally known as *bins*. The optimal number `NBINS` of bins and the optimal mapping `SIZE_TO_BIN` are up to debate, however, you are allowed to use the GNU libc `malloc`’s versions. They are provided as macros `NBINS` and `SIZE_TO_BIN` in the file `malloc.h`, you can thus use, for instance:

```c
size_t bin_number = SIZE_TO_BIN (payload_size (h));
for (bin_number++; bin_number < NBINS; ++bin_number) ...
```

### 8.2. GNU libc implementation

You are allowed to take inspiration from the GNU libc implementation. It is long, it is complex, but it is well documented. You can find it here. Note that you are not allowed to use `mmap(2)` in your implementation.

### 8.3. GDB

Refer to the Dict Lab writeup for info on GDB. To run one of the tests using your implementation of `malloc`, use the program name printed by the driver:

```shell
$ make test
* PART 0: Correctness
[KO] progs/grading-p0-amptjp
CTRL-c
$ cd tests
$ gdb progs/grading-p0-amptjp
(gdb) b malloc
Breakpoint 1 at 0x1040
(gdb) r
Starting program: /home/micha/src/malloclab/tests/progs/grading-p0-amptjp
Breakpoint 1, malloc (plsize=2040) at malloc.c:170
170       if (params.head == NULL)
```

You can also run *any* program in GDB and attach your library:

```shell
$ gdb emacs
(gdb) set environment LD_PRELOAD src/libmalloc.so
(gdb) b malloc
Breakpoint 1 at 0x43a70
(gdb) r -nw -Q
Starting program: /usr/local/bin/emacs -nw -Q
Breakpoint 1, malloc (plsize=4) at malloc.c:170
170       if (params.head == NULL)
```

### 8.4. Efficient debugging

Debugging an implementation of `malloc` is an art in itself. The tests of Part 0 can only detect so much and you may end up with unexpected crashes. As with any piece of code, it is better to make it crash yourself than having the user do it.

#### 8.4.1. Checking assumptions

You should make sure that your assumptions about the state of memory are always correct; there are two ways to do that:

- Use [`assert(3)`](https://man7.org/linux/man-pages/man3/assert.3.html). For instance, when you assume that a function returns a free block `b`, you may want to check this:

  ```c
  block = get_free_block (size);
  assert (block->free);
  ```

  Your program *will crash* with a useful error message if this condition is not satisfied:

  ```
  grading-p0-ls: malloc.c:128: main: Assertion `block->free' failed.
  [1]    650575 abort (core dumped)  ./a.out
  ```

  You can then run your program in GDB, have it crash, and go up the call stack to see what caused the assertion to be false.

- Use a consistency checker. This is a function you would call at the beginning and end of each of your `malloc`, [`free(1)`](https://man7.org/linux/man-pages/man1/free.1.html), and `realloc`. It would check that the memory is in the state you expect. For instance, you may want to check that all the blocks in the list of free blocks is marked free (traversing the list), and that every block that is free is in the free list (traversing the memory linearly).

The calls to `assert` are automatically removed when compiled as the optimized library: as indicated in the man page [`assert(3)`](https://man7.org/linux/man-pages/man3/assert.3.html), if the macro `NDEBUG` is defined, then `assert()` generates no code. Your optimized library is compiled using the command line flag `-DNDEBUG`, which defines the macro `NDEBUG`. However, the consistency checker is yours, so you should make sure that it does nothing when `NDEBUG` is defined:

```c
void check_consistency () {
#ifndef NDEBUG
  //...
#endif
}
```

**Note that all the tests run by the driver are using your library with `NDEBUG`. If your program fails a test, you will want to run the version linked with the nonoptimized library (see Evaluation Methodology).**

#### 8.4.2. Printing debug information

During development, you may want to use [`printf(1)`](https://man7.org/linux/man-pages/man1/printf.1.html). Note that [`printf(1)`](https://man7.org/linux/man-pages/man1/printf.1.html) calls `malloc`, so in theory, you should not be able to do this. However, your library is compiled with a simple reimplementation of [`printf(1)`](https://man7.org/linux/man-pages/man1/printf.1.html), so you *can* use it. Look in `mallocless.c` if you are curious.

## 9. Handing in Your Work

When you have completed the lab, you will submit it as follows:

```shell
$ git add src/malloc.c
$ git commit -m "update code"
$ git push
```

**You may commit and push your code as many times as you want. Just keep in mind that you will also need to submit the screenshot proof of your submission on D2L.**