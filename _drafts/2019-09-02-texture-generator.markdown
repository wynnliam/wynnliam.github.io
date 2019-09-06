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

# The problem (and how machine learning can solve it)
Okay with this background, we can now get an idea of what our problem is. As the name of the project implies,
I wanted to be able to generate these texture images. The grand vision would be that I could specify
the kind of texture I want and the system would spit out some kind of generated sample.

For example, I want a "honeycomb" texture, and perhaps it gives me one of the following:

![Figure]({{ "/assests/texture_generation_output/0.png" | absolute_url }})
![Figure]({{ "/assests/texture_generation_output/1.png" | absolute_url }})
![Figure]({{ "/assests/texture_generation_output/2.png" | absolute_url }})

(These are examples of what my model ended up actually generating)

Broadly speaking, how would this be done? We will basically need to take as input some noise, and spit out
a texture. Visually, our problem is as follows:

![Figure]({{ "/assests/tex_gen_overview/overview.png" | absolute_url }})

Now, as far as I can tell, there isn't an obvious function or algorithm that maps say noise to generated texture samples.
That's where machine learning comes in: it helps us find mappings between domains that aren't otherwise apparent. So
let's take a quick survey of models that I found weren't suitable, and then we'll get into the varational autoencoder.

## Models and techniques that I found insufficient
Texture generation (also called "texture synthesis") is not a new problem. Here are some of the algorithms
I thought about using or exploring and why I didn't use them

### Image Quilting
Essentially, you take a source texture image, and randomly sample from it. These samples you "quilt" together
onto a larger surface. This graphic from Douglas Lanman explains it:

![Figure]({{ "/assests/texture_generation_output/imagequilting.gif" | absolute_url}})

What I like about this algorithm is that it's simple: you don't need an elaborate model to get good results.
However, the problem is that it takes as input a sample texture and basically shuffles that. What I want is to
generate entirely new samples without relying off of given samples *at generation time*. You see, I will use
samples to learn a mapping between noise and textures. Once I have that mapping, I should no longer need samples
to generate textures.

### Gatys et. al. Method
In what was a follow up to their famous Style Transfer, Gatys et. al. employed the same style transfer technique,
but applied to noise. So given a source texture, and a noise image, they produces a new texture image. The paper
for this is [here](https://arxiv.org/abs/1505.07376)

As great as this method is, it has the same problem as quilting: at generation time, I have to give the system
a sample texture to work with.

### Deep Convolutional Generative Adversarial Networks
The current state of the art for such a generative approach is the GAN (Generative Adversarial Networks). Since
I'm doing something with images, I want a DCGAN. Essentially, you have two competing networks. One that is
training to identify real examples of the space you want to model from fake ones. The other network is trying
to generate phony examples that look like real ones. These two networks are essentially playing against each other.
In theory, as they play against each other, they both get better at their respective tasks.

If you're interested, [Alec Radford and Luke Metz](https://arxiv.org/pdf/1511.06434.pdf) published a seminal paper
showing the power of DCGANs.

I had used this model for my final in CS 446: Advance Topics in Machine Learning (as taught by [Anthony Rhodes](https://web.pdx.edu/~arhodes/))
in the Fall of 2018.

While this model worked, I found it had some annoying issues. First, as long as the generater network (the one
producing phony examples) could fool the discriminator network (the one seperating phonys from real ones), it
was "good". This means that for a long period of time, the generator can feed the discriminator noise and still
fool it. It's only after awhile that the discriminator can properly identify what it's looking for.

Second, I found the DCGAN was prone to collapse. I think this is what they call [modal collapse](https://aiden.nibali.org/blog/2017-01-18-mode-collapse-gans/).
Essentially, the training just falls apart and the discriminator fails entirely to learn the space of real samples. The generator
can give it some junk "example" and no matter what it passes.

## Variational Autoencoders
A
