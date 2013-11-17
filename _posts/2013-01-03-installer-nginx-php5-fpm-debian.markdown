---
layout: post
title:  "Installer nginx + php5-fpm sur Debian."
date:   2013-01-03 12:00:00
categories: php debian nginx
---

J'ai bien galéré à installer nginx ainsi que php5-fpm. Mais j'ai enfin réussi, après les bugs de pages blanches, etc.

Si vous venez juste d'installer le serveur, je vous conseille de faire un petit :

{% highlight bash %}
apt-get update && apt-get upgrade
{% endhighlight %}

Il vous suffit ensuite d'installer nginx et php :
{% highlight bash %}
apt-get install nginx php5-fpm
{% endhighlight %}

Au niveau de la config, rien de bien compliqué, les fichiers se trouvent dans /etc/nginx/  
Les confs des sites activés des sites sous nginx se trouvent dans /etc/nginx/sites-enabled/

Un exemple de fichier pour la conf d'un site, c'est tout simple :

{% highlight nginx %}
server {

        listen   80; ## listen for ipv4

        server_name  shre.me; # remplacez par votre ndd (ou localhost en local)

        access_log  /var/log/nginx/shre.access.log; # les logs d'accès à votre site
        root /home/shre.me; # l'endroit où se trouvent vos fichiers
        index index.php index.html; # les fichiers utilisés par défaut
        autoindex off; # pour éviter que quelqu'un puisse lister vos répertoires

        #error_page  404  /404.html;

        # La partie php
        location ~ \.php$ {
               include fastcgi_params;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
               # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

               # With php5-cgi alone:
               # fastcgi_pass 127.0.0.1:9000;
               # With php5-fpm:
               fastcgi_pass unix:/var/run/php5-fpm.sock; # si vous utilisez le socket
               fastcgi_index index.php;

        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        location ~ /\.ht {
                deny  all;
        }
}

{% endhighlight %}

Cette page est plus un mémo qu'autre chose. Si vous voulez plus de précisions, vous pouvez demander à [DuckDuckGo][ddg].

[ddg]:    https://duckduckgo.com/
