---
title: "Source RPMs"
slug: "source-rpms"
date: "2017-02-25"
---

This article will cover how to get the source package, setup your environment, then patch the sources for viewing.

<!--more-->

Sometimes you find yourself needing to look at the source code of a particular program. There are a variety of reasons you'd want to do this, but if you're using a yum based package manager, you're in luck. This article will cover how to get the source package, setup your environment, then patch the sources for viewing.

{% include tip.html content="You can do a lot of this work in either a docker container or virtual machine to prevent contaminating your system with build artifacts." %}

Let's start by setting up the environment. I always do this as a regular user (i.e. not root) to avoid accidentally installing the package. Begin by installing the `rpm-build` package which we'll use later to patch the sources.

{% highlight bash %}
$ sudo dnf install -y rpm-build
{% endhighlight %}

Next you need to tell the system where to place these sources you'll later unpack. You do that by setting the `%_topdir` variable in a file named `~/.rpmmacros`. I'll simply show you mine below.

{% highlight bash %}
$ cat /home/mw007/.rpmmacros
%_topdir /home/mw007/src/rpmbuild/
{% endhighlight %}

Now you'll need to create a few directories to sort of bootstrap the build environment. We'll create the directory structure that we referenced in the last step.

{% highlight bash %}
$ mkdir -p ~/src/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
$ mkdir ~/src/rpmbuild/RPMS/{i386,i486,i586,i686,noarch,athlon}
{% endhighlight %}

Next you need to actually get the source package. It's very simple and requires a single command.

{% highlight bash %}
$ yumdownloader --source <package-name>
{% endhighlight %}

Once you have the source package, you can unpack it with the command below. This will use the `%_topdir` setting and unpack the sources where ever you specified.

{% highlight bash %}
$ rpm -ivh package-X.X.X.src.rpm
{% endhighlight %}

Everything up to this point has been setting things up. With the sources unpacked, you're ready to start working them. The next step will have you enter the `~/src/rpmbuild` directory and patch the sources.

{% highlight bash %}
$ cd ~/src/rpmbuild
{% endhighlight %}

At this point you have a choice to make. Most packages will have some build dependencies, but we don't want to build; we want to view the source. Your choice is that you can edit what's called a SPEC file to remove the build dependencies, or you can install the build dependencies. I'll assume you're using a disposable environment (like a container or VM), and would like to take the path of least resistence. We'll be installing the build dependencies.

For the command below, you may need to add additional repositories in order to resolve all the dependencies. You'll also need to use the actual SPEC file name, not "package.spec".

{% highlight bash %}
$ sudo yum-builddep SPECS/package.spec
{% endhighlight %}

You'll now prepare the sources. This is `-bp` option to the rpmbuild command.

{% highlight bash %}
$ rpmbuild -bp SPECS/package.spec
{% endhighlight %}

Your patched sources are now under `~/src/rpmbuild/BUILD/package-X.X.X`!
