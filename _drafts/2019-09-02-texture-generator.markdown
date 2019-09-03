---
layout: post
title: "Texture Generation Using Variational Autoencoders"
date: 2019-09-02 17:12:00 +0000
categories: ml texture raycore
--- 

Hello all! In this post, I want to describe a fun experiment in machine learning I did
some months ago. This was using a variational autoencoder to create textures. In this
article, I will document the problem I had, the model I used, and then the results.

# What are textures?
To begin with, let's define what a "texture" is. The definition of "texture" is kind
of hazy. In games, a texture is a qualitative description of something's surface. Take
for example this picture I took from a simple level I made for Half Life 2.

![Figure]({{ "/assests/hl2_demo.jpg" | absolute_url }})

You can see the ground, a fence, and three blocks. We can describe the respective texture of each
of these as follows:

* The ground has a damp, dark green, grass texture
* The background has a metalic fence texture
* The three blocks (from left to right) have brick, wooden plank, and concrete textures

For these, the texture is represented by an image. I will briefly note that textures needn't be
images strictly. The sky has a texture that isn't an image unto itself. Depending on the engine used,
such textures may be elaborate pieces of code, or (in the case of Half Life 2's Source Engine) the use
of six images that form a "skybox".

In the case of my model, I wanted to generate texture images with a special property: being "locally similar".
Take the brick texture of the leftmost block in the image above. Now look at different areas of this block.
You'll notice that the texture (red brick) is similar no matter where you look. In other words, a texture
of the kind I want has repeating or similar patterns/shapes no matter what portion of it you're looking at.
