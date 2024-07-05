---
title: Interesting problems encountered working with satellites
layout: post
date: 2024-06-21 18:57:41
tags:  aerospace programming
description: Some interesting problems I've encountered working on satellites and how we solved them
---

### Intro

I'm considering making this a living document and writing down the interesting problems I encounter working with satellites as they come up.

1. CryoCooler State traversal

2. Subframe Set Covering test

3. Satellite commnad/telemetry database versioning

> *I won, because the gardener always stops to offer peace. And when they do, I always strike.* 
><br/><br/>
> *But by then, it didn't matter. The game was over. The garden had given birth to creation, the rules were in place, and there would never be a second chance. We played in the cosmos now. We played for everything...*
> <br/><br/>
> *...Some poor mutant discovered that it could collect carbon compounds much faster if it stopped grazing on the bacterial mat and started dissecting and eating the lumps of predigested carbon all around it: its neighbor oozeballs.*
><br/><br/>
> *It couldn't help but do it. It couldn't help but thrive. We don't get a choice about the rules. We just play the game.*
> *It was the first defector - the first predator. It changed everything.*
> <br/><br/>
> -- The Winnower. T=0. The Cambrian Explosion. *[Destiny 2: Unveiling](https://www.ishtar-collective.net/categories/book-unveiling)* 