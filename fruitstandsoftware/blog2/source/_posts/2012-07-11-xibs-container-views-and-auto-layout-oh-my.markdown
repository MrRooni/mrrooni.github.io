---
author: MrRooni
comments: false
date: 2012-07-11 20:23:01+00:00
layout: post
slug: xibs-container-views-and-auto-layout-oh-my
title: XIBs, Container Views, and Auto Layout (oh my)
wordpress_id: 596
categories:
- Cocoa
- Sample Code
---

While working on a major update for one of our products at [No Thirst](http://nothirst.com/) I ran across a small implementation question: In the world of auto layout, if you have a window whose subviews are managed by view controllers, what is the best way to layout these subviews? In the realm of springs and struts the answer is pretty straightforward: You add container views to your window, define their autoresizing masks in the XIB, add your view controller views as subviews of the container views in code, and set their frame sizes to match their container view frame sizes. As long as you set the autoresizing masks of the view controller views to be **NSViewHeightSizable | NSViewWidthSizable** in their XIBs you get the behavior you're expecting and the code is pretty minimal.

[caption id="attachment_605" align="aligncenter" width="513"][![](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/07/ShmetBencherWindow.png)](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/07/ShmetBencherWindow.png) This is the window we're trying to create[/caption]



If you try to follow a similar pattern while using auto layout and do most of your work in XIBs with very little code you hit a bit of a curve in the road. With springs and struts you set your autoresizing masks on individual views allowing you to define what each view controller's view should do when added to a super view. Layout constraints (instances of [NSLayoutConstraint](https://developer.apple.com/library/mac/#documentation/AppKit/Reference/NSLayoutConstraint_Class/NSLayoutConstraint/NSLayoutConstraint.html)) define relationships **_between_** views. This means that in order to create a layout constraint between two views both views need to be present at the time the relationship is defined. Thinking I could outsmart the system I had what I thought was a eureka moment. "Ah ha!" I thought, "I'll follow the same pattern of using container views, but in each view controller's awakeFromNib I'll set up some constraints that mimic NSViewHeightSizable | NSViewWidthSizable. I'm a GENIUS!"

```objective-c
    - (void)awakeFromNib
    {
        NSDictionary *viewsDictionary = @{ @"view":self.view };

        NSArray *horizontalMaximizingConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"|[view]|"
                                                                                           options:0
                                                                                           metrics:nil
                                                                                             views:viewsDictionary];

        NSArray *verticalMaximizingConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"V:|[view]|"
                                                                                         options:0
                                                                                         metrics:nil
                                                                                           views:viewsDictionary];
        [self.view addConstraints:horizontalMaximizingConstraints];
        [self.view addConstraints:verticalMaximizingConstraints];
    }
```


Uh, yeah, not so much:


    2012-07-11 14:02:07.339 Shmet Bencher[15158:303] *** Terminating app due to uncaught
    exception 'NSInvalidArgumentException', reason: 'Unable to parse constraint format:
    Unable to interpret '|' character, because the related view doesn't have a superview
    |[view]|
           ^'


Apparently there was a reason interface builder wouldn’t let me define those constraints visually.


## Possible Solutions


As far as I can see there are two possible solutions to the original question:







  1. Follow the container view pattern. In your XIB you define the relationships between the container views and then in code you add your view controller views to those container views and set up constraints similar to the ones I posted above. The difference being that the | character will now represent a super view that actually exists.


  2. Skip container views, leave your XIB alone, add all your view controller views as direct subviews of the window and then define the relationships between them.


Option 1 is great because working with auto layout in interface builder allows you to see the immediate results of changing constraints. However, interface builder will also inject constraints into your layout as you're working to try and make sure you don't end up with an ambiguous layout or unsatisfiable constraints. Option 1 is also nice because the code you end up writing to add the view controller views to their containers essentially becomes boilerplate. Yes, there is a lot of it, but because of the nature of it, it's easy to see when you've made a mistake.

Option 2 is great because you've reduced your view hierarchy and the constraints associated with each view were put there by you without interface builder getting in the way. Option 2 does suffer from the annoying side effects of having to write more code and not allowing you to play with your window size to see how constraints react until you've created a set of constraints that properly defines the layout for the entire window.

So what's the answer?






## The Answer


The answer, of course, is, "it depends". According to Apple you should define your constraints using these methods, in descending order of preference:




  * Within interface builder


  * Using the visual format language in code (as I did above)


  * Individually using the **constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:** class method on NSLayoutConstraint


In this particular case I'm going to charge forward with Option 2 and I will update this post if that turns out to be a horrible idea. If I had a mixture of other elements in this window like labels or other buttons I'd probably go for Option 1 but since it's just views, and only a handful of them at that, I'm going to code it. However, for all of the view controller views I will be doing their layout in interface builder.

If you've got some feedback, typos to point out, or just want to type obscenities at me, you should get in touch with me via Twitter: [@MrRooni](http://twitter.com/MrRooni)

If any of the information above is misinformed or just plain wrong definitely get in touch.
