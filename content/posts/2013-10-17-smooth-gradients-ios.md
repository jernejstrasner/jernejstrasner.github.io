---
title: "Smooth gradients on iOS/OSX"
date: 2014-01-08 23:00:00
images:
toc: false
draft: false
tags:
    - ios
    - osx,
    - objc
    - graphics
---

__UPDATE__: Updated for Swift, Swift 2 and packaged into a [drop-in replacement for UIView](https://github.com/jernejstrasner/Smooth-Gradient).

I thought drawing gradients on iOS is easy. Well, until our designer gave me this:

![Smooth gradient drawn in Photoshop](/images/2013-10-17/gradient_ps.png)

I went on and implemented the `drawRect:` method on a custom UIView. I used `CGGradient` and wrote the drawing code. I ran the project in the simulator and noticed that the gradient was quite different. Ran on the device, same thing.

![Gradient drawn with CGGradient](/images/2013-10-17/gradient_cg.png)

I played around with different colors and their positions but nothing turned out like the Photoshop gradient. After some head scratching I figured it must be something to do with how the colors are interpolated. `CGGradient` uses a linear interpolation of colors.

![Plot y = x](/images/2013-10-17/linear.png)

What I need is a slope (ease-in ease-out for you familiar with animation timings) function and apply that to each color component. I actually cared only about transparency so I had to only adjust the alpha channel.
I needed this:

![Plot slope function](/images/2013-10-17/slope_func.png)

![Plot slope](/images/2013-10-17/slope.png)

Looking at the documentation, `CGGradient` doesn't support custom color interpolation. There is something called `CGShading` that does. The documentation is not absolutely clear on how to use it, but after some experimenting I came up with a solution, which is presented below. There is also a sample project available on [GitHub](https://github.com/jernejstrasner/Smooth-Gradient) so you can compare CGGradient and CGShading.

Hope somebody finds this useful!

{{< gist jernejstrasner 6319645344d17e7df8aa >}}
