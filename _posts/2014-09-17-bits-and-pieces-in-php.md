---
layout: post
title: "Bits and Pieces in PHP"
description: ""
category: 
tags: [PHP, UTF-8]
---
{% include JB/setup %}

Something unusual happened this past week. A 
co-worker found a string which did not equal (`==`) another
string that read the exact same. The string was simple 
enough: `Earth's Systems`.

My first instinct was to go to [Write Code Online](http://writecodeonline.com/php/) and try `"Earth's Systems" == 
"Earth's Systems"`. This of course is (and was) true. Our
codebase appeared to say otherwise, so since this string
was in an array whose keys matched its values, I tried
`isset($array[$cat])`, and this was also `false`.

For a little context, `$cat` is the string `Earth's
Systems`, which is submitted by the browser via `POST`,
and `$array` is a collection of strings retrieved from the
database that is supposed to include `Earth's Systems` . And it appeared to be: when the array contents were printed via `print_r` I could read it clear as day, except
that both `in_array()` and `isset()` were saying otherwise. 

Next, I printed the `strlen()` of both `$cat` and each string in `$array`. Surprise! `strlen($cat)` was 15, while the 'equivalent' string found in `$array` was 21 (I should 
mention that we make use of the multibyte functions, so
`strlen` really is `mb_strlen`). Determined to find out what this difference was and echoing the string was going
nowhere, I wrote a for loop to build an array with the
actual string contents:
{% highlight php linenos %}
<?php
for ($array as $string) {
    $chrs = array();
    for ($i=0, $max = strlen($string); $i < $max; $i++) {
        $chrs[] = chr(ord($string[$i]));        
    }
    print_r($chrs);
}
?>
{% endhighlight %}

This was the contents of the relevant string:
{% highlight php %}
Array
(
    [0] => E
    [1] => a
    [2] => r
    [3] => t
    [4] => h
    [5] => &
    [6] => #
    [7] => 8
    [8] => 2
    [9] => 1
    [10] => 7
    [11] => ;
    [12] => s
    [13] => 
    [14] => S
    [15] => y
    [16] => s
    [17] => t
    [18] => e
    [19] => m
    [20] => s
)
{% endhighlight %}

And there it is, rearing its ugly head! The data had been entered into the database using `&#8217;` for `'` , which is the decimal HTML entity. Since the HTML-encoded version was not what we wanted, fixing it in the database was all that was needed.

Not since my dealings with Python 3 had I encountered an
encoding issue, so this was definitely fun (and this time
the encoding issue was my second guess, coming in only
after PHP being insane).