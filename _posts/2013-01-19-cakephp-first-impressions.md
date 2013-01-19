---
layout: post
title: "CakePHP: First Impressions"
description: "Thoughts after a brief survey of CakePHP"
category: frameworks
tags: [php, cakephp, frameworks, php frameworks]
---
{% include JB/setup %}
Overview
========
After successfully setting up CakePHP and doing a quick run-through on the blog
tutorial, my initial impression is that the framework does live up to the hype.

----

## The Good

While doing the tutorial, after setting up the add and view sections of the
controller, I was able to do the edit and delete sections without looking at
the tutorial. In my humble opinion, the level of abstraction is perfect for
quick data retrieval and storage. More than that, I'm mostly fond of its
conventions, with its predominant CamelCase convention, with the exception of
database tables. I'm slightly worried about non-obvious data retrieval, such
as the case of many-to-many relationships, but from what I've seen I'm
optimistic that such an issue has already been ironed out.

Over on the View side, I had the biggest shock of all. I was able to setup a
form with just this:
{% highlight php linenos %}
<h1>Add Post</h1>
<?php
echo $this->Form->create('Post');
echo $this->Form->input('title');
echo $this->Form->input('body', array('rows' => '3'));
echo $this->Form->end('Save Post');
?>
{% endhighlight %}
There was no need for me to write any HTML for a basic form, just the way it
should be! I'm always baffled by how much time I spend thinking on what id I
should issue for a form, and then refactoring when I decide to rename it. This
framework does away with all of that. I was initially concerned that the form
didn't have a `<fieldset>` set of tags, but a quick look to the documentation
for the Form object \(called [FormHelper](http://book.cakephp.org/2.0/en/core-libraries/helpers/form.html)\)
lead me to what I suspected: the option is there, if you want it. Which brings
up a new point, its documentation has so far been fantastic, with a short
summary of a function's outcome and use cases, including some edge cases.

Lastly, on the Controller side, I was most pleasantly surprised by the fact that
CakePHP has abstracted form validation, which is an often repetitive task prone
to error.

## The Bad

Though setting up a form was ridiculously quick and painless, having to setup
the same form view twice, once for adding and once for editing, felt terribly
unDRY. Though this was a very short and simple form, most forms are non-trivial
and depending on how many your application will need, I can see this becoming
very old very quickly. Hopefully this was thought of somewhere on Cake's API,
but the fact that it wasn't leaves a sour taste in my mouth.

I was also a bit disappointed that the data validation only took place server-
side. Again, I'm hoping this is something that's already found on the API,
because I was disappointed when I hit the 'Add' button on a blank form and
it took a network trip to determine that my blank request couldn't be added to
the database -- it should've never been sent in the first place.

## The Ugly

The HTML output is awful. I'm very uneasy looking at HTML output that isn't
properly formatted \(which is why I never look at my own blog's HTML, Jekyll
also doesn't concern with how its output looks\). In CakePHP's defense, if
you look at the HTML with modern developer tools such as Firebug, you don't
need to worry about what your output actually looks like, they will nest and
format it correctly. Still, I'm free to not like it \(I don't\), and hopefully
this can be addressed in time.

## In Short

CakePHP is so good that it has me thinking twice over the direction of the 
miniframework I was thinking of creating, Prepared. Cake already does everything
that I was thinking of implementing: abstraction of the more tedious portions
of HTML output, data validation \(again, not yet sure on client-side
validation\) and retrieval for MySQL using prepared statements, and fairly
rigorous file conventions. In fact, I'm reevaluating whether another lightweight
is needed, and perhaps my efforts could be better spent somewhere else, possibly
as a Cake contributor.
