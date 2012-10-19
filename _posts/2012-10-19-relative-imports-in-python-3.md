---
layout: post
title: "Relative Imports in Python 3"
description: ""
category: 
tags: []
---
{% include JB/setup %}
<p>I've noticed a good number of programmers who use python will structure their entire code into a single file, a practice that I'm not entirely too fond of. I like to structure my code into different packages. Python makes creating packages relatively painless - all you need is a __init__.py file in the folder where you wish to create a package.</p>
<p>This is great when you're creating a library that doesn't need to interact with its own packages, but what if you need to access subpackages? The answer isn't so straightforward anymore. Let's take look at a simple example:</p>
{% highlight bash %}package/
  __init__.py
  main.py
  foo/
    __init__.py
    foofirst.py
    foosecond.py
  bar/
    __init__.py
    barfirst.py
    barsecond.py{% endhighlight %}

<p>You might be fooled into thinking that <a href="http://www.python.org/dev/peps/pep-0328/">PEP 328</a> appropriately covers this, and you'd be wrong. Here's a case where it doesn't work:</p>
{% highlight python linenos %}Python 3.3.0 (default, Oct  3 2012, 11:57:03) 
[GCC 4.6.3] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import glob
>>> glob.glob("*.py")
['__init__.py', 'torrent.py', 'peer.py', 'connection.py', 'readfile.py', 'tracker.py', 'filequeue.py']
>>> import filequeue # worked
>>> import torrent
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "./torrent.py", line 6, in <module>
    from . import peer # the problematic line
SystemError: Parent module '' not loaded, cannot perform relative import
>>> from . import filequeue # equivalent to line 7, according to PEP 328
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
SystemError: Parent module '' not loaded, cannot perform relative import{% endhighlight %}
As you can see, "from . import filename" fails spectacularly -- oddly enough, '.' refers to the current directory. The error is deceivingly descriptive. My first instinct was to check sys.path for the existence of the current directory, and it was there. Before figuring how to get it to work, I received various ideas. One of them was to manually set the PYTHONPATH, but in my opinion that's the equivalent of burning the forest to scare a gnat. I was also told to just stick to absolute paths, which seems silly.

Suffice it to say that if you use relative imports, your python scripts can have only a single entry point - whatever resides in the parent package directory [in the sample file structure given above, that point would be main.py]. Anything else will have you pulling your hair out (or hacking away at __init__.py, which I would not endorse).

(I should like to thanks ballingt from hackerschool for getting this answer for me from #python-unregistered. I banged my head against my desk for a full day before he saved me.)
