---
layout: post
title: "Unit testing with CTest"
description: Setting up simple unit tests using CTest.
date: 2019-12-12
author: Bert Vandenbroucke
tags:
  - Code development
---

Let's talk about code testing. Testing is a very important aspect of 
developing a code: you wrote a function or class with a specific aim and 
now you need to make sure that this component of your program actually 
does what it should do. There is an important saying in (computer) 
science that data that is not backed up does not exist. Similarly, I 
would say that code that has not been tested does not work.

Apart from some anecdotes (like a student that spent an entire semester 
writing a bit of Python code and the week before his thesis deadline 
said: "Okay, I'm done. Now, how do I run Python?"), most people are 
acutely aware of the importance of testing. Any code development effort 
is usually broken down into a lot of small chunks of development, 
followed by a reasonable amount of testing. So that's all good.

Unfortunately, scientific code developers tend to be easily satisfied. I 
have seen a lot of examples of code where testing was definitely done at 
some point during the development (obvious because sometimes the code 
used for the test is still there - commented out). But once the testing 
phase was successfully completed, or - in other words - once the 
developer had convinced herself that the code was working, the test was 
simply removed or commented out.

That seems like a normal thing to do. You just tested that everything 
works, so why bother to ever test it again? Simple: because code 
changes. Granted, there are some very basic functions that do very 
simple things that can probably stay the way they are forever (there are 
only so many ways to implement a simple polynomial expression, so a 
function that simply returns that is probably safe). But most code is 
not as simple, and involves functionality from different other parts of 
the code that could all at some point change. Or code can be written in 
a naive and slightly inefficient way and then turn out to be the 
bottleneck for your program, so that it becomes important to rewrite it 
in a more efficient way... There are plenty of cases where you want to 
change existing code after it has been written, and in principle that 
means rerunning the tests you did before.

If you simply commented out your tests, then you can simply uncomment 
them and hope they still work. If you got rid of them all together, you 
need to rewrite them, which is more annoying. It becomes really 
problematic if you forgot that you need to run a test. If a function 
calls another function that calls another function that calls another 
function (and so on) and the last function in this series was changed, 
then it can become very hard to keep track of which tests could be 
potentially invalidated by the change you just made. Depending on what 
your function does, it can nonetheless still be necessary to rerun the 
tests for all functions in this series, as it can sometimes be very hard 
to predict all use cases of a function for a simple test of that 
function alone.

What I am really trying to say here is that it is generally a bad idea 
to get rid of code tests, since you never know when you might need them 
again. And it is generally also not such a good idea to hand pick the 
tests you want to run after you made a change to existing code, as that 
change could sometimes propagate to other parts of the code in 
unexpected ways. It is a good idea to keep tests and rerun them 
regularly, even if you don't think their results should be any 
different, just to convince yourself that the results are really not any 
different.

This is sort of what *unit testing* tries to achieve. The idea of unit 
testing is to consciously develop dedicated tests for *all* components 
of a program, and then keep them in place and run them every time 
something changes to the program. For this to work, the program has to 
be broken down into small components with a clear responsibility; they 
need to act as a *unit*. Small units can be combined into bigger units, 
and by testing each of these units for a range of input values against 
an expected range of output values, you can hope to cover the full 
complexity of the underlying code and test that your code really does 
what it should do.

While this principle sounds really nice, it is also incredibly hard. 
First of all, it is as good as impossible to cover the full complexity 
of an algorithm with a finite number of tests; large software projects 
can be very happy if they manage to cover more than 50% of their code 
base with unit tests. This is not necessarily a bad thing: some code is 
so trivial that it probably does not require testing (like error 
handling code, status output...) and other code is so rarely executed 
that it maybe never gets called in production runs anyway, but is just 
there because someone thought it might be necessary. If your unit tests 
test your code for a large enough range of sensible input values, then 
you can hope that they will still capture any bugs that might affect 
normal code users, and that is what really matters.

Second, writing and maintaining unit tests can seem very tedious at 
first. Because that is what you need to do if you want to use unit 
testing: you have to write additional code (small programs actually) 
that get executed and call a specific part of your code. While some ways 
of testing can be automated by using external libraries, you cannot get 
around actually writing a basic program that at least calls the 
functions in that library, and you will definitely need to provide all 
input values and expected output values for your test. Because you are 
the person that knows what your code is supposed to do.

From personal experience, I can say that having unit tests, however 
tedious they might be, does have significant long term benefits. I 
cannot count the number of bugs that I caught in a very early stage 
because they triggered an unexpected fail of one of my unit tests. Bugs 
that otherwise would have gone unnoticed until they caused some major 
issue somewhere else, by which time it would become very hard to find 
them. On top of that, the overhead of unit tests is not that big if you 
really think about it. Normal code needs to be tested too (because 
untested code does not work, see above), so you would need to spend time 
writing some kind of test anyway. The only overhead of the unit test is 
that you need to write it in a separate program, so you cannot just add 
some extra lines to the function you are testing. I would argue that 
this is actually a good think, because it forces you to think more about 
code structure and code inter-dependencies. But that is my opinion.

In the last couple of years I have developed a personal code development 
style that starts from the unit tests. Whenever I write a new component 
for a program, I start by writing the unit test program that calls it, 
and use that to test the basic functionality of the component before 
adding it to the main program. I find that this actually works very 
well: it helps me to think about the design of my components better, 
gives me a clear path to implement the components, and in the end gives 
me a unit test that will keep testing that component in the future.

All that being said, unit testing does require a good framework to 
implement, compile and run tests in an efficient and hopefully automated 
way. One of these frameworks, the CTest framework that ships with the 
configuration tool CMake, is the subject of this post. I spent an awful 
lot of paragraphs on this introduction, so I will try to keep the body 
of this post short.

# What is CTest?

CTest is an incredibly powerful tool that automates the task of running 
unit tests, but there are a lot of things it cannot do. So let me start 
by mentioning those, to avoid any future disappointments. First of all, 
CTest does not tell you how to implement unit tests. While I like to 
write my unit tests in the same language as my program and make them 
include the relevant parts of my code by including header files and 
calling functions, CTest does not make any assumptions about what your 
unit test is. For CTest, a unit test is anything that it can execute: an 
executable or a script that it can call and that will generate some 
output and will exit with an *exit code*. This exit code is all CTest 
cares about: if the exit code is 0, the test passed and CTest is happy. 
If it is anything else, then CTest will consider your test to have 
failed and will report this.

Second, since CTest does not know what your executable is, it will not 
help you in any way to create your executable. That is something that 
annoyed me when I first starting using CTest, but ultimately makes 
sense: to configure the compilation process for an executable, you can 
use the CMake framework itself. CTest is only concerned with the actual 
execution of the unit tests.

What CTest will do for you is simple but very important: it will keep 
track of all things that you list as tests during your CMake 
configuration, and then create a target for these in the Makefile that 
CMake generates. This target by default is called *test*. When you call 
`make test` from the directory where the CMake generated Makefile is 
located, CTest will automatically execute all unit tests, and report on 
their behaviour. It will automatically time all unit tests, and can be 
configured to run the tests in parallel. At the end, it will give you an 
overview of all tests that ran, passed and failed, and produces log 
files that contain the output of all tests and the output of all tests 
that failed.

The power of this cannot be understated: there is no need for you to do 
anything to keep track of unit tests. As soon as you configure a unit 
test, it will be there and it will be executed, unless you explicitly 
tell CTest not to execute it.

# How do I use CTest?

Simple: you first tell CMake to enable CTest by adding the following 
line somewhere near the top of you main `CMakeLists.txt` (usually right 
after the required `cmake_minimum_required()` and `project()` calls:

```
enable_testing()
```

Next, you can add tests anywhere else in you `CMakeLists.txt` or 
configuration files in subfolders that set up tests, like this:

```
add_test(NAME <name label>
         COMMAND <command that needs to be executed>
         WORKING_DIRECTORY <folder where the command needs to be executed>)
```

(there is an additional argument `CONFIGURATIONS` that I have never used 
and don't really understand myself). The option values in between `<>` 
obviously need to be set to sensible values (the label can be anything, 
it is simply what CTest displays in its reports). For the working 
directory, it is usually a good idea to use a subdirectory of 
`${PROJECT_BINARY_DIR}`. That's it!

# What does CTest output look like?

This will of course depend on your specific project. But to give you an 
idea, here is an example of a project I am currently working on:

```
> make test
Running tests...
Test project <PATH CENSORED>
    Start 1: testSpecialFunctions
1/9 Test #1: testSpecialFunctions .................   Passed    0.07 sec
    Start 2: testMatrix
2/9 Test #2: testMatrix ...........................   Passed    0.01 sec
    Start 3: testTMatrixCalculator
3/9 Test #3: testTMatrixCalculator ................   Passed   21.97 sec
    Start 4: testOrientationDistribution
4/9 Test #4: testOrientationDistribution ..........   Passed    0.05 sec
    Start 5: testTable
5/9 Test #5: testTable ............................   Passed    0.00 sec
    Start 6: testDraineDustProperties
6/9 Test #6: testDraineDustProperties .............   Passed    0.02 sec
    Start 7: testQuickSched
7/9 Test #7: testQuickSched .......................   Passed   25.72 sec
    Start 8: testTaskBasedTMatrixCalculation
8/9 Test #8: testTaskBasedTMatrixCalculation ......   Passed    6.50 sec
    Start 9: testDraineHensleyShapeDistribution
9/9 Test #9: testDraineHensleyShapeDistribution ...   Passed    0.00 sec

100% tests passed, 0 tests failed out of 9

Total Test time (real) =  54.36 sec
```

Apart from this console output, CTest will also produce 3 log files in a 
folder called `test/Testing/Temporary` located somewhere in the build 
directory:
 - `CTestCostData.txt`: this summarises the timing information and 
contains the time in seconds each test took to run.
 - `LastTest.log`: this contains the full terminal output of all tests that
were run the last time CTest was executed.
 - `LastTestsFailed.log`: this contains the names of the tests that 
failed, i.e. the last tests that failed the last time CTest was called 
and not all of its tests passed (so this file will still contain 
something, even if your tests currently all pass).

# How do I tweak CTest?

There are a few command line options for the CTest command that can 
tweak the way it behaves. For this to work, you need to call the CTest 
executable directly, using `ctest` instead of `make test` (both commands 
behave the same). The options I use are:
 - `--parallel <NUMBER OF THREADS>`: This runs multiple tests at the 
same time. CTest will use the timing information in `CTestCostData.txt` 
to determine an optimal order to schedule the tests in this case; they 
will not necessarily run in a fixed or predictable order.
 - `--output-on-failure`: This forces CTest to dump the terminal output 
of all tests that failed straight into the terminal instead of only 
putting it in the log files. This is useful if you want to quickly debug 
your failing tests, e.g. from within an IDE.
 - `-R <TEST NAME>`: This only runs the test with the given name. I 
actually use this command to set up custom `make` targets for individual 
tests (see below).

# How do I automatically compile unit tests?

As I mentioned before, CTest does not help you in any way to set up your 
unit test executables; it simply assumes that the tests you want the run 
exist and are up to date. If this is not the case, e.g. because your unit 
test is itself a small program that needs to be compiled after your code 
has changed, then you need to manually set up the CMake code that 
generates this executable.

Since this is the basic way I use unit tests, I have written a 
convenient CMake macro that automatically sets up unit tests. I'll 
simply give it here (this comes straight from [the CMacIonize 
code](https://github.com/bwvdnbro/CMacIonize/blob/master/test/CMakeLists.txt)):

```
# Add a new unit test
# A new target with the test sources is constructed, and a CTest test with the
# same name is created. The new test is also added to the global list of test
# contained in the check target
macro(add_unit_test)
    set(options PARALLEL)
    set(oneValueArgs NAME)
    set(multiValueArgs SOURCES LIBS)
    cmake_parse_arguments(TEST "${options}" "${oneValueArgs}"
                               "${multiValueArgs}" ${ARGN})
    message(STATUS "generating ${TEST_NAME}")
    add_executable(${TEST_NAME} EXCLUDE_FROM_ALL ${TEST_SOURCES})
    set_target_properties(${TEST_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY
                          ${PROJECT_BINARY_DIR}/rundir/test)
    target_link_libraries(${TEST_NAME} ${TEST_LIBS})

    if(TEST_PARALLEL AND HAVE_MPI)
      set(TESTCOMMAND ${MPIEXEC})
      set(TESTARGS ${MPIEXEC_NUMPROC_FLAG} 3 ${MPIEXEC_PREFLAGS}
                   "./${TEST_NAME}" ${MPIEXEC_POSTFLAGS})
      set(TESTCOMMAND ${TESTCOMMAND} ${TESTARGS})
    else(TEST_PARALLEL AND HAVE_MPI)
      set(TESTCOMMAND ${TEST_NAME})
    endif(TEST_PARALLEL AND HAVE_MPI)
    add_test(NAME ${TEST_NAME}
             WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/rundir/test
             COMMAND ${TESTCOMMAND})

    set(TESTNAMES ${TESTNAMES} ${TEST_NAME})
endmacro(add_unit_test)
```

This macro is quite heavy because of all the additional features I use, 
but the main idea is that it generates a CMake target that compiles the 
unit test executable, links in all relevant libraries, and then 
generates a command that calls the executable and registers it as a unit 
test using `add_test()`. Unfortunately, this does not automatically 
compile the unit test before it is executed. To do this, the name of the 
test created by the macro is added to a list (`TESTNAMES`). We need to 
manually make the CTest target depend on this list, by adding the 
following line to the `CMakeLists.txt`, *after all unit tests have been 
created*:

```
add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND}
                        DEPENDS ${TESTNAMES})
```

This will create a new target, `make check`, that first compiles all 
unit tests, and then executes them.

I later added the following call to the end of the `add_unit_test` 
macro:

```
add_custom_target(check_${TEST_NAME}
                  COMMAND ${CMAKE_CTEST_COMMAND}
                          -R ${TEST_NAME}
                  DEPENDS ${TEST_NAME})
```

This will set up an individual `make check_<test name>` target for each 
test that only compiles and executes that test.

With this macro, it now becomes very easy to add a unit test: you simply 
call the macro with the desired test name and source files, and CMake 
will generate all the relevant targets in the Makefile for you. You 
simply need to add `make check` to your default compilation command, and 
tests will get compiled and run every time you make a change to the 
code.
