---
layout: post
title: "Tools I use for scientific code execution"
description: An overview of the tools I use during scientific code execution
image: /assets/images/tools.jpg
author: Bert Vandenbroucke
tags: 
  - Scientific computing
  - Tools
  - Workflow management
---

In a [previous post]({% post_url 
2019-01-06-tools-i-use-for-scientific-code-development %}), I already 
covered the main tools I use during the scientific code development 
process. These tools were all closely related to writing, testing and 
maintaining code. In this post, I will cover tools that help with 
running code efficiently. These are tools that allow you to run whatever 
you want to run more efficiently, faster and more securely, and that 
also allow you to rerun things in a more effective way if required.

# Automation tools

Automation tools are all about efficiency: they allow you to decrease 
the number of commands you need to manually enter or execute, and hence 
allow you to spend your time on other things, especially if you have a 
long list of commands that can be automated. Additionally, automation 
tools are inherently part of large shared computing facilities where 
jobs cannot be executed in real time but have to be scheduled in a job 
queue.

## bash loops and scripts

A *shell* is the interactive program that runs in the background when 
you run a command-line terminal and that interprets and executes your 
commands. There are various shells you can choose from, but most of them 
offer basic *scripting* functionality, i.e. they have support for 
control structures like loops and conditions that allow you to write 
simple *programs* (called *scripts*) that are executed by the shell. 
Ubuntu-systems by default ship with the `bash` shell, and that is also 
my personal favourite.

A first level of automation can already be achieved by just using these 
control structures as actual commands. Suppose for example that you have 
a list of 500 files (`file001`, `file002`, etc.) and you want to run a 
program (or script) `compute_average_density` on each of these. Instead 
of manually calling the command on each of these files:

```
> compute_average_density --file file001
Done.
> compute_average_density --file file002
Done.
...
```

you could automate this process using a `for` loop:

```
> for file in file???
> do
> compute_average_density --file $file
> done
Done.
Done.
...
```

In this case, you only need to enter a single command, and `bash` will 
automatically call the `compute_average_density` command 500 times.

Note that the `bash` history (you can scroll through this history using 
the `up` and `down` arrows on your keyboard and search through it using 
`CTRL+R`) will store the entire loop as a single line, like this:

```
> for file in file???; do compute_average_density --file $file; done
```

You can always use this syntax as well if you like it better.

The basic syntax for the `for` command allows you to specify a 
space-separated list of elements over which to iterate:

```
> for i in 0 2 4; do echo $i; done
0
2
4
```

But you can also list files (and folders) in the file system using 
*wildcards* that are converted into such a list by `bash` before the 
`for` command is actually called:

```
> echo file???
file001 file002 ...
> echo file*
file0001 file001 file002 ...
> echo file??[!2]
file001 file003 ...
```

Where I have illustrated some of the wildcards that are available:

| wildcard | meaning |
| --- | --- |
| `?` | Replace by exactly one allowed character |
| `*` | Replace by any number of allowed characters |
| `[!...]` | Replace by exactly one allowed character that is not the character(s) listed in between brackets |


Additionally, you can generate ranges of (padded) values:

```
> echo {0..10}
0 1 2 3 4 5 6 7 8 9 10
> echo {00..10}
00 01 02 03 04 05 06 07 08 09 10
```

It can be quite useful to know some of these commands; learning them is 
a bit tedious at first, but you quickly get used to them and they 
facilitate basic tasks enormously when used correctly.

`bash` scripts (or more generally *shell* scripts) allow you to save the 
commands you would usually manually enter into a file (e.g. 
`commands.sh`). This file can then be executed by running

```
> bash commands.sh
```

This runs the commands in a new, isolated shell. You can also run the 
commands in the active shell:

```
> source commands.sh
```

The difference between both is quite technical. The first version will 
not allow changes to the shell *environment* made by the script to 
persist after the script finishes (e.g. setting an *environment 
variable* inside your script). Usually you don't care about that.

A final way to run a bash script is by converting it into an executable. 
This can be done by adding the following line to the *top* of the 
script:

```
#! /bin/bash
```

This tells the operating system to automatically execute the script with 
the `bash` program. You can then simply invoke the script as any other 
program:

```
./commands.sh
```

Note that Unix-systems are quite specific about executing files and by 
default set the file permissions so that this is not allowed. You can 
manually set the permission to execute the script by running

```
chmod +x commands.sh
```

The advantage of using scripts is *reproducibility*: if you often rerun 
the same set of commands, then having them stored saves you the effort 
of re-entering them every time. And you easily rerun the same set of 
commands for different sets of files.

Shell scripts are really like mini-programs, as they also allow you to 
use conditions:

```
> for i in {0..10}
> do
> if [ $i -lt 4 ]
> then
> echo $i
> fi
> done
0
1
2
3
```

The `bash` comparison operators themselves are unfortunately quite 
archaic. Here are some of them:

| comparison operator | meaning |
| --- | --- |
| `-eq` | Equal to |
| `-neq` | Not equal to |
| `-lt` | Less than |
| `-gt` | Greater than |
| `-le` | Less than or equal to |
| `-ge` | Greater than or equal to|

Also note the *spaces* in between the brackets of the conditional 
statement, as they are important.

There are additional commands like e.g. `-f` that can check if a file 
exists:

```
> if [ -f file001 ]
> then
> echo "File exists!"
> else
> echo "File does not exist!"
> fi
```

And of course, there is also a *not* operator:

```
> if [ ! -f file001]
> then
> echo "File does not exist!"
> fi
```

Inside a script, you can define variables:

```
> message="hello"
> echo $message
hello
```

And these variables can also be the result of another command if you 
enclose that command in `$()`:

```
> message=$(uname -o)
> echo $message
GNU/Linux
```

Note that the `uname` command gives you information about the operating 
system.

Lastly, `bash` scripts automatically receive additional command line 
arguments in special variables. The following script:

```
#! /bin/bash

echo "Number of command line argument: $#"
echo "First argument: $0"
echo "Third argument: $2"
```

illustrates this:

```
> ./script.sh these are words
Number of command line argument: 3
First argument: ./script.sh
Third argument: are
```

The `$0` argument contains the name of the script.

The overview above is just the tip of the iceberg that is shell 
programming; there are a lot of good online tutorials that can teach you 
more.

## batch systems

Batch systems are very similar to the shells introduced above, 

# Performance tools

## Python multiprocessing

## GNU parallel

# Workflow management

## Makeflow

## Other WMSs
