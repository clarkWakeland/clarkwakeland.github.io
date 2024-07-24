---
layout: page
title: A test writing tile
description: Test post for a write
img: assets/img/writing.webp
importance: 1
category: SciFi
related_publications: false
---

[Link to Github Page](https://github.com/clarkWakeland/cplusplusprojects/tree/master/Turing%20Machine)

<div class="row">
    <div class="col"></div>
    <div class="col-lg-">
        {% include figure.liquid loading="eager" path="assets/img/turingOpt.gif" title="Turing Machine Operation" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col"></div>
</div>

A simple Turing Machine defined as a Deterministic Finite Automota (DFA). This implementation takes in a string of *N* number of `a` characters and returns *N<sup>2</sup>* number of `a`'s. The full project can be found on my github [here](https://github.com/clarkWakeland/cplusplusprojects/tree/master/Turing%20Machine). The alphabet and transition states can be updated for any algorithm you can think of, and it will be performed by the DFA. Because this is Turing complete, it could theoretically be used to simulate anything a normal computer can, although that would be really time consuming and I wouldn't recommend it. That would be like [trying to travel around the world only by swimming](../../blog/2024/magellan).
