---
layout: post
title: "How I Did It: Rivers In Ulysses"
date: 2018-03-31 7:55:55 +0000
categories: news ulysses java geology rivers
---

Well I didn't quite get around to doing this yesterday. But I'd argue a day late
for a blog post is my best record to date!

So I promised a brief rundown on how I did river generation in Ulysses. Generating
rivers is part of the Hydrosphere component of the overall procedural terrain generation
system. The Hydrosphere is responsible for producing a precipitation map of the world.

In this post, I will first describe some of the geology behind river formation. Next, I
will explain how I use this to make a simple river building algorithm. Then, I will
discuss some of the additions and modifications I had to make to the system for more
aesthetically pleasing and plausible rivers. Finally, I will explore ways to make
this system have more depth.

# The Geology of River Formation

While the science of rivers is full of depth and incredibly interesting, they only play
a minor roll in the overall Ulysses terrain generation system. As such, there was really
only one component of their formation that was of use to me: gravity. As it turns out,
everything on planet Earth respects the laws of gravity. That is, if I hold something at
a higher elevation, and I let it go, it will fall to the lowest possible elevation
it can.

So what does this have to do with rivers? In sum, they begin at higher elevations, and then
wind their way down to the ocean, lakes, or other rivers.

PICTURE OF RIVER FORMATION HERE.

Basicaly, a river begins when precipitation collects in high altitude areas -- namely mountains.
This precipitation begins to spill over and work its way down to lower altitudes. As it does so,
and as it collects more and more water, it eventually grows into stronger and stronger streams
until it deposits into the sea or a lake. An important note here is that rivers can grow when
they merge with other rivers. This plays a role in the formation of drainage basins -- which
is an area where multiple sources of precipitation drain into a common outlet. As we will see, drainage
basins are somewhat implictily dealt with in Ulysses.

PICTURE OF A DRAINAGE BASIN

For reference, here is what a drainage basin looks like.

# Devising an Algorithm

So basically, to emulate the simple science above, we need two things:

1. Choose the best locations for rivers to form.
2. Choose a path for rivers to go.

Let's start with the first requirement.

# Selecting Where Rivers Form

As the broad overview of the science implied, rivers start in high altitude, high precipitation areas.
Now, you may notice a problem here: rivers are created in Ulysses because they affect the precipitation
of the world, but the precipitation of the world affects how rivers form. This is, of course, one of many
examples of the pitfalls that arise from making a complicated dynamic system into a simple static system.

But all is not lost, I'm happy to say. I can represent where precipitation will fall with something I call the
"Cloud Map". Basically, it's perlin noise that says where clouds frequently form. We once again grossly simplify
the real world science.

Here's an example of what this map would look like:

PICTURE OF CLOUD FREQ MAP

Note that this is nothing more than the "not"-Perlin-Noise mystery algorithm I used in the height map generation
system. Go [here]({% post_url 2018-01-26-how-i-did-it %})

TODO: FINISH ME!
