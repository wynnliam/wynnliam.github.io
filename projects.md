---
layout: page
title: Projects
permalink: /projects/
---

These are some of the projects I've worked on over the past few months. While there are
more on my github page, these are my most important ones. To see the rest of my work, go
[here]({{ "https://github.com/wynnliam" | absolute_url }})

# Ulysses
Ulysses is an on-going project of mine. At the moment, it is an experiment in procedural
terrain generation. In the future, I would like to add more world-building elements to give
it some depth. Part of the goal of this project is to produce worlds that are to some capacity
scientifically plausable, while also using relatively efficient algorithms.

For now, all it does is produce an image of a randomly generated world. Specifically, it
generates a colorized height map of a world. It does so by creating a series of maps that
specify different features of the world, and then combines these maps. Specifically, it
generates three maps: a tectonics map, crust thickness map, and an orogenic map. It then
combines these to produce a final height map of the world.

Click [here]({{ "https://github.com/wynnliam/ulysses" | absolute_url }}) to get it.

# Blazonry Parser
The Blazonry Parser is an experiment in language theory and functional programming.
Essentially, the user supplies a formal description of a blazon of arms, and the system
will verify that the description is valid. We are able to do this because blazons actually
have a formal grammar that dictate their structure. Although the official grammar is much more
in depth, the principle still applies.

Further, the grammar I use describes a regular language. In fact, the number of strings the grammar
accepts is six. Thus, this program could easily done with an if-statement. However, I used
a recursive function to practice iteration in a functional paradigm.

Click [here]({{"https://github.com/wynnliam/blazonry_parser" | absolute_url }})
