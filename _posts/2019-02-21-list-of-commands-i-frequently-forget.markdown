---
layout: post
title: "List of commands I frequently forget"
date: 2019-02-21
description: >-
  A list of Linux commands whose syntax I always forget, despite using them
  all the time.
image: https://imgs.xkcd.com/comics/tar.png
author: Bert Vandenbroucke
tags: 
  - Tools
---

One of the reasons I like the large amount of computational blog posts 
available on the world web is because of all the useful information they 
contain. Especially when it concerns very trivial tasks, there is bound 
to be a blog or forum post that covers that very topic and gives you the 
exact command you need to use to do whatever you want to do. Or that 
gives you all the necessary information to figure out the right command 
yourself.

Unfortunately, those blog and forum posts are hardly a complete, 
well-maintained set. Google does its best to tune its results to your 
taste (whether you like it or not), but this does not guarantee that you 
will always be able to retrace that magical post that made you see the 
light. And even if you do manage to find it, you will probably spend a 
lot of time and resources doing it.

For this reason, I decided to create this post that gathers a nice 
collection of commands that I tend forget (no matter how often I use 
them), and that, time and time again, make me try to find that one post 
that I need to tell me what to do. Again. Maybe this list will be useful 
for someone some day in the same way other people's posts have been 
useful for me. It will definitely be useful for me, as a personal 
reference.

# The classic: `ln`

The command that by far fits the description given above best is `ln`. 
`ln` is a GNU Linux command line tool that helps you create links in 
your file system. Suppose e.g. that you have a bunch of files stored in 
one folder, and you want to use one of these files as the input for 
something else. You could
 * move the file to the folder where you plan to do that something else, 
but then you would separate the file from its friends,
 * copy the file to that folder, but that will require more disk space 
(and what if the file is very large?),
 * link to the file, using minimal disk space and still ensuring the 
file is seemingly present in both folders

`ln` has many options, of which I only use one (and I always use it): 
`-s`, which creates a *soft link* (to be fair, I never even bothered 
understanding the difference between a *soft* and a *hard* link). The 
correct syntax is

```
ln -s TARGET LINK_NAME
```

But I always forget the order of `TARGET` and `LINK_NAME`. Not any more!

# The useful: `rsync`

`rsync` is the limousine of remote (and local) file transfer. Originally 
meant as a tool for creating backups, `rsync` allows you to copy a large 
number of (large) files from one location to another, using `ssh` in the 
background. It is a lot more powerful than `scp`, and also a lot safer 
(I refuse to use `scp` since it once mixed up the filenames for the 
snapshots of a simulation project I was transferring).

Usually, I run `rsync` like this:

```
rsync -rvth --progress ORIGINAL DESTINATION
```

For this one, I actually manage to remember the order. But don't ask me 
what all the flags do; I forgot that a long time ago. This command will 
copy all the files pointed to by `ORIGINAL` to `DESTINATION`. If 
`ORIGINAL` is a folder, it will copy the entire folder, including all 
subfolders and the folder itself. If `DESTINATION` is a folder, it will 
put everything in that folder.

What if you only want to copy a select number of files? You can get 
pretty far with the usual shell wildcards, but sometimes wildcards just 
don't cover it (or lead to commands that are just too long to be 
practical). Fortunately, `rsync` has builtin support for excluding and 
including files. Unfortunately, I always forget how this works. So here 
is an overview.

Suppose you want to exclude specific files. In this case you can add one 
or more `--exclude` patterns:

```
rsync --exclude "file.txt" --exclude "*.log" ORIGINAL DESTINATION
```

If the list is quite extensive, you can store it in a file and pass it 
on to the `--exclude-from` option.

Only including specific files is a bit more tricky (assuming a case 
where the files you want to include are not all in the same folder, 
because then a shell wildcard is sufficient). `rsync` has an `--include` 
option, but that option only works to negate an already present 
`--exclude` option. Additionally, `rsync` is quite picky about the order 
of these options. Long story short, this is the command to only include 
files that match a given pattern, including files in subfolders:

```
rsync --include "*/" --include "*.txt" --exclude "*" ORIGINAL DESTINATION
```

# Making movies

Making movies is an essential part of being a numerical astrophysicist. 
Movies are what sets us apart from other astronomers, who study static 
pictures and need to guess what is physically happening over the typical 
very long astronomical time scales. Despite this, I don't find myself 
making movies often enough to remember the relevant commands. On top of 
that, movie making software seems to evolve in between Linux versions 
and contains a whole list of options, most of which I don't understand. 
And then there are the additional options necessary to make the movie 
work on OSX...

So here is the command to convert all movie frames (`snapshot0000.png` 
to `snapshot1000.png`) into a single `.mp4` movie with a frame rate of 20 
frames per second (the `\` is only there to properly break the line in a 
shell):

```
ffmpeg -framerate 20 -f image2 -i snapshot%04d.png -c:v h264 -crf 1 \
  -pix_fmt yuv420p movie.mp4
```

# Trimming images

Another thing astrophysicists like me do a lot is making images. And 
then using these images in journal articles, presentations, grant 
proposals... Usually these images are made with plotting software 
(Python in my case) and look reasonably okay. Sometimes some additional 
image manipulation is required to make them suited for their purpose. 
For this, there is a powerful tool called 
[ImageMagick](https://www.imagemagick.org/). ImageMagick ships with a 
number of commands, the most important of which are `convert` and 
`mogrify`. These two commands are basically the same, except that the 
former takes one or multiple images as input and then generates a new 
image, while the latter takes a single image as input and then modifies 
it in-place.

ImageMagick commands are usually either very trivial:

```
convert image.something image.something_else
```

can convert pretty much any image format into pretty much any other 
image format, just based on extension. Or they are very complex (no 
example here, as I can't think of one of the top of my head).

The command to *trim* an image is an exception. *Trimming* is the 
operation whereby you remove white space surrounding an image, by 
shrinking the *canvas* of the image to the bit of the image that is 
interesting. Note that white space does not necessarily have to be white 
here. Trimming is very useful if your image has a very generous border 
you want to get rid of (e.g. because you really want to fit this image 
into your 1 page limited grant proposal).

ImageMagick has a `-trim` operator, but by default this does not do the 
trimming as you would expect: it still stores the original size of the 
image in the image header (for image formats that have a header). This 
can cause confusion when using the image later on for complicated 
operations. To fix this, you have to tell ImageMagick to also *repage* 
the image, so that it resets the header to the actual image size.

The command to trim an image then is either

```
convert image.png -trim +repage image_trimmed.png
```

or

```
mogrify -trim +repage image.png
```

# Another classic: `tar`

This particular command is known to be a problem, as already illustrated 
in the cover image for this post ([source](https://xkcd.com/1168/)):

![relevant xkcd](https://imgs.xkcd.com/comics/tar.png)

`tar` is a powerful combination and compression tool that can be used to 
create uncompressed (`.tar`) and compressed (`.tar.gz` or `.tar.bz`) 
archives containing a (usually large) number of files, in order to 
easily transfer these files together. The problem with `tar` is that its 
behaviour is completely dependent on the command line options you 
provide; this is what causes much of the confusion.

Suppose you have a compressed archive, `archive.tar.gz`. The following 
command lists the contents of the archive (option `-t`) (without 
extracting it):

```
tar -tzf archive.tar.gz
```

To extract the archive, use (option `-x`):

```
tar -xzvf archive.tar.gz
```

To create an archive from all files (and folders) in the current 
directory, use (option `-c`):

```
tar -czvf archive.tar.gz *
```

This command has the annoying side effect that it will extract the files 
into the current working directory when extracting the archive again 
(`tar` never creates new folders; it can unpack folders that are part of 
the archive). To create an archive that has a folder at its root, so 
that it extracts into its own folder, put all the files in the folder 
first, and then create the archive like this:

```
tar -czvf archive.tar.gz folder
```

Note that in all these commands, the `-f` option specifies the name of 
the archive. This is why it should immediately precede the archive name 
(the order of all other flags is arbitrary).

# The last one: `find`

Finding files in Linux is less straightforward than I think it should 
be. First of all, there is the easy to use `locate`:

```
locate bit_of_filename
```

Which will present you with a list of all paths in the system that 
contain `bit_of_filename`. There are two issues with this command:
 1. it literally returns results for the whole file system, so you might 
get a lot of results that you don't need.
 2. since searching the whole system would be impractical, `locate` 
depends on a database of file locations that is occasionally updated. 
New files (which are unfortunately often the files you want to find) are 
likely not to be in that database (yet). You can manually force a 
database update with `sudo updatedb`, but this (a) takes a while, and 
(b) requires root privileges.

`find` provides a more powerful alternative for finding files, as you 
can control where it looks for files. Unfortunately, its syntax is a bit
less straightforward. To find a file in a specific folder or all subfolders,
use

```
find folder -name "part_of_filename"
```

The folder is optional (defaults to the current working directory), the 
`-name` bit is not: if you omit it, `find` will just print the entire 
contents of the folder. The `-name` option accepts the regular 
wildcards, so once you know this syntax, file finding becomes a lot 
easier.
