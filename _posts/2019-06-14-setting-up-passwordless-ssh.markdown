---
layout: post
title: "Setting up passwordless SSH"
description: >-
  A quick tutorial on setting up passwordless SSH access between machines.
date: 2019-06-14
author: Bert Vandenbroucke
tags:
  - Tools
---

Secure Shell (SSH) is by far the most used protocol to provide safe 
network access to remote resources like HPC clusters or web servers. It 
has shipped with Unix based operating systems since forever, and is now 
even part of Windows since its version 10.

In its most basic usage, `ssh` is simply a command you run in the 
command line shell, with at least one additional command line parameter: 
the address of the remote resource you want to access. The software will 
then contact that resource, check if your user account (or the user 
account you specify as part of the address) has permissions to access 
that resource, and will then perform some sort of safety check to verify 
that it is actually you on the other side of the connection. By default, 
this means the software will prompt for a password. Once your identity 
is verified, SSH will exchange temporary encryption keys between your 
computer and the remote resource, so that all further network traffic 
between you and the resource can be safely encrypted.

In general, all of this is quite easy, except for one step: the password 
necessary to verify your identity. This is also the main weakness of the 
method: if someone manages to get hold of your password, they can fake 
your identity and gain access to the remote resource. Passwordless SSH 
access tries to remedy both of these problems by eliminating the 
password prompt from SSH calls, and by providing a more secure way of 
verifying the identity of the computer that requests a connection. In 
this brief tutorial, I will try to explain how passwordless SSH works 
and how you set it up.

# How does it work?

Passwordless SSH pretty much works the same way as most of the traffic 
that goes through SSH: it makes use of SSH keys. An SSH key consists of 
two parts: a *private* and a *public* key. The public key is nothing 
more than a very large number, usually stored as a pretty random-looking 
string of hexadecimal characters. The private key consists of two very 
large prime numbers stored in the same way; the product of these prime 
numbers is the number stored in the public key. The public key is used 
to encrypt messages, but cannot be used to decrypt them, this requires 
both private keys to be known. The security of the encryption comes from 
the fact that decomposing a large number into its prime factors is very 
computationally demanding and therefore entirely impractical for the 
typical life time of a key.

So in a typical SSH scenario, the client (your computer) and the server 
will exchange their public keys and then use these to encrypt all 
network traffic between them. Since the client and the server are the 
only machines that also have the respective private keys corresponding 
to their public keys, they are the only machines that can decrypt 
incoming network traffic on the connection.

But SSH keys do not necessarily need to be used for encryption: they can 
also be used to verify the client's identity. To this end, the client's 
public key can be stored on the server before the connection is made, 
and SSH can check whether or not the client also owns the private keys 
when a connection is requested. Since this mechanism uses (large) keys 
directly, it is more secure than relying on a much shorter password. A 
potential disadvantage of this method is that anyone with access to the 
client could in principle issue an `ssh` command, so the server could be 
vulnerable to attacks if the client itself is compromised. To protect 
against this, private keys on a client are usually password protected 
themselves; SSH will still ask for a password to *unlock* your private 
key. However, unlocked keys can be kept unlocked for a certain amount of 
time (e.g. for as long as you are logged in to your computer), so this 
still means much less passwords than without passwordless 
authentication.

# How do you set it up?

Setting up passwordless SSH requires two steps: generating an SSH key 
pair on your local computer, and then transferring the public part of 
that key to the remote computer you want access to.

## Generating an SSH key

To generate a key, you can use the `ssh-keygen` command (without any 
command line parameters). This will prompt three questions:
 1. The name of the file in which to save the key. By default, this will 
be `id_rsa`, in the `.ssh` folder of your home directory on the system. 
If you have never generated a key before, this file will not exist and 
it is best to keep the default.
 2. A password to protect the private key. This password will be asked 
whenever you try to unlock the private key on your system.
 3. The password, again. Just to make sure that you didn't make a typo 
the first time and locked yourself out of your private key.

Next, the command will generate two key files: the private key 
(`~/.ssh/id_rsa` by default), and the public key (`~/.ssh/id_rsa.pub` by 
default; if you changed the key file name, it will just be this + 
`.pub`). It will also give you a key fingerprint and a key random art 
image. I have never needed to use those, so I'm not sure what they are 
and why they are shown.

## Transferring your key to the remote system

This step depends a bit on what the remote system is. Some systems (e.g. 
the repository system on [GitHub](https://github.com/)) allow you to 
upload your public key to your online profile and then automatically 
make sure it ends up in the right location on their servers; other 
systems might ask you to send your public key over email and then have 
someone put it in the right location manually. Or you might need to 
transfer your public key yourself; the case I will handle here.

Obviously, putting your key on the remote system requires access to the 
system. So in this case, you will need to login to the system over SSH 
(with password) first. Once you have done that, you need to figure out 
where SSH stores files on the system. By default, this is an `.ssh` 
folder within your home folder (so `~/.ssh`). If this folder does not 
exist, then you will need to somehow figure out what non-default 
location is used by the system.

Public keys for clients are stored in a file called 
`.ssh/authorized_keys`. This file will likely not exist, in this case 
you can simply create it. There is no limit on the number of keys added 
to the file; you can simply add them on a new line and it should work.

To copy the key, you can either open the public key file on your 
computer using a text editor and copy it over (it is not very large). 
The key consists of three parts, separated by a white space:
 1. `ssh-rsa`: this identifies the key as a public key.
 2. A very long string of random characters: this is the actual key.
 3. A short label to identify the key, this defaults to 
`username@hostname`.

You need to copy over the full line. The label part is really useful to 
distinguish between different clients when you start adding more clients 
to `authorized_keys`. Once you have done that, save the file and 
everything should be ready to go.

## Testing passwordless SSH

Testing everything is straightforward: make sure you are logged out from 
the remote server (in case you manually added your key), and try to log 
in using the usual `ssh` command. If everything works, this will no 
longer prompt for a password and instead immediately log you in to the 
machine. Commands that use `ssh` in the background, like `rsync` and 
`scp` will also no longer prompt for a password.

# Caveats

While the above should in principle work fine, SSH can be configured to 
explicitly not allow passwordless authentication (on the remote system). 
In this case, the public key on the system will be ignored and SSH will 
always prompt for a password. Also note that for security reasons, SSH 
does not allow passwordless authentication if the `authorized_keys` file 
can be modified by users other than the root and login user, since then 
other users on the remote system might be able to gain access to your 
user account.

Also for security reasons, SSH does not usually allow you to unlock your 
private key indefinitely during a remote session, i.e. if you are 
already logged in on a remote server and you issue an `ssh` command from 
there, it might prompt you for the password to unlock your key every 
time you need the private key. Passwordless authentication is then no 
longer passwordless from the user point of view (but it is still safer 
than actually using a short password).
