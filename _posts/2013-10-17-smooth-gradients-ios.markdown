---
layout: post
title:  "Smooth color transition gradients with CoreGraphics"
date:   2014-01-8 23:00:00
categories: ios osx objc graphics
---

I thought drawing gradients on iOS is easy. Well, until our designer gave me this:

![Smooth gradient drawn in Photoshop]({{ site.url }}/images/2013-10-17/gradient_ps.png)

I went on and implemented the `drawRect:` method on a custom UIView. I used `CGGradient` and wrote the drawing code. I ran the project in the simulator and noticed that the gradient was quite different. Ran on the device, same thing.

![Gradient drawn with CGGradient]({{ site.url }}/images/2013-10-17/gradient_cg.png)

I played around with different colors and their positions but nothing turned out like the Photoshop gradient. After some head scratching I figured it must be something to do with how the colors are interpolated. `CGGradient` uses a linear interpolation of colors.

![Plot y = x]({{ site.url }}/images/2013-10-17/linear.png)

What I need is a slope (ease-in ease-out for you familiar with animation timings) function and apply that to each color component. I actually cared only about transparency so I had to only adjust the alpha channel.
I needed this: <img src="{{ site.url }}/images/2013-10-17/slope_func.png"/>

![Plot slope]({{ site.url }}/images/2013-10-17/slope.png)

Looking at the documentation, `CGGradient` doesn't support custom color interpolation. There is something called `CGShading` that does. The documentation is not absolutely clear on how to use it, but after some experimenting I came up with a solution, which is presented below. There is also a sample project available on [GitHub](https://github.com/jernejstrasner/Smooth-Gradient) so you can compare CGGradient and CGShading.

Hope somebody finds this useful!

{% highlight objc %}
- (void)drawRect:(CGRect)rect
{
	// Draw the gradient background
	CGContextRef context = UIGraphicsGetCurrentContext();
	CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceGray();

	// Define the shading callbacks
	CGFunctionCallbacks callbacks = {0, blackShade, NULL};

	// As input to our function we want 1 value in the range [0.0, 1.0].
	// This is our position within the 'gradient'.
	size_t domainDimension = 1;
	CGFloat domain[2] = {0.0f, 1.0f};

	// The output of our function is 2 values, each in the range [0.0, 1.0].
	// This is our selected color for the input position.
	// The 2 values are the white and alpha components.
	size_t rangeDimension = 2;
	CGFloat range[8] = {0.0f, 1.0f, 0.0f, 1.0f};

	// Create the shading finction
	CGFunctionRef function = CGFunctionCreate(NULL, domainDimension, domain, rangeDimension, range, &callbacks);

	// Create the shading object
	CGShadingRef shading = CGShadingCreateAxial(colorSpace, CGPointMake(1, rect.size.height), CGPointMake(1, rect.size.height*0.75f), function, YES, YES);

	// Draw the shading
	CGContextDrawShading(context, shading);

	// Clean up
	CGFunctionRelease(function);
	CGShadingRelease(shading);
	CGColorSpaceRelease(colorSpace);
}

// This is the callback of our shading function.
static void blackShade(void *info, const CGFloat *inData, CGFloat *outData)
{
	float p = inData[0];
	outData[0] = 0.0f;
	outData[1] = (1.0f-slope(p, 2.0f)) * 0.5f;
}

// ristributes values on a slope (ease-in ease-out)
static float slope(float x, float A)
{
	float p = powf(x, A);
	return p/(p + powf(1.0f-x, A));
}
{% endhighlight %}