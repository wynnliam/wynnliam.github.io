---
layout: page
title: Projects
permalink: /projects/
---

These are some of the projects I've done over the past few months. At some point, I'd
like to get all of these on github (all of them do use git for version control). In the
meantime, I will just have distributable tar files with the latest source code.

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

Click [here]({{ "/projects/ulysses.tar" | absolute_url }}) to get it.
