---
layout: post
title: Cacti, Better Cacti Graphs, and SSH Original Command 
categories:
---

# {{ page.title }}

So you're at the point where you want to monitor your servers performance. Actually let's take it a step further and you want to do this with [Cacti](http://www.cacti.net). Actually let's take _another_ step and say that you're going to use Cacti and [Better Cacti Graphs](http://code.google.com/p/mysql-cacti-templates/) (BCG)...and ssh.

We're getting pretty specific here aren't we?

So you setup Cacti, RPM up old BCG, and you configure [ssh logins](http://code.google.com/p/mysql-cacti-templates/wiki/SSHBasedTemplates) for BCG so that it can grab stats off the client system.

*But*, and here's the kicker--you want to make sure that you limit what the cacti user can do with a ssh login. (Possibly because your security analyst wants it to be "more secure" and yet has never actually used ssh.)

Well, there is fairly little known functionality built into OpenSSH that will allow you to lock down what the user can do with a ssh key based login. The best thing to do at this point to learn more about this functionality would be to google [SSH_ORIGINAL_COMMAND](http://lmgtfy.com/?q=SSH_ORIGINAL_COMMAND). Sorry, can't help but use [lmgtfy.com](http://lmgtry.com), no offense intended. *:)* It's just fun to say.

The point is that there are many (little known) options that can be put in a `authorized_keys` file to limit what the user can do with they login with that key.

eg. The beginning of the key that I put in the cacti user's `authorized_keys` file on the client server:

<pre>
<code>command="/usr/share/cacti-ssh-auth/ssh_commands_check.sh",from="SOME_IP_ADDRESS",
no-port-forwarding,no-X11-forwarding ssh-dss SNIP_REST_OF_KEY!
</code>
</pre>

What this setting does is:

* Only allows the user to run the script that comes after command, meaning they can *only* run `ssh_commands_check.sh`, and it runs by default.
* Only allow logins from SOME_IP_ADDRESS (eg. 10.0.4.30 or something), ie. the monitoring server where cacti is installed. Authentication via IP addresses isn't the best idea, but why not.
* Disable port forwarding and X11 forwarding for the session, always.

The contents of the `ssh_commands_check.sh` script look like this:

<pre>
<code>#!/bin/sh

# When using $SSH_ORIGINAL_COMMAND to watch what commands we get out
# of better-cacti-graphs, this is what we see:
# cat /proc/diskstats
# cat /proc/stat
# wget -U Cacti/1.0 -q -O - -T 5 "http://localhost/server-status?auto"
# uptime
# free -ob

case "$SSH_ORIGINAL_COMMAND" in
        'cat /proc/diskstats')
                cat /proc/diskstats
                ;;
        'wget -U Cacti/1.0 -q -O - -T 5 http://localhost/server-status?auto')
                wget -U Cacti/1.0 -q -O - -T 5 "http://localhost/server-status?auto"
                ;;
        'uptime')
                uptime
                ;;
        'free -ob')
                free -ob
                ;;
        *)
                # Then essentially do nothing b/c only the above
                # commands are allowed to run. :)
                # I don't really want to echo the actual command
                # until I can find some way to escape anything 
                # malicious. For another day!
                logger -i "$0 ERROR: disallowed command attempted"
                ;;
esac
</code>
</pre>

Great you say. But how did you find out _exactly_ what commands cacti is trying to run? Well SSH_ORIGINAL_COMMAND to the rescue!

I used something like:

<pre>
<code>command="echo $SSH_ORIGINAL_COMMAND >> /var/tmp/ssh_check_cmd.txt; $SSH_ORIGINAL_COMMAND",
from="SOME_IP_ADDRESS",no-port-forwarding,no-X11-forwarding ssh-dss SNIP_REST_OF_KEY!
</code>
</pre>

and then @tail -f /var/tmp/ssh_check_cmd.txt@ and saw this:

<pre>
<code>client_server$ tail -f ssh_orig_cmd.txt 
cat /proc/diskstats
cat /proc/stat
wget -U Cacti/1.0 -q -O - -T 5 "http://localhost/server-status?auto"
uptime
free -ob
</code>
</pre>

so we can be fairly sure that those are the commands the cacti monitor server is asking the client to run.

*NOTE* that I would suggest checking what commands the ssh cacti scripts are running and not blindly using the ones I put above because things could have changed since I put up this post. And in fact could be completely wrong. It will take a bit of time to figure this out.

Also you can review the [code](http://code.google.com/p/mysql-cacti-templates/source/browse/trunk/scripts/ss_get_by_ssh.php) for BCG's use of ssh to find out exactly what commands are running, but note that bash/ect might interpret things differently, so it's best to check with SSH_ORIGINAL_COMMAND.

