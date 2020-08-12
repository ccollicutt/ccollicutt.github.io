---
layout: post
title:  Remove Recent Commits from a Git Repo
categories:
header_image: "/img/git-commit.jpg"
summary: "Sometimes git is easy"

---

# {{ page.title }}

I've been building a workshop that uses the [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) example application.

I've made two commits updating the welcome message, and I've pushed those commits to a repo in github, but now I want to remove them.

Here are the commits:

```
$ git log
commit 9fec5226ec3dbc900bf3939c10f7f5e2b3c5ee94 (HEAD -> main, origin/main, origin/HEAD)
Author: curtis <curtis@serverascode.com>
Date:   Tue Aug 11 21:30:01 2020 -0400

    update welcome

commit 405d7567681871bd04439c2f49e13f96a0816d27
Author: curtis <curtis@serverascode.com>
Date:   Tue Aug 11 21:20:01 2020 -0400

    change welcome message

commit c42f95980a943634106e7584575c053265906978
Author: Martin Lippert <mlippert@gmail.com>
Date:   Wed Jul 29 16:43:31 2020 +0200

    remove push-to-pws button from guide, since pws free trials end

commit 02cc84223b296e7b401cdba74905b2e18a87e51e
Author: Stephane Nicoll <snicoll@pivotal.io>
Date:   Sat Jul 11 08:57:33 2020 +0200

    Polish
SNIP!
```

As you can see above there are two commits, 9fec5226ec3dbc900bf3939c10f7f5e2b3c5ee94 and 405d7567681871bd04439c2f49e13f96a0816d27 and I want to remove them from the remote repo.

It's actually quite easy! I'm going to reset to the commit prior to my two commits, c42f95980a943634106e7584575c053265906978.

```
$ git reset --hard c42f95980a943634106e7584575c053265906978
HEAD is now at c42f959 remove push-to-pws button from guide, since pws free trials end
```

Now git log shows...

```
$ git log
commit c42f95980a943634106e7584575c053265906978 (HEAD -> main)
Author: Martin Lippert <mlippert@gmail.com>
Date:   Wed Jul 29 16:43:31 2020 +0200

    remove push-to-pws button from guide, since pws free trials end

commit 02cc84223b296e7b401cdba74905b2e18a87e51e
Author: Stephane Nicoll <snicoll@pivotal.io>
Date:   Sat Jul 11 08:57:33 2020 +0200

    Polish

commit 5ad6bc3ccde1c61379d751f0751a1731114761af
Author: Stephane Nicoll <snicoll@pivotal.io>
Date:   Sat Jul 11 08:56:32 2020 +0200

    Upgrade to spring javaformat 0.0.22
SNIP!
```

And force push:

```
$ git push --force
Total 0 (delta 0), reused 0 (delta 0)
To github.com:ccollicutt/spring-petclinic.git
 + 9fec522...c42f959 main -> main (forced update)
```

Done!
