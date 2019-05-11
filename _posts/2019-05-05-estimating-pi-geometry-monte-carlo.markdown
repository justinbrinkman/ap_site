---
layout: post
title:  "Estimating Pi with Geometry and the Monte Carlo Method"
date:   2019-05-05 12:30:46 -0500
categories: python tutorials
tags: monte carlo simulation dimensions 2d 3d 4d geometry pi estimate
comments: true
---

Introduction
=======================

One cool application of the [Monte Carlo method](https://en.wikipedia.org/wiki/Monte_Carlo_method) is to estimate 
the value of pi. Simple geometric equations can be used in conjunction with Python's "random" module to do this. The
most common and straightforward method is to take a square inscribed in a circle (2D geometry), randomly generate 
points inside the square, and measure how many of those points fall inside the circle. I decided it would be 
fun and highly necessary to experiment with this method using 3D and 4D geometry.

This post will cover:

1. The use of Python to estimate pi using 2D, 3D, and 4D geometry
2. The analysis, comparison and improvement of the run-times of each of the algorithms

Please enjoy the semi-conspicuous use of engineering paper for drawings!

Basic Principles and 2D Method
===============================

![screenshot](/photos/estimatepi1.png){:class="img-responsive"}

Consider the drawing above. We have a circle (radius = 1) centered at (0,0) inscribed in a square (side length = 2). 
Their respective areas are as follows:

$A_{circle} = \pi r^{2} = \pi (1)^{2} = \pi $  
$A_{square} = length^{2} = 2^{2} = 4$ 

Thus, $\frac{A_{circle}}{A_{square}} = \frac{\pi}{4}$

By randomly generating $n$ points inside the square, we can expect the ratio of points within the circle to total 
points to be $\pi / 4$ for very large values of $n$. That is, $\frac{points\ in\ circle}{points\ in\ square} \approx \frac{\pi}{4}$. 
So we can rearrange to get the following equation: 

$\pi \approx 4*\frac{points\ in\ circle}{points\ in\ square}$

To generate points, we can perform a Monte Carlo simulation in Python. This is the first step:

``` python

import random
```
We will define a function ```montecarlo2d(num_simulations)``` so we can specify how many points (simulations) we want to 
generate. We will generate random points inside the square by randomly generating values for x and y between -1 and 1. If 
you take a look at the first drawing, you will notice that these points will, indeed, fall within the square.

Remember the general equation for a circle: $(x-h)^{2} + (y-k)^{2} = r^{2}$ where $(h,k)$ is the center point. In this case, 
the center is at $(0,0)$ with radius = 1, yielding the equation $x^{2}+y^{2} = 1^{2}$. This is derived from the Pythagorean 
theorem, and may be more obvious by taking another look at the first drawing.

So if $x^{2}+y^{2}\leq 1$, then the point is inside the circle.

The code for this function is as follows:

``` python

def montecarlo2d(num_simulations):
    num_cir = 0
    num_sq = 0
    for i in range(num_simulations):
        x = random.uniform(-1,1)
        y = random.uniform(-1,1)
        if x**2 + y**2 <= 1:
            num_cir += 1
        num_sq += 1
    return 4 * (num_cir / num_sq)
```
With ```num_simulations``` set to one million, I get the following output:

```
>>> montecarlo2d(1000000)
3.142092
```
Not a bad estimate! Now let's do it in 3-dimensional space.


3D Method
===============

The underlying principle of this method is the same as in 2-dimensions, but instead of relating circle and square areas
we will relate sphere and cube volumes. We will consider a sphere of radius = 1 inscribed in a cube of length = 2.


![screenshot](/photos/estimatepi2.png){:class="img-responsive"}

$V_{sphere} = \frac{4}{3}\pi r^3 = \frac{4}{3}\pi (1)^3 = \frac{4}{3}\pi$  
$V_{cube} = length^3 = 2^{3} = 8$

Thus, $\frac{V_{sphere}}{V_{cube}} = \frac{1}{6}\pi$  
$\pi \approx 6*\frac{points\ in\ sphere}{points\ in\ cube}$

Given the general equation of a sphere $(x-a)^{2}+(y-b)^{2}+(z-c)^{2} = r^{2}$, with $(a,b,c)$ replaced by our 
center point $(0,0,0)$ and radius = 1, we get $x^{2}+y^{2}+z^{2}=1$. If $x^{2}+y^{2}+z^{2}\leq1$ then the point is 
inside the sphere.

The code for this function is as follows:

``` python

def montecarlo3d(num_simulations):
    num_sph = 0
    num_cube = 0
    for i in range(num_simulations):
        x = random.uniform(-1,1)
        y = random.uniform(-1,1)
        z = random.uniform(-1,1)
        if x**2 + y**2 + z**2 <= 1:
            num_sph += 1
        num_cube += 1
    return 6 * (num_sph / num_cube)
```
Output:

```
>>> montecarlo3d(1000000)
3.1385879999999995
```
Seems simple enough. Why not try it in 4D space?

4D Method
================

We will consider a [4D hypersphere](https://en.wikipedia.org/wiki/N-sphere) of radius = 1 centered at the point $(0,0,0,0)$ 
inscribed in a [4D hypercube](https://en.wikipedia.org/wiki/Hypercube) (or tesseract) with length 2 in each direction. 
Please forgive the lack of a hand-drawing for this example. 

After doing some googling, I found the equation for hypervolume of a 4D hypersphere:

$V_{hypersphere} = \frac{1}{2}\pi^{2}r^{4}$  

And, as expected for the 4D hypercube: $V_{hypercube} = length^{4}$

Plugging in our values and simplifying:

$\frac{V_{hypersphere}}{V_{hypercube}} = \frac{\frac{1}{2}\pi^{2}1^{4}}{2^{4}} = \frac{1}{32}\pi^{2}$

So $\pi \approx \sqrt{32*\frac{points\ in\ hypersphere}{points\ in\ hypercube}}$

This will follow the same pattern as the previous methods in that $(x_{1})^{2}+(x_{2})^{2}+(x_{3})^{2}+(x_{4})^{2}=1$. 
If this sum is less than or equal to 1, then the point falls within the hypersphere.

The code for this function is as follows: 

``` python

import math
def montecarlo4d(num_simulations):
    num_nsphere = 0
    num_hypercube = 0
    for i in range(num_simulations):
        x1 = random.uniform(-1,1)
        x2 = random.uniform(-1,1)
        x3 = random.uniform(-1,1)
        x4 = random.uniform(-1,1)
        if (x1)**2 + (x2)**2 + (x3)**2 + (x4)**2 <= 1:
            num_nsphere += 1
        num_hypercube += 1
    return math.sqrt(32 * (num_nsphere/num_hypercube))
```

Output:

```
>>> montecarlo4d(1000000)
3.140491681249928
```
It actually works!

Algorithm Analysis
=====================

Now, let's measure the run-times of each of these functions using ```time.time()```, and plot with 
```matplotlib.plotly```. 

![screenshot](/photos/estimatepi3.png){:class="img-responsive"}

Nothing surprising here; run time increases as spatial dimensions are added. This is likely due to the increase 
in randomly generated values assigned to variables, and the general complexity of calculations.

These algorithms would still take a long time to run for, say, ```num_simulations = 100 million```. Let's 
see if we can make these functions run faster.

**IMPROVEMENTS**

-For each of the functions, we can replace ```random.uniform(-1,1)``` with ```random.random()``` which simply 
generates a decimal value between 0 and 1. This should be quicker than the former, and won't affect the outcome 
because these values are squared in the next step. 

-We can delete the variable that counts the number of points within the square (or cube or hypercube). This 
number is simply equal to ```num_simulations``` and its computation is wasting valuable time!

Let's see how much of a difference these changes make.

![screenshot](/photos/estimatepi4.png){:class="img-responsive"}

Wow! Compare the vertical scale on this plot with the previous plot. **These changes cut our run-time 
approximately in half!** This is a bit of an eye-opener to the impact of just a few lines of code. There are many 
solutions to the same problem, and speed can certainly be an important factor on large-scale projects or resource-
intensive scripts. In this case, it's not all that important unless you want to run a very large number of simulations.

Conclusion
================

A few takeaways from this project:

1. Monte Carlo simulations are highly interesting and very do-able in Python.
2. Even a few lines of code can have a major impact on run-time, and I will definitely look for ways to improve my 
algorithms in future projects. This is something I haven't cared too much about in the past.

Feel free to check out my code for this project in [my GitHub repository](https://github.com/justinbrinkman/applypython) 
and experiment with it or use it for your own project.

Thanks for reading and feel free to leave a comment or visit the 'Connect' page!
