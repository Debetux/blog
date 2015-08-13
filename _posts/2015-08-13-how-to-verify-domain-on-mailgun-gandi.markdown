---
layout: post
title:  "How to verify domain on Mailgun (Gandi)"
date:   2015-08-13 11:26:56
categories: mailgun dns bind gandi smtp
---

Here are some advices :

- pic.\_domainkey instead of pic.\_domainkey.yourdomain.com
- values in double quotes
- replacing yourdomainc.com by @ for the hostname

{% highlight text %}
@ 10800 IN SPF "v=spf1 include:mailgun.org ~all"
@ 10800 IN TXT "v=spf1 include:mailgun.org ~all"
pic._domainkey 10800 IN TXT "k=rsa; p=aaaaaaaaa"
{% endhighlight %}
