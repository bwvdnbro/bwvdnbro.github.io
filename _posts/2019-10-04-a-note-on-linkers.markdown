---
layout: post
title: "A note on linkers"
description: >-
  An attempt to demystify some of the more obscure aspects of code compilation.
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
multiple occasions in the past where I spent a considerable amount of 
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
computer can execute this function at run time.

Without this linking, the compute would still be able to execute 
`main.o`, but when it arrives at the line where `say_hello` is called, 
it would not know where to find this function. The `say_hello` function 
in `hello.o` would not work either, because it would not know where to 
find the C++ standard library functionality for writing to the standard 
output (`std::cout`). And it would never get called anyway.

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
(e.g. the `-lstdc++` but also `-lm` and `-lc`). Others are required for 
the specific C++ standard library we are using here: the version 
provided by the GCC compiler. This library version contains some 
additional features to help make the program more robust and some of 
these features require additional object files to be linked in, like 
`crti.o`, `crtbeginS.o` and so on. Since these objects and the library 
options for the linker are the same for every C++ program, the GCC 
compiler provides a convenient wrapper for the linker. The actual linker 
program called by GCC is called `collect2`, and this itself is only a 
wrapper for the actual linker, a program called `ld`.

If something goes wrong during linking, the command that will fail is 
hence called `collect2`. This is the command you will see in the error 
message that the compiler displays.

It is very instructive to try to link the example program yourself using 
`ld`, starting from the basic linker command and then adding the 
additional components on the detailed linking command above one by one 
until the executable is successfully linked. You will notice that this 
is not at all an easy procedure!

Note that because of the wrapper, all the paths and options for the C++ 
standard library are already guaranteed to be part of the linker 
command, and unless your compiler installation itself is broken, they 
will be correct. In general, this is hence not something you should 
worry about if you do not invoke the linker directly. And given how hard 
it is to add all these elements yourself, you should probably always use 
the compiler provided wrapper for the linker.

# Symbols

The only way the linker can know how to find the machine code for the 
functions that are referenced in object and library files, is by using 
some kind of identifier. These identifiers are called *symbols*. These 
symbols are somewhat similar to the *declarations* of functions and 
variables that you put in header files and that you use to tell the 
compiler what the correct signature is for a function call (or class) 
that you are using in one file, but that is actually defined in another.

Whenever the *compiler* encounters a new function or variable in a 
source code file that needs to be visible outside that file, it will 
prepend the bit of machine code generated for that function or variable 
with a unique symbol. Whenever that same variable or function is called 
in another source code file, it will simply check if the call matches 
the declaration for that function or variable in the header file, and 
then insert the same symbol in the machine code for that source code 
file. When the *linker* later links together the two object files, it 
will replace these symbols with actual machine code that calls the 
correct machine code for that function or variable where it is used.

In order for this to work, the symbols that are generated for the 
different source code files need to be consistent, i.e. they need to be 
generated in a deterministic way. On top of that, the symbols also need 
to be unique: if a variable or function with a specific symbol is 
created in one object file, then the same variable or function symbol 
cannot be created in another one, since then the linker would not know 
to which of the versions to link. The same goes for all additional 
libraries that are linked into the program: their symbols need to be 
generated in the same way and also need to be unique.

The rules for generating deterministic symbols depend on the compiler, 
and also on the programming language. For e.g. the C language function 
names need to be unique, so the function name itself can simply act as a 
unique and deterministic symbol. A language like C++ allows *function 
overloading*, i.e. multiple functions with the same name but with a 
different number of function arguments or function arguments of a 
different type. In this case, the symbol for a function also needs to 
encode information about the number and type of the function arguments 
to be unique.

A consequence of these different rules for different languages is that 
it is not generally possible to link together object files that were 
*compiled* for different languages. This seems obvious for object files 
of two very different languages like Fortran and C++, but is also true 
for languages that have very similar syntaxes, like C and C++. Note 
however that if your C code can be compiled as valid C++ code (which is 
quite likely), and you compile it with the C++ compiler, the generated 
symbols will use the C++ rules and linking will work. So everything 
depends on the compilation step.

That being said, it is often necessary to link together machine code 
that was generated for different languages. A lot of functionality of an 
operating system exists in the form of libraries that were written in a 
specific language (for Unix systems, this is generally C), and if you 
want to use this functionality, you will need to link against these 
libraries. Theoretically, this should be perfectly possible, as in the 
end all libraries just contain machine code, which is independent from 
the language it was originally written in. The problem is then not so 
much the fact that the library was written in another language, but the 
fact that the symbols for the functionality in that library were created 
using different rules.

If you do need to link code in different languages, it suffices to tell 
the compiler to generate symbols in a different way. For C and C++, this 
can be done by including the `extern` keyword in the declarations of 
functions. Suppose for example that the `say_hello` function in our 
example was written in C. Then we would have the following header file, 
`hello.h`:

```
extern "C"{
  void say_hello();
}
```

and the following implementation file (now using the C standard 
library):

```
#include <stdio.h>

void say_hello() {
  printf("C Hello!\n");
}
```

We now need to compile the C source code file using `gcc`:

```
> gcc -c hello.c
```

Replace the `hello.hpp` in `main.cpp` with `hello.h`, recompile 
`main.cpp` (still using `g++`) and link again (also using `g++`). The 
program will still work!

To see the difference between the symbols in both cases, we can use the 
`nm` command line tool. This tool will read the object file generated by 
the compiler, and print out all the symbols it contains. For the object 
file generated from `hello.c`, we get:

```
> nm hello.o
                 U _GLOBAL_OFFSET_TABLE_
                 U printf
0000000000000000 T say_hello
```

For the object file with source `hello.cpp`, this is:

```
> nm hello.o
                 U __cxa_atexit
                 U __dso_handle
                 U _GLOBAL_OFFSET_TABLE_
0000000000000078 t _GLOBAL__sub_I__Z9say_hellov
000000000000002f t _Z41__static_initialization_and_destruction_0ii
0000000000000000 T _Z9say_hellov
                 U _ZNSolsEPFRSoS_E
                 U _ZNSt8ios_base4InitC1Ev
                 U _ZNSt8ios_base4InitD1Ev
                 U _ZSt4cout
                 U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
0000000000000000 r _ZStL19piecewise_construct
0000000000000000 b _ZStL8__ioinit
                 U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
```

The latter can be made easier to read by using the `-C` option for `nm`:

```
> nm -C hello.o
                 U __cxa_atexit
                 U __dso_handle
                 U _GLOBAL_OFFSET_TABLE_
0000000000000078 t _GLOBAL__sub_I__Z9say_hellov
000000000000002f t __static_initialization_and_destruction_0(int, int)
0000000000000000 T say_hello()
                 U std::ostream::operator<<(std::ostream& (*)(std::ostream&))
                 U std::ios_base::Init::Init()
                 U std::ios_base::Init::~Init()
                 U std::cout
                 U std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)
0000000000000000 r std::piecewise_construct
0000000000000000 b std::__ioinit
                 U std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)
```

While in the C case, the symbol for the `say_hello` function is simply 
`say_hello`, the C++ version is `_Z9say_hellov`. By including the 
`extern "C"` keyword in the C header we include in `main.cpp`, we 
instruct the compiler to generate the C version rather than the C++ 
version of this symbol.

Something similar can be done in Fortran (using the `iso_c_binding` 
module) to make sure the Fortran compiler generates symbols that can be 
read in C(++) or vice versa. Many Unix C libraries include a check in 
their header files that activates an `extern "C"` statement whenever the 
library is included in a C++ project, so that you as a developer do not 
need to worry about correctly including that library.

Symbols are often mentioned in linker error messages, usually for two 
reasons. An *undefined reference to* linker error means that a symbol 
referenced in an object file does not have an actual implementation in 
another object file or library, usually because that object file or 
library is missing from the linker command. *multiple definition of* 
linker errors on the other hand mean that the same symbol is used for 
two different functions or variables, because you are reusing a function 
or variable name that already exists (are you linking two source code 
files that contain a `main` function?). This is often caused by unsafe 
use of header files: if you define a function or variable *inside* a 
header file and then include that header file from different source code 
files, then the compiler will include the entire definition of that 
function or variable *and its symbols* into the generated object files. 
That is why you should (a) never define variables inside a header file, 
and (b) always make sure that function definitions inside header files 
are *inlined*, i.e. the code they contain rather than the function 
definition is included in the object file, and no symbols are generated.

# Static versus dynamic

When multiple object files are linked together into an executable, the 
output of the linking process is a single executable that contains all 
the machine code that makes up that executable. In principle, the same 
can be done for executables that contain external libraries. This is 
called *static* linking: the object and library machine code is copied 
and linked as it is right now, and the resulting executable contains a 
static image of all this code that can only be changed by recompiling or 
relinking the executable. Since the executable contains *all* the 
machine code, it can become very large, but assuming it does not depend 
on any external input files, it can be copied and run from any other 
location.

Sometimes, this is not ideal. The external library you use could be very 
large, or the external library could change quite often and you want to 
be able to incorporate those changes into your code without having to 
recompile your executable every time. In this case, it is possible to 
*dynamically* link the library into your executable. In this case, the 
linker will not copy the machine code for the library, but will instead 
insert the location of the machine code within the external library file 
into the code. When the executable is executed, the external library 
will be loaded into memory, and the address of the linked functionality 
will be used to access the corresponding machine code. The advantage of 
this approach is that the executable does no longer need to contain all 
the machine code for the library, and also that the external library 
could in principle change without the need to recompile the executable.

Of course, this will only work if the linker can somehow generate a 
consistent address for the dynamically linked functionality that will 
keep working even if the external library itself changes. In practice, 
this means that different library files are generated for different 
types of linking: static libraries that can be used for static linking 
(with typical `.a` extensions on Unix), and shared libraries for dynamic 
linking (with a `.so` extension). It is very unlikely that you will ever 
actively need to think about what type of linking is required, but it is 
good to know these terms.

# Order is important

One last annoying thing about the linker is that it can be very peculiar 
about the order in which static libraries are linked in to your program. 
A static library itself is simply a collection (or *archive*) of object 
files, containing the same type of symbols you would find in the object 
files for your own source code. Static libraries can however contain a 
large amount of these object files, making looking through all of them 
during linking a reasonably expensive operation.

For that reason, the GCC linker (I don't know about other linkers) uses 
a very basic approach to find unresolved symbols during linking. It will 
simply take all the arguments provided to it, from left to right, and 
look through them, creating a list of unresolved symbols, i.e. symbols 
that are referenced but not yet defined. Whenever it finds a definition 
for a symbol, it will use the list to resolve all occurrences of that 
symbol. However, the linker does not generally store a list of resolved 
symbols for external static libraries, i.e. it will not generally keep 
track of the symbols that it already resolved. If a symbol is referenced 
after the linker encountered its definition, it might hence not be able 
to resolve it.

There are good reasons why the linker uses this strategy: system 
libraries can contain a huge number of symbols, and looking up symbols 
in a huge list is not a very efficient process. But unfortunately, this 
also means that you need to make sure that libraries are included in the 
right order: always after the object files that make up your program, 
and always in an order that makes sure dependencies between external 
libraries are resolved correctly. This is why it is always a good idea 
to use compiler wrappers for the linker (so that the wrapper can figure 
out the interdependencies for the standard libraries), and why I would 
also strongly recommend using configuration software like `autotools` or 
`CMake` to create linking commands.

Note that there are situations in which external libraries have circular 
dependencies, e.g. library `a.a` uses functionality from `b.a`, but 
`b.a` uses a function that is part of `c.a` that uses a function from 
`a.a`. There are various ways to resolve this: you can simply list `a.a` 
twice in the linker command, or you can force the linker to iteratively 
look through these three libraries until all symbols have been resolved, 
by using a group (`ld` has the `-(` and `-)` or `--start-group` and 
`--end-group` options to create cyclic groups of static libraries). This 
will have a performance impact, but can sometimes be necessary in order 
to successfully complete the linking process.

# Passing on additional linker options

If you use compiler wrappers and configuration tools for your project, 
then you will hopefully never need to touch the linker command. There 
are a few cases in which it can be useful to do this nonetheless, e.g. 
if you have cyclic library dependencies (see above) or if the linker 
does not find a library that is definitely installed on your system (a 
good configuration tool will help you sort out both of these).

The correct linker option to pass on the path where a library file can 
be found is `-L`, while the syntax to pass on a library file directly is 
`-l`. However, since you will probably invoke the linker command through 
the compiler wrapper (or at least, you should do this), you cannot 
directly pass on these options. Instead, you need to tell the wrapper to 
pass them on to the linker, using the `-Wl` option:

```
> g++ -o test test.o -Wl,-L/path/to/library -Wl,-llibrary
```

Multiple `-Wl` option arguments can be combined as a comma-separated 
list, while multiple `-Wl` options can be given as well. All of them 
will be passed on in the order they are provided as a space separated 
list. For shared libraries, you want to use `-R` rather than `-l`.
