---
layout: post
title: First look at Digital Ocean
categories: 
header_image: https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/digital_ocean_signup.png
---

# {{ page.title }}


Even though I have worked a great deal with OpenStack and various virtualization technologies, I haven't really used infrastructure as a service all that much. Being Canadian has made using Amazon Web Services unlikely as I normally work at organizations that have concerns (justified or not) about their data staying in Canada. Usually that means AWS is off the table. I've used Rackspace a bit to test out their Rackspace files product which is based on OpenStack Swift and I've also used Heroku to deploy a [hubot](https://hubot.github.com/), but that's about it.

## Digital Ocean

Digital Ocean (DO) keeps popping up in my RSS feeds, so I thought I'd give it a shot. From what I've read DO is a simplified IaaS provider that also separates itself from the rest of the pack by its use of only solid state drives--no spinning rust here. While I believe Rackspace now provides all SSDs, DO was kind of the first to really market themselves using SSDs (at least the first I was aware of at any rate).

I mention the word "simplified" above. This is another way that I believe DO is distinguishing itself from other IaaS providers--a simplified web interface, possibly a simplified product offering, and easy to understand pricing.

## Sign-up

The sign-up is very straight forward. Nothing but an email and a password. I don't think a second field to verify a password for a new sign-up is necessary, especially given how passwords can usually be reset with a link in an email, so the fact that DO only has one password field seems smart to me. Also the fact that the sign up is on the front page is telling.

## Credit card info

The first thing I wanted to do is find out what Operating Systems they support for virtual machines. I'm sure they support all the standard OSes, but the sysadmin in me wants to go to that page first. However, as far as I can tell I can't see that information without putting in a credit card first. So I enter my credit card info.

## Simple web interface

I'm definitely enjoying the simple web interface. I think there is a lot of room in IaaS to provide a very simple web gui. I think DO is doing a smart thing here, courting developers, simple web interface, simple usage, relatively low cost, and SSDs. AWS has tons of features and products that are difficult to understand, and what's more, it's hard to figure out what your bill is going to be (though again, I haven't used AWS much).


![](https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/webgui.png)


There's not much going on in the sidebar. Just:

- Create
- Droplets
- Images
- SSH keys
- Billing
- Support
- DNS
- API

Most of these concepts are going to be in every IaaS provider. Create a virtual machine/instance/droplet. What instances do I already have running? What images can these droplets be based off of? What SSH keys are automatically added to the image when it's booted? (Most IaaS providers inject an SSH key into the virtual machine and also turn off SSH password access--only access using SSH keys is allowed at first to remove the ability of bots to break the password and login.)

## Add an SSH key

A big button says "Add SSH key." It's calling to me. :)

![](https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/sshkey.png)

Surprisingly I can't just upload the file I have to paste it into the text box. Sure. Once I do that it shows my key in the SSH keys list.

![](https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/sshkeylist.png)

Now I'm going to guess that having entered my CC information, and having uploaded at least one SSH key, I can now boot a droplet/vm/instance.

But I don't want to do that with the web gui, because I don't really like to use gui interfaces. Looks like I need to find a command line application that can use the API that DO seems to have...

## Tugboat

So I google search for "digital ocean command line" and below are some of the results. Looks like there is an command line app called "tugboat."

![](https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/googlesearch.png)

Let's get that. The [github README](https://github.com/pearkes/tugboat) for tugboat says "gem install tugboat."

<pre>
<code>curtis$ sudo gem install tugboat
SNIP!
10 gems installed
</code>
</pre>

I noticed that during the install a digital_ocean gem was brought down too:

<pre>
<code>curtis$ gem list | grep digital
digital_ocean (1.0.1)
</code>
</pre>

I'm willing to bet that DO provides that gem, and Ruby code can take advantage of it, just like tugboat.

Now I've got a tugboat command.

<pre>
<code>curtis$ which tugboat
/usr/bin/tugboat
</code>
</pre>

Here are all the commands tugboat provides:

<pre>
<code>curtis$ tugboat
Commands:
  tugboat add-key NAME                                 # Upload an ssh public key.
  tugboat authorize                                    # Authorize a DigitalOcean account with tugboat
  tugboat create NAME                                  # Create a droplet.
  tugboat destroy FUZZY_NAME                           # Destroy a droplet
  tugboat destroy_image FUZZY_NAME                     # Destroy an image
  tugboat droplets                                     # Retrieve a list of your droplets
  tugboat halt FUZZY_NAME                              # Shutdown a droplet
  tugboat help [COMMAND]                               # Describe commands or a specific command
  tugboat images                                       # Retrieve a list of your images
  tugboat info FUZZY_NAME [OPTIONS]                    # Show a droplet's information
  tugboat info_image FUZZY_NAME [OPTIONS]              # Show an image's information
  tugboat keys                                         # Show available SSH keys
  tugboat password-reset FUZZY_NAME                    # Reset root password
  tugboat rebuild FUZZY_NAME IMAGE_NAME                # Rebuild a droplet.
  tugboat regions                                      # Show regions
  tugboat resize FUZZY_NAME -s, --size=N               # Resize a droplet
  tugboat restart FUZZY_NAME                           # Restart a droplet
  tugboat sizes                                        # Show available droplet sizes
  tugboat snapshot SNAPSHOT_NAME FUZZY_NAME [OPTIONS]  # Queue a snapshot of the droplet.
  tugboat ssh FUZZY_NAME                               # SSH into a droplet
  tugboat start FUZZY_NAME                             # Start a droplet
  tugboat verify                                       # Check your DigitalOcean credentials
  tugboat version                                      # Show version
  tugboat wait FUZZY_NAME                              # Wait for a droplet to reach a state
</code>
</pre>

Oops, looks like I could have added my key with tugboat. Oh well.

The tugboat README says use "tugboat authorized" to setup the API information to allow tugboat to access the DO API using my credentials.

Note that I found my API credentials and keys in the web gui under "API."

<pre>
<code>curtis$ tugboat authorize
Note: You can get this information from digitalocean.com/api_access

Enter your client key: <REDACTED>
Enter your API key: <REDACTED>
Enter your SSH key path (optional, defaults to ~/.ssh/id_rsa): ~/.ssh/id_dsa
Enter your SSH user (optional, defaults to curtis): 
Enter your SSH port number (optional, defaults to 22): 

To retrieve region, image, size and key ID's, you can use the corresponding tugboat command, such as `tugboat images`.
Defaults can be changed at any time in your ~/.tugboat configuration file.

Enter your default region ID (optional, defaults to 1 (New York)): 
Enter your default image ID (optional, defaults to 350076 (Ubuntu 13.04 x64)): 
Enter your default size ID (optional, defaults to 66 (512MB)): 
Enter your default ssh key ID (optional, defaults to none): curtis
Enter your default for private networking (optional, defaults to false): true
Enter your default for enabling backups (optional, defaults to false): 
Authentication with DigitalOcean was successful.
</code>
</pre>

I set everything as default except put the right location for my local key, and pointed it to the SSH key I already uploaded as default, and also enabled private networking. I know from having read a blog post on DO that they support a form of private networking in certain regions.

Lets see what I get from tugboat.

<pre>
<code>curtis$ tugboat droplets
You don't appear to have any droplets.
Try creating one with `tugboat create`
# so no droplets yet
curtis$ tugboat keys
SSH Keys:
curtis (id: 118429)
</code>
</pre>

What images are available? "--global" means show all images, not just my own images.

<pre>
<code>curtis$ tugboat images --global
My Images:
No images found

Global Images:
CentOS 5.8 x64 (id: 1601, distro: CentOS)
CentOS 5.8 x32 (id: 1602, distro: CentOS)
Debian 6.0 x64 (id: 12573, distro: Debian)
Debian 6.0 x32 (id: 12575, distro: Debian)
Ubuntu 10.04 x64 (id: 14097, distro: Ubuntu)
Ubuntu 10.04 x32 (id: 14098, distro: Ubuntu)
Arch Linux 2013.05 x64 (id: 350424, distro: Arch Linux)
Arch Linux 2013.05 x32 (id: 361740, distro: Arch Linux)
CentOS 6.4 x32 (id: 376568, distro: CentOS)
CentOS 6.4 x64 (id: 562354, distro: CentOS)
Ubuntu 12.04.4 x32 (id: 3100616, distro: Ubuntu)
Ubuntu 12.04.4 x64 (id: 3101045, distro: Ubuntu)
Ubuntu 13.10 x32 (id: 3101580, distro: Ubuntu)
Ubuntu 12.10 x32 (id: 3101888, distro: Ubuntu)
Ubuntu 12.10 x64 (id: 3101891, distro: Ubuntu)
Ubuntu 13.10 x64 (id: 3101918, distro: Ubuntu)
Debian 7.0 x32 (id: 3102384, distro: Debian)
Debian 7.0 x64 (id: 3102387, distro: Debian)
Fedora 19 x32 (id: 3102721, distro: Fedora)
Fedora 19 x64 (id: 3102879, distro: Fedora)
Ubuntu 12.10 x64 Desktop (id: 3104282, distro: Ubuntu)
Docker 0.10 on Ubuntu 13.10 x64 (id: 3104894, distro: Ubuntu)
MEAN on Ubuntu 12.04.4 (id: 3118235, distro: Ubuntu)
GitLab 6.6.5 CE (id: 3118238, distro: Ubuntu)
LAMP on Ubuntu 12.04 (id: 3120115, distro: Ubuntu)
Ghost 0.4.2 on Ubuntu 12.04 (id: 3121555, distro: Ubuntu)
Wordpress on Ubuntu 13.10 (id: 3135725, distro: Ubuntu)
Ruby on Rails on Ubuntu 12.10 (Nginx + Unicorn) (id: 3137635, distro: Ubuntu)
Redmine on Ubuntu 12.04 (id: 3137903, distro: Ubuntu)
Ubuntu 14.04 x32 (id: 3240033, distro: Ubuntu)
Ubuntu 14.04 x64 (id: 3240036, distro: Ubuntu)
CentOS 6.5 x32 (id: 3240847, distro: CentOS)
CentOS 6.5 x64 (id: 3240850, distro: CentOS)
Fedora 20 x32 (id: 3243143, distro: Fedora)
Fedora 20 x64 (id: 3243145, distro: Fedora)
Dokku v0.2.3 on Ubuntu 14.04 (id: 3288841, distro: Ubuntu)
</code>
</pre>

Lots of interesting images there. I can see ghost, wordpress, a basic LAMP stack, things I've never heard of, and more! Docker 0.10 too, that might be interesting to try out. Notice they each have an ID. This is all very similar to OpenStack.

## Spend some money

So now I assume I can create a droplet from the command line. But first lets look at how much this is going to cost me.

![](https://raw.githubusercontent.com/ccollicutt/ccollicutt.github.com/master/img/digital_ocean/pricing.png)

I should have clicked on hourly.

Hourly costs are:

- 512MB - $0.007 per hour
- 1GB - $0.015 per hour
- 2GB - $0.03 per hour

I'm not going to leave these running all the time. I just want to experiment. One note: Rackspace offers billing per minute, whereas DO just has monthly or per hour.

I'm going to boot a Ubuntu 14.04 32 bit image. First I need the image ID.

<pre>
<code>curtis$ tugboat images --global | grep 14.04
Ubuntu 14.04 x32 (id: 3240033, distro: Ubuntu)
Ubuntu 14.04 x64 (id: 3240036, distro: Ubuntu)
Dokku v0.2.3 on Ubuntu 14.04 (id: 3288841, distro: Ubuntu)
</code>
</pre>

Now I can see that the ID is 3240033. I'm going to use that and my default tugboat settings to boot an instance.

<pre>
<code>curtis$ tugboat create tester -i 3240033
Queueing creation of droplet 'tester'...the server responded with status 404!

You specified an invalid region for Droplet creation.

Double-check your parameters and configuration (in your ~/.tugboat file)
</code>
</pre>

Uh oh. Error message.

<pre>
<code>curtis$ tugboat regions
Regions:
San Francisco 1 (id: 3)
New York 2 (id: 4)
Amsterdam 2 (id: 5)
Singapore 1 (id: 6)
</code>
</pre>

Looks like I have New York 1 set, when I need New York 2, which has ID 4.

<pre>
<code>curtis$ tugboat create tester -i 3240033 -r 4
Queueing creation of droplet 'tester'...done
</code>
</pre>

tugboat exits. Guess I'll have to check the status with tugboat droplets.

<pre>
<code>tester (ip: 162.243.253.240, status: new, region: 4, id: 1539322)
# ... few seconds later
curtis$ tugboat droplets
tester (ip: 162.243.253.240, status: active, region: 4, id: 1539322)
</code>
</pre>

Ok, the droplet has status active. 

<pre>
<code>curtis$ ping -c 1 162.243.253.240
PING 162.243.253.240 (162.243.253.240): 56 data bytes
64 bytes from 162.243.253.240: icmp_seq=0 ttl=54 time=81.364 ms

--- 162.243.253.240 ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 81.364/81.364/81.364/0.000 ms
</code>
</pre>

I can ping it too.

I'm going to try to ssh into the droplet.

<pre>
<code>curtis$ ssh root@162.243.253.240
# hangs, and eventually asks for my password...
</code>
</pre>

Hanging. Wonder why. Check my email while I'm waiting, turns out DO sends and email when a new vm is created. There is a password in the email. Oh no. It says that if I'd prefer not to receive emails with passwords in them then to add an ssh key to DO and also create droplets with that key. I was assuming tugboat was using the default key I set up. Maybe I need to specifically put in the key.

<pre>
<code>curtis$ tugboat help create
Usage:
  tugboat create NAME

Options:
  -s, [--size=N]              # The size_id of the droplet
  -i, [--image=N]             # The image_id of the droplet
  -r, [--region=N]            # The region_id of the droplet
  -k, [--keys=KEYS]           # A comma separated list of SSH key ids to add to the droplet
  -p, [--private-networking]  # Enable private networking on the droplet
  -b, [--backups-enabled]     # Enable backups on the droplet
  -q, [--quiet]               

Create a droplet.
</code>
</pre>

I'll destroy and recreate my droplet.

<pre>
<code>curtis$ tugboat destroy tester
Droplet fuzzy name provided. Finding droplet ID...done, 1539346 (tester)
Warning! Potentially destructive action. Please confirm [y/n]: y
Queuing destroy for 1539346 (tester)...done
</code>
</pre>

Now lets create tester2.

But...first, I realized the API wants an SSH key ID, not key name. I edit ~/.tugboat and set my key to use the ID not the name "curtis."

<pre>
<code>curtis$ tugboat keys
SSH Keys:
curtis (id: 118429)
curtis$ vi ~/.tugboat
# make id change
curtis$ grep ssh_key ~/.tugboat 
  ssh_key_path: /Users/curtis/.ssh/id_dsa
  ssh_key: 118429
# looks good
</code>
</pre>

Ok, let's try that again.

<pre>
<code>curtis$ tugboat create tester2 -i 3240033 -r 4
Queueing creation of droplet 'tester2'...done
curtis$ tugboat droplets
tester2 (ip: 162.243.253.240, status: new, region: 4, id: 1539401)
curtis$ tugboat droplets
tester2 (ip: 162.243.253.240, status: active, region: 4, id: 1539401)
</code>
</pre>

Now it's active. Try ssh one more time...

<pre>
<code>curtis$ ssh root@162.243.253.240
SNIP!
root@tester2:~# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 04:01:17:7d:ad:01  
          inet addr:162.243.253.240  Bcast:162.243.253.255  Mask:255.255.255.0
          inet6 addr: fe80::601:17ff:fe7d:ad01/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1645 errors:0 dropped:0 overruns:0 frame:0
          TX packets:185 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:104117 (104.1 KB)  TX bytes:23629 (23.6 KB)

</code>
</pre>

I can login. Nice!

NOTE: It seemed like the IP took awhile to be available, longer than for the vm to become active. Not too sure what to make of that.

Further--DO did *not* send me an email with passwords in it. That suggests I have ssh keys setup properly in tugboat.

(Once the droplet is active and you've sshed in, I would suggest turning off password authentication in /etc/ssh/sshd_config and only use ssh keys to access servers.)

## Private network

DO recently announced that they support [private networks](https://www.digitalocean.com/company/blog/introducing-private-networking/). But they aren't private in the way most would consider...instead it's one big open internal non-routable network. Every droplet that requests a private network can talk to any other droplet. Seems kind of wild-west to me. But that is how it works. 

<pre>
<code>root@tester2:~# ifconfig eth1
eth1      Link encap:Ethernet  HWaddr 04:01:17:7d:ad:02  
          inet addr:10.128.183.80  Bcast:10.128.255.255  Mask:255.255.0.0
          inet6 addr: fe80::601:17ff:fe7d:ad02/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:133 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:5816 (5.8 KB)  TX bytes:578 (578.0 B)
</code>
</pre>

If I booted another droplet and it had a private IP I could access it that way. This would be good for things like small clusters, or high availability maybe, and the traffic doesn't count towards your bandwidth limit. It's important to note that if I boot a vm with a private network, and another user does to, technically I can connect to their server on the private network if I know its IP. So it's not a compartmentalized private network for each user or tenant--just one big network! Kind of like the public Internet...

## Destroy!

I'm going to destroy that droplet so I don't leave it up and running and have it cost me. (Though that said, a month of a 512MB droplet isn't much more than a couple coffees at $5.)

<pre>
<code>curtis$ tugboat destroy tester2
Droplet fuzzy name provided. Finding droplet ID...done, 1539401 (tester2)
Warning! Potentially destructive action. Please confirm [y/n]: y
Queuing destroy for 1539401 (tester2)...done
curtis$ tugboat droplets
You don't appear to have any droplets.
Try creating one with `tugboat create`
</code>
</pre>

Create...destroy...it's the DevOps way. :)

## Conclusion

I like the simplicity and ease of getting an account on Digital Ocean; that that aren't a ton of features to get in the way. I like that there is a command line application I can use to create droplets. As well, I think DO is doing smart to target developers. Developers are more important that ever, and part of cloud or utility computing is about making things easy to get going--sign up, input CC information, and boot servers to run code. Also simplifying charges is good too. AWS, Rackspace, etc, often have additional networking charges that are hard to calculate. With DO you get X terabytes of transfer, and moving data over the private network costs nothing. Not too much more to think about unless you hit the bandwidth limit.

One thing I see is that it's not possible to upload an image. I believe I can create a new image based off an existing droplet, but not upload a new image. So if I wanted an OpenBSD droplet I'm out of luck until they support a base OpenBSD image.

As far as disk space--I wonder if at some point they will have to offer volume storage, like what AWS does with EBS and what OpenStack does with Cinder volumes. I do think that EBS is a bit of a problem in that it's hard to offer a service like that without the possibility of it crashing big time--see the CAP theorem for more on that. But at some point users might run out of space, but then they can just move to a larger droplet size, though that will cost more obviously.  It will be interesting to see how DO copes with trying to keep it simple while also being profitable, ie. how to make more money without adding more features. 

I quite like DO, and I think it's interesting to compare their offering with what OpenStack or AWS provides.






