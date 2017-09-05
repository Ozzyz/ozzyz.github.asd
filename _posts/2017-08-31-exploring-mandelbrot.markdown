---
layout: post
title: "Exploring the Mandelbrot set"
date: 2017-08-31 22:14:00 +0200
categories: math, mandelbrot
---

[The Mandelbrot set][mandelbrot-set] is a set of complex numbers, where each number in the set, $$z_n$$ has the attribute that the sequence


$$z_{n+1} = z_{n}^2 + c, z_0 = 0$$


is bounded, meaning that is does not diverge. There are many interesting things about this set, but the one I find the most interesting is the numerous graphical representations of the set. The picture below is from the Wikimedia Commons Free Media Library.
![From Wikimedia Commons]({{ site.url }}/assets/images/mandelbrot.jpg)

I have always wanted to draw some of these myself, so today I'll do a quick rundown of how to achieve similar drawings. The only libraries I will be using is [matplotlib][matplotlib], and [numpy][numpy]. At the end, we will speed up the code by using [numba][numba]

We can start by defining the area the sequence will operate on. The whole mandelbrot set is interesting between $$\begin{equation} x \in [-2.5, 1] \end{equation}$$ and $$\begin{equation} y \in [-1, 1] \end{equation}$$, so this is where we will define our borders. Also, we need to decide how many points we will evaluate. We will divide our x and y values into N equally spaced points. The linspace function in numpy is perfect for this. 

{% highlight python %}

# Our resulting image will be NxN pixels
num_pixels = 7500
min_x, max_x = -2.5, 1
min_y, max_y = -1, 1

X = np.linspace(min_x, max_x, num_pixels)
Y = np.linspace(min_y, max_y, num_pixels)

max_len = 4
max_iters = 255
{% endhighlight %}

Now that we have defined our domain, we just need to iterate through the pixels and calculate the number of iterations for each complex number. The only thing that sticks out here is how we define a complex number. In Python, you don't need to create your own representation. You can represent imaginary numbers in an easy manner. The 'complex' class has everything we need (and more!) implemented.
{% highlight python %}
num = 3 + 4j

type(num)
#=> <class 'complex'>
abs(num)
#=> 5.0
num.real
#=> 3
num.imag
#=> 4
{% endhighlight %}

Our main loop will be really simple. We iterate through each value, calculating the mandelbrot sequence and seeing if it is bounded. The number of iterations the number takes will define the pixel intensity of the number. I will use matplotlibs colormap 'hot', which maps float values from 0 to 1 into RGBA-values. To make PIL accept this, we need to modify the floats to ints ranging from 0 to 255. 

{% highlight python %}
def mandelbrot(X, Y, max_iters, max_len):
    iters = np.zeros((X.size, Y.size), dtype=np.uint)

    for i in range(X.size):
        for j in range(Y.size):
            num_iters = 0
            c = X[i] + 1j*Y[j]
            z = c
            while(abs(z) < max_len and num_iters < max_iters):
                z = z**2 + c
                num_iters += 1
            iters[i, j] = num_iters
    return iters
{% endhighlight %}

Now we can show our image by calling plt.imshow(iters) or saving it by calling plt.imgsave(filename, iters).

This is all great, except one little thing. It is pretty slow! Ideally we want to create large pictures, which takes a lot of time. To estimate how long time the calculations take, we use the python time module. This is not a good method if we want to estimate short periods, but we are interested in approximately how many seconds the method takes, so it will do. 

{% highlight python %}

import time

start = time.time()
img = mandelbrot(X, Y, max_iters, max_len, num_pixels)
end = time.time()

print("%sx%s sized image finished in %s seconds" % (num_pixels, num_pixels,end-start))
    
{% endhighlight %}

When I set num_pixels to 2000, the method takes approximately 131 (!) seconds on my slow laptop. Fortunately for us, we can speed up this code drastically by a couple of lines of code! (There are probably a lot of other speedup options here, for example by not skipping 'boring' areas in the set and focus on the edges, but you can explore those on your own).

We are going to speed up our code by using a concept called 'just in time compiling', or 'jit'. We are going to use a library called Numba which does this, basically by compiling Python code into optimized machine code for the LLVM compiler. The library works best for C-like code, for example iterating over an array. Since our mandelbrot function just iterates over an array and computes stuff, this is a perfect target for Numba. To use the library (assuming you already have downloaded it), just import jit, and decorate the mandelbrot function we already have created.

{% highlight python %}
from numba import jit

@jit
def mandelbrot(X, Y, max_iters, max_len):
...
{% endhighlight %}

When creating the same 2000x2000 image, the code now runs in 2.9 seconds! That is a drastic speed-up for such few additional lines of code. 

To end of this post, lets look at some cool images! You can look at different areas of the mandelbrot set by adjusting the min and max limits for x and y.

![The whole mandelbrot set]({{ site.url }}/assets/images/mandelbrot_own.png)
First, the whole mandelbrot set $$ x \in [-2.5, 1], y \in [-1, 1] $$.


![Zoomed at x: [-0.8, -0.7] y: [0.1, 0.2]]({{ site.url }}/assets/images/mandelbrot_zoomed.png)

This is zoomed at $$ x \in [-0.8, -0.7], y \in [0.1, 0.2] $$


![Zoomed at x: [0, 0.002] y: [-0.823, -0.821]]({{ site.url }}/assets/images/mandelbrot_zoomed2.png)

This is zoomed at $$ x \in [0, 0.002], y \in [-0.823, -0.821] $$


[numba]: https://numba.pydata.org/
[pil]: https://pillow.readthedocs.io/en/4.2.x/
[numpy-mesh]: https://docs.scipy.org/doc/numpy/reference/generated/numpy.meshgrid.html
[python-pow]: https://docs.python.org/3/reference/datamodel.html#emulating-numeric-types
[numpy]: http://www.numpy.org/
[mandelbrot-set]: http://mathworld.wolfram.com/MandelbrotSet.html
[matplotlib]: https://matplotlib.org/