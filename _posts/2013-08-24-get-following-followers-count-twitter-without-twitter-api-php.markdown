---
layout: post
title:  "Get following/followers without Twitter API in PHP."
date:   2013-08-24 00:35:35
categories: php twitter
---

[FR]: Obtenir le nombre de following/followers d'un compte Twitter sans l'API en PHP.

Yay. It's simple as :

{% highlight php %}
<?php
/**
 * Get following/followers count of an Twitter account, without using 1.1 API
 * @author Lancelot HARDEL
 * @param username That's clear
 * @param cache If you want to enable cache (or not)
 * @param cachetime Time that the cachefile is valied
 * @param stat_name you can use two values : followers, or following
 * @return int
 */

function getTwitterStatsCount($username, $cache = false, $cachetime = 1800, $stat_name = 'followers'){

    $cachefile = 'cached-'.$stat_name.'-'.$username; # Name of the cached file 

    # Serve from the cache if it is younger than $cachetime
    if (file_exists($cachefile) && time() - $cachetime < filemtime($cachefile)):
        return file_get_contents($cachefile);
    else:
        # Get Twitter data :
        $twitter_data = file_get_contents('https://mobile.twitter.com/'.$username);

        # Regex to get follower count :
        preg_match('#'.$stat_name.'">\n.*<div class="statnum">([0-9,]*)</div>#', $twitter_data, $match);

        # Some operation :
        $twitter['count'] = str_replace(',', '', $match[1]);
        $twitter['count'] = intval($twitter['count']);
        
        # Write cache :
        if($cache){ $cached = fopen($cachefile, 'w'); fwrite($cached, $twitter['count']); fclose($cached); }

        return $twitter['count']; # Done !
    endif;
}

echo getTwitterStatsCount('debetux', true);
{% endhighlight %}
