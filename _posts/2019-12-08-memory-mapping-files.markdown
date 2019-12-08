---
layout: post
title: "Memory mapping files"
description: >-
  An introduction to the concept of memory mapping and why it can be useful.
date: 2019-12-08
author: Bert Vandenbroucke
tags:
  - Code development
---

Input and output, however tricky, are usually considered of secondary 
importance during a computationally intensive numerical simulation, 
since the time spent reading and writing files is usually a lot shorter 
than the time spent doing calculations. Having good input and output 
routines can however become important if these are called very 
frequently, since file manipulation involves (slow) hard drives and 
takes a long time compared to more basic CPU tasks like calculations.

In this post, I will discuss an interesting method to read and write 
files called *memory mapping*. In essence, this method replaces 
expensive file manipulations with much cheaper memory access, 
transferring the responsibility for how this memory is written to and 
read from an associated data file to the operating system, rather than 
your own software. This has a few important advantages, in terms of 
speed, number of reads/writes and even the possibility to share files 
between different programs without the need to write everything to disk.

# What is memory mapping?

When a program reads or writes data from a file, then a location 
(usually encoded as a number of bytes since the start of the file) in 
that file is accessed and transferred from the hard drive to a buffer in 
memory. Depending on how the file was *encoded*, this memory location is 
then used to access the data, either as simple binary contents (for 
binary files) or to perform some more complicated type conversions (for 
text files). Reading a single line of data (whichever way you want to 
define *line* in this context) hence requires a complex succession of 
instructions that copy from hard drive to memory and between different 
locations in memory. The details of this are pretty much hidden from you 
as a user or code developer, but you can be sure that any file reading 
or writing routine makes use of some intermediary memory buffers where 
data is stored after it is read or before it is written.

The idea of memory mapping is to formalise this idea of intermediate 
buffers and make these buffers directly accessible to the program, 
rather than hiding them and only providing some auxiliary functions that 
expose them. A reasonably large part of a file is mapped *directly* from 
the hard drive to the memory, and the corresponding buffer in memory is 
made directly accessible to the program and can be manipulated as any 
other memory array.

This has a few advantages. First of all, the direct access to the memory 
buffer means you can read and write data more efficiently and even in 
parallel, since you are just accessing memory. But there is more. When a 
file is memory-mapped, then the operating system (or more specifically, 
the *kernel*) becomes responsible for making sure the data in the file 
on the hard drive and the data in the memory buffer is the same. This 
means that the operating system can choose when to access the hard 
drive, which does not necessarily need to happen right away, but can 
happen at a time when the system has nothing better to do. This can 
alleviate common input/output bottlenecks in programs, when the program 
is just waiting for the hard drive to finish reading or writing data. 
And it reduces the load on the hard drive by spreading hard drive access 
operations better in time. Finally, the operating system will keep track 
of files that are memory-mapped; if two separate programs memory-map the 
same file, then they will effectively share the same memory buffer. This 
means that programs can exchange data before it is even written to disk!

# That sounds great, how do I do this?

The reason not everyone uses memory mapping for all input and output 
operations is that it is quite technical. And as with any input/output 
functionality, the exact details of how to memory map a file depend on 
the programming language you are using. Unsurprisingly, Python provides 
the most straightforward way to do this.

## Python

NumPy has a straightforward memory mapping function called `memmap`. 
This function takes the name of the file as input and makes the memory 
buffer accessible as a regular NumPy array. In order for it to know 
which type the elements of the array should have, there is an additional 
argument called `dtype` (the default is a 64-bit integer, which is very 
likely not what you want). Other arguments are the type of access you 
want (read, write or both), the offset of the data you want to read in 
the file, and the desired shape of the array. An example:

```
import numpy as np

d = np.memmap("test.dat", dtype = 'd', mode = 'w+', shape=(5))
d[0] = 1.
d[1] = 4.
d[2] = 5.
d[3] = 1.
d[4] = 40.
```

This will create a binary file containing 5 double precision numbers. 
Overall, this is all very straightforward.

There is one additional feature of `np.memmap` that is worth noting. 
Since memory mapping effectively creates a memory buffer from the 
contents of the file, it is a quick way to create an array based on a 
file. In normal use however, changing something in this memory buffer 
will also change the file. So what if you want to read the data and 
manipulate it inside your script, but still want to preserve the old 
data? To support exactly this behaviour, `np.memmap` supports a third 
access mode, apart from `r(+)` and `w+`: `c`. This mode will memory map 
the file so that its contents ends up in memory, but as soon as you 
change something to the memory buffer, it will copy that part of the 
buffer to another memory location without affecting the original memory 
buffer, a so called *copy-on-write*. This means that changes you make to 
the memory mapped buffer will only affect the memory buffer and will not 
affect the underlying file.

## C(++)

Since C and C++ are much more low-level languages than Python, memory 
mapping a file in these languages is a bit trickier. Luckily (for us) 
they both use the same syntax. Which essentially means that this feature 
only really exists in C, and that the C++ version uses the same C API.

Before I can explain how to do the memory mapping in C(++), it is 
important to introduce a concept called *page size*. The page size is 
the size (in bytes) of a single memory block used by the kernel. When 
the kernel allocates memory, the amount of memory that is allocated will 
always be a multiple of this page size, so the page size is basically a 
memory size unit for the kernel. Unfortunately, the page size is system 
and kernel dependent, although the most common value is $$4,096$$, or 
$$2^{12}$$ (4 kB). The kernel provides functionality to determine the 
page size.

When creating a memory map, the offset in the file and size of the 
memory mapped buffer have to use kernel units, which means they are 
required to be multiples of the page size. If you use memory mapping to 
write a file, then this file will also need to have a size that is a 
multiple of the page size, but it is always possible to reduce it to its 
smaller actual size when you are done with it.

Since memory mapping uses a C API, it requires files created using a C 
API too. For both reading and writing, we use the `open()` function from 
`fcntl.h` (part of the POSIX library, which means this only works on 
UNIX systems):

```
#include <fcntl.h>
...

// reading
int file_read = open("test.dat", O_RDONLY, 0);
// writing
int file_write = open("test.dat", O_CREAT | O_RDWR,
                      S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP);
```

The syntax is clearly very low-level. The third argument in the second 
function call denotes the file permissions created for the new file. In 
this case, read and write permission is given to both the current user 
and the group the user belongs to.

Once a *file descriptor* has been created like this, it can be used in 
the memory mapping functions. Before we can write anything however (so 
only if we want to write), we need to make sure the file has at least 
the size that we want for our memory mapped buffer. If we do not do 
this, then writing to the file could potentially fail because of a lack 
of disk space. To do this, we use the function `posix_fallocate`:

```
#include <fcntl.h>
...

// file was created above
posix_fallocate(file_write, 0, 4096);
```

In this case, we make sure all bytes from 0 (the offset) to 4096 (the 
length, here a single page size on a typical UNIX system) are available. 
The function will return zero if this allocation was successful, after 
which we are good to go. The same function can be used later to increase 
the size of the file to another 4096 bytes, by replacing 0 with the end 
of the current file.

Once the file is ready, we can finally start memory mapping. This is 
done using the function `mmap` which is part of the POSIX header 
`sys/mman.h`:

```
#include <sys/mman.h>
...

// file was opened or created/allocated here

char *buffer = reinterpret_cast<char*>(
  mmap(NULL, 4096, PROT_WRITE, MAP_SHARED, file_write, 0));
```

In the example here, we create a memory map for writing. The map is 
allocated at a memory location chosen by the kernel (`NULL`), has a size 
of 4096 bytes (1 page size), has write access (`PROT_WRITE`) and is 
shared between all processes (`MAP_SHARED`). The map is created on top 
of the file descriptor `file_write` that we created and allocated before 
and starts at offset 0 bytes within that file.

If the memory mapping was successful, this function will return a 
pointer to the start of the memory buffer that contains the file 
contents. If not, it will return `MAP_FAILED`, a constant that can be 
checked.

That's it! The memory buffer `buffer` can now be accessed as any other 
buffer. It could be cast to a type pointer so that it can be used as an 
ordinary array, or it can be used as a file buffer by copying data into 
it using `memcpy`.

When you are done with the memory buffer, you can unmap it using 
`munmap`, which simply takes the pointer to the start of the buffer and 
the size of the mapping as arguments. Strictly speaking, it can also be 
used to unmap *part* of an existing mapping, by providing the start 
address of the region that needs to be unmapped and an arbitrary length 
as arguments. The region will also be unmapped if the program finishes, 
but not if you close the file descriptor using `close()`.

There are two additional technicalities worth mentioning. The first one 
concerns the actual file writing. As explained before, the operating 
system kernel is responsible for making sure that the data in the memory 
buffer and the data in the file are the same. It is however not 
specified *when* the kernel needs to do this. The rationale behind this 
is that the contents of the file on disk does not really matter for the 
program that memory mapped the file, since it will only access the file 
contents using the memory buffer, and this is always up to date. The 
same goes for other processes that memory map the same file; they also 
read the same memory buffer.

It can however sometimes be useful to force the kernel to write to disk. 
For this purpose, the `msync` function is provided. Its arguments are 
similar to those for `munmap`: a pointer to the start of a memory buffer 
and its length (this is the part of the buffer that will be written to 
disk), and an additional flag that signals when to do the write. The 
latter can have the values `MS_SYNC` (write now and wait until the write 
is done) or `MS_ASYNC` (start writing in the background). Note that 
`munmap` on the same memory region will behave exactly as `MS_SYNC`, 
except that after `munmap` the corresponding region is no longer memory 
mapped.

The second technicality concerns the final file size when using memory 
mapping to write files. As mentioned before, all memory mapped files 
need to be created with a size that is a multiple of the page size, 
simply because that is the size unit used by the kernel for memory 
management. The hard disk does not have this restriction, so you can 
save hard disk space by shrinking the file to its actual size once the 
memory has been unmapped. This can be done using `ftruncate()`. This 
function takes the file descriptor and the desired file size as an 
argument. Be sure to provide the right size, since it will invalidate 
everything in the file beyond the given size!

# Can I see some examples of this?

Yes, you can. I have used memory mapping for a number of projects:
 - As parallel output mechanism for binary files in the [prototype 
version of CMacIonize 
2.0](https://github.com/bwvdnbro/ParallelCMacIonize/blob/master/MemoryMap.hpp)
 - As mechanism to write almost continuous output in a [1D spherically 
symmetric hydrodynamics 
solver](https://github.com/bwvdnbro/HydroCodeSpherical1D/blob/master/LogFile.hpp). 
The same repository also contains [a few example Python scripts that use 
memory mapping to read the continuous 
output](https://github.com/bwvdnbro/HydroCodeSpherical1D/tree/master/paper_workflows)

Most of my knowledge about memory mapping comes from the almost 
continuous output (called the *log file*) that my collaborators have 
been developing for the simulation code 
[SWIFT](https://github.com/SWIFTSIM/swiftsim/blob/master/src/logger.c).
