---
layout: post
title: "Implementing a Ray Caster Part 4: Optimization Techniques"
date: 2019-05-01 13:13:00 +0000
categories: raycaster news tutorial
--- 

TODO: Fix date!

Welcome back, everyone. Today I will discuss some of the optimization techniques
I used when implementing my raycaster. In broad strokes, this will cover two things:
how I represented sin, cosine, and tangent values to make them effecient, and then
how I transform the equations I derived in my previous posts. Specifically, we will
explore how I implement the computations for a ray in quadrant 1. This will be fairy
short because, truth be told. However, I hope you find this informative.

Before we begin, I'd like to cite this article [here](http://tigcc.ticalc.org/tut/raycasting.html).
I got the ideas for optimizing the ray caster from it, so please please please check it out.

Okay so with that out of the way, let's begin.

## Representing Trigonometric Functions in the system

In the previous articles, I showed how to implement rendering for walls, sprites, floors, ceilings,
and skies. However, the extent of these articles was to explain the theory behind them, so to speak.
It was just the high level mathematics behind it.

So how would it translate into code? Honestly, it's fairly direct, at least in my opinion. You can
use sin, cos, tan, etc as is. However, these operations are not the most efficient. For example,
[this](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/ieee754/dbl-64/s_sin.c;hb=HEAD) is IBM's
implementation of sin and cos.

Can we do better? Yes. I don't mean that I can write better sin and cos functions than IBM. I do mean that
we can avoid having to call this several times per second. And it's a fairly easy trick. Recall that all
my implementations are done with angles in degrees, not radians. This is because we can store the sin, cos,
tan, etc. values for the degrees between 0 and 359 (inclusive) as an array. The sample C code might look
like this:

```c
double sin_lookup[360];

int i;
for(i = 0; i < 360; i++) {
	sin_lookup[i] = sin((i * PI) / 180.0);
}
```

Now you see why I also insisted that every angle computation gets adjusted such that it's between 0 and 359.
Suppose I have the angle 361 degrees. The corresponding adjusted angle is 1 degree, and now when ever I need
to compute sin(1), and I can just look up the corresponding degree in the lookup table. This significantly
reduces the number of operations.

But we can actually take this a step further. Doubles are great, but doing computations with them is slow. Now,
modern machines are fast to the point that this slowness is insignificant for the most part. But I thought
being lazy and keeping doubles around went against the spirit of this project.

What if we instead used integers? Well our resulting code might look like this:

```c
int sin_lookup[360];

int i;
for(i = 0; i < 360; i++) {
	sin_lookup[i] = (int)sin((i * PI) / 180.0);
}
```

Not bad, right? Well let's consider a simple operation. I want the sin of 23 degrees. When I plug
that into C's sin function, I get 0.390731, as expected. However, when I use my integer lookup table,
I get 0. What gives? Anyone whose programmed for awhile understands that integers don't store the decimal
part of a number, it just cuts it off. So if you take 0.390731 and cast that to an integer, you get 0.
So how do we rectify this? We store the result of sin(23) * 128! I'll show you how and why this works
in the overall ray casting system, but first let's update our lookup table:

```c
int sin_lookup[360];

int i;
for(i = 0; i < 360; i++) {
	sin_lookup[i] = (int)(128 * sin((i * PI) / 180.0));
}
```

Now, when I use my lookup table, instead of 0.390731 or 0, we get 50. In terms of memory effeciency, it beats
doubles typically, since a double is usually about 8 bytes, whereas an int is typically 4. Computational effeciency
is better too. You could store these as chars if you wanted, and get even better memory effeciency. However,
you will get some problems at say 128 * cos(0), or 128 * sin(90), since your answer will be 128, which for signed bytes
you can't store. An easy fix is for these cases to subtract 1, and consider it good enough. Looking back, I should
have done this because I don't think the resulting visual artifacts would be that bad (more on this later).

So if you store say 128 * sin(23) instead of just sin(23), you'll inevitably get some problems. Consider the following
example. Suppose I want to compute:

```c
10 * sin_lookup[23]
```

Since our lookup table holds 128 * sin(23), our computation would look like this:

```c
10 * 128 * sin(23)
```

So all we have to do is just divide. That is:

```c
10 * 128 * sin(23) / 128
```

Using our lookup table, this gives us a value of 3. The correct answer is about 3.9073.
Now, obviously this is wrong, since we lose the decimal part. If we just stored the integer
result of sin(23) without the 128 multiplication, our answer would be 0.

By multiplying the result of sin by 128, we are able encode a more accurate result within an
integer. As long as we remember to divide by 128 when doing our operations, this works
well enough for my application. 

We're not quite done yet, the computation above uses a division operation. This is still
too inefficient. I will now show you how to do the above operation without division.

### An Aside: Binary Numbers, Bits, and Bit Shifting

As you hopefully know, everything in a computer is represented as binary numbers. So when
you see the value 6, the number in the machine is actually 0110. Each digit in the binary
representation is called a bit. So if I have the binary value 0110, we can say this is a 4 bit
binary representation of the decimal value 6.

Why does 0110 in binary equal 6 in decimal? Like decimal, each digit represents a power.
So the decimal value 6 is 6 * 10 ^ (0) = 6 * 1 = 6. Hence, we have a 6 in the "ones" place.
In principle, the same thing applies to decimal values, albeit each place is a power of two
instead of a power of 10. That is, 0110 translates to 0 * 2 ^ (3) + 1 * 2 ^ (2) + 1 * 2 ^ (1) + 0 * 2 ^ (0)
which equals 0 * 8 + 1 * 4 + 1 * 2 + 0 * 1 which simplifies to 0 + 4 + 2 + 0 which thus equals 6 in decimal.


