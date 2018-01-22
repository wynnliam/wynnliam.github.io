---
layout: post
title: "Ulysses First Release"
date: 2018-01-22 8:31:00 +0000
categories: ulysses news
---

I have my first project up and running: Ulysses. 

Check it out [here]({{ "/projects/" | absolute_url }}).

Ulysses will be an on-going development. So what you see is by no means
a final product. At the moment, it only generates a colorized height map.
Later on, I'd like to add more features to it. For now, in a sentence, it's
just a fun experiment in procedural terrain generation that tries to
make scientifically plausible worlds.

I'll write up a post on how I generated these worlds over the course of
this week. There really isn't a whole lot at play here, but it's still
worth it to know how I did these things.

In the mean time, here are a couple of worlds of size 256 x 128:

![World 1]({{"/assests/1_22_2018_world_1.png" | absolute_url}})

![World_2]({{"/assests/1_22_2018_world_2.png" | absolute_url}})

![World 3]({{"/assests/1_22_2018_world_3.png" | absolute_url}})

Note that blue is oceans, green is land, and white is mountainous land. In these
worlds, there are 60 tectonic plates used, 20% of the world is land, and 5% of the
world is mountains. The other 75% of the world is oceanic.

What I love most about this is how configurable it is. You can not only change
the dimensions of the world, but also stuff like the number of tectonic plates,
the amount of land, mountains, etc. This was a goal of mine, and I'm really happy
to see this manifest into a final product. Further, the diversity of the worlds
is awesome. A lot of other algorithms that just use perlin noise, for example,
produce worlds that look the same. In addition, these worlds have all kinds of
features, from continents to island chains and everything in between.

An interesting thing to note: these worlds wrap around the edges. So the top
of a world and its bottom are seamless (as well as the left and right sides).
Also, the generation process is very quick, even with worlds as large as 1024
pixels by 1024 pixels. I have not benchmarked this in any way though. That
would probably be a worth-while endeavor.
