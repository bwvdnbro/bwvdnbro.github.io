---
layout: post
title: "SSH tunnels"
description: >-
  An introduction to SSH tunnels and how to set them up in a user-friendly way
date: 2019-06-23
author: Bert Vandenbroucke
tags:
  - Tools
---

This week's post continues on the same theme as last week's: SSH. Last 
week I discussed how to set up passwordless SSH, so that connecting over 
SSH to a remote computer becomes as simple as a single `ssh` command. 
This week I will explain how to do the same if your laptop or desktop 
and the remote computer cannot directly talk to each other.

# Tunnels

There are many reasons why two computers might not be able to talk to 
each other. In fact, this is pretty much the default. When you want a 
computer to be accessible to other computers over a network, you have to 
actually configure it to be accessible, and while doing so, you will 
probably think twice about how to make it accessible. You don't usually 
want a computer to be open to the whole world, as this significantly 
increases the risk of it being accessed by people with bad intentions.

As a result, remote computers are usually only open to a specific range 
of computers that are all part of the same network that is shielded from 
the global internet. A computer might for example only accept incoming 
SSH requests from computers within the internal network of your 
institution, or, for an HPC cluster, might only accept incoming 
connections from a specific login node. If you are not on the same 
network or even on the right computer, the remote computer will simply 
refuse to listen to your SSH requests.

This is a very safe setup, but unfortunately also very impractical: if 
you can only access the computer from another computer within the same 
institution, then you have to be in a specific location to do anything 
with it, which sort of defies the point of it being remotely accessible. 
On the other hand: almost all institutional networks are actually 
somehow exposed to the global internet anyway: there will be some 
computers on the internal network that do accept incoming SSH requests 
from outside the network, or, in the case of an HPC cluster, there will 
be login nodes that are open to the world. These computers can act as 
*gateways* to the internal network: once you are connected to one of 
these, you can use their status as members of the internal network to 
connect to other computers on the internal network. This is called 
*tunnelling*.

So the basic idea of SSH tunnelling is very simple: you simply connect to 
a computer you have access to (the gateway computer above), and then 
from that computer connect to the computer you actually want to connect 
to. Since connecting to a computer (when set up right) only requires an 
`ssh` command, and an SSH connection to a remote computer allows you to 
run *any* command on that remote computer, it is perfectly possible to 
run a second `ssh` command from a computer you are connected to, to 
connect to yet another computer. In this case, the first remote computer 
acts as a *tunnel*: it will simply forward all your commands to the 
second remote computer, and will forward the response from the second 
remote computer back to you.

# Hidden tunnels

The problem with the approach above, where you simply run a remote `ssh` 
command to set up the tunnel manually, is that it does not create a 
direct link between your computer and the second remote computer. You 
will be able to run commands and receive their output directly through 
the tunnel, but you will not be able to run file transfer commands (like 
`scp` or `rsync`) directly; if you were to transfer files between remote 
computer 2 and your computer, you would first need to transfer them to 
remote computer 1, and then transfer them from there to your computer. 
This intermediate step means that you need (some) hard drive space on 
remote computer 1 to actually store the files temporarily, and is 
obviously a bit tedious as well.

To overcome this, you can configure SSH so that it automatically sets up 
a tunnel whenever it connects to remote computer 2. In essence, you tell 
SSH that remote computer 2 exists and that, in order to access it, it 
first needs to connect to remote computer 1. To set up this 
configuration, you need to edit the SSH configuration file, by default 
stored in `~/.ssh/config`. If you have never set up any custom 
configuration for SSH, this file will likely not exist. If the `~/.ssh` 
folder does not exist, you need to figure out where that is stored on 
your system. If the file does not exist, you can simply create it, and 
SSH will automatically detect its presence and use its contents (unless 
SSH is configured otherwise on your system, in which case none of this 
will work).

A typical `.ssh/config` file will look like this:

```
Host remote1
  HostName remote1.network.com
  User me42

Host remote2
  HostName remote2.network.com
  User vierzwei
  Port 2222
```

Each `Host` block in the file contains configuration options for a 
specific remote computer. The `HostName` is the IP address of that 
computer or its name on the network, i.e. the address you pass on to the 
`ssh` command. The `User` is the user name on the remote computer: it is 
the name of the user account on that computer you will log in with. 
Finally, `Port` tells SSH which network port on the remote computer to 
connect to (by default this is 22, but some systems use a different port 
for security reasons). Once a block like this has been added to 
`.ssh/config`, you will be able to connect to that computer by simply 
running `ssh remote1`, without having to specify the full address, user 
name or port for that machine (e.g. `ssh -p 2222 
vierzwei@remote2.network.com`).

To add a third computer (`remote11`) to this, which can only be accessed 
using a tunnel through `remote1`, we add the following lines:

```
Host remote11
  HostName remote1.internalnetwork.com
  User me42
  ProxyCommand ssh remote1@network.com -W %h:%p
```

The `ProxyCommand` will tell SSH that it first needs to connect to 
`remote1`, and from there it will forward all input and output of the 
connection to host `%h` (`remote11`) on port `%p`. More recent versions 
of SSH also support a `ProxyJump` configuration flag with similar 
properties.

That's it! Now SSH will automatically set up the tunnel whenever you try 
to access `remote11`, also when you use SSH implicitly by using `scp` or 
`rsync`. To you, it will now seem as if you were connecting straight to 
`remote11`, while in the background, you would still be tunnelling all 
your network traffic through `remote1`.

# Passwordless SSH

If you apply the technique above without setting up passwordless SSH, 
every connection to `remote11` will prompt for two passwords: one for 
`remote1`, and one for `remote11`. To avoid this, you will need to set 
up passwordless SSH twice: once to allow passwordless SSH between your 
computer and `remote1`, and once to allow passwordless SSH between 
`remote1` and `remote11`. This might be important to know, as it adds 
some complexity.

# Tunnelception

The example here just shows you how to tunnel through a single computer. 
Some networks might require multiple layers of tunnels. This works: 
there is really no limit on how many layers of nested `ssh` commands you 
can perform. Just be aware that configuring many layers of tunnels might 
become very complex, and that each additional tunnel means that your 
network traffic will need to pass through that computer. The final 
connection between you and your deeply embedded target computer will 
only be as fast as the slowest network connection between any of the 
tunnels. And only as secure as any of the intermediate connections too.
