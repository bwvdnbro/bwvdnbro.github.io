---
layout: post
title: "Exposing C++ to Python: an introduction"
description: A brief summary of methods to expose C++ to Python.
date: 2019-11-20
author: Bert Vandenbroucke
tags:
  - Code development
---

Python is a very popular tool among many scientists, mainly because it 
is very easy to use and because there exists a large number of powerful 
libraries that can be used from Python. Writing efficient Python is 
however not as easy, especially if the algorithm you want to write is 
complex and cannot exploit the power of NumPy or SciPy. For this reason, 
it can be quite useful to write the crucial parts of an algorithm in a 
low-level language like C++, and then somehow expose this code to 
Python. In this post, I will list a few ways to achieve this.

# The easy way: running software from Python

Before we embark on the difficult journey to actually couple C++ to 
Python, it is worth pointing out that there are lazy solutions. If you 
have a C++ executable that can be easily controlled through command line 
arguments or an input parameter file, and this executable generates the 
output you want for those parameters without much overhead, then 
arguably the easiest solution to achieve this is to simply run the 
executable from Python.

Running commands from Python can be done in many ways, e.g. using the 
`subprocess` module. To run an executable called `test` with additional 
arguments `a.in` and `0.001`, and collect its output to the terminal for 
example, you can use:

```
import subprocess

output = subprocess.run(["./test", "a.in", "0.001"], stdout = subprocess.PIPE,
                        universal_newlines = True).stdout
```

This assumes the `test` executable is located in the same directory 
where you are running Python. The arguments to `subprocess.run` are 
quite straightforward: the first argument is a list containing all parts 
of the command to run (usually an executable followed by additional 
arguments). These will be joined together with spaces to create the 
command that will be executed. `stdout = subprocess.PIPE` will capture 
the output to the terminal instead of simply writing it to the terminal 
window. Finally, `universal_newlines = True` makes sure line breaks in 
the terminal output are processed correctly.

Both the input and output for `subprocess.run()` assumes strings, so you 
will need to convert all input arguments to strings before passing them 
on, and you will need to parse the output if you want to extract values 
from it. This might not be very efficient and definitely does not make 
for very easy code. If your executable reads parameters from a file, 
then your script either needs to create that file, or somehow needs to 
change parameters in some template file, which again requires regular 
expression parsing.

Alternatively, you can share values between Python and the executable 
using binary files, since those can store *any* data type. NumPy has 
convenient functions to read and write these files in Python, and 
reading and writing binary files in C++ is also not hard. Even more 
advanced is the use of *memory mapped files*. These are binary files 
that the operating system maps to a specific location in memory. 
Multiple processes running on the operating system can memory map the 
same file, and this means that they will access the same location in 
memory, without necessarily requiring any interaction with the file 
system itself. This is quite advanced, and I will definitely write a 
post about this topic at some point.

Even with all these powerful tools, this lazy (but easy) way to 
interface C++ and Python is far from ideal, since either you need to do 
a lot of string parsing and formatting, or you need to somehow exchange 
binary data between different processes. Surely there must be a better 
way?

# The better way: writing Python modules in C++

Python itself is an executable. When you run `python` (or the 
recommended `python3`), you are actually running a program called the 
*Python interpreter* that reads, interprets and executes your commands 
written in the Python scripting language. The Python interpreter itself 
is written in C ([source code](https://github.com/python/cpython)), 
which means that all objects that you create in Python scripts actually 
correspond to C structures. To extend Python with your own modules in 
C++, you only need to know how to access and use these C structures.

To make this possible, Python exposes a lot of its functionality through 
a so called [API](https://docs.python.org/3/c-api/index.html). This API 
makes it possible to write your own libraries that can then be imported 
as any other module from a Python script. Unfortunately, the process to 
do this is quite complicated.

Because the process is so complicated, many solutions have been created 
over the years to simplify this process. I have encountered some of 
these myself over the years, and will give a chronological overview and 
impression below. All of these solutions assume that you have a working 
C++ code, and that you want to expose parts of it (functions, 
classes...) to Python.

## SIP and related packages

My first encounter with Python/C++ interfaces used the 
[SIP](https://www.riverbankcomputing.com/static/Docs/sip/) bindings. 
These bindings equip existing C++ code with a number of directives that 
are then used by an intermediary interpreter to generate additional C 
files that contain the module code. SIP is only one of many such 
bindings.

My experience with SIP was not a very good one: the analysis software 
used by the group where I did my PhD used it, and after every update of 
our operating systems this software would break because of SIP issues. 
SIP also makes it a lot harder to compile code because of the additional 
steps that are required, which increases the risk of these failures. 
Because of all this, I will not give any examples of how to use SIP, and 
simply mention that it (and many alternatives) exists.

## Boost

The [C++ Boost libraries](https://www.boost.org/) are a collection of 
open-source peer-reviewed library functions that extend the standard C++ 
library with more powerful features. Some of these features eventually 
even make it to the standard library. Boost is the go to library for any 
complicated functionality that would be provided by more obscure 
libraries in C and Fortran, e.g. extended precision floating point 
operations, command line argument parsing, regular expression parsing... 
And it also contains its own [Python 
bindings](https://www.boost.org/doc/libs/1_71_0/libs/python/doc/html/index.html).

These Python bindings are very similar to the other Python bindings I 
mentioned above, with one distinction: they are entirely based on macros 
(directives that are executed by the compiler during compilation), so 
that they do not require any additional steps during the compilation 
process. If you have the Boost Python headers and libraries installed on 
your system, then you can simply write a new C++ code file that contains 
the code required to expose your C++ routines and classes, or even add 
them to the existing code files.

A simple example (`test.cpp`):

```
#include <boost/python.hpp>

int get_answer_c(){
  return 42;
}

BOOST_PYTHON_MODULE(answer){
  boost::python::def("get_answer", get_answer_c);
}
```

This creates a simple module that contains a single function that 
returns a number. To compile the example, you need to create a shared 
library:

```
g++ -fPIC -shared test.cpp -o answer.so
```

This will likely fail with an error that complains about missing 
`pyconfig.h` header files. You can fix these by adding the `pyconfig.h` header
location for the Python version you want to use to the command, e.g.

```
g++ -fPIC -shared -I /usr/include/python3.6m test.cpp -o answer.so
```

You will probably also need to link the Boost Python (shared) library 
(your library will compile without doing this, but will fail to run):

```
g++ -fPIC -shared -I /usr/include/python3.6m test.cpp \
  /usr/lib/x86_64-linux-gnu/libboost_python3.so -o answer.so
```

Once you manage to compile, you can import the new module as any other 
Python module:

```
import answer
print(answer.get_answer())
```

Note that Boost recommends using their own tool to compile your module, 
but I think it is more instructive to do it manually. For production 
runs, I prefer using CMake to configure my Makefiles and then this 
becomes very easy.

While the approach above is quite simple, there are a few things to keep 
in mind. First of all, you need to make sure to import the library using 
its full name; shared libraries on Unix are typically called 
`libSOMETHING`, so this means you would need to import using `import 
libSOMETHING` (in our example, the library does not include the `lib`). 
The name you give to the `BOOST_PYTHON_MODULE` macro needs to match the 
name of the Python module, and should hence be the same as the name of 
the shared library file.

To expose classes, you can use similar syntax:

```
#include <boost/python.hpp>

class ClassAnswer{
private:
  int _answer;
public:
  ClassAnswer(int answer) : _answer(answer) {}

  int get_answer_c() {
    return _answer;
  }
};

BOOST_PYTHON_MODULE(answer){
  boost::python::class_<ClassAnswer>("Answer", boost::python::init<int>())
    .def("get_answer", &ClassAnswer::get_answer_c);
}
```

```
import answer
aobj = answer.Answer(42)
print(aobj.get_answer())
```

As in the first example, I have deliberately given different names to 
the C++ and Python class and its member method to illustrate that the 
names in Python are completely set by the Boost Python binding and do 
not need to match the internal C++ names.

While the examples above seem pretty simple, even Boost Python bindings 
can get very complicated very quickly, especially if you want to pass on 
arguments from Python to C++ that are not basic types (e.g. Python lists 
or even NumPy arrays). In this case, it is often not possible to 
directly expose the C++ class member functions. Instead, you need to 
write dedicated wrapper functions that take care of the conversion from 
Python types to C++ types (and vice versa). This can make things quite 
complicated and my own experience with this is not necessarily all good.

That being said, since Boost is specifically aimed at C++, Boost Python 
is probably the most natural way to expose C++ to Python. But of course 
it creates an important dependency on the Boost library.

## The Python C API

The most low-level way to bind C++ to Python is by using the C API 
provided by the Python interpreter itself. Within this API, *every* 
object or value that is exchanged with Python is a `PyObject`, i.e. a 
low-level Python object that stores the information that Python requires 
for its memory management. Using this API, the first example above 
becomes

```
#include <Python.h>

static PyObject *get_answer_c(PyObject *self, PyObject *args) {
  return PyLong_FromLong(42);
}

static PyMethodDef TestMethods[] = {
  {"get_answer", get_answer_c, METH_VARARGS,
   "Get the answer to life, Universe and everything."},
  {nullptr, nullptr, 0, nullptr}
};

static struct PyModuleDef testmodule = {
  PyModuleDef_HEAD_INIT,
  "answer",
  "Answer module.",
  -1,
  TestMethods
};

PyMODINIT_FUNC PyInit_answer() {
  return PyModule_Create(&testmodule);
}
```

Quite a bit longer than the Boost Python version! Note that even the 
simple integer that we return from the function needs to be converted 
into a Python Long object; even integers are objects in Python. To 
compile this example, we can use Python itself and write a `setup.py` 
script:

```
from distutils.core import setup, Extension

def main():
    setup(name="answer",
          version="1.0.0",
          description="Answer module",
          ext_modules=[Extension("answer", ["test.cpp"])])

if __name__ == "__main__":
    main()
```

Where `test.cpp` is the name of the file that contains the module. We
can compile the example using

```
python3 setup.py build
```

This will create a new folder called `build` within the current working 
directory; inside this folder a folder called `lib.SYSTEM_ARCHITECTURE` 
is created that contains the shared library module. You can use this 
module as before from within that build directory.

Using `install` instead of `build` would first compile the module (in 
the same location) and then install the new module so that it can be 
used from within Python anywhere on your system. This usually requires
root privileges. You can also just install the module for the current user
by using

```
python3 setup.py install --user
```

This does not require any special privileges.

Despite being significantly more complicated than the Boost Python 
example, using the C API directly has a few advantages. Firstly, you can 
very easily control for which Python version your module is compiled by 
running the `setup.py` script with that version; no need to figure out 
the location of the header and library files specifically for that 
version on your system. Secondly, the `install` option makes it 
incredibly easy to install the module on your system and make it 
available everywhere. Modules compiled with Boost Python will be located 
in a specific location and need to be manually installed. Thirdly, the 
`setup.py` based install process is exactly the same one as used by the 
Python package manager (`pip`), so that this approach makes it very easy 
to distribute your module if you would want that.

However, the fact that the syntax is a lot more complex makes it much 
harder to learn and use. I only just started this process myself, and I 
don't feel I am in a position to give very detailed examples yet, 
including the second example above that exposes a class. I might come 
back to this in a future post.

# Interfacing with other Python libraries

Most of the power of Python comes from specific libraries like NumPy and 
SciPy, and your own modules will probably only become powerful 
themselves if they can somehow interface with these as well. It would be 
especially useful for example if your Python module functions could 
accept NumPy arrays as input arguments and return them as output.

Both Boost Python and the Python C API have ways to read, create and 
write NumPy arrays, but none of these are particularly well documented 
(in my opinion) nor very easy. Given my limited experience with this, I 
will again refrain from giving any specific examples, and simply mention 
that this is indeed possible. The Python `distutils` and `setuptools` 
packages have options to enforce dependencies for the modules you 
create, which can ensure that your module will work as intended.
