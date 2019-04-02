---
layout: post
title: "Implementing a Ray Caster Part 2: Sprite Rendering"
date: 2019-04-02 14:41:00 +0000
categories: raycaster news tutorial
--- 

Welcome back. In this part, we will explore sprite rendering. Originally, I was going
to do floor, ceiling, and skybox rendering for this part. The reason being was that
in my implementation, I did floors, ceilings and sky rendering after wall rendering.
However, sprite rendering is a fundamental part of the overall rendering system. We use
sprites to render objects in worlds. Things like trees, pots, tables, and so on cannot
be done well without sprites.

Before we get started, I would like to direct you to a post by one [David Layne](https://www.allegro.cc/forums/thread/355015).
I will admit, when I first did my implementation, I used this and didn't understand *why* it worked.
So pretty much all credit goes to him. However, I've since taken the time to read his code and better
understand it. Much of this post will be a more in-depth explanation of Layne's work.

So with that out of the way, let's get right into sprite casting!

### Section 1: The Overall Algorithm

In broad strokes, to render sprites in our environment we do the following:

```
1. Sort sprites according to distance from player
2. For each sprite:
	1. Find sprite coordinates on screen
	2. Scale sprite according to distance
```

Like ray casting for rendering walls, we open up with a deceptively easy algorithm.
As you will see, pretty much all of these steps will require a bit of math. The part
in particular that killed me (were it not for Layne's implementation) was finding
the sprite coordinates on the screen.

For this post, I will dedicate a section to each step.

### Section 2: Sprite Sorting
For anyone familiar with CS (algorithms in particular), there isn't that much interesting
that happens here. I did a very quick and dirty implementation of [Quicksort](https://en.wikipedia.org/wiki/Quicksort).

Rather than detailing my implementation of Quicksort, I think it's more instructive to
know why sorting sprites by distance is needed. Like my last post, I'll employ extensive
use of poorly drawn graphics to explain this. In this vain, consider this scenario:


![Figure]({{"/assests/raycaster_implementation/part_2/01.jpg" | absolute_url}})

The red circle is the player. The arrow indicates the way the player is facing. The blue A and green B are sprites
in our environment. In this case, we would want to see something like this in our 3D rendering:


![Figure]({{"/assests/raycaster_implementation/part_2/02.jpg" | absolute_url}})

That is, the A is drawn behind the B. On the other hand, consider this scenario:


![Figure]({{"/assests/raycaster_implementation/part_2/03.jpg" | absolute_url}})

Then naturally, we'd want to see the sprites rendered like:


![Figure]({{"/assests/raycaster_implementation/part_2/04.jpg" | absolute_url}})


This is a rather quick and dirty mockup, but I think it does a sufficient job explaining
the ideal behavior. In the first scenario, we would render the A onto the screen, then render
the B over it. So when you perform quicksort, we want to sort from furthest sprite to closest
sprite. That way, as we loop through each sprite to render them, we render the furthest
first, then the closest over that.

### Section 3: Computing Sprite Screen Coordinates

For each sprite, we now have to figure out where on the screen we render. The screen coordinates
specifically tell us where the center of the sprite is. In my implementation,
I did this computation even for sprites outside the view of the player, because we may be able
to see a part of a sprite, even if its slightly off screen. I didn't want to deal with special
cases where a sprite is partially or wholly off screen.

Finding the screen coordinates of a sprite is not immediately intuitive. It essntially requires
you transform the sprite's world coordinates to screen coordinates. Some implementations require
matrix transformations and a whole slew of linear algebra. Although this may be an intuitive approach
to vector transformations, we won't use that here. Instead, we'll begin with some assumptions about
walls and sprites in the world:

* As I detailed in the last post: *everything* is 64 pixels by 64 pixels in dimensions
* Everything is vertically fixed as I will explain below

The first point isn't knew, but coupling it with the second point is huge. When I say that
everything is vertically fixed/centered, I mean the following graphic:


![Figure]({{"/assests/raycaster_implementation/part_2/05.jpg" | absolute_url}})

So the player is 32 pixels in height, but everything else is 64 pixels in height. Furthermore,
everything is vertically fixed/centered. That is, no matter where anything in the world is,
be it sprites or walls or what have you, it means that y coordinate of the center of everything
is half of the projection plane's height. Since my implementation assumes the projection plane
has a height of 200, then the y coordinate of the center of everything is 100.

This means right off the bat, we already have the y coodinate of sprite's screen position. All
that remains to find is the x coordinate of the center of the sprite.

This is a much trickier problem, since we don't have an obvious way to relate a sprite's world position
to the screen. We could do something similar to walls. For each column of pixels in our screen, fire a ray,
and move it until it hits a sprite. However, since sprites are a lot less constrained as to where they can
be in the environment, we'll hit some nasty cases we'd have to handle.

Instead we'll need to do a reverse computation to walls. With walls, we start with columns of pixels and find
the world coordinates. This time, we know the world coordinates, and need to find the column of pixels we
want to render.

For this, we'll draw from trigonometry, and use the [atan2](https://en.wikipedia.org/wiki/Atan2) function.
Essentially, given a vector, we can compute the angle formed between it and the x-axis. Suppose we've the
situtation:

![Figure]({{"/assests/raycaster_implementation/part_2/06.jpg" | absolute_url}})

Where the red circle is of course the player, and the blue one is a sprite in the world. If we know the
position of each, we can figure out the vector between them, which is the following:


![Figure]({{"/assests/raycaster_implementation/part_2/07.jpg" | absolute_url}})

I would call this vector d for "difference" or perhaps i for "increment" (as Layne does), but I thought
they would be confusing (we used d extensively in the previous article, and i is associated with imaginary
numbers). Instead, we'll use h. Computing h is a simple vector operation:

h.x = sprite.x - player.x
h.y = sprite.y - player.y

Now why do we need this? Basically, we can compute the angle between the x-axis and h. Here is the
graphic redrawn with the x-axis superimposed:

![Figure]({{"/assests/raycaster_implementation/part_2/08.jpg" | absolute_url}})

Also included is an angle p. This angle is the one we are interested in. To compute
p we do the following computation:

p = atan2(h.y, h.x)

Note that typical implementations of this have the y coordinate as the first argument.
Also, since our y axis is flipped, we need to negate this. Thus:

p = atan2(-h.y, h.x)

And finally, since we are dealing in degrees, our resulting angle will be:

p = atan2(-h.y, h.x) * (180 / PI)

So great, Liam. You made everyone solve for this angle p, but what was the point?
Recall that there is a relationship between columns of pixels and ray angles. If our
projection plane width was 320, and our field of view was 60 degrees, we could say
that there was 60 degrees for 320 columns of pixels, or 0.1875 degrees per column
of pixels. Similarly, you could say there was 320 columns of pixels for 60 degrees.
Simplifying this would tells us there are about 5.33 columns of pixels for every
1 degree.

This relationship gives us a means convert say an angle to a corresponding column of pixels.
Since the x coordinate of our sprite is a column of pixels, we just use this conversion factor
on p and we're all set, right? That is, we just need to do p * 320 columns of pixels / 60 degrees, right?

Not quite. Let's draw h, p, and the player's field of view in one graphic:

![Figure]({{"/assests/raycaster_implementation/part_2/09.jpg" | absolute_url}})

TODO: Finish me!
