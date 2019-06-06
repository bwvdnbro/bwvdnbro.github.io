---
layout: post
title: "Using a large scale cluster"
description: Some thoughts about large scale clusters and how to use them.
date: 2019-06-06
author: Bert Vandenbroucke
tags:
  - Scientific computing
---

As with some of my previous posts, this week's post is HPC cluster 
themed. I thought it would be a good idea to share some of my (limited) 
experience with using large HPC clusters, so that other people could 
benefit from it.

# What is a large scale cluster?

When I use the term *large scale cluster*, I mean a shared HPC resource 
that needs to be accessed remotely, runs jobs using a scheduler and a 
queue, and requires some sort of application process before you can use 
it. So generally speaking: a machine on which computation time is 
valuable and you have to compete with many other users to get it. Or in 
other words, a machine on which you only want to run *good* 
computations: jobs that do something useful (i.e. the output can be used 
for scientific publications), use the machine efficiently, and - above 
all - do not crash. Or at least, that is the idea I always had, and also 
what I think we should strive for.

The exact layout of a large cluster depends a bit on the cluster, but 
there are some general trends. A cluster will always consist of many 
different computers (*nodes*) that are interlinked using a fast network. 
Most of these nodes are not directly accessible to users, and jobs can 
only be started on these nodes by the scheduler. Users can log in to one 
or a few dedicated *login nodes*. These nodes are not used to run jobs, 
and users can use them to submit their jobs and do all the small tasks 
that go with that (e.g. creating directory structures, moving small 
files, configuring and setting up software...). They are hence the way 
the cluster presents itself to the outside world.

Depending on the cluster, data storage is provided on the individual 
nodes or on a central data server. Clusters usually have three levels of 
data storage:
 * user space (usually called *home*) that can be used by individual 
users to store software, configuration files... The volume of this space 
is usually very limited (a few GB), and persistent: it is linked to the 
user account on the machine rather than any specific allocation assigned 
to that user.
 * scratch space that is not (!) backed up, but that is very fast. Jobs 
running on the nodes can (and should) use this storage for all their 
intermediate files and large output, as writing output is orders of 
magnitude slower than running computations or moving data within memory, 
and scratch space guarantees the most efficient use of the file system. 
Scratch space is usually very large (at least of the order of the total 
memory size of the system), but should not be used for anything that 
requires backing up. Scratch space is usually linked to a specific node 
and/or a specific allocation on the machine.
 * data space that should be used for persistent data, i.e. data that 
needs to be stored securely for some amount of time. This storage is 
usually slower to access and should therefore only be used for important 
data. The total volume of data storage is usually very big, and is also 
linked to a specific allocation.

Really advanced systems can provide additional levels of data storage 
that are larger and slower (because of the use of tape storage), and 
that should be used for data that needs to be stored but is not used 
very often.

Clusters can be very uniform (one type of node), but can also consist of 
different types of node, e.g. a number of nodes with little memory and a 
fast network connection, a number of nodes with large memory and a large 
number of cores, a number of nodes with GPUs... Different nodes are 
usually accessed through different queues in the scheduler. They can 
have separate scratch file systems, but should all share the same user 
and data space.

# How do I use a large scale cluster?

The outline above makes using a large scale cluster sound a lot scarier 
than it is. Which in a way is a good thing, as it makes you more careful 
about what you do on a cluster. But it could also be a bad thing, as 
many people treat clusters as a black box and by doing this actually use 
them in a very inefficient and bad way, by for example running jobs with 
the wrong hardware specifications or by being scared of submitting jobs 
and running large computations on the login nodes.

The first thing to know when you are using a cluster, is that it is just 
a computer, and is in many ways the same as your laptop or desktop. The 
main difference between a cluster and a laptop/desktop is the *remote 
access* aspect I mentioned before: while your laptop is on your lap and 
your desktop on your desk, the cluster will be housed in some dedicated 
building (probably provided with a state-of-the-art cooling system) that 
is not close to you. Which means that the only way to access the cluster 
is through a network connection. As a result, the default way in which 
you use a cluster might be a bit different from what you are used to: 
while every laptop and desktop by default runs a graphical environment, 
clusters by default present themselves as a basic, low-level login shell 
to the user.

This difference is however purely in the default usage: you can use your 
laptop/desktop using text-based shells (in fact, I do that most of the 
time). And you can run a remote graphical environment on the login node 
of a cluster through the network. This of course requires some kind of 
graphical environment to be installed on the cluster. When you realise 
these possibilities (and maybe try them out), it is easier to convince 
yourself that the cluster is just another computer.

Just as for any other computer, what you can do on the cluster depends 
on the available software. The operating system on your laptop or 
desktop ships with a lot of software from the start, and you can install 
a lot more later on. After using it for some time, it will probably 
contain a wealth of different tools and libraries that are interlinked 
in various ways, and that are quite specific to how you use your 
computer. Since a cluster is used by many different users, it needs to 
provide a larger variety of tools and libraries, and therefore some kind 
of software management system is required. This is called a *module 
system*.

A module system is in essence a large database of all available software 
and libraries on the cluster, and their mutual dependencies. Each entry 
in the database (called a *module*) will contain information about where 
a specific tool is located in the file system, what other libraries are 
required to run it, and what system settings need to be changed in order 
to make jobs use this tool. It will also contain information about which 
modules are mutually exclusive, e.g. modules that contain different 
versions of the same library. A module needs to be *loaded* before it 
can be used, by giving a specific instruction (e.g. `module load`) to 
the module system.

The use of a module system makes the process of installing new software 
a bit more involved than it is on a personal computer, as it requires 
setting up the right dependencies and making sure the new entry in the 
database is correct and up to date. This task is therefore usually 
delegated to dedicated specialists (system administrators). But once 
installed, using software is pretty much the same as on a normal 
computer: once loaded, you can simply run commands in the remote shell 
(or in the graphical environment if you prefer), and they will still do 
the same thing and generate output in the usual way.

Large computations that require a lot of resources need to be managed, 
so that many users can use the system simultaneously in an efficient 
way. This is why they are managed by a scheduler. But the scheduler 
itself is pretty much just a manager: it decides when and where a job 
can run, but that is it. When the scheduler decides it is time to run 
your job, it will simply hand over full control of the requested 
resources to your job script, and your job script can do whatever it 
wants with those resources (within the limits of the system of course). 
There is very little magic in this: if you job script tells the system 
to move to a specific directory in the cluster file system, it will do 
so. If it tells the module system to load specific software, it will do 
so. If it tells the system to execute a specific command in parallel 
using all the available resources, it will also do so. The scheduler 
will not interfere with any of this, but will also not help with any of 
this. It will start your job in a specific point within the cluster file 
system (the *working directory*) and will store the usual output of your 
job into a specific file, but apart from that, it will do nothing. So if your
job script runs fine outside the scheduler, it will do so when run by the
scheduler.

Once the job script is done, it will notify the scheduler so that 
another job can use the resources. While the job runs, the scheduler 
will keep track of how long the job runs, and if it exceeds some 
threshold, it will kill the job. These thresholds are called *time 
limits* and they are absolutely essential for efficient usage of a 
cluster. There are a few reasons for this. First of all, time limits 
make sure that resources are not used indefinitely by a select number of 
users, by periodically freeing them up automatically. This guarantees 
that even small jobs with a low priority eventually get run within a 
reasonable time frame. Secondly, they are essential for maintenance of 
the cluster. When the system administrators want to install new software 
or fix hardware issues, they can simply drain the queue (not accept any 
new jobs that require more time than some limit) and wait for the 
longest running jobs to finish. With reasonable time limits, this 
imposes little overhead for the users when a well-planned maintenance is 
carried out. Lastly, time limits provide a good incentive for users to 
make sure that their jobs generate regular output. If a job gets 
interrupted for technical reasons, or a scratch drive (not backed up!) 
is lost, then the impact of this event stays limited to the time limit 
for a single job.

# Cluster etiquette

I hope it is now clear that clusters are just like other computers, and 
that using them is as easy as using your desktop from a command line 
window, provided that you know how to use the module system. Running 
large jobs is pretty much the same as running a script that 
automatically does the things you would do manually. The scheduler and 
its limits are only there to make sure the system is used efficiently. 
Despite them being so ordinary, there are however a few things to keep 
in mind when using clusters.

## Login node etiquette

This was literally the subject of an email I received earlier this week 
from the administrator of a DiRAC cluster. It concerns - as might be 
clear - the usage of the login nodes. As I outlined above, the login 
nodes are a set of dedicated nodes that are used by the cluster users to 
perform basic tasks and submit large jobs. As such, they are shared by 
all users of the cluster, and are not managed by the scheduler. This 
means that there is no real control over what users do on the login 
nodes.

As a general rule, users should not perform computationally intensive 
tasks on the login nodes, as this might affect performance for other 
users and in extreme cases could even hamper the scheduler. Different 
clusters however have different interpretations of the term 
*computationally intensive*. And different users might also have 
different interpretations of the specific interpretation of a cluster. I 
will not go into too much detail, but I will instead explain why people 
might want to use login nodes for things that might be too 
computationally intensive, and why there are better ways to do these.

One very good reason for wanting to run things on the login node is 
control. When you send a job to the scheduler, it might run immediately, 
but it might also run half a day later (or even later). When you know 
your job will work fine, and you are not particularly curious about the 
results, that is acceptable. If you have some doubts about whether or 
not your job script works (because you did not test it properly), or you 
are not sure that your software will run smoothly (because you are 
debugging), then this potential delay can seriously hamper your work. In 
this case, direct control has two benefits: it allows you to make small 
changes to what you want to do based on the output of what you already 
did (control over *input*), and it allows you to add additional 
instructions to get more value out of what you did (control over 
*output*). Exactly the same kind of control you have on your personal 
computer.

Another potential reason for using the login node is fear of the 
scheduler. The documentation of any scheduler will tell you that the 
scheduler makes decisions about what job to schedule next based on a 
variety of criteria, including past usage of the cluster. No cluster 
will tell you exactly what the latter phrase means, so you might be 
worried about what happens to your ranking when you submit a lot of 
small jobs to the scheduler. On top of that, the scheduler will count 
the time you use in scheduled jobs and might subtract it from an 
allocation. Better be careful about what you submit!

Good clusters usually address both these issues. They might provide you 
with a separate queue for small jobs that is managed separately from the 
main queue. And they might allow you to run jobs in *interactive mode*. 
The latter means that you submit a job request to the scheduler as 
usual, but instead of running a script when the request is awarded, the 
scheduler will instead open a login shell in which you can work 
interactively.

Small job queues are very useful to test job scripts or to debug 
software, as they are usually faster. They can also be useful to run 
jobs that require less resources, but should nonetheless not be run on 
the login nodes, like analysis scripts. Interactive jobs are very useful 
for compiling software, especially on inhomogeneous clusters where the 
login node does not necessarily have the same hardware as the nodes you 
want to run your jobs on. And they can be useful to run large live 
analysis tools that use a lot of memory and resources (like Jupyter 
notebooks).

## Data transfers

Another issue with the usage of the login nodes is network usage. Since 
the login node is usually the only access port users have to the 
cluster, all data traffic (including file transfers) has to go through 
them. When transferring large data files from the cluster to your own 
machine (or another file server), you put a lot of strain on the 
network, which can affect other users. It is therefore worth considering 
whether you really need to transfer the files from the system, as it is 
sometimes a lot easier to just run the analysis you want to perform on 
the cluster, and only transfer the small data products. It is also worth 
checking if it is possible to *compress* your data before you transfer 
it; output generated by your jobs is probably optimised for writing 
speed, and that might lead to output that is not optimised for file 
transfer. Again, it might be easier to run a compression script on the 
cluster before transferring the data.

Some clusters have dedicated nodes to handle file transfers and that use 
a different network connection from the one used by the login nodes. 
Again, it is worth checking the documentation of the cluster to find out 
what the optimal way is to use it.

## Checkpointing

One of the main consequences of having time limits is that jobs might 
not be able to finish within the allocated time. In this case, the job 
will be aborted by the scheduler in a very ungracious manner (the 
scheduler will literally just kill the job). If you want to make sure 
the job can later be restarted (with as little as possible loss of 
progress, of course), you need to provide a checkpoint: a state of the 
job as it was right before it was killed.

Some clusters might be able to do the checkpointing and restarting for 
you, although this is rare. So in many instances, you will need to make 
sure yourself that your jobs provide regular output that can be used to 
restart them if they suddenly stop. This is not very hard: you simply 
choose a specific point in the execution of your software, and then 
record the state of the memory as it is at that point to a file, bit by 
bit. If you later restore the state of the memory from the file and 
start the software at the exact same point, the job should continue as 
if it never stopped.

Note that choosing a good checkpointing strategy is tricky, as it 
depends on many factors. If your job uses a lot of memory, then writing 
a checkpoint file can be very expensive and you probably want to keep 
the number of checkpoint files small. If your job generates a lot of 
additional output, it is further worth checking if you can restart your 
job from this output and save the trouble of explicitly checkpointing. 
If possible, it is also a good idea to gracefully exit a job some time 
before the scheduler will kill it; in this case you can write a 
checkpoint file, stop the job, maybe automatically resubmit it, and then 
gracefully hand back control to the scheduler. It the queue is very 
quiet, it will almost seem as if the job never stopped.

## Hardware requirements

The last thing to think about when using a cluster is whether or not you 
should actually be using that cluster. Or so much of that cluster. There 
are two reasons why people run jobs on large machines: because it makes 
the jobs run faster (efficiency), or because the jobs do not fit in 
memory on a smaller machine (size). It is quite easy to figure out how 
much resources to use for a given memory requirement. The efficiency 
requirement can be a bit trickier to assess.

Not all problems necessarily get solved faster by throwing more 
computing power at them; in fact the rate at which they do is called 
*strong scaling* and can be measured. Any given software tool or problem 
has some scalability limit. When you use more resources than the limit, 
the job will no longer run faster, or might actually start running 
slower. This makes sense: if your job consists of 1000 tasks and you use 
2000 CPU cores to execute it, 1000 CPU cores will sit around doing 
nothing while the other 1000 work. It is worth knowing what the limit 
for your job is when choosing a number of cores to execute it.

Apart from this hard limit, there is a much more subtle soft limit on 
the number of cores you can use. If your job no longer gets executed any 
faster, not using more resources is a no-brainer. If doubling the number 
of cores makes your job run 10% faster, you still get something, but you 
might consider not doubling the resources, as you might just as well 
spend the same resources to run two jobs and get more done overall. 
While all of this is pretty straightforward, it is surprising how 
unaware most people are of the scaling behaviour of the software they 
use. And judging from the ever stricter technical application forms I 
see for cluster time applications, cluster administrators are more and 
more aware of this fact.
