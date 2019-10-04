---
layout: post
title: "A note on linkers"
description: >-
  An attempt to demistify some of the more obscure aspects of code compilation.
date: 2019-10-04
author: Bert Vandenbroucke
tags: 
  - Code development
---

Compiling a code is always a bit of an exciting event, especially if you 
just finished implementing something new. You thought about what you 
want to do, you built up a logical structure for the new code in your 
head, did the necessary online research to see how to exactly implement 
a function call, and now you will ask the compiler to actually turn this 
into a real program that does something.

And more often than not, this will fail at the first attempt. If you are 
new to a language, you are likely to make syntax errors (a forgotten `;` 
somewhere for example). If you were in a hurry while writing the code, 
you might have made a typo. Or you might be making some slightly more 
complicated programming mistake that the compiler will tell you about. 
In most cases, the compiler errors or warnings you get (I personally 
recommend turning all warnings into errors using the `-Werror` flag for 
the GCC, Clang and Intel compilers) will be pretty easy to decipher, and 
after a few iterations, the code will usually happily compile.

However, as I already mentioned in a post a long time ago, the 
compilation process for a complex project involving multiple source code 
files and libraries is a two step process. The first step is actually 
compiling all the individual source code files into machine code (so 
called *object files*, with a typical `.o` extension). During the second 
step, these object files are combined into an executable, together with 
all additional libraries the executable depends on. This is done by a 
*linker*, a separate program (although usually shipped with the 
compiler) that makes sure all interdependencies between different object 
files and object files and libraries are correctly resolved.

If something goes wrong during this step, the error messages will 
usually be a lot harder to understand, making it more difficult to 
understand what the problem is and how to resolve it. There have been 
multiple ocassions in the past where I spent a considerable amount of 
time trying to figure out a linker error, eventually ending up copying 
some solution from the internet that seemed to work, as if it was magic. 
Although this is good from a pragmatic point of view, it is not very 
satisfying, nor is it very helpful towards future problems.

By now, I have sufficient experience with linker problems to actually 
have a better understanding of what a linker exactly does and how this 
can go wrong, and I thought it would be good if I could share some of 
this understanding, in the hope that it might help other people 
understand the linker magic.

# Why do we need a linker?

To understand what a linker does, it is first of all important to know 
why we need it. Imagine you have a minimal C++ program, consisting of:

A header file, "hello.hpp":

```
void say_hello();
```

an implementation file, `hello.cpp`:

```
#include <iostream>

void say_hello() {
  std::cout << "Hello!" << std::endl;
}
```

and a main program file, `main.cpp`:

```
#include "hello.hpp"

int main(int argc, char **argv){
  say_hello();
  return 0;
}
```

To compile this simple program, you first need to compile the individual source
code files (assuming you use the GCC compiler:

```
> g++ -c hello.cpp
> g++ -c main.cpp
```

Then, you need to link them:

```
> g++ -o hello hello.o main.o
```

This will produce the executable `hello`, which you can run:

```
> ./hello
Hello!
```

The structure of this program is very simple: the main function will 
place a single call to the `say_hello` function and exit. The 
`say_hello` function itself is compiled into the `hello.o` object file. 
What the linker needs to do is make sure that the machine code in 
`main.o` that calls `say_hello`, actually points to the exact location 
in `hello.o` where `say_hello` is actually implemented, so that the 
computer can execute this function at runtime.

Without this linking, the compute would still be able to execute 
`main.o`, but when it arrives at the line where `say_hello` is called, 
it would not know where to find this function. The `say_hello` function 
in `hello.o` would work, except that it would never get called, because 
the executable part of the program does not know where to find it.

So the task of the linker is to make sure that calls in one object file 
to a function or variable defined in another object file are correctly 
resolved. The same is true for calls to other functions, e.g. functions 
that are part of some external library or the standard C or C++ library 
(like the `std::cout` call in `hello.cpp`). To do this, the linker needs 
to know where all relevant object and library files are stored in the 
system, what kind of linking is required (static or dynamic, I will come 
back to this), and where in the object and library files specific 
functions can be found.

# The explicit linker command

It looks like the link step in the above example was performed by the 
GCC compiler, but actually the compiler called another program, the 
linker. We can figure out what command was actually called by adding the `-v`
flag to the linking command:

```
> g++ -v -o hello hello.o main.o
Using built-in specs.
COLLECT_GCC=g++
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 7.4.0-1ubuntu1~18.04.1' --with-bugurl=file:///usr/share/doc/gcc-7/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-7 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-libmpx --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1) 
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/7/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/7/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-shared-libgcc' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/7/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper -plugin-opt=-fresolution=/tmp/cccMNTeb.res -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie -z now -z relro -o hello /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o -L/usr/lib/gcc/x86_64-linux-gnu/7 -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/7/../../.. hello.o main.o -lstdc++ -lm -lgcc_s -lgcc -lc -lgcc_s -lgcc /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o
COLLECT_GCC_OPTIONS='-v' '-o' 'hello' '-shared-libgcc' '-mtune=generic' '-march=x86-64'
```

The relevant line is the penultimate one. As you might be able to see, 
the basic linking command `-o hello hello.o main.o` is still there, but 
the compiler has added a lot of additional object and library files to 
this. Some of these are necessary to link in the C++ standard library 
(e.g. the `-lstdc++` but also `-lm` and `-lc`). Others are due to the 
specific C++ standard library we are using here: the version provided by 
the GCC compiler. This library version contains some additional features 
to help make the program more robbust and some of these features require 
additional object files to be linked in, like `crti.o`, `crtbeginS.o` 
and so on. Since these objects and the library options for the linker 
are the same for every C++ program, the GCC compiler provides a 
convenient wrapper for the linker. The actual linker program called by 
GCC is called `collect2`, and this itself is only a wrapper for the 
actual linker, a program called `ld`.

If something goes wrong during linking, the command that will fail is 
hence called `collect2`. This is the command you will see in the error 
message that the compiler displays.

Note that because of the wrapper, all the paths and options for the C++ 
standard library are already guaranteed to be part of the linker 
command, and unless your compiler installation itself is broken, they 
will be correct. In general, this is hence not something you should 
worry about if you do not invoke the linker directly.

# Symbols
