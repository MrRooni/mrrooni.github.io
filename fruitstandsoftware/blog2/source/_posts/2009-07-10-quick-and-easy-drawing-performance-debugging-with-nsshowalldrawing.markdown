---
author: Michael
comments: true
date: 2009-07-10 14:59:06+00:00
layout: post
slug: quick-and-easy-drawing-performance-debugging-with-nsshowalldrawing
title: Quick and Easy Drawing Performance Debugging with NSShowAllDrawing
wordpress_id: 324
categories:
- Cocoa
- Debugging
tags:
- Bezipped
- Debugging
- drawing
- interface_builder
- performance
- WWDC
---

While watching one of the WWDC09 session videos I was informed of a great tip that I had been previously unknown to me: Pass `-NSShowAllDrawing YES` as an argument to your application in Xcode to see a visual representation of the drawing that your application performs as it runs.

To illustrate how NSShowAllDrawing works and the issues it can help you correct I've put together two videos.  The first shows my app, Bezipped, in its current 1.0 state and its current drawing behavior.



This second video shows how I improved the drawing in Bezipped simply by setting the top-level container to be backed by a Core Animation layer:



I highly recommend giving your app a spin with NSShowAllDrawing if you haven't already, it was certainly a real eye-opener for me.  There are some additional resources for debugging your drawing performance on OS X (as pointed out to me by [André Pang](http://twitter.com/AndrePang)) provided by Apple here: [Drawing Performance Guidelines: Measuring Drawing Performance](http://developer.apple.com/documentation/Performance/Conceptual/Drawing/Articles/MeasuringPerformance.html#//apple_ref/doc/uid/20001875-98880)

Lastly, both [Alan Rogers](http://twitter.com/alancse) and [Steve Streza](http://twitter.com/SteveStreza) pointed me towards Quartz Debug.app (included with the developer tools) as another means to see similar redrawing behavior.  I found Quartz Debug's options to be a bit heavy-handed as the drawing performance of the entire OS was shown instead of just my app, but your mileage may vary.
