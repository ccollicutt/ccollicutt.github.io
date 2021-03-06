---
layout: post
title: OpenBSD pf and set limit states 
categories:
---

# {{ page.title }}

So you have a OpenBSD firewall. Actually you have at least two because you are doing [carp](http://www.openbsd.org/faq/pf/carp.html) for high availability (not load balancing but HA), right? 

Awesome! It's fun isn't it? I suppose I have to admit it's more fun testing it in a lab environment than in production. *:)*

One thing I noticed when doing a bit of *non-scientific* load testing on a pair of small carped firewalls is that in OpenBSD the size of the state table is limited to 10000 entries by default. I would imagine that most people won't run into the limit, but I was surprised at how easy it was to hit 10000 sessions using something like [siege](http://www.joedog.org/index/siege-home).

Using a client laptop--an older core 2 duo with 4 gigs of ram and a 60 gig SSD drive (a Lenovo T61 specifically) on one side of the firewall and a virtualized web server with 512MB of RAM and one CPU on the other--I was able to hit the state limit in a couple of seconds with a command such as:

<pre>
<code>laptop$ siege -b -c 40 -r 100 http://testserver/testpage
</code>
</pre>

where the resulting test page looks like:

<pre>
<code><html>
<body>
hi there
</body>
</html>
</code>
</pre>

When the `siege` command is run I watch the state tables on the OpenBSD firewalls with a command such as:

<pre>
<code>openbsd_fw{1,2}#  while true; do pfctl -s info; sleep 1; done
</code>
</pre>

With nothing happening the result of that command looks about like this:

<pre>
<code>openbsd_fw1$ pfctl -s info
Status: Enabled for 0 days 00:22:05              Debug: err

State Table                          Total             Rate
  current entries                       38               
  searches                          215881          162.9/s
  inserts                            30364           22.9/s
  removals                           46342           35.0/s
Counters
  match                              30804           23.2/s
  bad-offset                             0            0.0/s
  fragment                               0            0.0/s
  short                                362            0.3/s
  normalize                              0            0.0/s
  memory                                 0            0.0/s
  bad-timestamp                          0            0.0/s
  congestion                             0            0.0/s
  ip-option                              0            0.0/s
  proto-cksum                            0            0.0/s
  state-mismatch                         0            0.0/s
  state-insert                           0            0.0/s
  state-limit                            0            0.0/s
  src-limit                              0            0.0/s
  synproxy                               0            0.0/s
</code>
</pre>

We can see that the current limit is 10000:

<pre>
<code>openbsd_fw1$ pfctl -sm     
states        hard limit    10000
src-nodes     hard limit    10000
frags         hard limit     5000
tables        hard limit     1000
table-entries hard limit   200000
</code>
</pre>

So lets fire up that `siege` command and see what happens by watching the current entries on the firewall that has the master carp IP. (Note that with `pfsync` all the states will be transferred to the backup firewall as well, but for simplicity let's focus on the master.)

<pre>
<code>openbsd_fw1$ ifconfig | grep -i master
        carp: MASTER carpdev fxp2 vhid 1 advbase 1 advskew 0
        status: master
        carp: MASTER carpdev fxp0 vhid 2 advbase 1 advskew 0
        status: master
openbsd_fw1$ while true; do pfctl -s info | grep "current entries"; sleep 1; done
  current entries                       17               
  current entries                       15               
  current entries                       13               
  current entries                       12               
  current entries                       12 
# siege starts up here              
  current entries                     4820               
  current entries                    10000               
  current entries                    10000               
  current entries                    10000               
  current entries                    10000               
  current entries                    10000 
^C
</code>
</pre>

So you can see it only takes a couple seconds to hit that limit.

Let's up it to 200000 and see what happens.

<pre>
<code>openbsd_fw1$ grep "set limit" /etc/pf.conf
set limit states 200000
openbsd_fw1$ pfctl -nf /etc/pf.conf
openbsd_fw1$ pfctl -f /etc/pf.conf
openbsd_fw1$ pfctl -sm
states        hard limit   200000
src-nodes     hard limit    10000
frags         hard limit     5000
tables        hard limit     1000
table-entries hard limit   200000
</code>
</pre>

Run the siege command on the client:

<pre>
<code>openbsd_fw1$ while true; do pfctl -s info | grep "current entries"; sleep 1; done 
  current entries                       40               
  current entries                       40               
  current entries                       40               
  current entries                       40  
# siege starts up here             
  current entries                      560               
  current entries                     6686               
  current entries                    12480               
  current entries                    17728               
  current entries                    23060               
  current entries                    27116               
  current entries                    28332               
  current entries                    29498               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
  current entries                    29884               
^C
</code>
</pre>

And looks like we max out the `siege` command now at about 30k sessions. Nice!

To conclude, this was just a quick look at session limits on OpenBSD. If you're running a pf firewall it may be something to consider looking at to make sure you're not hitting the limit which would reduce the effectiveness of your firewall.

Note that I haven't shown any memory usage from the firewall, but the small boxes have 512MB of RAM and even at 200K sessions the memory usage only went up very slightly so I don't think it's constrained for memory reasons.


