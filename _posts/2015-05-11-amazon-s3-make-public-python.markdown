---
layout: post
title:  "Set public-read permission to an entire folder in Amazon S3."
date:   2015-05-11 15:25:12
categories: python boto aws3 amazon web services acl public folder
---


Make sure to install [boto][boto-python].  
`pip install boto`.

{% highlight python %}
AWS_ACCESS_KEY_ID = ''
AWS_SECRET_ACCESS_KEY = ''
AWS_STORAGE_BUCKET_NAME = 'your-bucket'
AWS_FOLDER = 'media/'

from boto.s3.connection import S3Connection
conn = S3Connection(AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
bucket = conn.get_bucket(AWS_STORAGE_BUCKET_NAME)

for key in bucket.list(AWS_FOLDER):
    key.set_acl('public-read')
{% endhighlight %}

[boto-python]: https://github.com/boto/boto "Python interface to Amazon Web Services"
