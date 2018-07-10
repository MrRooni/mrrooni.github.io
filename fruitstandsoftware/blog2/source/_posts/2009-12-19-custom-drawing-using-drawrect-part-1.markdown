---
author: Michael
comments: false
date: 2009-12-19 02:01:28+00:00
layout: post
slug: custom-drawing-using-drawrect-part-1
title: Custom Drawing Using drawRect, Part 1
wordpress_id: 437
categories:
- Cocoa
- Design
- Sample Code
tags:
- Cocoa
- drawing
- Objective-C
- Sample Code
- UIView
---

One of the more advanced techniques for creating custom user interfaces on the Mac is the use of NSView's drawRect method.  Many answers to questions on StackOverflow and Apple's mailing lists include recommendations to "just override drawRect and do the drawing yourself".  Some folks see this recommendation and their eyes glaze over, thinking that it's too advanced of a technique for them to wrap their heads around.  Over the next few days I'm going to go over some basic techniques that can yield powerful results.




Let's start by setting up the Xcode project that will be the basis of the rest of these posts.







  * Open Xcode and create a new Cocoa Application project called DrawingSample.


  * Create a new NSView subclass called CustomDrawingView.


  * Open MainMenu.xib, add a new Custom View to the Main Window, set its class to be CustomDrawingView, and set it's autosizing flags as seen here:


[![DrawingSample IB Layout](http://fruitstandsoftware.com/blog/wp-content/uploads/2009/12/Screen-shot-2009-12-18-at-8.14.10-PM.png)](http://fruitstandsoftware.com/blog/wp-content/uploads/2009/12/Screen-shot-2009-12-18-at-8.14.10-PM.png)


Save and Quit Interface Builder and switch back to Xcode.  Open CustomDrawingView.m, it should look like so:


```objective-c
    @implementation CustomDrawingView

    - (id)initWithFrame:(NSRect)frame {
        self = [super initWithFrame:frame];
        if (self) {
            // Initialization code here.
        }
        return self;
    }

    - (void)drawRect:(NSRect)dirtyRect {
        // Drawing code here.
    }

    @end
```

We're going to start (and finish) today with just a simple concept and some basic drawing that will set the stage for the future posts. All drawing in Cocoa is done by first setting up the environment in which you want to draw, and then doing the actual drawing. For instance, if we want to draw a blue box, we first have to setup the color blue, define the bounds of the box, and then draw it.  In this case we are using the NSRect that is passed to the drawRect method as the box we want to draw, and we setup the color blue by calling [[NSColor blueColor] set].  We then use the convenience method NSRectFill to fill the dirtyRect with the color blue.  Notice that we didn't pass the color to NSRectFill, we set it, and from then on anything we draw will be blue until we change the color.




You can think of drawing in Cocoa much the same way as you would think of painting with a brush.  You dip your brush in a certain paint color, paint the shape you want to paint, and then dip your brush in a new color and paint some more.



```objective-c
    - (void)drawRect:(NSRect)dirtyRect {
    	[[NSColor blueColor] set];
    	NSRectFill(dirtyRect);
    }
```




The preceding code, when run, will generate a view that looks like this:




[![NSRectFill](http://fruitstandsoftware.com/blog/wp-content/uploads/2009/12/Screen-shot-2009-12-18-at-8.51.12-PM.png)](http://fruitstandsoftware.com/blog/wp-content/uploads/2009/12/Screen-shot-2009-12-18-at-8.51.12-PM.png)Now, this may not look like much, but in future posts we will build on these concepts and, hopefully, by the end have drawn some pretty cool and useful things.
