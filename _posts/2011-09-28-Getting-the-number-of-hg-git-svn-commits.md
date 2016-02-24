---
layout: post
title: Getting the number of commits in mercurial, git, and svn 
---

# {{ page.title }} 

This is a short post on one way to get the number of commits in hg, git, and svn.

One of the things that I wanted to graph with Cacti is the number of commits that happen in our git, mercurial, and svn repositories. Yup we use all three. 

I know that the number of commits isn't the best metric in terms of figuring out how much our repos are being used, but it's certainly one of the numbers to look at, and it's easy to start with. I'm aware of things like churn in hg, but haven't looked into them fully. Obviously one could make one large commit, or many smaller ones. I prefer many smaller ones, but that's just me. Basically I'm saying I'll add more metrics later.

In order to graph the number of commits, I need to find the number of commits.

At this point I'm most interested in the number of commits that happened in the last 24 hours, one day ago, or yesterday, which I'm aware are not necessarily all the same thing. ;) I'll run the cronjob that checks commits just after midnight, so the numbers should be kinda accurate.

*hg*

<pre>
<code>$ cd some_hg_repo
$ hg log --template '{rev}:{node|short}\n'  --date -1 | wc -l
</code>
</pre>

*git*

<pre>
<code>$ cd some_git_repo
$ git log --since="24 hours ago" | grep "^commit" | wc -l
</code>
</pre>

*svn*

<notextile>
<pre>
<code>$ cd some_svn_repo
# and SVN_REPO=`pwd` or something like that
# Where YESTERDAY=`date --date yesterday +\{\%Y-\%m-\%d\}`
$ svn log -q -r $YESTERDAY file:///$SVN_REPO | grep "^r" | wc -l
</code>
</pre>
</notextile>

Note that with svn, if there hasn't been a commit in since `YESTERDAY` it will return that last commit before that--could be two days ago or more--so unless there are no commits, the number of commits will be at least one, which may not be what you are expecting.


