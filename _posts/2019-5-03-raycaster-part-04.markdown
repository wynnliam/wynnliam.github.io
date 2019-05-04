---
layout: post
title: "Implementing a Ray Caster Part 4: Optimization Techniques"
date: 2019-05-03 18:30:00 +0000
categories: raycaster news tutorial
--- 

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

Next, let's think about the number ten, which is 10 in decimal. That is, ten in decimal is 1 * 10 ^ (1) + 0 * 10 ^ (0).
What happens when we multiply ten by ten? We get 100 in decimal. One hundred is 1 * 10 ^ (2) + 0 * 10 ^ (1) + 0 * 10 ^ (0).
In other words, by shifting the one to the left, we increased our number by ten.

Likewise, suppose we have the binary value 10, which is 2 in decimal. Now let's perform this shifting operation
on binary 10. As before, we get 100, but since this is a binary value, the resulting decimal value is 2.
In other words, doing a shift to the left doubles our value. If we shift to the right, it divides the value
by two. We call this bit-shifting.

You can perform bit-shifting in code, of course. Here's a snippet of C doing left shifting:

```C
int i = 1 << 1;
```

This takes 1 and shifts all the bits representing it to the left by 1. 1 in 4-bit binary would be this:

```
0001
```

So shifting it to the left once gives us:

```
0010
```

Which gives us decimal 2.

### Using Bit Shifting for Division
There's a lot of nuance I'm omitting here. But we now have enough theoretical and technical background to
further optimize our system. It turns out that using 128 was not a coincidence. 128 is a power of two.
That is, 128 = 2 ^ 7. In binary, that would be

```
10000000
```

Now let's take this number and bit-shift it right by 7:

```
00000001
```

We took 128 in decimal and divided it by 128! In other words, if we take any number and bit-shift it to the right,
we divide it by 128. Concretely, lets examine our example of doing cosine with a look up table:

```c
10 * sin_lookup[23]
```

Since the sin\_lookup holds the sin value of each degree from 0 to 359 multiplied by 128, we have to
divide the expression by 128 to get our final value. That is:

```c
10 * sin_lookup[23] \ 128
```

Division operations are notorious for their inefficiency. So we can impove this by doing:

```c
(10 * sin_lookup[23]) >> 7
```

Which is about as fast as we can make it. Admittedly, this is a contrived example. So let's see
how these techniques can be used in a computation for the ray caster.

## Applying Optimizations to Computations for a Ray in Quadrant 1
Recall from the first part in the series the series of equations we needed to do for a ray in quadrant 1.
I will list them here for you:

* c\_h.x = ((c\_h.y - player.y) / Tan(alpha)) + player.x
* c\_h.y = (floor(player\_pos.y / 64) * 64) - 1
* d\_h.x = 64 / Tan(alpha)
* d\_h.y = -64 (Since we travel in the negative direction along the y-axis)
* c\_v.x = (floor(player.x / 64) * 64) + 64
* c\_v.y = Tan(alpha)(c\_v.x - player.x) + player.y
* d\_v.x = 64
* d\_v.y = -Tan(alpha)(d\_v.x)

As a brief sidenote, 64 is also a power of two. That is, 64 = 2^6. Also, d\_h.y and d\_v.x are
constants, so there isn't a lot to do. Let's go through a few of these and see what we can't do.

I'll start with the second equation. That is:

```
c_h.y = (floor(player_pos.y / 64) * 64) - 1
```

Now, c\_h.y is an integer in our system, and division will always floor your result. Right off the bat
we get:

```
c_h.y =  (player_pos.y / 64) * 64 - 1
```

You might think that the multiplication and division cancel out. Not quite. Since this is integer division,
if you divided say 158 (not a multiple of 64) by 64, you get 2. The correct answer is actually 2.46875. However,
we can use the fact that 64 is a power of two to simplify this. Using bit shifting, the result becomes:

```
c_h.y = ((player_pos.y >> 6) << 6) - 1;
```

Next, let's find c\_h.x. We start with this:

```
c_h.x = ((c_h.y - player.y) / Tan(alpha)) + player.x
```

First, Tan is done with a lookup table. However, I made a lookup table for 128 divided by the tangent, which I called
tan1. Note that A / B = A * (1/B). Using this fact, we can rewrite this as:

```
c_h.x = ((c_h.y - player.y) * tan1[alpha] / 128) + player.x
```

Now with out bit-shifting tricks, this becomes:

```
c_h.x = ((c_h.y - player.y) * tan1[alpha] >> 7) + player.x
```

We can do a similar thing for d\_h.x:

```
d_h.x = 64 / Tan(alpha)
```

Can become:

```
d_h.x = 64 * tan1[alpha] >> 7
```

Now this is interesting. Note that we computee 64 * tan1[alpha]. As we showed before, this
is equivalent to tan1[alpha] << 6. But then we bit shift to the right by 7. We go left 6, then 7 right,
so the net shift is 1 to the right. Hence:

```
d_h.x = tan1[alpha] >> 1
```

c\_v.x is similar to c\_h.y, so I won't bother deriving the optimized solution. It is:

```
c_v.x = ((player.x >> 6) << 6) + 64
```

Now we can optimize c\_v.y. Note that this will use a lookup table tan[alpha], which is 128 * tan(alpha).
This is distinct from tan1[alpha], which is 128 / tan(alpha). Thus, we need only to divide by 128 like so:

```
c_v.y = ((tan[alpha] * (c\_v.x - player.x)) >> 7) + player.y
```

Lastly, we have d\_v.y. That is:

```
d_v.y = -Tan(alpha)(d_v.x)
```

This is fairly easy to optimize. The result is:

```
d_v.y = (-tan[alpha] * d_v.x) >> 7
```

And there we have it! All of our values computed in an optimized form. Below is a summary of the calculations:

```
c_h.y = ((player_pos.y >> 6) << 6) - 1;
c_h.x = ((c_h.y - player.y) * tan1[alpha] >> 7) + player.x

d_h.x = tan1[alpha] >> 1
d_h.y = -64

c_v.x = ((player.x >> 6) << 6) + 64
c_v.y = ((tan[alpha] * (c_v.x - player.x)) >> 7) + player.y

d_v.x = 64
d_v.y = (-tan[alpha] * d_v.x) >> 7
```

## Final Remarks
This is the end of my little series! I hope you found it enjoyable as well as informative. While it is
unlikely I'll add new features to the ray caster, if I do I'll write similar posts to the ones in this
series. Until next time, thank you for reading!
