---
layout: post
title: Mariadb Galera 5.5.42 Crash
categories:
header_image: /img/tvglitch.jpg
header_permalink: https://www.flickr.com/photos/silvertje/2073591109/in/photolist-4aeGoR-78CZMF-87rL21-nBfKXy-7n3An2-qLtWCb-JwVMC-FYzSv-wXBEBx-7ECMwa-Jcg8u-eP3H41-67UWrm-qLG8qZ-8ZRGME-7KaqEu-73VkiS-iY9AFi-qpuAvE-eJiGUw-pbGi9b-m4Y2B1-dDjZRb-7T9nYJ-dF5L6R-5RehTe-ofcNyW-nq4qvx-68x4en-cF5rhA-5Ea85-muAuiS-7hAKZe-7mA9Ho-8teLnE-muAEW5-8dtdiS-68BgVs-2nmWfr-558yb7-KU3Kt-oDz3Ld-2nnafK-dDjZS9-eHGJCT-fcBPNK-dodGqG-snSWzg-e1GoXU-8f45uD
---

# {{ page.title }}

Recently I have been working on setting up a "global" MariaDB 5.5.42 Galera cluster, which means a cluster that has to sync up across a wide area network (WAN), one that is several thousand kilometers long. Normally I'd try to avoid doing this kind of thing, but in this particular situation, running a multi-region OpenStack deployment, I don't have much choice. 

Unfortunately during my initial test, yes the very first test, the cluster crashed. It didn't partition due to a network issue or anything obvious like that, it just simply hard crashed.

In this situation I had 3 regions:

Region 1) 3 initial nodes in a cluster
Region 2) 1 galera arbitrator
Region 3) 1 new node

I added the new node and after a couple of hours, the entire cluster had crashed. The arbitrator and the new node were still running, but the databases were inaccessible. The 3 nodes in region #1 had all crashed with the below message. I had planned on adding the other 2 nodes in region #3 later, but didn't have the chance before it crashed.

<pre>
<code>ubuntu@node2:/var/log/mysql$ tail -100 galera_server_error.log 
    eb91a487,1
} joined {
} left {
} partitioned {
})
151014 22:50:09 [Note] WSREP: save pc into disk
151014 22:50:09 [Note] WSREP: New COMPONENT: primary = yes, bootstrap = no, my_idx = 2, memb_num = 5
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: Waiting for state UUID.
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: sent state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: got state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580 from 0 (garb)
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: got state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580 from 1 (node3)
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: got state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580 from 2 (node2)
151014 22:50:09 [Note] WSREP: STATE EXCHANGE: got state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580 from 3 (node5)
151014 22:50:10 [Note] WSREP: STATE EXCHANGE: got state msg: 1afbdabb-728b-11e5-a0f7-7b10174ed580 from 4 (node1-zone-1)
151014 22:50:10 [Note] WSREP: Quorum results:
    version    = 3,
    component  = PRIMARY,
    conf_id    = 20,
    members    = 4/5 (joined/total),
    act_id     = 134798064,
    last_appl. = 134797950,
    protocols  = 0/7/3 (gcs/repl/appl),
    group UUID = 16e6742c-d7b9-11e4-8f72-af6e67877366
151014 22:50:10 [Note] WSREP: Flow-control interval: [36, 36]
151014 22:50:10 [Note] WSREP: New cluster view: global state: 16e6742c-d7b9-11e4-8f72-af6e67877366:134798064, view# 21: Primary, number of nodes: 5, my index: 2, protocol version 3
151014 22:50:10 [Note] WSREP: wsrep_notify_cmd is not defined, skipping notification.
151014 22:50:10 [Note] WSREP: REPL Protocols: 7 (3, 2)
151014 22:50:10 [Note] WSREP: Service thread queue flushed.
151014 22:50:10 [Note] WSREP: Assign initial position for certification: 134798064, protocol version: 3
151014 22:50:10 [Note] WSREP: Service thread queue flushed.
151014 22:50:10 [Note] WSREP: Member 4.1 (node1-zone-1) requested state transfer from '*any*'. Selected 3.0 (node5)(SYNCED) as donor.
151014 22:50:12 [Note] WSREP: (acc7d568, 'tcp://0.0.0.0:4567') turning message relay requesting off
151014 22:50:15 [Note] WSREP: 3.0 (node5): State transfer to 4.1 (node1-zone-1) complete.
151014 22:50:15 [Note] WSREP: Member 3.0 (node5) synced with group.
151014 22:50:29 [Note] WSREP: 4.1 (node1-zone-1): State transfer from 3.0 (node5) complete.
151014 22:50:29 [Note] WSREP: Member 4.1 (node1-zone-1) synced with group.
pure virtual method called
terminate called without an active exception
151015  0:48:27 [ERROR] mysqld got signal 6 ;
This could be because you hit a bug. It is also possible that this binary
or one of the libraries it was linked against is corrupt, improperly built,
or misconfigured. This error can also be caused by malfunctioning hardware.

To report this bug, see http://kb.askmonty.org/en/reporting-bugs

We will try our best to scrape up some info that will hopefully help
diagnose the problem, but since we have already crashed, 
something is definitely wrong and this may fail.

Server version: 5.5.42-MariaDB-1~trusty-wsrep-log
key_buffer_size=134217728
read_buffer_size=131072
max_used_connections=551
max_threads=2002
thread_count=15
It is possible that mysqld could use up to 
key_buffer_size + (read_buffer_size + sort_buffer_size)*max_threads = 4523910 K  bytes of memory
Hope that's ok; if not, decrease some variables in the equation.

Thread pointer: 0x0x7f1406812000
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 0x7f144f95ea00 thread_stack 0x48000
/usr/sbin/mysqld(my_print_stacktrace+0x2e)[0x7f14502bfa0e]
/usr/sbin/mysqld(handle_fatal_signal+0x457)[0x7f144fea6f57]
/lib/x86_64-linux-gnu/libpthread.so.0(+0x10340)[0x7f144e8fa340]
/lib/x86_64-linux-gnu/libc.so.6(gsignal+0x39)[0x7f144df51cc9]
/lib/x86_64-linux-gnu/libc.so.6(abort+0x148)[0x7f144df550d8]
/usr/lib/x86_64-linux-gnu/libstdc++.so.6(_ZN9__gnu_cxx27__verbose_terminate_handlerEv+0x155)[0x7f144e6466b5]
/usr/lib/x86_64-linux-gnu/libstdc++.so.6(+0x5e836)[0x7f144e644836]
/usr/lib/x86_64-linux-gnu/libstdc++.so.6(+0x5e863)[0x7f144e644863]
/usr/lib/x86_64-linux-gnu/libstdc++.so.6(+0x5f33f)[0x7f144e64533f]
/usr/sbin/mysqld(_Z14wsrep_apply_cbPvPKvmjPK14wsrep_trx_meta+0x7c7)[0x7f144fe57c37]
/usr/lib/galera/libgalera_smm.so(_ZNK6galera9TrxHandle5applyEPvPF15wsrep_cb_statusS1_PKvmjPK14wsrep_trx_metaERS6_+0xd8)[0x7f144bb188f8]
/usr/lib/galera/libgalera_smm.so(+0x1df27d)[0x7f144bb4f27d]
/usr/lib/galera/libgalera_smm.so(_ZN6galera13ReplicatorSMM9apply_trxEPvPNS_9TrxHandleE+0xd2)[0x7f144bb51b32]
/usr/lib/galera/libgalera_smm.so(_ZN6galera13ReplicatorSMM11process_trxEPvPNS_9TrxHandleE+0x10e)[0x7f144bb5498e]
/usr/lib/galera/libgalera_smm.so(_ZN6galera15GcsActionSource8dispatchEPvRK10gcs_actionRb+0x1b8)[0x7f144bb33668]
/usr/lib/galera/libgalera_smm.so(_ZN6galera15GcsActionSource7processEPvRb+0x58)[0x7f144bb33ef8]
/usr/lib/galera/libgalera_smm.so(_ZN6galera13ReplicatorSMM10async_recvEPv+0x73)[0x7f144bb54ef3]
/usr/lib/galera/libgalera_smm.so(galera_recv+0x18)[0x7f144bb634e8]
/usr/sbin/mysqld(+0x49c5a1)[0x7f144fe585a1]
/usr/sbin/mysqld(start_wsrep_THD+0x514)[0x7f144fcb3cc4]
/lib/x86_64-linux-gnu/libpthread.so.0(+0x8182)[0x7f144e8f2182]
/lib/x86_64-linux-gnu/libc.so.6(clone+0x6d)[0x7f144e01547d]

Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (0x0): is an invalid pointer
Connection ID (thread ID): 2
Status: NOT_KILLED

Optimizer switch: index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,index_merge_sort_intersection=off,engine_condition_pushdown=off,index_condition_pushdown=on,derived_merge=on,derived_with_keys=on,firstmatch=on,loosescan=on,materialization=on,in_to_exists=on,semijoin=on,partial_match_rowid_merge=on,partial_match_table_scan=on,subquery_cache=on,mrr=off,mrr_cost_based=off,mrr_sort_keys=off,outer_join_with_cache=on,semijoin_with_cache=on,join_cache_incremental=on,join_cache_hashed=on,join_cache_bka=on,optimize_join_buffer_size=off,table_elimination=on,extended_keys=off

The manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
151015 00:48:27 mysqld_safe Number of processes running now: 0
151015 00:48:27 mysqld_safe WSREP: not restarting wsrep node automatically
151015 00:48:27 mysqld_safe mysqld from pid file /var/lib/mysql/node2.pid ended
</code>
</pre>

## Similar errors

I found this [error](https://bugs.launchpad.net/galera/+bug/1330622) in the Galera bug tracker. The submitter notes:

> One node in my 5-node cluster suddenly failed today with the following backtrace. I haven't seen a node go down abruptly like this before, and don't know how to reproduce.

Interesting that they also experienced the bug with a 5 node cluster. That bug leads to this Codership github [issue](https://github.com/codership/galera/issues/66).

## Minor versions

One thing I noticed is that I had 5.5.45 in the new node, not 5.5.42 like all the other nodes. Presumably that is not a good thing to do. So I setup my automation to ensure I'm pulling from the right archive because the standard MariaDB mirrors only include the latest version.

You can find archived versions of MariaDB software [here](http://archive.mariadb.org/).

While I can't be sure the version difference was the actual issue, as I haven't had time to try to replicate this bug, I'm sure it didn't help.

Even small version increments can be problematic. It's not good enough to install mariadb-galera-server-5.5, I have to ensure the minor version number is the same as well. This has me looking at all the other major software packages I'm running and I know I'll have to setup my automation to include minor versions.


## Current status

Now that all the nodes have the same version of MariaDB Galera Server and I have expanded the nodes from 5 to the full 7 (including the arbitrator) things seem to be running ok.

<pre>
<code>MariaDB [(none)]> show status like 'wsrep_cluster%';
+--------------------------+--------------------------------------+
| Variable_name            | Value                                |
+--------------------------+--------------------------------------+
| wsrep_cluster_conf_id    | 17                                   |
| wsrep_cluster_size       | 7                                    |
| wsrep_cluster_state_uuid | 16e6742c-d7b9-11e4-8f72-af6e67877366 |
| wsrep_cluster_status     | Primary                              |
+--------------------------+--------------------------------------+
4 rows in set (0.01 sec)
</pre>
</code>

## 5.5.46 is out

I would like to eventually get to a newer version of MariaDB. I know that 10.0 is out, it'd be great to get there soon. I am considering going to 5.5.46 if I have any other issues with the cluster in testing.
