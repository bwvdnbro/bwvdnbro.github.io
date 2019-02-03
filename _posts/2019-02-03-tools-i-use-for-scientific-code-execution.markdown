---
layout: post
title: "Tools I use for scientific code execution"
date: 2019-02-03
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
Ubuntu-systems by default ship with the [`bash` 
shell](https://www.gnu.org/software/bash/), and that is also my personal 
favourite.

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

> Note that Unix-systems are quite specific about executing files and by 
> default set the file permissions so that this is not allowed. You can 
> manually set the permission to execute the script by running
>
> ```
> chmod +x commands.sh
> ```

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

> Note that the `uname` command gives you information about the 
> operating system.

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

Batch systems are very similar to the shells introduced above, except 
that you don't usually use them interactively. Batch systems are 
inherently part of shared computing facilities, where you have to use 
them. A batch system shell script is referred to as a *job script*. You 
need to *submit* this job script to the batch system (using an 
appropriate command) and this will then put your job in a shared *queue* 
that contains all the jobs of all the other people that want to run 
commands on the facility. The batch system will then *schedule* your job 
on (a part of) the machine when there are sufficient resources 
available. Depending on how many jobs are in the queue, this can take a 
while. Most batch systems additionally assign a *priority* to each job 
that is scheduled, with higher priority jobs getting preference over low 
priority jobs. These priorities can depend on many factors, and usually 
funding is one of them (someone has to pay for the facility).

Unfortunately, there are many batch systems, and it is not up to you to 
pick one; the choice of batch system depends on the shared computing 
facility. Most batch systems have very similar features however, and 
they all use an underlying shell to actually execute your job. Knowing 
shell commands hence also pays off in this case. And knowing one batch 
system is usually enough to be able to use another one, if you can get a 
hold of the documentation for the system-specific feature syntax, that 
is.

Figuring out which batch system you are using can be tricky. In my 
experience, small facilities (especially computing nodes owned by 
university departments or research groups) don't bother documenting 
their system very well. In this case, it helps to try locate 
system-specific commands. The table below can be useful:

| commands | likely batch system |
| --- | --- |
| `sbatch`, `sinfo`, `squeue`, `scancel` | [Slurm Workload Manager](https://slurm.schedmd.com/) |
| `qsub`, `qstat`, `qdel` | [Portable Batch System](https://www.pbspro.org/) or [TORQUE](http://www.adaptivecomputing.com/products/torque/) |

If you can find these commands on the login shell of the remote 
facility, you can be quite confident that they are using the 
corresponding batch system.

Independent of the batch system, each system offers commands to submit 
(`sbatch`, `qsub`), monitor (`sinfo` and `squeue`, `qstat`) and cancel 
(`scancel`, `qdel`) jobs.

A batch script (job script) is very similar to a normal shell script, 
except for some batch system directives at the start. Below is an 
example script for the Slurm system:

```
#! /bin/bash
#
#SBATCH --job-name=test
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --output=test.out
#SBATCH --error=test.err

echo "Hello expensive computing facility!"
```

The lines starting with `#SBATCH` contain directives for the Slurm 
system, e.g. the name of the job that will be used in status reports, 
the number of computing nodes to use, the number of threads to use per 
computing node, and the names of the files in which the standard output 
and standard error output will be stored (this is the output you would 
normally see in the command line terminal if you were to execute the 
script in a shell). Most other batch systems have similar directives 
(but with different names). For the PBS and TORQUE systems, directive 
lines start with `#PBS` instead of `#SBATCH`.

Using a batch system can seem a bit intimidating at first, but once you 
realise it is just a somewhat more complicated shell, it is actually not 
that difficult. It is quite normal that things go wrong when you submit 
a script for the first time, and knowing how to interpret error output 
and to get information about how the script was executed can help a lot. 
Just make sure that you test your script properly before submitting 100 
jobs using the same script!

# Performance tools

The tools below are similar to the automation tools above, except that 
they introduce a new introducing feature: parallelisation. All systems 
nowadays (even your laptop) have a number (usually something like 4, 8 
or 16) cores to their disposal that can execute commands simultaneously. 
Unfortunately, you cannot normally use these cores manually from the 
command line; the responsibility for using the parallel environment 
correctly is left to the individual programs you use. If your program 
does not support using multiple threads (a *thread* is a single active 
core, or the equivalent of a single serial program), it will just run on 
a single core, and you will be wasting a large fraction of your 
available resources. The tools discussed below can help you exploit the 
additional available computing power.

## Python multiprocessing

The first tool is not really a tool in itself, but is a useful module 
that is part of Python: the `multiprocessing` module. This module allows 
you to execute a function within Python using multiple, completely 
independent threads, and hence offers you a lightweight way to introduce 
parallelisation into your Python scripts. Since a lot of commands I run 
nowadays are Python scripts, this addresses the problem with serial 
commands at the core: by making the commands themselves use the 
available threads efficiently.

Despite being useful, I recently stopped using this option in favour of 
the solution I will introduce next. The main reason is that parallel 
Python scripts usually have complex dependencies, which makes it harder 
to use them in workflows ([see below](#workflow-management)). On top of 
that, it is quite tricky to interrupt parallel Python scripts; just 
killing the Python script does not properly close all the threads and 
leads to ghost Python processes still running in the background. I'm 
sure parallel processing within Python can be useful for some 
applications, but for the purposes I usually use Python (simulation 
analysis), it is not ideal.

## GNU parallel

[GNU parallel](https://www.gnu.org/software/parallel/) is a more elegant 
solution for the problem that shells do not execute commands in 
parallel; it is a (powerful) program that takes a list of commands and 
then executes them in parallel within a copy of the shell and sends the 
output back to the shell.

A basic example looks like this

```
> parallel -j 4 echo ::: a b c d
a
c
d
b
```

The program takes the `echo` command, and then runs it with the four 
different arguments provided in the list that starts with `:::`. The `-j 
4` argument tells `parallel` to use 4 parallel threads to execute this 
command. The output will not necessarily be in the same order as the 
order of the list, as it depends on how fast the corresponding thread is 
at executing the `echo` command.

`parallel` has a lot more ways of specifying a list of commands, and I 
very much suggest that you consult its [extensive 
tutorial](https://www.gnu.org/software/parallel/parallel_tutorial.html) 
to find out more. I usually use it as a parallel alternative for the 
shell loops introduced above:

```
> ls file??? | parallel -j 8 compute_average_density
Done.
Done.
...
```

In this case, the list generated by the `ls` command is *piped* directly 
into the `parallel` command and used as arguments for the 
`compute_average_density` program.

For more complicated commands that require multiple arguments, you can 
use argument substitution:

```
> ls file??? | parallel -j 8 plot_density --input {} --output {}.png
Plotting file001 as file001.png...
Plotting file004 as file004.png...
...
```

In this case, each `{}` in the command is replaced by the respective 
element in the list. An individual command issued by `parallel` will 
look like

```
plot_density --input file001 --output file001.png
```

There are many variants of the `{}` directive that allow you to apply 
filters to the list elements, e.g. filtering out file extensions, folder 
names...

To get a parallel equivalent for a loop that has a numerical counter, you
can use the `seq` command:

```
> seq 0 3 | parallel -j 4 echo {}
0
2
1
3
```

To get the equivalent of a fixed width counter (`{00..03}`), you can use 
`seq -w`:

```
> seq -w 00 03 | parallel -j 4 echo {}
00
03
02
01
```

> Note that if you use `parallel` for the first time, it will ask you to 
> promise that you will cite the underlying publication if you use it 
> for your work:
>
> ```
> Academic tradition requires you to cite works you base your article on.
> When using programs that use GNU Parallel to process data for publication
> please cite:
>
>   O. Tange (2011): GNU Parallel - The Command-Line Power Tool,
>   ;login: The USENIX Magazine, February 2011:42-47.
>
> This helps funding further development; AND IT WON'T COST YOU A CENT.
> If you pay 10000 EUR you should feel free to use GNU Parallel without citing.
> ```
>
> Once you do this, it will work as illustrated above.

# Workflow management

The tools above allow you to automate some of the tasks you want to do 
during code execution, and to make more efficient use of the available 
resources on your computer. They still require a lot of manual 
intervention however, and still limit you to using a single computer.

Workflow Management Systems (or WMSs) offer a more complete solution to 
automating and optimising your tasks. They do however require an 
additional step of abstraction, which we call a *workflow*.

I already briefly mentioned workflows in [a previous post]({% post_url 
2019-01-27-the-five-steps-of-an-astrophysical-simulation-project %}). In 
essence, a workflow is a structured representation of the commands you 
need to run in order to execute a task that details the *dependencies* 
between these commands: required input and output files, and software 
and hardware requirements. This representation is usually visualised as 
a flowchart, and allows you to see the order in which commands need to 
be run, and which files need to be present in order for a command to 
work. It also shows you which parts of your task get invalidated if a 
file changes, i.e. they make it possible to easily figure out which 
commands you need to rerun if a small part of your input files changed.

Workflows are incredibly powerful, and I should probably discuss them in 
more detail in a future post. For now, I will limit myself to the 
software that uses workflows: WMSs. In short, these WMSs take your 
workflow and execute it for you, *using all available resources, 
including remote shared computing facilities, and in the most optimal 
way*. This is obviously a quite complicated thing to do; and much 
depends on which WMS you use and how your computer and other facilities 
are configured. I am still very much new to these tools myself. Below is 
what I know so far.

## Makeflow

[Makeflow](http://ccl.cse.nd.edu/software/makeflow/) is probably one of 
the most basic WMSs, and is also the only system I have thus far managed 
to successfully use myself. The tool itself is very similar to [GNU 
make](https://www.gnu.org/software/make/) (hence its name), and is quite 
easy to manually install (which is usually the only way to make it work 
consistently on a number of different machines). It consists of the WMS 
itself, `makeflow`, and an additional batch system, `workqueue`, that is 
responsible for executing `makeflow` jobs efficiently (similar to a 
traditional batch system). `makeflow` can also be used in conjunction 
with existing batch systems, like the ones discussed 
[above](#batch-systems), and with a standard shell.

I will explain the usage of `makeflow` in more detail in a future post 
and limit myself to a very basic example. Suppose we have a single input 
file, `input.txt`, and we want to split this file into two parts using 
`split`. We then want to store the number of lines in each file in a 
file (using `wc`), and then join these two files back into a final 
output file `output.txt` (using `cat`). A corresponding Makeflow file, 
`makeflow.makeflow` looks like this:

```
input00 input01: input.txt
  split -n 2 -d input.txt input

input00count: input00
  wc input00 > input00count

input01count: input01
  wc input01 > input01count

output.txt: input00count input01count
  cat input00count input01count > output.txt
```

Despite its absurdity, this example shows you how Makeflow works: each 
block in this file has the general structure

```
OUTPUT: INPUT
  COMMAND
```

(note that the `COMMAND` needs to be preceded by a `tab`). The `OUTPUT` 
should contain a list of all output that the command generates that you 
will use in other commands or want at the end of the task (output that 
is not listed will be deleted by Makeflow as soon as the command 
finishes). `INPUT` should specify all the input needed for the command 
(if this input is not present in the file system, Makeflow will look for 
a block that has the corresponding file as output and run that block 
first). The `COMMAND` specifies the actual command to run. Makeflow will 
complain if this command does not generate all of the expected output.

To run this flow using Makeflow, we need to do two things. First, we 
need to start the WMS, using the `makeflow` command:

```
> makeflow -T wq makeflow.makeflow -p 9000
```

The `-T wq` argument specifies the batch system to use to submit jobs; 
`wq` stands for WorkQueue, Makeflow's own batch system. `-p 9000` 
specifies the network port that Makeflow will use to communicate with 
WorkQueue. If you omit this argument, Makeflow will choose a random port 
and tell you its number. You need this number for the second step.

The second step is launching the batch system itself. This can be done 
using the `work_queue_worker` command (from a new command line 
terminal):

```
> work_queue_worker --cores 8 --memory 1000 --disk 1000 localhost 9000
```

This will launch a batch system with 8 threads (`--cores 8`). This 
system will use a maximum of 1000 MB of RAM memory (`--memory 1000`) and 
1000 MB of hard drive space (`--disk 1000`), and will try to contact 
Makeflow on the `localhost` using network port `9000`. Note that the 
system parameters do not have to match those that are actually 
available, but that WorkQueue will fail to launch if you use values that 
are larger than what is available. If WorkQueue has multiple threads 
available, it will use all of these to execute Makeflow jobs. Makeflow 
allows you to specify the computational requirements for a job in the 
Makeflow file.

If all went well, the Makeflow and WorkQueue processes will now start 
communicating with each other, and will execute the entire workflow. If 
something goes wrong, Makeflow will tell you about this. In this case, 
you will be able to restart the workflow where you left it.

You can also run the WorkQueue command on a remote cluster and still 
make it communicate with your local Makeflow command. This however 
requires appropriate network settings and is not so easy to set up. 
Makeflow has additional features that make it possible to execute a 
workflow using multiple WorkQueue systems on multiple clusters. This is 
just a very brief introduction to Makeflow, so I will tell you more 
about this in a future post.

## Other WMSs

Makeflow is quite limited in terms of what it can do and the amount of 
diagnostic output it generates. This makes it less suitable for really 
large projects. There are many other WMSs that are much more powerful, 
but unfortunately, I have not been able to successfully use any of 
these. Here is a list of some of these:
 * [Pegasus WMS](https://pegasus.isi.edu/)
 * [Kepler](https://kepler-project.org/)
 * [Galaxy](https://galaxyproject.org/)

I would also recommend the [Blue Waters webinar series on Scientific 
Workflow Management 
Systems](https://bluewaters.ncsa.illinois.edu/webinars/workflows).
