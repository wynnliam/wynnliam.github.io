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
will explain how I use this to make a simple river building algorithm. Finally, I will
discuss some of the additions and modifications I had to make to the system for more
aesthetically pleasing and plausible rivers.


