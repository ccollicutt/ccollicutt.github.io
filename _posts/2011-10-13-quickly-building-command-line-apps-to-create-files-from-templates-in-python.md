---
layout: post
title: Quickly building command line apps to create files from templates in python 
categories:
- python

---

# {{ page.title }} 

First, let me say that I am not an expert in Python. :) That said, I have been working on a [script](http://github.com/ccollicutt/kicker) for a while now that creates a kickstart file from a command line application that uses:

# a configuration file to get defaults,
# command line options to add or override options, and
# [cheetah template](http://www.cheetahtemplate.org/) files.

I think I finally have a good system for doing this, and could see it being useful to others in terms of writing their own quick application to generate some kind of text file from a command line application using the above three points, and what I have done is commited a set of [skeleton files](https://github.com/ccollicutt/cli-template-generator) so that you could do this yourself very easily.

## Dependencies

`cli-template-generator` was written to run on RHEL5 which means python 2.4.3.

It also requires `python-argparse` and `python-cheetah` if you are on RHEL5.

I think it will work in later python versions.

## Using the cli-template-generator

First, clone the `cli-template-generator` repository.

<pre>
<code>$ git clone git@github.com:ccollicutt/cli-template-generator.git
Cloning into cli-template-generator...
remote: Counting objects: 8, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 8 (delta 0), reused 8 (delta 0)
Receiving objects: 100% (8/8), done.
$ cd cli-template-generator/
$ ls
README  skeleton.conf  skeleton.py  skeleton.tpl
</code>
</pre>

There are three files that work together to create a text file from a template: skeleton.{conf,py,tpl}.

The tpl file contains the Cheetah template information:

<pre>
<code>$ cat skeleton.tpl 
Hello $hello 
</code>
</pre>

Which means the `hello` variable will be replaced with the string it's set to in either the config file or the command line argument.

The conf file holds the default configuration option(s).

<pre>
<code>$ cat skeleton.conf 
[default]
#
# Add default configuration options in this file
# Eg.
# Key:	Value

hello:	World!
</code>
</pre>

By default, the skeleton.conf file sets the variable `hello` to the string `World!`. So if we run `skeleton.py` we will see:

<pre>
<code>$ ./skeleton.py 
Hello World! 

</code>
</pre>

Also by default we have one command line configuration option, `--hello`, that we can set to whatever we want. We can see what options are available using `--help`.

<pre>
<code>$ ./skeleton.py --help
usage: skeleton.py [-h] [-c CONFIGFILE] [--hello HELLO]

optional arguments:
  -h, --help            show this help message and exit
  -c CONFIGFILE, --config-file CONFIGFILE
                        Use a different config file than ./skeleton.conf
  --hello HELLO         Who are you saying hello to?
</code>
</pre>

So if we run `skeleton.py` with the `--hello` option we should get different results.

<pre>
<code>$ ./skeleton.py --hello Curtis!
Hello Curtis! 
</code>
</pre>

So, as you can see, it should be fairly easy to copy the skeleton files, replace some of the default locations for the files, add some configuration options to the configuration file and also to the parser in the py file, and edit the template so that it uses your new variables.

Bam! You have a custom cli text file generator! Given the amount of text files in Unix/Linux, there could be a lot of good uses for this.
