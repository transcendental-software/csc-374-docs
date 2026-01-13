---
title: Shell Lab
---

## 1. Lab Assignment 1 Shell Lab

Fluency with the shell is a life skill. How can you list 10 header files in `/usr/include/` that have at least 100 lines? How can you get all the unique second fields in a bunch of [CSV files](https://en.wikipedia.org/wiki/Comma-separated_values)? How do you add a `#include` at the top of several files automatically? All of these tasks can be done in one line of shell scripting:

```shell
$ cd /usr/include; wc -l *.h | grep ' [1-9]\{3\}' | head -n 10
     957 aalib.h
     244 aio.h
    1596 alpm.h
     436 ansidecl.h
     138 a.out.h
     721 archive_entry.h
     437 argon2.h
     564 argp.h
     156 argz.h
     813 aspell.h
$ cd ~/src/algs4/data; cut -d, -f2 *.csv | sort | uniq -u
[...]
$ cd ~/src/mycode; sed -i '1s/^/#include\n/' *.c
```

The kind of commands in the above example are limited. To iterate through files, for instance, I have used [globbing](https://tldp.org/LDP/abs/html/globbingref.html), i.e., simple wildcards: `*` is all files (including directories), `*.h` is all files ending with `.h`, and `[a-z]*` would be all files starting with a lowercase letter. These are expanded by the shell, in the sense that they are replaced by the actual list of files before calling the program. For instance, `echo` just prints its arguments, nothing else, yet:

```shell
$ echo hello world
hello world
$ echo *.c
main.c lib.c
```

If we need to iterate through files, rather than pass all filenames to a program (as with `wc`, `cut`, and `sed` above), we need an actual programming language. This is provided by your shell: it has `if` statements, `for` and `while` loops, and `case` switches. We will go over these and over a few commands in this lab.

Shell scripts are slow, inefficient, in the same way a Swiss-army knife is bad at all the jobs you throw at it, but it can do them. Its power resides in the ability to express complicated tasks easily, making it extraordinarily common and useful for one-time or simple tasks. As with most scripting languages, there are several ways of doing any task (for the examples above, see [this](https://stackoverflow.com/questions/17058184/file-with-the-most-lines-in-a-directory-not-bytes), [this](https://www.unix.com/shell-programming-and-scripting/121170-return-list-unique-values-column-csv-format-file.html), and [this](https://superuser.com/questions/246837/how-do-i-add-text-to-the-beginning-of-a-file-in-bash)), and although there are better ways than others, shell scripting is mostly about saving time, so the first working solution is the best.

This project is divided into 6 parts and a bonus part. The commands presented in each Part must be used in your solutions, except for the bonus part.

## 2. Downloading the assignment

Start by accepting the GitHub Assignment by clicking on the invitation link from the assignment description on D2L. Then clone your repository in your home directory on the **matrix.cdm.depaul.edu** machine:

```shell
$ cd ~/
$ git clone git@github.com:transcendental-software/csc-374-lab1-USER.git
```

This will cause a number of files to be unpacked into the directory. *You should only modify the files `src/*.sh`, and these are the only files that are submitted.* The `test` folder contains the different tests and grading examples for your program. Use the `make test` command to run the test driver. This would run all the tests for all parts. See Section 11.2 for more finely grained testing options.

## 3. Concepts used across multiple parts

### 3.1. Scripts

Rather than typing commands directly in the prompt, we will write script files. These are text files bearing the extension `.sh` (by convention). They contain a sequence of commands, separated by newlines. If you want to put two commands on a single line, you can separate them with a semicolon. To run your script, you should always indicate the directory in which it is, even if you are currently in that directory (in that case, type `./myscript.sh` as `./` indicates the current directory).

To debug a script, you can issue, within it, the command `setopt xtrace`. This will print each line that the shell executes:

```shell
$ cat test.sh
#!/bin/zsh -f
setopt xtrace # this is a comment!
var=value     # var is a variable
echo $var     # $var means the value of var
$ ./test.sh
+./test.sh:3> var=value 
+./test.sh:4> echo value
value
```

The first line of the script file should indicate which shell we will be using. Each of the solution files already contains that line, called a [shebang](https://tldp.org/LDP/abs/html/sha-bang.html):

```shell
#!/bin/zsh -f
```

This lets the kernel know that it should use the [Z shell](https://en.wikipedia.org/wiki/Z_shell), a standard shell, for interpreting the program. The file is then processed as if it were typed into a shell. To read the documentation of the Z shell, look at its info page (type `info zsh`; search by typing `s`, an index is accessible with key `i`). Use `info info` to learn more about navigation into info pages.

> **Note:** The Z shell is now fairly standard. Since 2019, it is the default shell on macOS, and it is easily installed on all Linux distributions. Its scripting language alleviates a lot of the weird behaviors, peculiarities, and inconsistencies of the older shells (in particular the completely standard [Bourne shell (`sh`)](https://en.wikipedia.org/wiki/Bourne_shell) and [Bourne Again shell (`bash`)](https://en.wikipedia.org/wiki/Bourne_again_shell)), at the expense of backward compatibility.

> **Note:** All of your scripts already have “execution rights,” which is a technicality of the file system. If you write your own scripts and obtain `zsh: permission denied: ./myscript.sh`, this is likely due to the lack of execution rights. You can solve this by adding the right, using `chmod +x ./myscript.sh` (see the manpage of `chmod` for more).

### 3.2. Standard input: `<` and `|`

At this point, you should be familiar with the standard input in C. This is the “file” that corresponds to the user interactive input which we read to get commands in the Dict Lab. Most text processing commands read by default the standard input, for instance, here is `sort`, a program that sorts the lines in input:

```shell
$ sort
3
1
4
1
[Ctrl-d]
1
1
3
4
```

> **Note:** As we’ve seen in the Dict Lab, when the input is provided from the shell, the key combo `Ctrl-d` ends the input. The program `sort` needs to read all of its input in order to produce any output. With programs that can produce output as they read input, looking at the terminal can be a bit weird. For instance, the program `cat`, without arguments, reads the standard input and repeats it on the standard output (it’s of nearly no use):
>
> ```shell
> $ cat
> hello
> hello
> how do you do
> how do you do
> good talk
> good talk
> [Ctrl-d]
>```
>
> It takes a good understanding of what the command is doing in order to distinguish the input, typed by the user, and the output, printed by the program. Can you tell which is which?

In a shell, we can replace the standard input with a file. This is done with the `<` operator:

```shell
$ cat myfile
lorem
ipsum
dolor
$ sort < myfile
dolor
ipsum
lorem
```

(This is equivalent to `sort myfile` since `sort` can read the contents from files passed as arguments, see its manpage. This is true of lots of commands.)

We can also replace the standard input of a program with the standard output of another. This fundamental operation is called [piping or pipelining](https://tldp.org/LDP/lpg/node10.html) and uses the `|` operator:

```shell
$ echo hello | sort
hello
```

Thus, the statement `cat file | sort` is functionally equivalent to `sort < file`. We strongly prefer the latter, since it doesn’t require running an extra command.

**Note:** The program `echo` (which is actually a builtin, not a program) prints its arguments followed by a newline. If we wanted multiple lines, we could do:

```shell
$ echo 'line3\nline1\nline2' | sort
line1
line2
line3
```

This is because `echo` interprets the two characters `\n` as a single newline. Another, more advanced option, is to use a subshell, which would have a sequence of `echo` separated by semicolons:

```shell
$ { echo line1; echo line2; echo line3 } | sort
line1
line2
line3
```

Try to answer these questions: What would `cat | cat` do? What about `cat < myfile | cat`?

**Note:** What do the words program, executable, builtin, and command mean?

- **Program, executable**: These are synonymous. They refer to a file on the disk that can be executed (run). This can be a native assembly program (we will see how these files are laid out during the course) or a script (the interpreter of which being given in the shebang).
- **Builtin**: You can “call” builtins in a shell just like a program, but instead of having the shell find the executable file and running it, the shell itself implements that feature. When you type `$ echo hello`, the program `/bin/echo` is not called; instead, the shell just prints `hello`. This is more efficient. Here are some other builtins implemented by the Z shell: `alias`, `bg`, `break`, `cd`, `continue`, `declare`, `echo`, `eval`, `exec`, `exit`, `export`, `false`, `fg`, `history`, `jobs`, `kill`, `printf`, `pwd`, `read`, `return`, `shift`, `source`, `stat`, `trap`, `true`, `wait`, `which`.
- **Command**: This is the general term for a single instruction in a shell. They can either run a program or a builtin.

### 3.3. Quoting

Shell scripting is a fairly loose language in terms of typing. A string `"echo"` can be used as a command and the strings `'test'` and `"test"` are the same from the point of view of the shell. So why and when do we use quotes?

- **Grouping**. If we have a file named `file with spaces.txt`, then if we pass it to `cat` (which just dumps the contents of the file to the standard output), we need to make sure that `cat` takes this filename as one argument. Compare:

```shell
$ cat filename with spaces.txt
cat: filename: No such file or directory
cat: with: No such file or directory
cat: spaces.txt: No such file or directory
$ cat "filename with spaces.txt"
[contents of file]
```

- **Preventing expansion**. Variables (starting with `$`, see The variables `$#` and `$@`) and some special characters (mostly backslash) are interpreted in a different way by the shell. Compare the following, where `echo -E` is used to prevent any kind of interpretation by `echo` itself:

```shell
$ myvar=value
$ echo -E 'the\ val:\ \"$myvar\"'
the\ val:\ \"$myvar\"
$ echo -E "the\ val:\ \"$myvar\""
the\ val:\ "value"
$ echo -E the\ val:\ \"$myvar\"
the val: "value"
```

In words, single quotes do not change their contents at all, double quotes interpret `\"` as double quotes and evaluate variables, and without any quotes, backslashes are removed and variables evaluated.

- **Raw spaces**. It makes sense for some commands to pass a space as argument; this is the case for `cut` for instance, which selects fields in a line given a delimiter. If you want the third word of a line, you might say “use space as delimiter.” To do so, you need quotes: `cut -d ' '`. See the Bonus Part for more. An even more extreme situation is when you want to pass an empty argument to a program; you would write `cmd ''`.

In the following, `echo` receives only one argument; can you explain the behaviors?

```shell
$ echo -E lorem"'"ipsum
lorem'ipsum
$ echo -E 'lo '\'' rem'
lo ' rem
$ echo -E ''""''""''

$ echo -E '\'\'"' \""
\'' "
```

### 3.4. Regular expressions

Two of the most common programs used in scripting are `sed` and `grep`. They both rely on patterns to match lines of input files, and do something with them (`sed` transforms them, while `grep` just prints the matching lines). Patterns are universally expressed using regular expressions (regex), not to be confused with globbing. We will make sparse use of regexes in this lab, but it is an indispensable computer skill that you will sharpen over the years. Regexes are implemented in each and every programming language.

A regex is a string that describes a simple string property, for instance “starts with some repetitions (possibly 0) of `b`, followed by a sequence of 3 digits and ends with a nondigit” (e.g., `bb314c` and `2a` match it, but `314` and `c1b` don’t). Rather than introducing regexes completely and formally, we will go through examples.

| Regex | Meaning | Match examples | Nonmatch examples |
| :--- | :--- | :--- | :--- |
| `abc` | the exact string abc | `abc` | `lol`, `ok` |
| `\(ab\)*` | any repetition of ab | `abab`, the empty string | `a`, `ba` |
| `ab*` | a followed by any number of b | `a`, `ab`, `abb` | `abab`, the empty string |
| `.` | any single character | `a`, `8`, `#` | `az`, the empty string |
| `.*` | any number of character | `ab3k`, the empty string | none, all strings match |
| `a.*b` | starts with a, ends with b | `ab`, `aloremb` | `a`, `b`, `lorem` |
| `[abc]` | either a, b, or c | `a`, `b`, `c` | `ab`, `bc`, the empty string |
| `l[ou]c` | either loc or luc | `loc`, `luc` | `lc`, `louc` |
| `l\(o\|u\)c` | same as previous with or `\|` and grouping | same as previous | same as previous |
| `ab\|bc` | either ab or bc | `ab`, `bc` | `abc`, `abbc` |
| `x\(ab\|ba\)*y` | x, then sequence of ab or ba, then y | `xy`, `xaby`, `xbaaby` | `xaay`, `xbby`, the empty string |
| `[a-z]` | any lowercase letter | `a`, `z`, `t` | `0`, `#`, the empty string |
| `[a-zA-Z0-9]` | any alphanumeric character | `c`, `H`, `7` | `#`, `lo`, the empty string |
| `[^a-z]` | any character but lowercase letters | `8`, `#`, `K` | `t`, `KO`, the empty string |

Regexes are used in `grep`, `sed`, and many applications to find a substring that follows a pattern within a string (e.g., find `a.*e` in `kloatre`: the substring `atre` matches it). We can thus additionally ask for the regex to match only at the beginning or the end of the string:

| Regex | Meaning | Match examples | Nonmatch examples |
| :--- | :--- | :--- | :--- |
| `^[0-9]` | starts with a digit | `0t`, `44tro1` | `t0lt`, the empty string |
| `[a-z]$` | ends with a lowercase letter | `0k b`, `lo#l` | `a0k8`, the empty string |

Let’s now cover some more involved regexes:

- `/[^/]*$`: This finds a sequence of characters that are not slashes at the end of the string. If the string where a path `/usr/include/file.h`, then this would match the substring `/file.h`.
- `^[a-zA-Z0-9]*$`: This only matches alphanumeric strings.
- `ff\(-[0-9]\|\)$`: The string ends with `ff` or with `ff-N` with `N` a digit.
- `\(^\|,\)[^,]*`: A substring that matches this regex is a sequence of characters that are not commas that either appears at the beginning of the string or after a comma. This is useful, for instance, to find fields in a CSV field.
- `[^,]*\($\|,\)`: This has the same effect as the previous one, with a different strategy: we match a sequence of noncommas that is either at the end of the string or followed by a comma.

It is often much easier to write a regex than to read one, so you can practice with the following (these are harder than what you’ll need in the lab):

1. Write a regex for strings that start with `a`, end with `b`, and contain at least one `c`.
2. Write a regex for strings that are IPv4 addresses (0.0.0.0 to 255.255.255.255).
3. Write a regex for strings that are dates (say from 1/1/1900 to 12/31/2099).
4. Write a regex that matches the substring of a CSV line that corresponds to the first two fields (e.g., matching `x,y` in `x,y,z,t`).
5. We’ve seen that `^.*$` is any string. How would you write a regex for strings that contain only one kind of letter? The strings `aaaaaa`, `bb`, and `ccccc`, for instance, should match it, but not `ab` or `bc`. (There’s no “good” way of doing it with the regexes we’ve seen.)

To test regexes, you can use `grep`. Its first argument is a regex and it reads from the standard input by default. `grep` will print the matching input lines:

```shell
$ echo "lorem 314" | grep '^[^0-9]*1'
$ echo "ipsum 159" | grep '^[^0-9]*1'
ipsum 159
$ grep '[a-z]'
NOTREPRINTED BY GREP
Reprinted by grep
Reprinted by grep
314159 (NO LOWERCASE)
314159 (with lowercase)
314159 (with lowercase)
```

(Note that the repetitions are printed by `grep`.)

Programming languages will always implement at least the regexes we just saw. However, many will implement all sorts of extensions, see for instance the Python module for regex and the info page for `grep` (run `info grep` then go to section 3).

**Note:** The backslashes in grouping and disjunction (e.g., `\(a\|b\)`) are cumbersome. The simplest extension of regexes drops them (`grep -E` implements that, and so does Python). Further extensions allow nongreedy iterations (e.g., `^.*a` matches the last `a`, as `.*` is taken for as long as possible; some languages implement `^.*?a` to say “match `.*` on the shortest possible string”), and backward references (with Perl-style regexes, that `grep` understands with the `-P` flag, the regex `^(.)\1*$` matches any repeated sequence of the same letter). To go further, you can look at the regex howto of Python.

## 4. Part 1: Most frequent line

In this part, you will write a script that reads lines from the standard input and prints out the line that appears most often at the end.

### 4.1. Example use

```shell
$ ./maxocc.sh
lorem
ipsum
dolor
ipsum
sit
amet
sit
ipsum
[Ctrl-d]
ipsum
```

### 4.2. sort, uniq, tail, sed

The solution (or, really, a solution) for this part combines a few programs on a single line, using pipes.

- **sort**: We’ve seen that one, but as with every program, options can change its behavior. Using `sort -n`, the program will sort according to the numerical value of the line, so that if a line starts with `34 xyz...` it appears before the line starting with `188 xyz...`, even though the latter would appear first in a dictionary, as 1 appears before 3. Other very useful options are `-u` that flushes the repeating lines and `-r` that reverses the order.
- **uniq**: Without options, this deletes repeated adjacent lines, so as to keep only a unique occurrence of each. Note that In particular `sort -u` and `sort | uniq` do the same thing. With option `-c`, it counts how many occurrences of the repeated lines appear:

```shell
$ uniq -c
hello
dup
dup
hello
[Ctrl-d]
      1 hello
      2 dup
      1 hello
```

- **tail**: The programs `head` and `tail` print the first few or last few lines of a file (or the standard input). The number of lines is indicated with the `-n` option. The program `tail` has a very useful option `-f` to follow a file (not used in this lab): every time the file grows, `tail` would print the new lines; this is often used to watch log files.

```shell
$ tail -n 1
hello
world
[Ctrl-d]
world
```

- **sed**: We could do a whole lab on `sed` (and its siblings `perl` and `awk`), and I encourage you to look at the info pages to better understand the power of these tools. We will use it in its most common form: substitute a pattern with a string in each line of a file. The command `sed` takes as first argument its instructions and reads from the standard input (if no other arguments are provided), applying the program on each input line. The only instruction we will use is `s/PATTERN/REPLACEMENT/` which replaces substrings matching `PATTERN` with `REPLACEMENT` (only the first match within the line is replaced by default). For instance, if we want to prefix each line with `X:`, we can use:

```shell
$ cat myfile
lorem
ipsum
dolor
$ sed 's/^/X:/' < myfile
X:lorem
X:ipsum
X:dolor
```

If we want to erase a bunch of spaces, a bunch of digits, and a bunch of spaces at the beginning of each line, we can do:

```shell
$ echo '  18  lorem' | sed 's/^ *[0-9]* *//'
lorem
```

### 4.3. Writing the script

Your script should be in the file `maxocc.sh`. As it should read from the standard input, the first command you start can simply read from the standard input. For instance, if your script only contains the command `sort`, then it will behave like `sort`.

The strategy to obtain the line that appears most often is to:

1. Sort the input lines
2. Use `uniq -c` to count how many times each line occurs
3. Use `sort -n` to sort the output of `uniq` numerically
4. Get the line corresponding to the highest count using `head` or `tail`
5. Remove the prefix of the line that countains the count.

## 5. Part 2: Print all arguments

In this part, you will write a script that prints all of its arguments, one per line.

### 5.1. Example use

```shell
$ ./println.sh a b c
a
b
c
$ ./println.sh a 'b c'
a
b c
$ ./println.sh a'\ x' 'b c'
a\ x
b c
```

### 5.2. The variables `$#` and `$@`

Shell scripts can define variables and use them:

```shell
$ var=value; echo $var
value
$ var2=$var; echo $var2
value
```

In a shell script written in a file, there are two variables defined by default, `@` and `#`. The first one is a special variable that contains each of the arguments to the script, the second one contains the number of arguments (`argv` and `argc` in C). For instance, suppose we have a script called `args.sh` that contains the following:

```shell
#!/bin/zsh -f
echo Number of args: $#
echo These args are: $@
```

Then executing it with different arguments give:

```shell
$ ./args.sh
Number of args: 0
These args are:
$ ./args.sh hello world
Number of args: 2
These args are: hello world
$ ./args.sh 'hello world'
Number of args: 1
These args are: hello world
```

Note that `hello world` is two arguments, while `'hello world'` (or `"hello world"`) is one argument. Note that using `echo "$@"` doesn’t allow us to distinguish between these two cases. Separating the arguments with newlines would.

To access a specific argument, say the third, one can use either `$3` (in classic notation) or `$@[3]` (in array notation, starting at 1).

### 5.3. for loops

The Z shell and other shells have a vast diversity of different `for` loop constructions, but we will focus on the classical one. Its syntax is:

```shell
for elt in elt1 elt2 elt3 elt4; do
    # use $elt
done
```

The list of items can be replaced by:

- **An array**: The only array we will be using is `$@`, the array of arguments to your program. To avoid any weird behavior with empty arguments (`myprog '' ''` has two arguments!), we should use `for elt in "$@";` instead of `for elt in $@;`.
- **The output of a program** (not used in this lab). The syntax is then `for elt in $(cmd);`. To “split” the output of `cmd` into elements, the shell uses a special variable `$IFS` that contains the list of characters at which it should split.
- **A globbing pattern** that matches filenames (not used in this lab), for instance `*` that matches all filenames, or `*.h` which matches all filenames with extension `.h`.

### 5.4. Writing the script

Your script should be in the file `println.sh`. You simply have to iterate through the array `$@` and print each of the items you see. To print them even if they contain special characters like backslashes, you will have to use `echo -E` (see Quoting).

## 6. Part 3: Lowercase

The script for this part should turn its input(s) to lowercase. It follows the usual UNIX convention:

1. If no argument is provided, it reads the standard input, and outputs the lowercased version on the standard output.
2. If arguments are provided, these are files that should be read, and their contents printed on the standard output, in lowercase.

### 6.1. Example use

```shell
$ echo 'lOrem' | ./lowercase.sh 
lorem
$ cat t1
HELLo
worlD
$ ./lowercase.sh t1 t1
hello
world
hello
world
```

### 6.2. tr, if

- **tr**: Shell scripting is very much line driven, and `tr` is one standard command that breaks that rule. The command `tr` translates a set of characters to another one. The set of characters can be expressed like the ranges of the regexes `[a-z]`. For instance, uppercasing a word can be done using:

```shell
$ echo hello | tr 'a-z' 'A-Z'
HELLO
```

The i-th element of the first set is mapped to the i-th element of the second (here, `k`, say, is mapped to `K`). Useful options (not for this lab) include `-d`, used to delete a set of characters, and `-c`, used to complement a set. The option `-s` will be seen in the Bonus Part.

- **if**: This builtin of the shell behaves like a normal `if` statement. Its syntax is:

```shell
if CONDITION; then
    ...
else
    ...
fi
```

In there, `CONDITION` can either be:

1. **A program**. The return value of a program is a small integer (0 to 127). In your script, you can exit with a return value `n` using `exit n`. In shell parlance, a program succeeds when it returns 0 and fails otherwise. This means that the true part of the `if` statement is executed if the program returns 0. There are two standard builtins called `true` and `false` that just return 0 and 1, respectively, hence:

```shell
$ if true; then echo hello; fi
hello
$ if false; then echo X; else echo Y; fi
Y
```

After executing any statement, the shell stores the return value of the previous program in the variable `$?`:

```shell
$ true; echo $?
0
$ false; echo $?
1
```

2. **A numerical condition** `(( COND ))`. For instance, one can use `(( $# == 0 ))` or `(( 43 > 88 ))`. Numerical conditions are fairly powerful, as they can also use for instance the ternary conditional operator.
3. **A test condition** `[[ COND ]]` (used once in this lab, in the next exercise; `[[` is an alias for the program `test`). This is used to test file properties and properties of strings. For instance, `[[ -e myfile ]]` tests that a file named `myfile` exists, and `[[ $var == "" ]]` tests that `$var` is the empty string.
4. **A logical combination** of the above (not used in this lab), using `||` (or) `&&` (and) and `!` (not):

```shell
$ if true && ! false; then echo X; else echo Y; fi
X
```

### 6.3. Writing the script

Your script should be in the file `lowercase.sh`. You first want to test whether there are arguments provided to your program; if there is none, just call `tr`, which will read the standard input. If not, you should iterate through all the arguments provided and use `tr` on each. Funnily enough, `tr` does not follow the convention of accepting filenames as arguments, so you should replace its standard input with each of the input filenames, in turn.

## 7. Part 4: Validate input

For this part, your program should check that it receives exactly one argument, and verify that it is alphanumeric using `grep`. On success, it will print `accepted` and return 0, otherwise it will print `rejected` and return 1.

### 7.1. Example use

```shell
$ ./validate.sh
usage: ./validate.sh PASSWD
$ ./validate.sh a b c
usage: ./validate.sh PASSWD
$ ./validate.sh loremipsum
accepted
$ echo $?
0
$ ./validate.sh to+be_rejected
rejected
$ echo $?
1
```

### 7.2. grep, variable as standard input

- **grep**: We have quickly seen `grep` before: it lists all the lines on its input that match a given regex. To use it as a “yes/no” program, we will use the `-q` option. With this option (standing for quiet), `grep` will exit immediately with return value 0 if any match is found, and with value 1 if no match is found. For instance, checking that the input contains an `a`:

```shell
$ echo 'hello' | grep -q 'a'; echo $?
1
$ echo 'target' | grep -q 'a'; echo $?
0
```

- **Variables as standard input**: You will receive the argument as the variable `$1` (that is, the first element of the array `$@` of arguments). To pass it on the standard input of a command `cmd`, the usual way is with `echo "$1" | cmd`. For technical reasons, it is not efficient to use a pipe simply to put a variable on the standard input; modern shells implement the better-looking construct `cmd <<< $1`. Can you explain the difference between this and `cmd < $1`?

**Note:** The “technical reasons” mentioned above have to do with how pipes are traditionally implemented. In a command `cmd1 | cmd2`, the shell would create two new processes, for `cmd1` and `cmd2`, and branch the standard output of `cmd1` to the standard input of `cmd2`, which is a lot of work. Avoiding doing this just for `echo` can save a bit of time. More on this “branching” and “creating processes” during the session!

### 7.3. Writing the script

Your script should be in the file `validate.sh`. You will first test that the number of arguments is correct; if it isn’t, you should print the message `usage: ./validate.sh PASSWD` and exit with code 1. Note that it is bad practice to write `./validate.sh` verbatim in your script, as your program may be called from somewhere else. You should use the variable `$0` which contains the name of the program:

```shell
$ cat t.sh
echo $0
$ ./t.sh
./t.sh
$ cd child_directory
$ ../t.sh
../t.sh
$ cd ..
$ mv t.sh othername.sh
$ ./othername.sh
./othername.sh
$ ././././othername.sh
././././othername.sh
```

After this, you should check that your only argument is alphanumeric using `grep -q` as the condition of an `if` statement. If it is, you should print `accepted` and exit with code 0, otherwise, print `rejected` with code 1.

## 8. Part 5: REPL (Read, Evaluate, Print, Loop)

In this part, you will implement a mid-layer between the user and the shell. Your script should prompt the user for input using `>`, read the user input, evaluate it as a command, and print the outputs of the command. In addition, it should log its activity in a file named `log`, prefixing input commands with `0:`, the standard output with `1:`, and the standard error output with `2:`.

### 8.1. Example use

```shell
$ rm -f log  # delete the file log
$ ./repl.sh
> echo hello\ world
hello world
> cat existing nonexisting
I'm the contents of the file 'existing'
cat: nonexisting: No such file or directory
> [Ctrl-d]
$ cat log
0:echo hello\ world
1:hello world
0:cat existing nonexisting
1:I'm the contents of the file 'existing'
2:cat: nonexisting: No such file or directory
```

### 8.2. while, read, eval, redirecting output to file

- **while**: Your main looping mechanism in this script is a `while` loop, that will say “as long as there is input from the user.” The syntax is:

```shell
while CONDITION; do
    ...
done
```

The `CONDITION` is the same as in an `if` statement.

- **read**: The builtin `read` reads one line of standard input and puts it in a variable. Consider:

```shell
$ echo hello | read var
$ echo $var
hello
```

To avoid the input to be treated specially (with backslashes, field splitting, etc.), `read` takes the option `-r` (for raw):

```shell
$ echo -E 'read\ me' | read var; echo -E $var
read me
$ echo -E 'read\ me' | read -r var; echo -E $var
read\ me
```

The command `read` returns a nonzero status when reaching the end of the input, there is thus this very idiomatic use of it to read each line of the standard input:

```shell
while read -r line; do
    # use $line
done
```

**Note:** To read from a file instead, we would have written `done < myfile` instead of `done`, or equivalently, `< myfile while` instead of `while`. If we had written `while read -r line < myfile` instead, every time `read` would have been called, `myfile` would have been reopened, and this would result in an infinite loop.

- **eval**: This builtin takes one argument and executes it as is, as if interpreted by the shell. Consider:

```shell
$ var='t=8; echo "$t\ x" | sed s/8/9/'
$ echo -E $var
t=8; echo "$t\ x" | sed s/8/9/
$ eval $var
9\ x
```

- **Redirecting output to file**: We have seen one way to redirect the output of a program, using pipes. To redirect to a file we use the `>` redirection:

```shell
$ echo hello > myfile
$ cat myfile
hello
```

The `>` redirection first erases the file, then writes to it. To append to a file, we use `>>`:

```shell
$ echo hello > myfile; echo world > myfile; cat myfile
world
$ echo wide >> myfile; echo web >> myfile; cat myfile
world
wide
web
```

**Note:** There is an alternative, that comes in handy in multiple situations. The program `tee` reads its standard input and writes it to both the standard output and to a provided file. Consider:

```shell
$ echo hello | tee myfile
hello
$ cat myfile
hello
```

Programs can output on the standard output (`echo hello`) or the error output (`echo hello >&2`, not used in this lab). To redirect the error output to a file, we use `2>` instead of `>`:

```shell
$ cat t.sh
echo 'to std out' 
echo 'to error out' >&2
$ ./t.sh 2> stderr
to std out
$ cat stderr
to std err
$ ./t.sh > stdout 2> stderr
$ cat stdout
to std out
$ cat stderr
to error out
```

**Note:** When outputting to the error output (with `>&2`) or taking the error output to a file (with `2>`), the number 2 indicates the so-called file descriptor of the output. Three numbers are reserved: 0 is the standard input, 1 the standard output, and 2 the error output. More on this later in the session.

### 8.3. Writing the script

Your script should be in the file `repl.sh`. It will first print the prompt, using `echo -n '> '` (the `-n` option does not add a trailing newline, and the quotes are necessary since `>` is a special symbol for the shell and we need a space). After this, we enter the loop, reading each line of the standard input using the `while read` construction. In the body of the loop, we:

1. Evaluate the line we just read, redirecting its standard output to a temporary file, and the error output to another. You can use `/tmp/out` and `/tmp/err`, or look into the manpage of `mktemp`.
2. The user should see the outputs, so we `cat` both temporary files (the standard output first); both outputs go on the standard output.
3. We populate the file `log`: we write the line we received, preceded with `0:`, this can be done using `echo -E "0:$line"` and the appropriate redirection. Then we take care of the standard and error outputs of the command just evaluated. We can use `sed` to add a prefix to each line of `/tmp/out` and `/tmp/err` and redirect them to the file `log` (see Part 1 for how to use `sed` to do this).
4. We can now print the prompt `> ` again.

## 9. Part 6: Most similar files

In this part, your script is provided with at least 3 files in argument, and should return the pair that is “most similar.” Our definition of “most similar” is the two files that differ at the furthest character. For instance, `lorem` and `loram` differ at the fourth character, while `lorem` and `lorel` do so at the fifth, so they are more similar.

### 9.1. Example use

```shell
$ grep . f*   # print the contents of all the files with name f...
f1:lolem ipsum dolor sit amet
f2:lorem ipsam dolor sit amet
f3:lorem ipsum dolqr sit amet
f4:lorem ipsum dolor sit amet
f5:lorem ipsum dolor sit apet
$ ./mostsim.sh f*
f4
f5
```

Indeed, the files `f4` and `f5` differ on character 24, all others differ before that. You are guaranteed that the pair of files that are most similar is unique. When a file appears twice, we say that it differs at its size plus one:

```shell
$ ./mostsim.sh f* f2
f2
f2
$ echo hello > f6
$ ./mostsim.sh f4 f5 f6 f6
f4
f5
```

The files `f6` and `f6` are only similar “up to” byte 6.

### 9.2. cmp, wc, seq, more on variables

- **cmp**: This is the most basic tool to check whether two files differ. It simply tells you if they do, and where is the first mismatch; it exits with error code 0 and no output only if they have the same contents. More involved commands, such as `diff`, list the differences between two files, and can be extra useful when you have multiple versions of a file and you’re searching for a change that broke a feature. Here is an example use of `cmp`:

```shell
$ cmp f4 f5
f4 f5 differ: byte 24, line 1
$ echo $?
1
$ cmp f4 f4
$ echo $?
0
```

Note that the output is always the same. In particular, to retrieve the byte indicated by `cmp`, one can replace everything up to and including `byte ` with the empty string (in `sed` parlance, `s/^.*byte //`), and everything including and after the comma with the empty string (in `sed` parlance, `s/,.*$//`).

- **wc**: This command provides statistics on the number of bytes, words, and lines of a file:

```shell
$ wc myfile
 3  3 15 myfile
```

In this part, we are only interested in the `-c` option, that counts the number of bytes (`c` is for character). To have a clean output, without extra spaces and the filename, we can use the following trick:

```shell
$ wc -c < myfile
15
```

This is the filesize of the file. This will be used in case `cmp` does not return any information, meaning that its two input files have the same contents.

- **seq**: You will have to compare each pair of files from `$@`. To avoid comparing a file with itself, or comparing them twice, we will address `$@` as an array (indexed from 1). The `seq` command prints the sequence of numbers between two integers:

```shell
$ seq 1 10
1 2 3 4 5 6 7 8 9 10
$ i=30; seq $((i + 1)) 34
31 32 33 34
```

This last example also illustrates how simple arithmetic computations can be made (although you shouldn’t need more than what is in that example). You can then iterate through the indices of `$@` using `for idx in $(seq 1 $#); do ...; done`.

**Note:** Using `seq` is… old school. Modern alternatives are:
- To use a range: `{1..5}` evaluates to the string `1 2 3 4 5`, and can be used as `for idx in {1..$#}; do ...; done`.
- Use an arithmetic `for` loop:

```shell
for (( i = 1; i <= $#; ++i )) do
    # use $@[i]
done
```

- **More on variables**. Setting a variable is done using the syntax `var=value`, as we’ve seen. We are not allowed to use spaces around the equal sign (`var = value` in a shell means: execute the program `var` with two arguments, `=` and `value`). To set a variable to the output of a program, we can either use `read` as above, or the more idiomatic: `var=$(cmd)`. Note that `cmd` can be any command, for instance:

```shell
$ var=$(cmp f1 f2 | sed 's/byte/char/'); echo $var
f1 f2 differ: char 3, line 1
```

Similarly, to set a variable to, say, the third argument of the script, one can use `var=$@[3]` (or `var=$3`).

### 9.3. Writing the script

Your script should be in the file `mostsim.sh`. First, as in any computation of a maximum, we define a variable that will contain the current maximum number of common bytes between two files; we can set it to -1 at first. Then two nested loops will iterate through `$@`, the outer loop with index `i` from 1 to `$# - 1`, the inner loop with index `j` from `i + 1` to `$#`.

In the body of the inner loop, we compute the “similarity” of the files at `$@[i]` and `$@[j]` (note that `$` is not needed in array indices, but `$@[$i]` would still work). To compute this value:

1. We set a variable to the output of `cmp` on the files `$@[i]` and `$@[j]`, where we isolate with `sed` the byte number of the first difference (refer to `cmp` above to see how `sed` can do that).
2. If that variable ends up empty (e.g., `[[ $var == "" ]]`), this means that the two files have same contents. In that case, we set our variable to the filesize of, say, `$@[i]`, plus one.
3. Now we can check if our variable is bigger than the current maximum: `if (( var > curmax )); then`. If it is, then we can save `$i` and `$j` in some variables `$save1` and `$save2`, then update our current maximum.

At the end, we can print the two filenames at our saved positions within `$@`, using `echo $@[save1]; echo $@[save2]`.

**Note:** As we have seen, arithmetic is done through `$((math expr))`. Everything behaves pretty much like you would expect, including when mixing floating point numbers:

```shell
$ var=3
$ echo $((var / 2))
1
$ echo $((var / 2.0))
1.5
```

Note that the `$` sign in front of variables is optional and that spaces don’t matter. One bug you may experience is with the expression `$(($# - 1))`. This works as expected, but not if you remove the spaces; this is because `$#-` is a variable on its own (its meaning is not important) and `$#-1` is then interpreted as that variable followed by 1. Hence make sure to use either `$((# - 1))` (with our without spaces) or `$(($# - 1))` (with spaces).

## 10. Bonus Part: Reformat the output of wc

The output of `wc` is human-friendly, but it may not be the best format for, say, a spreadsheet application. In this part, we will reformat the output of `wc`. I propose one strategy to do it, but you are free to use any strategy that works.

### 10.1. Example use

```shell
$ wc *.sh
   9   25  124 lowercase.sh
   4   14   94 maxocc.sh
  18   52  382 mostsim.sh
   5   11   51 println.sh
  15   47  297 repl.sh
  13   32  188 validate.sh
   9   26  176 wcreformat.sh
  73  207 1312 total
$ ./wcreformat.sh *.sh
124,25,9,lowercase.sh
94,14,4,maxocc.sh
382,52,18,mostsim.sh
51,11,5,println.sh
297,47,15,repl.sh
188,32,13,validate.sh
176,26,9,wcreformat.sh
```

As you can see, the new format gives, for each file, its number of bytes, words, and lines (which is the reversed order from `wc`), and then the filename, all separated with commas instead of spaces.

### 10.2. tr -s, cut, case, and wc "$@"

- **tr -s**: This option of `tr` “squeezes” the characters that are provided. The output of `wc`, as seen in the example above, adds a variable number of spaces between the columns, which makes field splitting a bit awkward. By using `tr -s`, we avoid this:

```shell
$ echo "   3  1  4 1    5" | tr -s ' '
 3 1 4 1 5
```

- **cut**: This command extracts fields from each input line. To indicate how fields appear, we indicated a delimiter using the `-d` option; to select a field, we use the `-f` option:

```shell
$ echo 'hello:world' | cut -d':' -f1
hello
$ echo 'hello:world' | cut -d':' -f2
world
```

**Note:** Modern shells have an array type, that is less error-prone than trusting the shell to “do the right thing” when it comes to splitting. To convert a string into an array, we indicate how we want to split it. The syntax is a bit awkward, and oftentimes `cut` does the job. But in a case where you’d be interested in extracting multiple fields, it may be better to split with the shell feature; the syntax reads:

```shell
$ mystring='split:me::at:colons'
$ ar=("${(@s/:/)mystring}")
$ echo "1: $ar[1], 2: $ar[2], 3: $ar[3], 4: $ar[4], 5: $ar[5]"
1: split, 2: me, 3: , 4: at, 5: colons
```

The `:` in the splitting can be changed to any separator.

- **case**: The `case` switch in shell scripting is used to compare a string against very simple patterns (globbing, just like for files). For instance:

```shell
case $myvar; in
    a*)    echo 'myvar starts with a';;
    *k*)   echo 'myvar contains a k'
           echo 'but it does not start with a!';;
    [0-9]) echo 'myvar is a single digit';;
    lorem) echo 'myvar is the string lorem';;
    *)     echo 'myvar is something else';;
esac
```

Note the double semicolon at the end of the sequence of instructions for each case. This is somewhat equivalent to the `break` instruction in C, but it is mandatory here.

- **wc "$@"**: The syntax `"$@"` means “reproduce all the arguments to the current script as is.” Hence the command `wc "$@"` passes all the arguments to `wc`. Note that when `wc` receives more than one file, it adds an extra line showing the “total” of bytes, words, and lines. If you are reading the output of `wc` line by line, you may want to skip this extra line if it appears. One way to do it is to check whether the current line ends with `total`, using a `case` switch.

### 10.3. Writing the script

Your script should be in the file `wcreformat.sh`. Here is a possible strategy:

1. Use `wc "$@"` to pass all of your arguments to `wc`.
2. Pipe this into a `tr` that would squeeze the spaces.
3. Pipe this into a `while read` loop (yes, you can do that!).
4. In the loop, check that you are not reading the `total` line. If not, then split the line into its four fields using four `cut <<< $line`, and print them in the right order, separated with commas.

## 11. Evaluation

### 11.1. Points

The parts are graded as follows:

- Parts 1 to 6 are worth 10 points each,
- The bonus part is worth 20 points.

### 11.2. Driver

This project is evaluated by a test driver. You can run the full driver by typing, at the root of your project:

```shell
$ make test
```

This moves to the directory `tests` and runs the driver `./driver.sh`.

**Note:** The driver itself is a shell script. All of your labs are evaluated using shell scripts and you have full access to the sources. They use, at times, advanced constructions, but reading them can be educational.

As you work on each successive parts, you may want to run the driver on each part separately and on smaller test files:

```shell
$ cd tests/
$ ./driver.sh -h
usage: ./driver.sh [-P PART]...
Grade the dictlab.
  -P PART       Grade only part PART (1-6, B).  Default is all.
$ ./driver.sh -P 1
* PART 1: MOST FREQUENT LINE
  ...
PART 1 SCORE  10 / 10
FINAL  10 / 10
```

For each test, the driver prints what is the command executed. For instance, when the driver prints:

```
[OK] ../src/maxocc.sh < files/part1-3
```

it ran `../src/maxocc.sh` with `files/part1-3` as input.

The result, for each test, is readily printed; in bracket, there’s a shorthand notation for the result:

- `OK`: Test passed
- `KO`: Test failed

**Note:** The driver is executed from the `test` directory, hence to access your scripts, it needs to call them as, for instance, `../src/maxocc.sh`, which means “go to the parent directory, then to the `src` directory, then find `maxocc.sh`.”

## 12. Handing in Your Work

When you have completed the lab, you will submit it as follows:

```shell
$ make submit
```

This will package your whole `src/` folder into a tarball that is saved in `~/submissions/`, where the instructor will get it. Since this is simply saved on your account, you can use `make submit` as many times as you want up until the deadline.
