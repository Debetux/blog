---
layout: post
title:  "Configuration serveur Debian post-install."
date:   2013-11-13 12:00:00
categories: debian
---

J'ai tendance à recommander l'installation de Debian Stable pour un serveur en production. Debian, par préférence tout simplement. Stable, car je n'ai jamais eu aucun problème, et en général les logiciels s'accordent bien entre eux.

Une fois que vous avez commandé votre dédié, qu'il est tout chaud, tout beau, comme configurer tout cela ?

Tout d'abord, commençons par mettre à jour le serveur, et a install ntp pour avoir un serveur toujours à l'heure :
{% highlight bash %}
apt-get update && apt-get upgrade;
apt-get install ntp ntpdate;
{% endhighlight %}

Certains indiquent de toucher aux fichiers `/etc/hostname` ou `/etc/hosts`, mais ils sont souvent déjà préconfigurés. Je préciserais cette partie plus tard, mais vous en aurez sûrement pas besoin.

// ECRITURE EN COURS