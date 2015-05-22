---
layout: post
title:  "Parse CSV exported with Mac Excel (wrong encoding)."
date:   2015-05-22 10:39:40
categories: python ruby encoding excel csv mac_roman utf-8
---


If you already tried to parse a csv exported with Mac Excel, you probably ran into few issues, like :
{% highlight python %}
  File "/usr/local/Cellar/python3/3.4.3/Frameworks/Python.framework/Versions/3.4/lib/python3.4/csv.py", line 96, in fieldnames
    self._fieldnames = next(self.reader)
  File "/usr/local/Cellar/python3/3.4.3/Frameworks/Python.framework/Versions/3.4/lib/python3.4/codecs.py", line 319, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x8e in position 44: invalid start byte  
{% endhighlight %}

It's because this file use `mac_roman` encoding, and `;` delimiter.
{% highlight python %}
import csv

file = 'ing.csv'
with open(file, encoding='mac_roman') as csvfile:
    reader = csv.DictReader(csvfile, delimiter=';')

    headers = reader.fieldnames
    print(headers)

    for row in reader:
        print(row)
{% endhighlight %}
