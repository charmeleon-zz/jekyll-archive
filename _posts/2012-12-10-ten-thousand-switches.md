---
layout: post
title: "Ten thousand switches"
description: ""
category: Thoughts
tags: ["javascript", "performance"]
---
{% include JB/setup %}
For roughly the past two weeks I've been familiarizing myself with Javascript;
it's an interesting language and it happens to dominate browser-based
interaction, in case you haven't heard. Though I'm by no means an expert
I'd like to fuel some discussion on a pet peeve of mine.

I've perused some scripts, and it pains me to see blocks of code that made of
if statements comparing integer values. Something like this:
{% highlight javascript linenos %}var resultVar;

if (someVar === 1){
  resultVar = "one";
} else if (someVar === 2) {
  resultVar = "two";
} else if (someVar === 3) {
  resultVar = "three";
} else if (someVar === 4) {
  resultVar = "four";
}{% endhighlight %}
This is the gist of logic that I've been witness to. To avoid sounding
condescending, I'd like to be succint: __this is a case for a switch statement.
__ This is one of those rare cases where speeding up your script does not come
at the cost of making your code harder to read, if anything a switch statement
here would make this much more legible. Not only that, but there's considerable
speed to be gained by using switch statements \(depending on how many times a
script is accessed\).

To show this, I profiled [a simple script](https://gist.github.com/4255776),
and ran it five times. The script loops over an if condition block, then after
a switch block, 10,000 times, and the time is logged on the console. Over five
runs, the switch block completed with average time of __0.2898s__, while the
if block ran an average of __1.1226s__. Note that any sense of security
we may have gained by previously using `===` is equally afforded by a
`parseInt` function call. In short, if you are testing equality on integer
values, please use `switch`!

__EDIT:__ There's a decent post on the front page of Hacker News that expands
a bit on JS conditionals:
[Find it here](http://rmurphey.com/blog/2012/12/10/js-conditionals/)
