---
layout: post
title: "Input and output with strings, part 2: input with Python"
description: About formatted string input in Python
date: 2019-10-30
author: Bert Vandenbroucke
tags:
  - Code development
---

A short while ago, I wrote a first post in a series about string input 
and output in various (programming) languages, and I started with the 
easy part: string output. Since then, I realised that providing a full 
reference for string input for all languages covered in this first post 
(Python, C(++) and Fortran) would constitute too much material for a 
single post. Hence why this second post in the series will be devoted to 
string input for a single language: Python.

As hinted at before, this is a complicated topic: strings can contain 
many things in various formats (all the ones covered in the post about 
string output), and it is impossible to read anything other than strings 
(e.g. integers or floating point values) from a string without having at 
least some knowledge about this formatting. Some Python functions are 
capable of recognising whether a specific part of a string contains a 
valid number, but these functions will only do this when instructed to 
do so, and even then, they will still not be able to use this number as 
an integer/floating point value if you don't tell them which kind of 
number you want (or they will use a default that might surprise you).

That being said, Python is Python, so there are a lot of easy hacks that 
allow you to many powerful things with strings without having to go 
through the trouble of actually *parsing* the string, which is the 
correct and most powerful way to read strings.

# Some easy solutions

So let's focus on some of these easy solutions before embarking upon the 
difficult journey to understand string parsing. Suppose for example that 
you want to read a text file that contains a number of data values, 
arranged in columns. In principle, reading a text file requires you to 
read the lines from the file as strings, and then reading the values in 
the columns from those strings. However, since this kind of file input 
and output is very common, Python modules like `numpy` actually offer 
convenient functions that do this for you: `numpy.loadtxt` reads data 
from a well-behaved text file (i.e. each row has exactly the same number 
of columns and all values in a column have the same type and can be 
parsed as values of that type), while `numpy.genfromtxt` offers a more 
flexible (and slower) version that can also handle missing or weirdly 
formatted data.

Assuming that all values in the file are numbers, reading a text file is 
as easy as

```
import numpy as np
data = np.loadtxt("filename.txt")
```

This will automatically use a 32-bit floating point data type for the 
values. If your values have another type (e.g. `int`), you can simply 
specify this as an additional argument:

```
data = np.loadtxt("filename.txt", dtype = int)
```

This function will also automatically ignore lines that start with a `#` 
(or any other character passed on to the additional argument 
`comments`). If you want to manually skip some rows at the start of the 
file, or only read specific columns, this is also possible.

In practice, these basic forms of `numpy.loadtxt` have proven sufficient 
for about 90% of the cases where I read in data from text files. 
However, sometimes text files contain data that cannot easily be read in 
as the same type, e.g. some columns contain floating point values and 
others (long) integers. Since floating point values cannot be parsed as 
integers, you can sometimes get around this by simply reading all values 
as floating point values and then later converting the integer elements 
to integers from floats. If the integers are really large, this can 
however lead to round off error which would not occur if you would 
simply read the integers as integers.

In order to deal with these cases, it is possible to specify a custom 
`numpy` data type:

```
data = np.loadtxt("filename.txt",
                  dtype = {
                    "names": ("longInt", "longFloat"),
                    "formats": ("i8", "f8"),
                  },
)
```

This will read each line in the file as a `tuple` with the corresponding 
listed data types. Individual columns can be easily accessed through the 
additional names provided in the data type:

```
print(data["longInt"]) # prints all long integer elements
```

This even works when one or more columns contain string values:

```
data = np.loadtxt("filename.txt",
                  dtype = {
                    "names": ("label", "longInt", "longFloat"),
                    "formats": ("s100", "i8", "f8"),
                  },
)
```

Note that the `s100` in this case contains the maximum size (in 
characters) of a string that can be parsed. This number should be chosen 
carefully.

What if the data you want to read is not contained in a text file, but 
already part of a string? Easy: there is a `numpy.fromstring` method 
that works in a very similar way to `numpy.loadtxt`. If the string has a 
predictable layout (e.g. a fixed number of columns with predictable data 
types in each column), then this provides a very flexible way of reading 
it.

What if the string does not really have a column layout? Or you simply 
want to read a single number from it? If the string has a predictable 
layout, then a very easy (and dirty) hack is the following:

```
text = "I want to read the number 42 from this string"
number = int(text.split()[6])
```

You simply split the string into individual words using `split()`, and 
then use `int()` to convert the 7th (6th when you start counting from 0 
like Python) element into an integer. This will work no matter what the 
value of the number, but will only work as long as the number is always 
the 7th word in the sentence, and is well-behaved. This has been my go 
to solution for string parsing in more cases than I would ever want to 
admit.

# The hardcore solution

The solutions above work in almost all cases that I care about, but that 
is mostly because I usually need to parse strings that I wrote myself, 
i.e. I can guarantee that the input is well-behaved and has a 
predictable format. These solutions will not work if
 - the input has no predictable format: e.g. you know that the string 
contains a single number, but you cannot predict in what position this 
number will be
 - the number itself is contaminated: e.g. you are trying to read a value 
that has a unit attached to it (literally; the unit is not separated 
from the value with a white space), or you are parsing a label that 
encodes some values in a single word: `n10e7s42` (scientists tend to 
construct file names in this way).

When this is the case, there is really only one solution to still 
retrieve the values from the string, and that is by using *regular 
expressions*. In essence, a regular expression is a combination of 
characters of various types that tell the *parser* that reads the string 
how to recognise a certain part of the string. An unsigned integer for 
example will always consist of a combination of digits (0123456789) and 
optional exponential signs (eE), while a signed integer can also contain 
a sign (+-). Whenever the parser encounters a group of characters that 
consists solely of these characters, it could decide that these 
characters make up an integer, and parse that integer, no matter where 
it is in the string.

While in theory this sounds fairly easy, I have noticed that it usually 
is not. One of the reasons for this is that there are many regular 
expression parsers, and that all of them tend to have a slightly 
different syntax. Furthermore, regular expressions are very basic 
constructs. This gives them maximum flexibility (which is good), but 
also means they can be very pedantic. If you want to parse numbers 
consisting of multiple digits for example, you need to explicitly 
instruct the parser to do this. This is very useful if you want to parse 
two different numbers that are not separated by a white space, but very 
annoying in the more obvious case where you just want it to parse the 
whole number.

Python has a very powerful regular expression module called `re`, that 
is devoted to regular expression parsing. It contains some basic 
functions that take a string and a regular expression as input and 
simply output the matching parts of the string, but also has 
functionality to *compile* regular expressions that are used multiple 
times. In this case, a regular expression object is created and 
optimised and then applied to many strings using the same regular 
expression.

Let's start with a simple example:

```
import re
text = "this string contains the number 43 somewhere in its body"
print(re.findall("[0-9]", text))
```

This will print the following:

```
['4', '3']
```

This is how it works: `re.findall` will parse the string provided as 
second argument, and *match* every occurrence of the regular expression 
`[0-9]`, i.e. all characters in the range 0-9 (all digits). It will then 
return a list of all matches as individual strings. This is probably not 
entirely what you want in this case, since you probably do not want to 
match *individual* digits of the number. To match all digits as a single 
number, we need to change the regular expression to `[0-9]+`, where the 
`+` indicates *repeat until the next character does no longer match*.

This will work for any integer number containing only digits, and will 
also work if the string contains multiple numbers (since `findall` 
matches *all* occurrences of the regular expression). It will not work 
if the integer is written in exponential notation, e.g. `2e7`. To match 
this, we need to add the exponent part: `[0-9]+e[0-9]+`. However, this 
will no longer match numbers that do not have an exponent... Note that 
we also cannot add the `e` to the square brackets, as this would 
literally match every character *e* in the string:

```
> re.findall("[0-9e]+", text)
['e', 'e', '43', 'e', 'e', 'e']
```

To allow for both integers with and without an exponent, we need to make 
the exponent bit *optional*. To this end, we first need to create a 
group: `[0-9]+(?:e[0-9]+)`. The group is created by using the round 
brackets, the additional `?:` is required to turn this into a 
*non-capturing* group, because `re` by default only displays the 
*captured* part of a match if such a part exists (I will get back to 
this in a minute). The entire group is made optional by appending a `?` 
to it: `[0-9]+(?:e[0-9]+)?`. To also allow for capital *E* exponents, we 
can change this to `[0-9]+(?:[eE][0-9]+)?`.

This expression will match all unsigned integers. We can add an optional 
sign very easily: `[-+]?[0-9]+(?:[eE][0-9]+)?`. This will now match 
*all* integers. For floating point values, we need to go a step further, 
and add an optional decimal point, followed by additional (optional) 
digits: `[-+]?[0-9]+\.?[0-9]*(?:[eE][-+]?[0-9]+)?`. The `\` before the 
`.` is required to *escape* this character, since a `.` is already used 
by `re` to denote a regular expression that matches any character except 
a white space. Note that we also added an optional sign to the exponent 
in this case. We also introduced the `*` that, unlike `+`, also matches 
0 repetitions of the preceding expression. This expression now matches 
almost all possible numbers. However, it will fail in some cases, e.g. 
when you omit the leading 0 in a purely decimal number like 0.05: `.05`. 
This expression will be matched (since the `[0-9]+` bit matches the 
`05`) but it will omit the decimal point itself from the match. And this 
of course leads to a completely different number...

To solve this, we could replace the `+` for the leading digits with a 
`*`. However, this will go horribly wrong, since then the regular 
expression could match literally anything (since it matches a character 
containing no digits at all). To have a useful regular expression, we 
really need to keep at least one `+`. One possibility is to put the `+` 
after the decimal digits, since then the same problem we had above will 
simply omit the decimal point from expressions like `5.`, which parses 
as the correct number. This will still fail to parse numbers like 
`2.e5`, as it will split them in two parts. More correct would be to add 
an additional `\.?` after the decimal digits. This works, but will now 
also match wrongly formatted numbers like `2.4.e5`, which might not be 
desired.

Another option is to allow for two possible floating point signatures, 
using the regular expression *or*, `|`: 
`[-+]?(?:(?:[0-9]*\.?[0-9]+)|(?:[0-9]+\.?[0-9]*))(?:[eE][-+]?[0-9]+)?`. 
Unfortunately, this expression will still split values like `2.e5` in 
half... The reason for that is that `|` first tries to match the first 
expression, and only parses the second if that fails. Since the first 
expression provides a correct match for `2.`, it will not even try to 
match the second expression, even though that expression provides a more 
complete match. The solution is to swap the order of the two 
expressions:

```
> text = "5 5. .4 1e5 1.435e-9 .2E3 2.e+5"
> re.findall(
    "[-+]?(?:(?:[0-9]+\.?[0-9]*)|(?:[0-9]*\.?[0-9]+))(?:[eE][-+]?[0-9]+)?",
    text)
['5', '5.', '.4', '1e5', '1.435e-9', '.2E3', '2.e+5']
```

As you can see, this now matches pretty much anything that we would 
like. Unfortunately, the regular expression has become quite large by 
now...

If all of this was not complicated enough, there are an extensive number 
of additional options to add more constraints to regular expressions, as 
well as *aliases* for common groups of characters, like `\d` for 
`[0-9]`. A full list can be found in the [documentation for 
`re`](https://docs.python.org/3/library/re.html).

As mentioned before, `re` can be instructed to only *capture* specific 
parts of the matching string, e.g. the contents of *groups* within the 
regular expression. If we were to omit the `?:` in our above expression 
for example, then the expression would still match all the numbers, but 
its output would be more interesting:

```
> re.findall(
    "[-+]?(([0-9]+\.?[0-9]*)|([0-9]*\.?[0-9]+))([eE][-+]?[0-9]+)?",
    text)
[('5', '5', '', ''),
 ('5.', '5.', '', ''),
 ('.4', '', '.4', ''),
 ('1', '1', '', 'e5'),
 ('1.435', '1.435', '', 'e-9'),
 ('.2', '', '.2', 'E3'),
 ('2.', '2.', '', 'e+5')]
```

In this case, for each of the 7 matches, the contents of the individual 
groups that match is displayed. Great for debugging, but not very 
useful, unless you are interested in matching specific parts of the 
floating point number, of course. In that case, it could also be useful 
to provide name labels for specific parts of the match. This can be 
achieved by naming groups: `?P<name>`:

```
> re.findall(
    "[-+]?(?P<base>(?:[0-9]+\.?[0-9]*)|(?:[0-9]*\.?[0-9]+))"
    "(?P<exponent>[eE][-+]?[0-9]+)?",
    text)
[('5', ''), ('5.', ''), ('.4', ''), ('1', 'e5'), ('1.435', 'e-9'),
 ('.2', 'E3'), ('2.', 'e+5')]
```

`re.findall` will ignore the name labels, but other `re` functions do 
not. Most other functions do not return a string (or list of strings), 
but instead return a `Match` object (or list of these objects). This 
object contains functionality to retrieve the original string and 
regular expression that were used to create it, and also offers lots of 
options to retrieve the different groups contained in the match, e.g. 
the labels created above. For the example above, we could for example 
use

```
> for match in re.finditer(
>    "[-+]?(?P<base>(?:[0-9]+\.?[0-9]*)|(?:[0-9]*\.?[0-9]+))"
>    "(?P<exponent>[eE][-+]?[0-9]+)?", text):
>   print(match["base"], match["exponent"])
5 None
5. None
.4 None
1 e5
1.435 e-9
.2 E3
2. e+5
```

As already mentioned before, regular expressions can be *compiled* into 
a regular expression object, so that you can reuse them more 
efficiently. An example:

```
import re
floatregexp = re.compile(
  "[-+]?(?:(?:[0-9]+\.?[0-9]*)|(?:[0-9]*\.?[0-9]+))(?:[eE][-+]?[0-9]+)?"
)
text = "5 5. .4 1e5 1.435e-9 .2E3 2.e+5"
floatregexp.findall(text)
```

The compiled regular expression objects have the same functions as the 
`re` module itself, but always use the same regular expression.

Note that apart from searching specific expressions in a string, you can 
also use regular expressions to check if a string (as a whole) matches a 
regular expression, using `re.fullmatch`. This will only produce a match 
if the entire string matches the expression.

The regular expressions above allow you to very flexibly locate and 
retrieve numbers (and other parts) of a string. However, they still only 
return strings themselves, i.e. you will still need to convert the 
resulting strings into the actual data types you want (using `int()` and 
`float()` for example). This is usually easy enough, but can get quite 
cumbersome (and slow) if you are reading a large number of values e.g. 
from a file. That is why `numpy` also provides an alternative for 
`loadtxt` that use regular expressions: `numpy.fromregex`. I have never 
used this function myself (I only discovered it while doing my research 
for this post), but here is an example. Suppose you have a text file, 
`test.txt`, with the following contents:

```
This line contains the number 5.
And a 672 is hidden somewhere in this line.
```

Then you can retrieve the numbers in an array as follows:

```
> import numpy as np
> test = np.fromregex("test.txt", "[0-9]+", dtype = [("number", "u4")])
> print(test)
[(  5,) (672,)]
```

Note that the `dtype` argument is required in this case and needs to be 
a custom data type like the one we already encountered before.
