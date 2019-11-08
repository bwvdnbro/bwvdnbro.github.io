---
layout: post
title: "Configuring projects using CMake"
description: An introduction to CMake for astrophysicists
date: 2019-11-08
author: Bert Vandenbroucke
tags:
  - Code development
  - Tools
---

I have mentioned the use of [CMake](https://cmake.org/) in the past, but 
have never really explained how to use it. However, I have recently 
noticed that such an introduction would really helpful, since the lack 
of proper configuration tools in my own scientific field (computational 
astrophysics) is just painful.

As a case study, I will take the public moving-mesh code 
[Arepo](https://arepo-code.org/), available from 
<https://gitlab.mpcdf.mpg.de/vrs/arepo>. This code has only recently 
been made public and is incredibly popular in computational 
astrophysics, so I think it is fair to say that it is very 
representative for the field nowadays.

If you read [the 
documentation](https://arepo-code.org/wp-content/userguide/running.html), 
you will notice that the entire compilation of the code consists of one 
step: `make`. This is weird, as the code depends on at least two 
required libraries (MPI and GSL) that can be installed in very different 
ways depending on your system. It seems very unlikely that the person 
that wrote the `Makefile` required for GNU Make is aware of all possible 
locations of these libraries on all possible systems. This is obviously 
not the case. As a "solution", the developers have provided an 
additional file, `Makefile.systype` that hardcodes the library paths and 
compiler options for different systems. Good luck figuring out these 
things on your system!

A second weird thing that immediately strikes you is that the code 
depends on a large number of parameters in an additional file, 
`Config.sh` that are then parsed by a Perl script and written to a 
number of additional source code files that are placed in the *build* 
directory for the code. This script is somehow run as part of the 
compilation process.

All of these things can be done just as well (and a lot more easy) by 
using CMake. So here a very brief example of how to do this for this 
particular code.

# The basics: source code files and target executable

To use CMake, you need to create a CMake configuration file, called 
`CMakeLists.txt`. For Arepo, we can create the following skeleton that 
simply creates a *target* executable for all the Arepo source code 
files:

```
cmake_minimum_required(VERSION 3.0)

project(arepo C)

file(GLOB SOURCES_LEVEL2 "${PROJECT_SOURCE_DIR}/src/*/*.c")
file(GLOB SOURCES_LEVEL3 "${PROJECT_SOURCE_DIR}/src/*/*/*.c")

add_executable(Arepo ${SOURCES_LEVEL2} ${SOURCES_LEVEL3})
```

The first line, containing `cmake_minimum_required`, tells CMake which 
version of CMake you expect to find on the system. If the version of 
CMake that your system uses is older than this, then CMake will display 
an error and stop the configuration. This is useful if you are using 
features of CMake that only became available from a certain version. 
Unfortunately, there is no default value, so we need to choose something 
here. CMake 3.0 is reasonably well supported on modern systems, so that 
is why I choose it here (older versions could also work).

Next, we define a name for the CMake project, `arepo`. This line is very 
important, as it will instruct CMake to start looking for compilers on 
the system. The optional additional argument `C` tells CMake we have a 
pure C project; by default CMake will look for both C and C++ compilers.

On the following two lines, we find all the source code files. For the 
specific case of Arepo, those are located in subfolders of `src/`. Some 
of these subfolders in their turn have subfolders, so that in total we 
have two levels of source folders. The `file(GLOB` command will 
interpret the regular path expressions `src/*/*.c` and `src/*/*/*.c` and 
put their results in the CMake *variables* `SOURCES_LEVEL2` and 
`SOURCES_LEVEL3`. The `src` folder is located in the same folder as our 
`CMakeLists.txt`, and in CMake we can refer to this folder as 
`PROJECT_SOURCE_DIR`. The `${}` syntax expands this variable and hence 
displays this path. Note that CMake variables are also expanded when 
they are part of a string.

The last line in this skeleton creates the actual executable, named 
`Arepo`, and tells CMake on what source files it depends. It will 
automatically generate a Makefile that contains rules to compile all of 
these source files, and a rule to link them together into the `Arepo` 
executable.

To try out this skeleton file, we first need to create a `build` folder. 
This could be anywhere, but I will assume it is another subfolder of the 
folder that contains the `CMakeLists.txt`. Once created, enter the build 
folder and run CMake, using the path to the folder that contains the 
`CMakeLists.txt` as additional argument:

```
cmake ..
```

This should work and will create a `Makefile` that you can use to 
compile the program. However, if you try to actually run `make`, you 
will get a lot of compilation errors. This makes sense, since we did not 
tell CMake about any of the library dependencies yet.

# Adding library dependencies

To add library dependencies, CMake provides the convenient macro 
`find_package`. Typical usage is illustrated using the required GSL 
library:

```
find_package(GSL REQUIRED)
include_directories(${GSL_INCLUDE_DIRS})

target_link_libraries(Arepo ${GSL_LIBRARIES})
```

The last line needs to be added *after* the `add_executable` line, while 
the other two can be put somewhere at the top, e.g. right after the 
`project` command.

In a nutshell, `find_package` takes the name of the library you want to 
find (`GSL`) and optional version numbers and subcomponents. The 
`REQUIRED` keyword tells CMake this library is required; if not found, 
CMake will display an error and abort the configuration. 

The `find_package` macro will then use another file that is part of the 
CMake installation, `findGSL.cmake`, to find the library and set a 
number of CMake variables: `GSL_FOUND` is set to `True` or `False` (this 
is useful for *optional* libraries), while `GSL_INCLUDE_DIRS` and 
`GSL_LIBRARIES` contain respectively the path to the folder containing 
the GSL header files and the path to the GSL library itself. By adding 
the former to the `include_directories` we make sure the header files 
can be included in the program source files. The latter makes sure that 
the *linker* (see an older post) can include the library machine code in 
the executable.

While recent versions of CMake ship with a lot of `findLIBRARY.cmake` 
files, there is no guarantee that such a file will exist for all 
libraries, especially exotic ones. `find_package` can alternatively look 
for a custom `findLIBRARY.cmake` file in your project source folder, but 
then you need to write this yourself (or find one on the internet, which 
is usually quite easy). Note that while most of these will set variables 
with standard names, this is not guaranteed. This is e.g. the case for 
the MPI library. You can find full documentation for all the `.cmake` 
files that are part of CMake online.

The MPI library has different flavours depending on the language of your 
code. As a result, `find_package(MPI REQUIRED)` will look for all of 
these, and then set slightly different variables to deal with different 
languages. This means that the header file directory in our case is 
called `MPI_C_INCLUDE_PATH` and that the library itself is contained in 
`MPI_C_LIBRARIES`. Since the C++ or Fortran libraries could be present 
without the C library (unlikely, but still), we need to check if the C 
library was actually found, and deal with the case where it was not:

```
find_package(MPI REQUIRED)
if(MPI_C_FOUND)
  include_directories(${MPI_C_INCLUDE_PATH})
else(MPI_C_FOUND)
  message(FATAL_ERROR "No MPI C compiler found!")
endif(MPI_C_FOUND)
```

We now explicitly check if the library was found, and only include it if 
it was. The `message(FATAL_ERROR` call will cause CMake to exit with the 
given error message. `message` can also be used with `STATUS` and 
`WARNING` for respectively normal standard output and colourful warning 
output. Only `FATAL_ERROR` will cause CMake to abort.

If you add both GSL and MPI to the `CMakeLists.txt`, the configuration 
and compilation should progress somewhat further, until one of the 
source code files tries to include the configuration header, 
`arepoconfig.h`, that is created by the Perl script. Let's see how we 
can add this.

# Executing additional commands as part of the configuration process

To run the Perl script, we first need to find the Perl executable. This 
is easy:

```
find_package(Perl REQUIRED)
```

This will set the variable `PERL_EXECUTABLE`. We can call the Perl 
script using the `execute_process` macro:

```
execute_process(COMMAND ${PERL_EXECUTABLE}
                ${PROJECT_SOURCE_DIR}/prepare-config.perl
                ${PROJECT_SOURCE_DIR}/Config.sh
                ${PROJECT_BINARY_DIR})
```

This simply runs the Perl executable and passes the three additional 
parameters as command line arguments: the name of the Perl script, the 
name of the configuration file (located in the source folder) and the 
build folder, which we can access here as the `PROJECT_BINARY_DIR` 
variable (CMake sets this to the folder from which the `cmake` command 
is run).

The Perl script will create the additional header file `arepoconfig.h` 
and will put it in the `PROJECT_BINARY_DIR`. We need to make sure this 
folder is included:

```
include_directories(${PROJECT_BINARY_DIR})
```

The script also generates two additional source code files, 
`compile_time_info.c` and `compile_time_info_hdf5.c`. We need to add 
these to the executable:

```
set(EXTRA_SOURCES ${PROJECT_BINARY_DIR}/compile_time_info.c
                  ${PROJECT_BINARY_DIR}/compile_time_info_hdf5.c)

add_executable(Arepo ${SOURCES_LEVEL2} ${SOURCES_LEVEL3} ${EXTRA_SOURCES})
```

The last line replaces the `add_executable` we had before.

If you now run `cmake` again, the Perl script will be executed as one of 
the steps, and it will create the additional source code files as 
expected. `make` will now successfully compile all source code files, 
but you should still get a linker error in the end.

# System libraries

In the case of Arepo, the linker errors are caused by missing library 
dependencies for `libm` (the standard maths library) and `libgmp` (the 
GNU big number library). Unfortunately, there are no `findLIBRARY.cmake` 
files for these libraries. If you are on a standard Ubuntu system (like 
I am), you can easily link them by using

```
target_link_libraries(Arepo m)
target_link_libraries(Arepo gmp)
```

If this does not work, then you will need to find the appropriate 
`.cmake` files online and use the normal library inclusion approach. It 
is unlikely you will need to do this for `libm`, but for `libgmp` it is 
a real possibility.

Once you have added these two missing libraries, the code will finally 
compile!

# More configuration options

The `Config.h` file does not exist in a clean clone of the Arepo 
repository, so in this case the Perl script will just create empty 
versions of the extra source code files. You can copy the 
`Template-Config.sh` and enable some additional options and then the 
code will be compiled with those options. Note that some options will 
requires additional libraries (e.g. `HAVE_HDF5`), which you then need to 
include and link using the same approach as above.

One problem with the approach as outlined above is that the Perl script 
is only run if `cmake` is run, and not when you simply run `make`. So if 
you change `Config.sh` but don't rerun `cmake`, the code will still use 
the old configuration options! To overcome this, we can create a custom 
target for the Perl script:

```
add_custom_target(Config ALL
                  COMMAND ${PERL_EXECUTABLE}
                    ${PROJECT_SOURCE_DIR}/prepare-config.perl
                    ${PROJECT_SOURCE_DIR}/Config.sh
                    ${PROJECT_BINARY_DIR}
                  COMMENT "Running Perl script..."
                  BYPRODUCTS ${EXTRA_SOURCES})
```

This replaces the `execute_process` command and has to go below the 
`set(EXTRA_SOURCES` (since it uses this variable).

This command will create an additional target with the name `Config` 
that is automatically executed whenever `make all` (or simply `make`, 
which defaults to `make all`) is executed. The command is the same as 
before, `COMMENT` is a string that is displayed while the Perl script 
runs, and `BYPRODUCTS` makes sure that CMake knows the `EXTRA_SOURCES` 
are created by this target (so they will not be present when `cmake` 
runs). If we don't add this, then CMake will complain that it cannot 
find the `EXTRA_SOURCES` files.

This works, but unfortunately also means that the Perl script is run 
*every time* `make` is called, irrespective of whether the `Config.sh` 
file was changed or not. To overcome this, and make the command actually 
*depend* on the state of the `Config.sh` file, we can instead create a 
custom command:

```
add_custom_command(OUTPUT ${EXTRA_SOURCES}
                   COMMAND ${PERL_EXECUTABLE}
                     ${PROJECT_SOURCE_DIR}/prepare-config.perl
                     ${PROJECT_SOURCE_DIR}/Config.sh
                     ${PROJECT_BINARY_DIR}
                   COMMENT "Running Perl script..."
                   DEPENDS ${PROJECT_SOURCE_DIR}/Config.sh)
```

The syntax is similar to before, but now we promote `EXTRA_SOURCES` to 
`OUTPUT` (meaning that CMake will automatically rerun the command if the 
extra source code files changes) and we add an explicit dependency for 
`Config.sh` (`DEPENDS`). This will make sure that the script is rerun 
when `Config.sh` changes (but only then). There is one catch: this only 
works if there is a `Config.sh` file present in the source folder.

With that, our CMake configuration file is complete!

# The full file

For reference:

```
cmake_minimum_required(VERSION 3.0)
project(arepo C)

# library dependencies
find_package(GSL REQUIRED)
include_directories(${GSL_INCLUDE_DIRS})
find_package(MPI REQUIRED)
if(MPI_C_FOUND)
  include_directories(${MPI_C_INCLUDE_PATH})
else(MPI_C_FOUND)
  message(FATAL_ERROR "No MPI C compiler found!")
endif(MPI_C_FOUND)

# run the perl script to parse configuration options
find_package(Perl REQUIRED)
include_directories(${PROJECT_BINARY_DIR})
set(EXTRA_SOURCES ${PROJECT_BINARY_DIR}/compile_time_info.c
                  ${PROJECT_BINARY_DIR}/compile_time_info_hdf5.c)

add_custom_command(OUTPUT ${EXTRA_SOURCES}
                   COMMAND ${PERL_EXECUTABLE}
                     ${PROJECT_SOURCE_DIR}/prepare-config.perl
                     ${PROJECT_SOURCE_DIR}/Config.sh
                     ${PROJECT_BINARY_DIR}
                   COMMENT "Running Perl script..."
                   DEPENDS ${PROJECT_SOURCE_DIR}/Config.sh)

# list the source files
file(GLOB SOURCES_LEVEL2 "${PROJECT_SOURCE_DIR}/src/*/*.c")
file(GLOB SOURCES_LEVEL3 "${PROJECT_SOURCE_DIR}/src/*/*/*.c")

add_executable(Arepo ${SOURCES_LEVEL2} ${SOURCES_LEVEL3} ${EXTRA_SOURCES})
target_link_libraries(Arepo ${MPI_C_LIBRARIES})
target_link_libraries(Arepo ${GSL_LIBRARIES})
target_link_libraries(Arepo m)
target_link_libraries(Arepo gmp)
```
