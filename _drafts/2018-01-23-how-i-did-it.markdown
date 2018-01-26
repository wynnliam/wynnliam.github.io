---
layout: post
title: "How I Did It: Generating a Height Map in Ulysses"
date: 2018-01-23 17:29:00 +0000
categories: ulysses how-to
---

Now that Ulysses is live, I think it'd be a neat idea to explain how it does
its only feature: generating heightmaps. My goal was to create worlds that
were not only interesting, but also scientifically plausible.

Before I get into how I did this current version of Ulysses, I think it
would be neat to get into how I originally attempted Ulysses.

# Worlds With Just Perlin Noise

Some of my earliest experiments were using nothing other than Perlin Noise.
For many purposes, this will likely work just fine. However, I began to notice
similar patterns emerge in the kinds of worlds being produced. As an example,
here are some worlds I whipped up using the [not-Perlin-Noise-algorithm](devmag.org.za/2009/04/25/perlin-noise/).

![Example-1]({{"/assests/hidi_1_26_2018/1.png" | absolute_url}})

![Example-4]({{"/assests/hidi_1_26_2018/4.png" | absolute_url}})

I should breifly mention that of these five examples, these two were the most
distinct looking.

![Example-2]({{"/assests/hidi_1_26_2018/2.png" | absolute_url}})

![Example-3]({{"/assests/hidi_1_26_2018/3.png" | absolute_url}})

![Example-5]({{"/assests/hidi_1_26_2018/5.png" | absolute_url}})

Now, to be fair, simple Perlin noise worlds have a lot going for them. First,
their simple to generate, and very efficient. In fact, using Minecraft as an
example (which I think early versions used a form of the [Diamond-Square Algorithm](https://en.wikipedia.org/wiki/Diamond-square_algorithm)), you can have
worlds that are infinite. Since a fractal world is quick and easy, you
don't even need to keep them in memory.

However, it did not take long for me to start getting worlds that begin
to show similarities. While the first two worlds are fairly unique, the
latter three show commonalities. Mainly, they begin to have a gaping ocean
in the middle of the world. Also, the mountain ranges only appear as you
move inland. In my early experiments, I went through upwards of a hundred
worlds as I tweaked parameters. I would bet that if you did the same
you'd start to think every world was the same too.

Further, I'm not all too convinced these are realistic worlds too, at least
if the goal is to mimic Earth.

I tried other algorithms and tweaked all kinds of parameters for a long time
before I decided that a fractal or simple noise approach was not going to
cut it. I did a little digging and found a neat system that does a
full-fledged plate tectonics simulation. In fact, my two favorite ones
are found [here](https://www.gamedev.net/forums/topic/623145-terrain-generation-with-plate-tectonics/) and [here](https://davidson16807.github.io/tectonics.js/).

A long time ago, I tried doing my own implementation of something like those
two. However, I occured to me how woefully outclassed I was. Thus, I decided
to try a new approach.

# Snapshots of Earth

I got an idea for my generator when I was reading some texts on Geology.
I asked myself, "What if, instead of a realtime plate tectonics simulation,
we took a snapshot of the world at some time t, and tried to 'guess' what
the height map would look like?"

To make this idea work, I would need something to the effect of 'metadata'
about the world. Essentially, I would have to quantify the height of any
given point on the world as a function of other pieces of information. In
other words, "if I new everything about a point except its actual elevation,
could I guess if it's submurged by an ocean or not?" This seems like a strange
idea, but as it turns out, there's a lot of metrics by which this idea holds
weight.

For this idea to work, we need to select the data we're interested in. Before
I do this, however, let's define a couple of things. First, we say that
a given point is "continental crust" if it is not submerged by an ocean. Second, we say that a given point is "oceanic crust" if it is submerged by an ocean.

Throughout Ulysses, I use these terms fairly regularly, even if they are pretty
self-explanatory. So it's a good idea to define them.

Now that we have that tangent out of the way, lets figure out what data we
will use to guess if a point is continental or oceanic. Does any such data exist? As far as I could tell, no. Not in the sense that crust is a direct function of. However, there are some really great indicators that work well enough. Let's explore these.

# The Moho Discontinuity

The first, and arguably best indicator of continental crust is what is called
The Moho Discontinuity. It is the boundary between the Earth's crust and the
Earth's mantle. Essentially, it is a measurement of the depth from a given point on the Lithosphere (where all crust resides) to the end of the mantle. On average, from the ocean floor to the Moho is 5 to 10 kilometers, and for continental crust it's about 20 to 90 km. Looking at a map of it, we see just how well it maps to the height of a point:

![Moho]({{"assests/hidi_1_26_2018/Mohomap.png" | absolute_url}})

Credit to Wikipedia. For more information, go [here](https://en.wikipedia.org/wiki/Mohorovi%C4%8Di%C4%87_discontinuity)

# Age of Crust

The next piece of data used is the age of crust. This primarily applies to oceanic crust, as you will see. Before we get into the age of crust, we need to have a bit of background on Plate Tectonics.

# TODO: Finish me!
