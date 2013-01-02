---
layout: post
title: "What I Wish I'd Known About the Terminal"
tagline: From Day One
description: ""
category: 
tags: [bash, terminal, linux]
---
{% include JB/setup %}
## Overview

The Linux terminal gives you a lot of power over your operating system, but most
beginner tutorials are of the type "type _x_ into the terminal and _y_ shall
happen," which I've often found to be insufficient.

## Background

To be fair, I wasn't entirely a beginner in command-line interfaces \(CLIs\). I
had previously used MS-DOS before the Windows 95 GUI interface, so I was
familiar with creating/copying/deleting/changing files and directories \(at
least the basics\), so suffice to say that I'm not scared of accidentally
running `rm -rf /`, the StackOverflow joke used to trick newbies into deleting
everything.

For the few times when I've done something along the same vein I've
so far been able to recover, but there are slightly more advanced \(maybe
intermediate?\) tricks and concepts that I feel would've accelerated my command
of the Linux terminal if I had known them when I first started.

#### 1. The default Linux terminal runs bash

If I were to run across nothing else about the terminal on my first attempt at
commanding it, it would be that the magic behind the terminal is the command
processor named bash. I would've come across the correct tutorials much faster,
and it makes more sense to think of the terminal as a live interpreter of a
scripting language than "the magic box that runs Linux."

#### 2. You can chain input to certain commands

I've slowly started discovering that the terminal is much more flexible than
I suspect. Take the `touch` command, which is used to update access and
modification timestamps of a file, but will create the file if the supplied
name doesn't exist. When I needed three files, I would chain the commands
{% highlight bash %}user@host:~$ touch file1
user@host:~$ touch file2
user@host:~$ touch file3{% endhighlight %}
But one day I decided to be adventurous and simply run
{% highlight bash %}user@host:~$ touch file1 file2 file3{% endhighlight %}
And it worked! In fact, it will work for other utilities such as `mkdir`, `rm`,
and surely many others that I haven't tried.

#### 3. grep

The `grep` command is popular enough to be a verb. Unfortunately the command
is much more extensive than I can write in a few lines, but think of it as
the search function. Ignoring optional flags, a grep expression has the form
`grep "EXPRESSION" FILE`. grep will attempt to match the expression, and
print the entire line of any matches within the given file.

It should be noted that `"EXRESSION"` can be a regular expression, but I am no
RegEx expert so for now they are beyond the scope of this post. For beginners,
it's enough to know that `"EXPRESSION"` is a keyword and you can search the web
for more complex patterns.

#### 4. Piping and other redirection

Piping is another functionality popular enough to be its own verb. Piping is a
form of redirection, and its purpose is to send the output of one process as the
input of another. As an example, we could use `printenv | grep HOME`. The
vertical bar is called the pipe. In this example, `printenv` would normally
print text to the terminal, but instead we redirect that text so that we can
search for the keyword HOME.

There are other redirection keys, which are covered more or less thoroughly
[on this page](http://www.ee.surrey.ac.uk/Teaching/Unix/unix3.html).

#### 5. bash has environment \(global\) variables

This is probably well-known after a few sessions with the terminal, but the
importance of this is that there is an easy way to see which variables are
designed and what their values are. The command `printenv` will print the
environment variables as `KEY=value` pairs. But if you know the environment
variable that you're after, simply typing `echo $KEY` in the terminal will
give you the value if you've typed a valid key.

Alternatively, you could search the current environment variables by piping
with grep: `printenv | grep KEY`

#### 6. You can search man pages

The `man` command, used to search the manual for any command accessible from
the terminal, brings up a page that competes with Wikipedia for thoroughness.
The interface, however, seems rigid and hard to navigate, and so I shied away
from them until I found that you can search them.

Run the man command by using `man <command name>`. Then, press the forward
slash key \(`/`\) and type the expression you'd like to search. This makes the
process of searching for needles in a haystack much faster. A favorite use case
of mine is when I've found a command on the web that uses multiple flags and
I'd like to see what a particular flag does.

#### 7. Check which binaries are run

At some point, if you're using the terminal extensively, it will be necessary
to know which file executes when you run a certain command \(let's say python\).
In this area, I would say that the `which` command is unmatched. Simply run
`which <command name>`, and on your screen you will see some /path/to/binary.

For completeness' sake, I should mention that the /path/to/bin will be one of
several paths contained in your PATH variable. Check by running `which python`
and then verify that the given path is listed when you run `echo $PATH`.

## The End

And that's it! It's possible something has escaped me, but in truth this is a
fairly small and powerful list to make you a competent terminal user.
Alternatively, you could work your way through the
[Advanced Bash Scripting Guide](http://tldp.org/LDP/abs/html/) \(in fact at some
point you definitely should\), but it probably won't be the first thing you do,
especially if you're like me and find bash scripting to be a fairly dull,
though necessary, area.
