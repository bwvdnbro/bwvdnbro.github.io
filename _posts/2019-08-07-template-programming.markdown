---
layout: post
title: "Template programming"
description: >-
  A brief introduction to C++ templates and what they can be used for.
date: 2019-08-07
author: Bert Vandenbroucke
tags: 
  - Code development
---

Nowadays, computers can be programmed using a large variety of computing 
languages, with very different degrees of complexity and sometimes with 
a very strong fan base of enthusiastic users. Fundamentally though, 
computers and their CPUs only understand one language: the machine code 
that contains the binary instructions for the CPU that tell it what to 
do. This machine code only depends on the type of CPU, and has no link 
to any specific programming language. To convert human readable code 
(usually written using ASCII characters in some kind of text editor) 
into machine code (long series of bits that can be fed straight into the 
CPU) some kind of code *interpreter* or *compiler* is required. If an 
*interpreter* is used, the conversion is done while the code is already 
running, and overall code execution will be slow (this is what happens 
for Python and Java). If a *compiler* is used, the human readable code 
is translated into machine code before the program is executed, 
resulting in a *program binary* that makes optimal use of the CPU.

To make the conversion from program code to machine code, the compiler 
can make a number of assumptions that allow it to generate more 
efficient machine code. If you for example have a loop that needs to be 
executed a certain number of times, and the compiler can deduce what 
that number is during compilation, it can choose to get rid of the loop 
altogether and just generate machine code for that fixed number of 
iterations. This way, all the instructions necessary to control the loop 
can be completely removed from the machine code. Similarly, the compiler 
can decide to *inline* a small function call by replacing the explicit 
call to another bit of machine code with the instructions that are in 
that function, removing the overhead of actually calling a function.

A programmer can also directly give instructions to the compiler. In 
C(++), these compiler instructions are preceded by a hashtag (#). One 
typical example is the `#include` statement, which is used to include 
*header* files (that contain information about functions that are 
provided in external source code files or the C(++) standard library). 
When a compiler encounters such an instruction, it will immediately 
interpret and execute it; in the case of an `#include` statement, this 
means it will copy the entire contents of the header file into the 
currently compiled file. Other compiler instructions are e.g. `#define`, 
which allows you to define a *variable* or *macro* that can be 
referenced elsewhere in the code, and that will be literally replaced by 
whatever the `#define` defines by the compiler before it actually 
compiles the code, or the `#pragma` statements that are typically used 
to include OpenMP instructions for shared memory parallelisation.

In C++, these basic compiler instructions have been expanded with a very 
powerful language concept: *templates*. As with the compiler 
instructions above, templates are entirely handled by the compiler at 
compilation time, which means they are not present in the final machine 
code. They do however allow you to write more modular and more concise 
code, and are very useful in certain situations. An introduction.

# An example template

If you have ever used any of the standard C++ library functionality, 
you very likely encountered templates without realising it. Consider for 
example the following bit of code:

```
#include <iostream>
#include <vector>

int main(int argc, char **argv){

  std::vector<double> positions(4);
  positions[0] = 0.;
  positions[1] = 3.;
  positions[2] = 9.;
  positions[3] = 6.;

  for(unsigned int i = 0; i < positions.size(); ++i){
    std::cout << "positions[" << i << "]: " << positions[i] << std::endl;
  }

  return 0;
}
```

This snippet creates a four element vector with double precision 
floating point values, assigns values to the individual elements, and 
then displays the contents of the vector. The reason we know the vector 
contains floating point elements is because of its declaration: 
`std::vector<double> positions(4)`. This bit of code calls the 
*constructor* of the `std::vector` class with a single argument (the 
number of elements: 4). The type of the elements is declared using the 
`<double>` part of the statement. This is a template declaration!

So why do we use a template here (and how does this work)? Basically, a 
C++ vector is nothing more than a memory block for which the memory 
allocation and deallocation are hidden, and with some additional 
functionality (like the possibility to query the size of the vector 
using the `size()` function). Without using vectors, we could rewrite 
the same bit of code as

```
double *positions = new double[4];
positions[0] = 0.;
...
for(unsigned int i = 0; i < 4; ++i){
...
}
delete [] positions;
```

Suppose that instead of double precision floating point elements, we 
wanted to use 32-bit integers as the data type for the vector elements. 
Then the only thing we would need to do is replace `double` with `int` 
twice in the above code snippet. If we were to write our own 
`std::vector` class, then the only change required between the vector 
with floating point values and the one with integer values would be a 
similar change from `double` to `int`. All the other functionality of 
the vector class would be completely unaffected by this change.

When we write the above piece of code, we know, while writing it, that 
we want the data type of the vector elements to be `double`. Since we 
know this, this fact is also known *at compile time* by the compiler. 
This means that the compiler could in principle also make the change 
from `double` to `int` if we were to change our mind about the data 
type. Of course, the compiler is not allowed to simply change every 
`double` to `int` (as this might break other parts of the code). We can 
however tell it which occurrences of a specific value can be replaced at 
compile time. This is what templates do.

To be more specific, the C++ `std::vector` class does not actually use a 
specific data type internally. Instead, it uses a *template data type*: 
it replaces all occurrences of the data type with a placeholder name, so 
that the compiler knows which variables to replace with a specific type 
at compile time. To tell the compiler which data type we want to use, we 
simply provide it as an additional instruction to the `std::vector` 
class, by means of the `<>`.

# Writing your own template classes

So you can use template classes by providing a *template argument* in 
between `<>` when constructing a template class. But how do you 
*declare* a template class? Let's write our own version of the C++ 
standard vector class:

```
template <typename _data_type_>
class OurVector{
private:
  unsigned int _size;

  _data_type_ *_elements;

public:
  OurVector(unsigned int size) : _size(size) {
    _elements = new _data_type_(size);
  }

  ~OurVector() {
    delete [] _elements;
  }

  _data_type_ &operator[](unsigned int index){
    return _elements[index];
  }

  unsigned int size() {
    return _size;
  }
};
```

This class does everything the `std::vector` in our previous example had 
to do: it contains a constructor that allocates the vector in memory, a 
destructor that takes care of deallocating the memory when the vector is 
no longer used, a `size()` function that returns the size of the vector, 
and an indexing operator function (`operator[]`) that allows accessing 
individual elements of the vector. To turn this class into a template 
class, the only thing we had to do was replace all specific occurrences 
of the vector data type with the label `_data_type_`, and add the 
template declaration `template <typename _data_type_>` to the class 
definition. That's all!

Note however that there is an additional subtlety: since the template 
class uses the label `_data_type_` as data type, it cannot be compiled 
on its own (`_data_type_` is not a valid C++ type!). This means that the 
compiler first needs to replace `_data_type_` with an actual type 
whenever the template class is used, before the class can be compiled. 
Since the compiler only knows what the data type should be when you call 
the constructor of the vector class in a source code file, the class 
needs to be compiled as part of that same source code file. Or in other 
words: you need to provide the full class inside a header file that can 
be included in the source code file that is being compiled. So unless 
you only plan to use the template class in one source code file, you 
always need to put the full class definition, including all 
template-dependent code, in a header file.

Note also that the label `_data_type_` can be anything that is not a C++ 
key word. This can make it hard to distinguish between ordinary C++ 
variables and template labels. For this reason, I personally prefer to 
use the double underscore syntax (`_label_`) for template labels, but 
this is by no means standard practice.

In the example above, the template *type* was `typename`, which means 
the compiler is allowed to replace the template label with anything that 
could be used to declare a variable type (standard types like `double` 
or `int`, but also classes, as they can also act as type). It is also 
possible to use the template type `class`, in which case the compiler is 
allowed to replace the label with any class name, but not basic types 
like `double` or `int`. Finally, it is also possible to use the template 
type `int`, in which case the compiler will replace the label with an 
integer. This is useful to create classes with a variable number of 
elements that is known at compile time:

```
template <int _size_>
class Values{
private:
  double _values[_size_];

public:
  double &operator[](unsigned int index){
    return _values[index];
  }

  unsigned int size(){
    return _size_;
  }
};
```

This class can provide an alternative for a standard C++ array with an 
additional `size()` function (similar to `std::array`).

# Template functions

The example above illustrated how to create template classes. In this 
case, the template label is used throughout the class definition, and a 
separate class will be created by the compiler each time the class 
constructor is called with a different template type. It is also 
possible to use a template in conjunction with an individual function (a 
*function template*), in which case the compiler will generate separate 
versions of that function for different template types.

Function templates are extremely useful when a function can perform the 
same action on a variety of variable types. Consider the following 
example:

```
#include <iostream>

template <typename _data_type_>
_data_type_ add(_data_type_ a, _data_type_ b) {
  _data_type_ result = a + b;
  std::cout << a << " + " << b << " = " << result << std::endl;
  return result;
}
```

In this case, we provided our own addition function that provides some 
output while performing the addition. Since both an addition and 
terminal output are independent of the variable type (`_data_type_` 
could be `double`, `int`, `float`...), this function will work for any 
of those types, and apart from the `_data_type_`, the code would be 
exactly the same. By using a template function, we avoid unnecessary 
code duplication and leave this task to the compiler.

What if you do want to provide a separate function to add two double 
precision floating point values? In this case, a concept called 
*template specialisation* comes into play. By default, the compiler will 
only create a function (or class) from a template when it cannot find a 
matching function or class for the code signature it encountered. This 
makes sense, as the compiler is only allowed to create the same class or 
function once during the compilation of a single file (if you use a lot 
of `std::vector<double>` variables, it will only create the code for 
that class once). You are allowed to provide the code for a specific 
specialisation of a template yourself, in which case the compiler will 
always use your custom specialisation. This is done as follows:

```
template<>
double add<double>(double a, double b){
  double result = a + b;
  std::cout << "Double addition: " << a << " + " << b << " = " << result
            << std::endl;
  return result;
}
```

Note that we need to tell the compiler that this is a template function 
`template<>`, before we tell it that we explicitly assume the template 
type `double`.

# What can I use templates for?

Templates can be useful in a variety of scenarios. A first (obvious) 
scenario is that of a *container class*, i.e. a class that is used to 
store values or objects (like a `std::vector`). Very often, these 
classes can be written in a way that is pretty much agnostic of what the 
object type is, and then a template allows you to reuse that same class 
for different object types without having to duplicate any code.

Another scenario in which template functions are very powerful is one 
where a function needs to read or write a value to and or from a 
terminal window or file. In such a function, the interface between the 
terminal window or file and the code will make use of a `std::string`, 
independent of the type of the variable that is read or written. The 
only thing that differs between different variable types is the 
conversion to and from a `std::string`. In this case, you can provide a 
single template input/output function that itself calls a template 
conversion function. That conversion function can then be specialised 
for different variable types, while only a single input/output function 
needs to be written.

The final scenario I want to mention here is one were templates are used 
to mimic object inheritance. Since template labels can also be classes, 
it is possible to call member functions on them. If multiple classes 
provide the same function (with exactly the same signature), then the 
compiler can in principle interchange these at compile time. This is 
similar to normal class inheritance, where a *parent class* defines a 
function signature that can then be implemented by different *child 
classes*. However, in normal inheritance, the compiler will always 
create an explicit function call to the parent function, and the 
decision on which function to call will be made at run time by the CPU, 
based on which child class is actually used. If you already know which 
child class will be used, then it is better to use a template: the 
compiler will then create a direct call to the actual child class 
function, avoiding the additional call to the parent function.

# Limitations

As repeatedly pointed out above, templates are completely handled by the 
compiler. This means that they can (and should) only be used when the 
information required to compile them is available at compile time. You 
can use a template size for a custom array class when you know what 
sizes of arrays you want to use in your code. You cannot use the 
template array when the size depends on the run time (it is e.g. read 
from the command line).

A good way to think about it is in terms of code duplication. In 
principle, you should always be able to do the same job as the compiler, 
and explicitly write the template specialisations you use yourself. If 
you are using templates correctly, this will mean a lot of code 
duplication, where you need to write a lot of classes/functions that are 
almost identical up to a single number or variable type. If you are 
somehow unable to write the code for a template specialisation (because 
it e.g. depends on a number that is not available before you start 
running the code), then you cannot use a template. If you notice that 
explicitly writing the template specialisations does not cause any 
significant code duplication, then it might be better to convert the 
class/function into a non-template version, as that makes for clearer 
code (unless you plan to use the same function with a different type 
later, of course).
