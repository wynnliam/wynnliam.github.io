---
layout: post
title: "Real Time Pathfinding Using Flow Fields Optimized with Navigation Meshes"
date: 2019-12-01 00:30:00 +0000
categories: alexander
--- 

Hello all! It’s been about three months since my last post, so I figured I would do one on my latest AI project: Flow Fields.

This post will begin with a brief overview of steering forces and flock systems, as these are crucial to understanding flow fields.
Next, we will go into flow fields proper: how they are created for pathfinding, why you want to use them as opposed to say A-star
pathfinding, and a historical usage of flow fields in games. Finally, I will detail how you can optimize flow fields with navigation
meshes so that their generation is done quickly. Here is a simple video detailing what the result of all this looks likes:


FLOW FIELD BOID SYSTEM IN ACTION


Before we get started, I wanted to touch on a couple of things. This entire post is detailing the flock system used in my project
[Alexander](https://github.com/wynnliam/alexander), which is basically an implementation of steering forces and flock behaviors
in the Unity Engine. This is important because it will define some assumptions I am making about the systems we are dealing with:
namely that the space you are working in is 2D and can be described by a grid of tiles. Despite this, you can extend this to
3D environments if you so desire.

 

Background – Steering Forces and Flock Systems

First, what exactly are “flow fields” and “steering forces” and “flock systems” for? As these names may imply, we are dealing with real time AI systems. Whether it be a game, simulation, or something else, the goal is to resolve paths through spaces with many agents running around.

 

So what is a “steering force”? They are vectors you apply to objects that move them. The important thing is that it does these movements smoothly. Let’s look at the graphic below:

 

GRAPHIC OF AGENT MOVING IN DIRECTION INDICATED BY BLUE ARROW

 

The blue arrow represents the velocity of the agent. Suppose now something happens in the simulation, and the agent want to go towards some goal:

 

GRAPHIC OF AGENT AS ABOVE BUT WITH A RED FLAG

 

The red flag indicates the desired destination of the agent. Given this target, we can compute the desired velocity for the agent to move towards that goal:

 

GRAPHIC AS ABOVE BUT WITH RED ARROW GOING FROM AGENT TO TARGET

 

Now we have two velocities: one for the agent currently (indicated in blue) and one for what the agent wants to have (indicated in red). For each tick of your simulation, you can push the agent closer towards the desired velocity until it has it. In doing so, the agent will have a smooth transition from its current velocity to its desired one. With only a few lines of code, you can simulate birds, airplanes, horses, people moving, and so much more.

 

Another cool fact: steering forces can be combined! You could have one steering force that pushes you towards a goal, and some that push you away from hazards. The power of steering forces lies in its ability to create emergent behavior from simple rules:

 

IMAGE DEPICTING THE POWER OF STEERING FORCES

 

For the sake of this article, I won’t go into the math of them. You can find a million of such tutorials and articles online, and one of my favorite is a series by [Fernando Bevilacqua]( https://gamedevelopment.tutsplus.com/tutorials/understanding-steering-behaviors-movement-manager--gamedev-4278).

 

Steering forces give you the power to easily simulate individual agents. You can extend steering forces to simulate a flock of birds (hence the names “flock” or “boid” systems), thanks to the work of Craig Reynolds: http://www.cs.toronto.edu/~dt/siggraph97-course/cwr87/

 

Essentially, you begin with the idea that agents (from here on out called “boids”) are clustered into flocks. Then, using these flocks you can come up with new steering forces to govern groups of boids. The three main ones are:

 

·         Cohesion: a steering force that encourages nearby boids to be closer to each other

·         Separation: a steering force that, the larger its magnitude, encourages boids to maintain a distance from each other more

·         Alignment: encourages boids to line up with other boids nearby in its flock

 

TODO: ADD GRAPHIC FOR EACH OF THESE

 

The article I posted above dives more deeply into the math of these, if you are interested in reading more about them. With a brief understanding of these concepts in mind, let us now dive into flow fields themselves.

 

Flow Fields

So we know how to move an individual boid, and we know how to move a flock of boids. This is great, but suppose you want to navigate a space with obstacles? We need some way to have our flocks navigate. It may be tempting to hit this with something like A-star, but there are runtime problems with this.

 

For one, suppose we did A-star on individual boids. As absurd as this may be, it does have some benefits. Consider a scenario where, for some reason, a member of the flock was separated from the rest of its group:

 

IMAGE OF BOID SEPARATED FROM THE REST OF THE GROUP

 

If we did A-star on the entire flock, we run into an issue where this boid can’t navigate like the rest of the group because the path it would need is different. I’m sure there are ways to rectify this, but we can just avoid this problem by using a simpler method: flow fields.

 

Suppose you have a tile grid of N rows and M columns. Then a flow field would be a 2D array of vectors with dimensions N rows and M columns. Given the entry (i, j), the corresponding vector would be a steering force that pushes you in the direction of the next tile. Ideally, if you follow these forces, they will eventually guide you towards a goal as depicted here:

 

IMAGE OF A FLOW FIELD

 

The dark grey tiles are walls. What makes flow fields attractive is the fact that they are easy to generate. Suppose you have a goal position that is the tile at (i, j). The algorithm for generating the flow field is as follows:

 

*BREADTH FIRST SEARCH STARTING AT END GOAL GOING OUTWARDS

 

For a given flock, we need only one flow field to find the goal. This way, we can use the power of steering forces and flock behaviors while seamlessly adding pathfinding. Flow fields are not new: they were used in [Supreme Commander 2] (http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter23_Crowd_Pathfinding_and_Steering_Using_Flow_Field_Tiles.pdf) for pathfinding with groups of units.

 

Optimizing Flow Fields with a Navigation Mesh

While flow fields are great, we can make them a little faster. If you’re space is really large, you are going to be search a lot of tiles. Is there some way we can make this faster? If you’re willing to eat some costs up front, then yes. What I mean is that we can spend some time at the start of the simulation computing a “navigation mesh”, which is a graph that groups walkable spaces together (the edges denoting which groups are adjacent). So in our space, groups of walkable areas would be groups of tiles. We do not group tiles arbitrarily – the next little example will hopefully give you an intuition for how we build our navigation region. Consider a level that looks like this:

 

IMAGE OF LEVEL THAT LOOKS LIKE AN L-SHAPED CORRIDOR

 

The red flag denotes the goal. Since we will compute a vector for all walkable tiles that can reach the goal, we don’t bother putting an agent/flock in this graphic. I will now show the same image overlaid with a navigation mesh:

 

IMAGE OF SAME LEVEL WITH NAVIGATION MESH

 

I’d like to interject with some history. I actually used a similar navigation mesh system like the one I detail here, but with A-star path finding. I did this with An Engine of Ice and Fire about [five years ago] (https://github.com/wynnliam/an_engine_of_ice_and_fire).

 

The walkable tiles are now colored according to the “navigation region” they are a part of. This partition was not arbitrary. First, all the regions are rectangles. This means that for any two tiles in the same rectangle, there is a direct path between them. Thus, you now have a guarantee that so long as your destination is in the same region, you will not bump into any walls. Furthermore, there is a relationship between the rectangular regions: If you are to combine a region with one that is directly adjacent to it, they will form a rectangle. This is important because it gives us another guarantee: there is a direct path of travel between any point in one region, and any point in an adjacent region. This latter guarantee doesn’t hold for every region. For instance, the blue region and the red region, while both being adjacent to the green region, would not form a single rectangular region all together.

 

Now what is the significance of these navigation regions? In the context of flow fields, we can say “what is true for one tile in this region is true for all tiles in this region”. Thus, we need only one steering force vector for each of these regions. This reduces our search space from several tiles to three regions. Generating a flow field is now done on the regions themselves, not the tiles they hold.

 

There are two issues with this method: first is finding an algorithm to generate these navigation regions, which must be ran up front before the simulation begins. The second is that you will only have an approximation to the most efficient path. There are some cases where the corresponding steering force for a particular tile will not point directly at the goal when it otherwise could have. I personally found that such losses in accuracy in path solutions to be miniscule for a real time simulation like Alexander. There are ways to rectify this, but I will not detail them here. I will, however, give a brief overview of how you may generate such navigation meshes.

 

Generating Navigation Meshes

I’d like to preface this by saying there are countless ways to create these navigation meshes (if you wanted, you could build them by hand too). As such, I am detailing the algorithm I designed for this. If you would like to see the source files in particular that implement this in Alexander, go [here](https://github.com/wynnliam/alexander/tree/master/alexander/Assets/Scripts/Grid).

 

My algorithm has two phases to it:

 

1.       Generating the basic navigation regions such that they are all rectangle regions with no wall tiles in them.

2.       Splitting these navigation regions until every pair of adjacent regions is such that, if combined, form a larger rectangle that also has no wall tiles in it.

 

The first phase can be done in several different ways, so I will pass over how I did this part. It is the second part that, in my opinion, is far more salient.

 

It’s overall structure is:

 

* SPLITTING ALGORITHM

 

Essentially, we continue to “split” these regions until it is no longer possible. What does it mean to “split”? Suppose I have two adjacent regions that are as follows:

 

TWO BAD ADJACENT REGIONS

 

I need to pick one of these pieces and break it in either two or three pieces, perhaps as follows:

 

TWO BETTER REGIONS

 

There are several cases of splitting, and sometimes we need to split both regions until the navigation mesh looks right. Suppose this did happen: we split a region but the resulting navigation mesh didn’t look right:

 

CASE OF IMPERFECT REGIONS

 

This is fine because our splitting algorithm will keep going as long as we need to keep splitting, so we just save the next region for a future iteration of our loop. There are about ten different cases for splitting, and that would be an article unto itself. Hopefully you at least have an intuition for how navigation meshes are created, and how we can use them for fast flow fields.  

 

And that’s it for this post. I hope you found this useful. Please do checkout my project [Alexander]( https://github.com/wynnliam/alexander) if you want to see this system in action.
