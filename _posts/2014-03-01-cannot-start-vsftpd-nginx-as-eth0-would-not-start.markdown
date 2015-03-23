---
layout: post
title:  "Solving: \"ERROR: cannot start vsftp as net.eth0 would not start\""
date:   2014-03-01 14:59:18
categories: gentoo nginx vsftpd error
---

Since systemd/udev [automatically assign predictable interface names][PredictableNetworkInterfaceNames] for all local Ethernet, WLAN and WWAN interfaces, you might find some difficulties to start services like nginx, or vsftpd.

{% highlight bash %}
 * Caching service dependencies ...                                    [ ok ]
 * Bringing up interface eth0
 *   ERROR: interface eth0 does not exist
 *   Ensure that you have loaded the correct kernel module for your hardware
 * ERROR: net.eth0 failed to start
 * Bringing up interface eth0
 *   ERROR: interface eth0 does not exist
 *   Ensure that you have loaded the correct kernel module for your hardware
 * ERROR: net.eth0 failed to start
 * ERROR: cannot start netmount as net.eth0 would not start
 * ERROR: cannot start nginx as net.eth0 would not start
{% endhighlight %}


  
  
First of all, spot the name of your interface, with ifconfig :
{% highlight bash %}
➜  ~  ifconfig
enp3s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.94  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::be5f:f4ff:fe45:84ad  prefixlen 64  scopeid 0x20<link>
        ether bc:5f:f4:45:84:ad  txqueuelen 1000  (Ethernet)
        RX packets 73401  bytes 82817338 (78.9 MiB)
        RX errors 0  dropped 273  overruns 0  frame 0
        TX packets 53573  bytes 6415454 (6.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16
{% endhighlight %}

So mine is enp3s0.  
nginx, vsftpd (etc...) try to bring up eth0, by doing something like : `/etc/init.d/net.eth0 start`. But, I don't have an interface named eth0.  
`/etc/init.d/net.eth0` is simply a bash script, which try to do something with net.\<interface_name\>.  

We could simply rename the script `net.enp3s0`, but the service will try to start `net.eth0`. The only solution I found, is to replace all the occurences of `${RC_SVCNAME#*.}` by `enp3s0`.

[PredictableNetworkInterfaceNames]: http://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/ "Predictable Network Interface Names"
