---
layout: post
title: Environment variables with Docker
categories: 
---

# {{ page.title }}

I've been working with [Docker](http://docker.io) for a little while, and was wondering how environment variables work, so I took a few minutes to look into it.

Docker supports setting environment variables with the -e switch.

Below is the simplest example. As can be seen, the FOO variable is indeed set to bar within the container.

<pre>
<code># docker run -e FOO=bar busybox env
HOME=/
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=9c6f6cd077b2
FOO=bar
</code>
</pre>

Multiple variables can be added with multiple -e switches.

<pre>
<code># docker run -e FOO=bar -e BAR=foo busybox env
HOME=/
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=6cf2d6e8acb3
FOO=bar
BAR=foo
</code>
</pre>

## Using environment variables in a script in a docker image

Below is a simple docker file that adds a script called run.sh to the image.

<pre>
<code># file Dockerfile
FROM busybox
ADD run.sh run.sh
RUN chmod +x run.sh
CMD ./run.sh
</code>
</pre>

This is what is in the run.sh script. If FOO is not set, the container should report "FOO is empty" and otherwise it should print what the variable has been set to.

<pre>
<code># file run.sh
#!/bin/sh

if [ -z "$FOO" ]; then
	echo "FOO is empty"
else
	echo "FOO is $FOO"
fi
</code>
</pre>

I can now build that Dockerfile.

<pre>
<code># ls
Dockerfile  run.sh
# docker build -t testenv .
Uploading context 3.584 kB
Uploading context 
Step 0 : FROM busybox
 ---> 4c0f792ebd1e
Step 1 : ADD run.sh run.sh
 ---> 6154a355bdd6
Step 2 : RUN chmod +x run.sh
 ---> Running in 20784036cde1
 ---> 784a1682769b
Step 3 : CMD ./run.sh
 ---> Running in d86187fc3a6f
 ---> 1c952ed5cc6d
Successfully built 1c952ed5cc6d
Removing intermediate container c598898e1706
Removing intermediate container 20784036cde1
Removing intermediate container d86187fc3a6f
</code>
</pre>

Now that the image has been built, I can run a container based off it, and the run.sh script should execute.

<pre>
<code># docker run testenv
FOO is empty
</code>
</pre>

And now adding the FOO environment variable to the docker run command:

<pre>
<code># docker run -e FOO=bar testenv
FOO is bar
# docker run -e FOO=$RANDOM testenv
FOO is 10567
# docker run -e FOO=$RANDOM testenv
FOO is 8898
</code>
</pre>

Awesome. Now with this ability, we could, for example, create one Wordpress container and have it configure wp-config.php with the correct MySQL connection variables, and have that container connect out to the MySQL server with the right user and password, rather than creating an image for each Wordpress site.




