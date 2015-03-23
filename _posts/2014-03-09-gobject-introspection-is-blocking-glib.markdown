---
layout: post
title:  "Gentoo : gobject-introspection is blocking dev-libs/glib"
date:   2014-03-09 15:10:40
categories: gentoo emerge glib gobject
---

Yesterday, I've done :
{% highlight bash %}
emerge --update --ask @world
{% endhighlight %}

And I've got something like this :
{% highlight bash %}
<dev-libs/gobject-introspection-1.38" is blocking dev-libs/glib-2.38.2
{% endhighlight %}

Solved with something like that :
{% highlight bash %}
emerge --ask =dev-libs/gobject-introspection-1.38.0
# Or the latest version of the package available :
# http://packages.gentoo.org/package/dev-libs/gobject-introspection
{% endhighlight %}
