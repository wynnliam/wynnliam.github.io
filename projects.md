---
layout: page
title: Projects
permalink: /projects/
---

These are some of the projects I've worked on over the past few months. While there are
more on my github page, these are my most important ones. To see the rest of my work, go
[here]({{ "https://github.com/wynnliam" | absolute_url }})

# Raycaster
For my Full Stack Web Development final project, I implemented a ray caster based renderer
for pseudo-3D environments. It was entirely front-end. It render both indoor and outdoor
environments, including textures floors, ceilings, and walls. Also, it supports sprite
rendering, so levels can have props and other objects in them. In addition, you can have
rendered sky textures. After I completed the class, I designed a simple level file parser
so you can load levels from file and render them.

Click [here]({{ "https://web.cecs.pdx.edu/~wynnliam" | absolute_url }}) to see it in action.

Click [here]({{ "https://github.com/wynnliam/CS410p_Raycaster" }}) to see the source code.

# An Engine of Ice and Fire
An Engine of Ice and Fire (AEOIAF) is a near fully-functioning tile based 2D top down game
engine written in C++. It features an rich actor system, a game state manager, basic texture
rendering, AI, and a working weapon system. I developed it over the course of about two years.

Click [here]({{ "https://github.com/wynnliam/an_engine_of_ice_and_fire" | absolute_url }}) to get it.

# Blazonry Parser
The Blazonry Parser is an experiment in language theory and functional programming.
Essentially, the user supplies a formal description of a blazon of arms, and the system
will verify that the description is valid. We are able to do this because blazons actually
have a formal grammar that dictate their structure. Although the official grammar is much more
in depth, the principle still applies.

Further, the grammar I use describes a regular language. In fact, the number of strings the grammar
accepts is six. Thus, this program could easily done with an if-statement. However, I used
a recursive function to practice iteration in a functional paradigm.

Click [here]({{"https://github.com/wynnliam/blazonry_parser" | absolute_url }}) to get it.
