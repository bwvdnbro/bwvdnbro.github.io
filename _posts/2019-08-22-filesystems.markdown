---
layout: post
title: "File systems"
description: A short rant about file systems.
date: 2019-08-22
author: Bert Vandenbroucke
tags: 
  - Tools
---

As a very frequent user of computer systems, there are a lot of things 
about a computer that I simply take for granted. I do care a lot about 
how fast my CPU(s) execute my software, but I never question the fact 
that what they do is correct. I care a lot about how much memory is 
available, but I never question that what is stored in memory is stored 
correctly (and doesn't for example change while it's being stored).

This week, I was forced to think a bit more about another part of my 
computer I simply took for granted: the file system. In the end, I ended 
up reading a bit about file systems, and had to do something not quite 
standard to end up with the result I desired. A short story of my 
experience.

# Context

As it happens, I am currently serving my last weeks as a postdoc at the 
University of St Andrews, and I will soon move on to another postdoc in 
Ghent (Belgium; also written as Gent in the native Flemish language). I 
could write a very politically motivated post about this move, but I'll 
save that for a time when I really don't have anything else to write 
about.

The fact I am moving is however relevant to the story, as it means that 
I am currently in the process of consolidating some of the work I have 
done in my current position. As things go in academia, you never really 
move *out* of a job, but rather just move from one place to another and 
take on additional projects. As such, I really want to make sure that I 
still have access to the data and files I am currently working with two 
weeks from now, so that I can still finish all those papers that I 
promised my boss (and myself) I would write.

So long story short: a few weeks ago I bought myself an external hard 
drive, and this week I wanted to move some of my work files to this 
drive to make sure I can access them in my new job. Unfortunately, I 
have quite a lot of files on my current computer (and on some other 
network drives scattered around the university), totalling a few TB 
worth of data. So the hard drive I bought was quite big: 4 TB. All 
connected to a computer of choice with a single USB cable (USB 3, 
although it turns out all the computers I have only go up to USB 2).

The hard drive came (as those things go) completely ready to be used 
with Windows, and shipped with a file system called exFAT. Unfortunately, 
I don't use Windows unless I really have to, and my Linux machine wasn't 
able to read this file system. So I quickly reformatted the drive to 
NTFS (*to use with both Windows and Linux systems*, as the Linux 
formatting instructions nicely explained), and thought I was done with 
it. Until I started really using it earlier this week.

Limited by the USB 2 connection, I noticed that the writing speed to the 
hard drive was limited to about 25 MB/s. Knowing that I have to copy 
about 3 TB of data, that translates into about 35 hours of copying. 
Slow, but not really unexpected, and also the reason I started doing 
this early enough. So far so good.

After a full day of copying files (roughly 7 hours), I had managed to 
copy about 500 GB. A bit slower than expected, but I did have a lot of 
small files to copy and for those you copy at slightly below the average 
speed. So nothing to worry about. I left the hard drive to do its thing 
over night, fully expecting to reach at least 1 TB by the next morning.

I was a bit surprised then when I arrived back the next morning and 
noticed the progress over night had been quite slow: only 200 GB had 
been copied during the whole night. According to the `rsync` output, 
average writing speed was down to about 2 MB/s, and I was even copying 
large files! When I interrupted the `rsync` command to see if things 
would speed up after restarting it, things took even longer. And when I 
tried to unmount the drive to try again, it didn't even want to mount 
any more!

Checking the disk with the *Disks* tool revealed that somehow the 
file system was corrupted. The same tool allowed me to repair the 
corruptions, but this did not improve the speed. What had happened?

# File systems and partitions

Before I can explain my explanation for the weird and annoying behaviour 
I encountered, I need to explain something about how hard drives work. A 
hard drive itself is a clearly defined thing: it is either a type of 
very densely packed magnetic (?) disk with a needle that reads and 
writes data to it, or a flash-drive type solid state disk that uses 
quantum tunnelling to trap electrons in specific configurations to 
represent bits (much faster than the needle version, but a lot more 
expensive and usually smaller in size). What happens on the software 
side with a hard drive is somewhat more complicated.

First of all, there are *partitions*. A partition is in essence a large 
chunk of memory on the hard drive that presents itself as a single 
storage space to the operating system. It has a specific size and 
location on the hard drive, and the operating system can write data to 
it, simply by requesting a specific address in that partition. When the 
partition is used in this way, we call it *swap* space; it is the hard 
drive equivalent of RAM memory. A single drive can contain many 
partitions.

While partitions are useful for the kernel, they are not really useful 
to store files in an organised way. The reason for this is simple: when 
you store a file on a hard drive, you usually intend to use and read that 
file again at a later point, which means you need to find it again. If 
the file was only stored on the hard drive (meaning its individual bits 
were written to it, but nothing else), then it would be impossible to 
find again later, unless you somehow know where on the hard drive those 
individual bits are stored.

Of course, you as a user are not going to store that specific 
information. Instead, you expect the operating system itself to do this. 
A file system is the tool that the operating system uses to achieve 
exactly this. In essence, it is a piece of software that acts as an 
intermediary between the operating system and a partition on a hard 
drive that stores the exact location on the partition of a specific file 
in some kind of easily accessible database, along with additional 
*metadata* about the file that help retrieve it later on. This metadata 
can be a lot of things, but it is at least a *file name*; a unique label 
attached to that specific file that allows you to refer to it later.

Obviously, there are many ways of achieving this. One way is to reserve 
a part of the storage space within the partition to store a database 
table containing all the files stored on the system and the locations on 
the partition where they are stored. This is how the FAT file system 
works, which was used on many old Windows computers. This type of 
file system is quite restricted: the fixed size of the database table 
puts severe limits on the number of files that can be stored on the 
system (as well as the total size a single partition can have), and on 
how easy it is to use the file system in a parallel context (multiple CPU 
cores trying to write to almost the same part of the disk at the same 
time is not a very good idea). On top of that, it is very susceptible to 
data corruption: if somehow a bit in the database table gets corrupted, 
this can cause entire files to go missing, despite the files themselves 
still being completely intact (but what good is that if you don't know 
where the intact file is stored?).

As hard drives got bigger (at quite a significant rate), these size 
restrictions and relative vulnerability to corruptions prompted the 
development of more sophisticated file systems, the previously mentioned 
NTFS being one of them. Different file systems can have very different 
ways of storing the information required to manage files, and vary 
immensely in writing/reading speed, how vulnerable they are to 
corruptions and how their performance scales with increasing hard drive 
size.

# Large hard drives

And exactly that is what was causing the troubles I encountered. The 
NTFS file system I used for my new hard drive without really thinking 
about it works very well for hard drives with sizes of the order of 100 
GB, like the ones I always used in the past. It also works reasonably 
well if you slowly fill up the hard drive, and regularly *defragment* 
the drive, i.e. you rearrange the storage space so that files that were 
spread out across the drive when they were originally written, are put 
in more sensible locations on the drive, making sure the leftover space 
is organised in a more continuous fashion that allows for faster 
writing.

However, when you use it to quickly fill up a TB size drive, you quickly 
run into the limitations of the NTFS system: as more of the space gets 
used up, the leftover available space gets less and less continuous, 
making it increasingly harder to write and read files. This problem gets 
especially bad if you try to write *a lot* of small files to the drive, 
as they lead to a higher degree of file system fragmentation.

The solution was hence simple: I had to stop using the NTFS system, and 
switch to a system that does support larger hard drives. My desktop at 
work very happily uses a 2 TB hard drive with the native Linux ext4 
system, so that seemed like an obvious choice.

However, turns out that much of the power of the ext4 system comes from 
the fact that its original layout is already quite organised. Or in 
other words: while writing/reading to an existing ext4 file system is 
relatively fast and not susceptible to fragmentation (meaning it will 
still work for large drives and if you have a lot of small files), 
creating the system is not, especially not over a slow USB 2 connection. 
I tried multiples times to reformat my hard drive into ext4, and 
eventually had to give up because it took too long.

After reading up on the problem for a while, I eventually ended up 
creating a new BTRFS (*better file system*) file system on the disk. This 
system was not natively supported on my computer (I had to install an 
additional package), but its creation was a matter of seconds, rather 
than the - probably - days it would have taken to create the ext4 
system. The writing speed turns out to be about the same as it was 
before (20 MB/s), but after writing 1 TB of data already, this speed is 
still the same. Better still, reading the hard drive seems to work 
faster than when it was an NTFS file system. So it looks like I might 
finish copying my files by the end of this week after all.

Conclusion: it is very easy to take your hard drive and its performance 
for granted. Turns out there is a lot of hidden software behind the 
scenes that can have a very significant impact on this, and once you 
start really using a hard drive, it really matters what this software 
does.
