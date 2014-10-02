---
layout: post
title: "Parsers... Parsers All The Way Down"
description: ""
category: Misc
tags: [epiphany]
---
{% include JB/setup %}

## Turtles all the way down

In order to not bore you with my attempt at an explanation,
here's the first Wikipedia paragraph for [Turtles all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down):

> **"Turtles all the way down"** is a jocular expression of the infinite regress problem in cosmology posed by the "unmoved mover" paradox. The metaphor in the anecdote represents a popular notion of the myth that Earth is actually flat and is supported on the back of a World Turtle, which itself is propped up by a chain of larger and larger turtles. Questioning what the final turtle might be standing on, the anecdote humorously concludes that it is "turtles all the way down".

## Parsers... Parsers all the way down

Today I was working on a scoring engine for problems
which are imported by parsing XML data. After only a few 
hours I had a working engine, I think the fact that I've
started writing parsers for nand2tetris helped a lot.
That's when I realized that this would be the third parser
I would write this week. One for `.asm` to `.hack`, one 
for `.vm` to `.asm`, and this one, which was for work, 
but was a parser all the same. Not only that, but I was
using SimpleXML, an XML parser.

I think people tend to focus on the mystery of binary so 
much that until now, I have naively imagined my computer 
as being powered mostly thanks to these billions of
transistors. While it's true that transistor states and
boolean logic serve as the foundation of programming
logic, I think that not enough credit goes to the
ingenuity and levels of abstraction that parsers
enable. After all, given the abundance of modern day
languages and runtimes, modern programming seems to really
be driven by parsers... parsers all the way down.

\(In the end, it turns out that I cannot automate the 
scoring, but that's only because the data is not in the 
standardized format that it's supposed to be in\). 
