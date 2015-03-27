---
layout: post
title:  "Receive and send JSON data with Django."
date:   2015-03-26 23:06:02
categories: django python json jquery ajax
---


With an ajax request (jQuery here):

{% highlight javascript %}
var form_data = {
    name: 'Sherlock HOLMES',
    age: 34,
};

$.ajax({
    type: "POST",
    url: '/ajax/',
    data: JSON.stringify(form_data),
    success: function(data) {
        console.log('data');
    },
    dataType: 'json'
});
{% endhighlight %}

You can load the json data easily in your view, using `request.body`:

{% highlight python %}
from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
import json

@csrf_exempt
def add_ajax(request):
    form_data = json.loads(request.body.decode('utf-8'))

    return JsonResponse({'success': 'true'})
{% endhighlight %}
