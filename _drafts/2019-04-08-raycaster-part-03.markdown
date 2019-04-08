---
layout: post
title: "Implementing a Ray Caster Part 3: Floor, Ceiling, and Sky Rendering"
date: 2019-04-08 13:13:00 +0000
categories: raycaster news tutorial
--- 

Welcome back, everyone. In this article, I will discuss how to render floors,
ceilings, and skies.

### Why do we want to render floors, ceilings, and skies?
I suppose this question answers itself. Nonetheless, I'll take an opportunity
here to present two screenshots from my engine. The first one has floors, ceilings
and sky present. The second, of course, does not:

The first image:

![Figure]({{"/assests/raycaster_implementation/part_3/sky_floor_ceil.jpg" | absolute_url}})

The second image:

![Figure]({{"/assests/raycaster_implementation/part_3/no_sky_floor_ceil.jpg" | absolute_url}})

I don't think there's much else to say here other than being able to render floors, ceilings,
and skies dramatically improve the environments you can depict. Now let's jump right into it!

## Part 1: Floor Casting
Floor casting is an inverse operation to ray casting, in some sense. That is, you begin with pixels
on the screen, project them into the world, and then render floors, if the resulting location
has a floor texture to render.

Why do we start with the screen pixels? I suppose you don't have to, but it's more effecient. If you
started with world coordinates, you could only reasonably do so as rays are being casted to check for walls.
So at every step of your ray casting, you would check if a floor pixel is present. However, recall that the
ray casting goes between gridlines. What about the points in between gridlines? No no, this won't do.

Instead, let's take advantage of the fact that there is no vertical variation in our levels, and everything
is vertically centered. What does this mean for floors? It means they're always below wall slices that we
render when performing ray casting. In other words, our algorithm looks like the following for a single column
of pixels:

```
1. Cast a ray
2. If ray does not hit a wall: return
3. Otherwise, render wall slice ray hits.
4. For each pixel below the bottom of the wall slice:
	1. Project pixel location onto world.
	2. Render pixel of floor texture at screen position, but only
	if there is a floor texture to render.
```

So in my implementation, we only render floors if a ray actually hit a wall. Naturally, this
is a limitation of my engine, but it does mean floor rendering is about as effecient as can be.

So the crux of floor casting is projecting a screen position onto the world. Let's figure out how
to do that.

Here is a graphical representation of our goal:

![Figure]({{"/assests/raycaster_implementation/part_3/00.jpg" | absolute_url}})

Where the red circle with the arrow is the player. We know the player's rotation, the ray angle,
and the screen space position of the floor. What we want is the floor pixel in world space. Since
we do not know this, we do not know the distance of the ray.

Like every problem, we can employ trigonometry to help us here. What if we had the straight line
distance to the floor point in world space? That is:

![Figure]({{"/assests/raycaster_implementation/part_3/01.jpg" | absolute_url}})

I've also included an angle beta, which is the ray angle relative to the player. To compute
beta, do the following:

```
Beta = abs(ray angle - player rotation)
```

Now if we knew the straight line distance to the floor point in world space, then we could
employ Soh Cah Toa to get the "true" distance to the point. Let's first find the straight
line distance to the point. Let's re-render the above but from the side:

![Figure]({{"/assests/raycaster_implementation/part_3/02.jpg" | absolute_url}})

Now we see a relationship between several values that we know and the desired value that we
wish to solve for. Using the Similar Triangle Formula, we can represent this relationship as:

straight line dist to floor point in world space / dist to projection plane = player height / r

Now wait a minute, what is r? Graphically, r is this value:

![Figure]({{"/assests/raycaster_implementation/part_3/03.jpg" | absolute_url}})

Where the black line is the projection plane. So r is not the row of floor point in screen
space (the floor point's screen y coordinate, in other words), but the difference between
the floor point row and the middle row of the projection plane. This the projection
plane height / 2, or 100 in my implementation Putting everything together
that we have so far, we get:

* r = projected floor point row - center row of projection plane = projected floor point row - 100
* straight line dist / dist to proj plane = player height / r
* and thus, staight line dist / 277 = 32 / r

TODO: Finish me!
