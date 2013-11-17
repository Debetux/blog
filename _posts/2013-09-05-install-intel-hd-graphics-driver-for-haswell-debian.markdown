---
layout: post
title:  "Install the lastest Intel HD Graphics Driver for Haswell in Debian."
date:   2013-08-24 00:35:35
categories: debian intel driver
---

The brand new Intel processors, and their Intel HD Graphics 4600 are not supported yet in Debian Stable nor Debian Testing.

You'll need to intall xserver-xorg-video-intel from experimental. Here's how to do :

Add this new line to /etc/apt/sources.list :
{% highlight bash %}
  deb http://ftp.debian.org/debian experimental main
{% endhighlight %}

Then :
{% highlight bash %}
  apt-get update
  apt-get -t experimental install xserver-xorg-video-intel
{% endhighlight %}

Enjoy !
