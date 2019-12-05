---
layout: default
title: Projects
permalink: /projects/
---

These are some of the projects I've worked on over the past few months. While there are
more on my github page, these are my most important ones. To see the rest of my work, go
[here]({{ "https://github.com/wynnliam" | absolute_url }})

# raycore
A simple, ray casting based, pseudo-3d game engine. It can handle both indoor and outdoor
scenes, including textured floors, ceilings, and walls. In addition, it supports sprite rendering
so environments can have props and other objects in them.

In addition to rendering environments, there is an entity system that allows game logic to be
included in levels. Now you can actually do things in the game, instead of just walking around.

Finally, I wrote a simple file loader specifically for raycore. This allows you to load levels
from an external file without having to recompile the entire project.

For the source code, go [here]({{ "https://github.com/wynnliam/raycore" | absolute_url }})

![Figure]({{"/assests/raycaster_implementation/part_3/sky_floor_ceil.jpg" | absolute_url}})
![Figure]({{"/assests/raycaster_implementation/part_2/sprites.png" | absolute_url}})


---


# An Engine of Ice and Fire
An Engine of Ice and Fire (AEOIAF) is a near fully-functioning tile based 2D top down game
engine written in C++. It features an rich actor system, a game state manager, basic texture
rendering, AI, and a working weapon system. I developed it over the course of about two years.

Click [here]({{ "https://github.com/wynnliam/an_engine_of_ice_and_fire" | absolute_url }}) to get it.

![Figure]({{ "/assests/aeoiaf_demo.png" | absolute_url }})


---

# Texture Generator
Texture Generator is a machine learning experiment that creates texture images
using the variational autoencoder model. Essentially, I give the model a series
of images of the kinds of textures I want, and then it will use those to build
new samples from normally distrubted noise. The kinds of textures it produces
are 64 x 64 pixel grayscale bitmaps.


Click [here]({{"https://github.com/wynnliam/texture_generator" | absolute_url }}) to get it.


![Figure]({{"/assests/texture_generation_output/0.png" | absolute_url}})
![Figure]({{"/assests/texture_generation_output/1.png" | absolute_url}})
![Figure]({{"/assests/texture_generation_output/2.png" | absolute_url}})

---

# alexander
alexander is a on-going experiment in realtime AI simulations using flock/boid behaviors, flow fields, and continuum crowds.
I want to simulate environments with large groups of agents moving around. This is created
with the help of the Unity Engine, so collisions between agents work seamlessly. The best
description is a neat little video:

[![Watch the video]({{ "/assests/flow-field-post/boid-demo.png" | absolute_url}} )]({{ "/assests/flow-field-post/boids-in-action.webm" | absolute_url }})
(Click to watch video)

Click [here](https://github.com/wynnliam/alexander) to get it.


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
