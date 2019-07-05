---
layout: post
title: "Command line arguments"
description: >-
  A rant about command line arguments and how to use them in programs 
  and scripts.
date: 2019-07-05
author: Bert Vandenbroucke
tags:
  - Tools
  - Code Development
---

Command line arguments are very important, as they provide (initially) 
the only way for a program or script to interact with its users. True, a 
program or script could just read all its input from a specific file or 
interactively query for user input at run time (both of these happen in 
science), but those are not very transparent or portable approaches. 
Much cleaner is to provide at least some run time parameters directly to 
the program, and then command line arguments are the way to do this.

In this post, I will briefly explain my view on command line arguments 
and how to use them. I will also introduce some of the tools that greatly
facilitate the use of command line arguments, both in C(++) code and in
Python scripts.

# The basics

Anyone who has ever written a program in C or C++ will be painfully 
aware of what happens to command line arguments: they are the additional 
arguments passed on to the prototype `main` function that forms the 
entry point of any C(++) program:

```
int main(int argc, char **argv) {
  return 0;
}
```

The argument `argc` will receive the total number of command line 
arguments, while the `argv` argument will contain the individual command 
line arguments, as unformatted C strings. These two arguments are the 
only way for the operating system kernel to communicate with your 
program directly, all other communication needs to be initiated 
explicitly by your program. The `return` statement is the only way for 
the program to directly communicate back to the kernel and contains the 
*exit code* of the program. An exit code of zero means the program 
successfully terminated without errors, any other code indicates a 
program during execution. Other operating system tools might do 
something useful with this exit code, so it is important to make sure it 
is accurate.

In Python, there is no such a thing as a *script entry point*, but 
nonetheless command line arguments are available in a very similar 
manner: they are stored in a list called `sys.argv`:

```
import sys

argc = len(sys.argv)
argv = sys.argv
```

The `argc` and `argv` in the above example will contain exactly the same 
values as the C(++) equivalent.

Upon program entry, the command line arguments are hence simply stored 
(in order) in an array of unformatted strings, and the only information 
your program has is the number of arguments. If you want to do anything 
useful with these, you will have to access the individual arguments and 
*parse* them.

A naive approach (widely used in science) is to simply remember in what 
order the command line arguments are expected to be, and then simply 
feed them in that order. So if your first argument is for example the 
name of a parameter file that contains all the simulation parameters, 
then you simply make sure that name is always the first argument, and 
always parse it as a string. There are two problems with this. First of 
all, this approach relies a lot on a good (external) documentation, as 
there is no way of knowing that the first argument is supposed to be the 
name of a parameter file. Second, this approach does not work for 
optional arguments, because omitting them will change the inherent order 
of other arguments. So while this naive approach is very useful for some 
cases (there are plenty of small GNU tools that use this approach), I 
would advise against it in many cases.

# Command line parsers

To facilitate working with command line arguments, a variety of tools 
has been developed, along with a standard that streamlines the approach 
used by for example GNU programs. This standard distinguishes between 
command line options that act as *flags* and have meaning simply because 
of their presence, and command line options that have additional 
arguments that provide input for the program. The standard also 
specifies the difference between *optional* and *required* arguments, 
and sets out how to deal with *default values* for optional command line 
arguments that have been omitted. Lastly, the standard dictates how 
these options can be combined. A general program call that adheres to 
this standard could look like this:

```
program-name -f flag-value --output output-file-name required-argument
```

In this case, the program takes a single required argument that is 
expected to be the last command line argument. Additional arguments are 
optional and are preceded by option flags that have either one dash and 
a one character name (`-f`) or two dashes and a long name (`--output`). 
Both of them are followed by a specific value that goes together with 
that specific option. Most programs will provide both a short and long 
name for the same option, e.g. `-f` and `--file`. Older programs might 
only use short options. Short options can usually be put together in a 
single block with a single leading `-`, although this does depend on the 
program and only works for options that do not take additional 
arguments.

The advantage of this standard should be clear: since most command line 
arguments are now preceded by an option flag, it is much clearer what 
they are. On top of that, the option flags make it possible to rearrange 
the command line arguments, so that the order is no longer important. 
This also makes it possible to omit arguments without affecting the way 
the remaining arguments are parsed.

To parse arguments in C(++), a tool called `getopt` was developed. This 
tool only allows for short option flags; an extension called 
`getopt_long` also deals with long options. The tool is documented on 
[the relevant man page](https://linux.die.net/man/3/getopt_long). I will 
not give any examples here, but simply explain the basic gist.

Before parsing the arguments, you need to specify what arguments you 
expect. For `getopt`, this information is stored in an array. For each 
expected argument, you need to provide a long option name and optionally 
a short option name, and you need to state whether or not it takes an 
additional input argument. You then pass this array on to the 
`getopt_long` function, together with the command line arguments as they 
were received. `getopt` will automatically parse the command line 
arguments, and will allow you to access all of the ones that were found. 
It will also let you know if command line options were missing an 
argument, or if command line options that were not in the list were 
passed on to the program.

Despite its power, `getopt` is still a very basic tool that leaves a lot 
to the programmer. It will for example not perform any of the type 
conversions necessary to convert a command line argument into a floating 
point value, nor will it complain about required options that are 
missing. It also does not provide any functionality to list available 
command line options and default values. For this reason, I decided not 
to use `getopt` in my C++ projects, and instead developed my own 
[`CommandLineParser`](https://github.com/bwvdnbro/CMacIonize/blob/master/src/CommandLineParser.cpp).

In Python, there is a very powerful command line parser, called 
`argparse`, which pretty much does all of the things I mentioned above:

```
import argparse

argparser = argparse.ArgumentParser(
  "Name of the program displayed in help messages")
argparser.add_argument("-n", "--name", action = "store", required = True)
argparser.add_argument("-t", "--true", action = "store_true")
argparser.add_argument("-f", "--float", action = "store", default = 0.1,
                       type = float)
args = argparser.parse_args()

name_value = args.name
true_value = args.true
float_value = args.float
```

`argparse` will automatically do all the necessary type conversions, 
will complain about missing required arguments, will set default values 
for missing optional arguments, and will even provide a full list of 
options when you provide the `--help` option. Note that it also 
automatically accesses `sys.argv` without the need to import `sys` 
explicitly. [The 
documentation](https://docs.python.org/2/library/argparse.html) contains 
many more examples and explains some more advanced features that I have 
personally never used.

# Conclusion

Dealing with command line arguments in C(++) will always be tricky, as 
these languages are simply too low-level to provide a good interface. It 
is therefore very much worthwhile to invest some time developing a good 
command line parser at the start of a large development project.

In Python, dealing with command line arguments is extremely easy thanks 
to `argparse`. To add and parse one command line option, you need only 
four lines of code, and adding more options amounts to one line per 
additional argument. There is hence really no reason not to use 
`argparse`. Furthermore, `argparse` has a lot of options to properly 
document command line options, which make your scripts a lot more 
user-friendly.

In terms of good practice, I believe that most programs/scripts require 
at least two arguments: the name of some file that contains input, and 
the name of a file to store output in. Especially for analysis scripts, 
this is true: such a script usually reads data from a single data file, 
and creates an image based on this. By making sure both the input and 
output are command line arguments, you ensure maximal flexibility in 
using the script on arbitrary input data and with arbitrary output file 
names. I would hence very much advocate in favour of using `argparse` in 
any somewhat important Python script.
