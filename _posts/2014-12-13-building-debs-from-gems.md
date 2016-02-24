---
layout: post
title: Providing gems as debs using fpm and Package Cloud
categories:
---

# {{ page.title }}

There are a lot of package managers.

Some of them provide binary packages, which are pre-compiled. But others don't, especially ones around programming language packages such as pip (python) and rubygems (ruby). Basically if you want to install a pip or gem package you have to have a build environment (ie. compilers, make, etc) installed on each instance or container in order to install the packages. That increases the size of images, and magnifies the attack surface of the instance/container, among other things.

## Before I get too far...

Many of the pip and gem packages are available already as OS packages. Usually pip packages start with "python-" and ruby gems with "ruby-". But not all of them are packaged. Most of the common packages can be installed with apt/yum, but not all. I did a brief search and [this page](http://askubuntu.com/questions/431780/apt-get-install-vs-pip-install) listed a few good differences between getting python libraries via pip and apt-get.

## Sensu

Recently I went to install sensu. Sensu provides a repository that provides the base sensu install. (It gets installed in /opt, as an "omnibus" package.)

However, if you want to use plugins, many of them use the "sensu-plugins" gem, so that needs to be installed. The instructions say that you don't need to install the sensu-plugins gem if you're using the omnibus package.

> If you have installed Sensu from the omnibus packages you can continue to installing the check-procs.rb plugin. Otherwise we need to install the sensu-plugin gem which has various helper classes used by many of the community plugins.

Unfortunately, I believe that would mean ensuring plugins are called using the "embedded" ruby.

Example:

<pre>
<code>ubuntu@curtis-sensu:/etc/sensu/plugins$ /opt/sensu/embedded/bin/ruby check-procs.rb
CheckProcs OK: Found 83 matching processes
</code>
</pre>

But without the sensu-plugin gem...

<pre>
<code>ubuntu@curtis-build:/etc/sensu/plugins$ ./check-procs.rb
/usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file -- sensu-plugin/check/cli (LoadError)
	from /usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
	from ./check-procs.rb:29:in `<main>'
</code>
</pre>

Most plugins will use the ruby pointed to by env.

<pre>
<code>ubuntu@curtis-sensu:/etc/sensu/plugins$ head -1 check-procs.rb
#!/usr/bin/env ruby
</code>
</pre>

I guess one question would be as to what environment plugin developers are writing plugins for--a base OS ruby or the embedded ruby in Sensu's omnibus. Probably the OS ruby would be specified, but that would depend on your developers and organization. The plugin files could be altered to use the embedded ruby.

Thus, I decided to package up that gem.

## Packaging gems with fpm

[fpm](https://github.com/jordansissel/fpm) to the rescue!

One of the first things I do at any job is setup a packaging server. Typically I'm the only one that uses it, but I still do it. :)

For my build/packaging server I installed:

 * ruby-dev
 * gcc
 * make

from the OS packages, and:

 * fpm
 * package_cloud

from rubygems.

I've been using [packagecloud](http://packagecloud.io) for a while now, nd it's a very handy and easy way to setup your own custom repository. Once you build a pacakge you can just use the package cloud CLI to push that package up to your own "cloudy" repo. But I'm getting ahead of myself...

[fpm](https://github.com/jordansissel/fpm) is a great tool to easily build packages. You can tell the developer of fpm takes this very seriously by his all caps usage. :)

> It helps you build packages quickly and easily (Packages like RPM and DEB formats). FUNDAMENTAL PRINCIPLE: IF FPM IS NOT HELPING YOU MAKE PACKAGES EASILY, THEN THERE IS A BUG IN FPM.

Using fpm we can build debian packages from gems.

First, install the gems.

<pre>
<code>ubuntu@curtis-build:~$ gem install --no-ri --no-rdoc --install-dir /tmp/gems sensu-plugin
</code>
</pre>

Next, build the debs using fpm.

<pre>
<code>ubuntu@curtis-build:~$ mkdir debs; cd debs
ubuntu@curtis-build:~/debs$ find /tmp/gems/cache -name '*.gem' | xargs -rn1 fpm -d ruby -s gem -t deb
</code>
</pre>

That will build all the dependencies of sensu-plugin as well.

<pre>
<code>ubuntu@curtis-build:~/debs$ ls
rubygem-json_1.8.1_amd64.deb      rubygem-sensu-plugin_1.1.0_all.deb
rubygem-mixlib-cli_1.5.0_all.deb
</code>
</pre>

## Package cloud

Next we can push those debs to package cloud.

<pre>
<code>ubuntu@curtis-build:~/debs$ package_cloud push serverascode/custom/ubuntu/trusty rubygem-mixlib-cli_1.5.0_all.deb
</code>
</pre>

I do that will all the gems in that dir. Now they show up in my [package cloud repo](https://packagecloud.io/serverascode/custom).

Package cloud has some examples as to how configure your custom repos on your server, but for me it basically looks like this:

<pre>
<code>ubuntu@curtis-sensu:/etc/apt/sources.list.d$ cat packagecloud_io_serverascode_custom_ubuntu.list
deb https://packagecloud.io/serverascode/custom/ubuntu/ trusty main
</code>
</pre>

(Don't forget the package cloud GPG key too.)

I usually use Ansible to setup my servers, including configuring this repository. Below is a snippet from a playbook.

<pre>
<code>- name: install the serverascode custom repository
  apt_repository: >
    repo='deb https://packagecloud.io/serverascode/custom/ubuntu/ trusty main'
    state=present

- name: ensure the packagecloud reposity gpg key is installed
  apt_key: >
    url=https://packagecloud.io/gpg.key
    state=present
</code>
</pre>

Now that the repo is setup, I can install my custom built packages.

<pre>
<code>ubuntu@curtis-build:~/debs$ apt-cache policy rubygem-sensu-plugin
rubygem-sensu-plugin:
  Installed: 1.1.0
  Candidate: 1.1.0
  Version table:
 *** 1.1.0 0
        500 https://packagecloud.io/serverascode/custom/ubuntu/ trusty/main amd64 Packages
        100 /var/lib/dpkg/status
</code>
</pre>

And, the ultimate goal, run the sensu plugin.

<pre>
<code>ubuntu@curtis-build:/etc/sensu/plugins$ ./check-procs.rb
CheckProcs OK: Found 68 matching processes
</code>
</pre>

## Conclusion

So, certainly there are still some issues here. I haven't used this process a lot yet, but I plan on doing a lot more with it because I don't want to be installing compilers and other build tools in every virtual machine or (especially) containers. Containers, as an example, should be as small as is reasonably possible, which I think is going to preclude any build tools. Also there are some security concerns with having build tools in the OS, and a few other infosec issues that I should refamiliarize myself with.

At any rate, at some point most systems administrators will need to package their own debs or rpms, and using fpm and [Package Cloud](http://packagecloud.io) to package and provide repositories is quick and painless.
