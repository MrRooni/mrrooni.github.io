---
author: MrRooni
comments: false
date: 2012-08-22 18:13:39+00:00
layout: post
slug: quick-and-easy-debugging-of-unrecognized-selector-sent-to-instance
title: Quick and easy debugging of unrecognized selector sent to instance
wordpress_id: 629
categories:
- Cocoa
- Debugging
---

It's happened to all of us; we're merrily trucking down the development road, building and testing our app when all of sudden everything grinds to a screeching halt and the console tells us something like:

**-[MoneyWellAppDelegate doThatThingYouDo]: unrecognized selector sent to instance 0xa275230**

Looking at the Xcode backtrace doesn't seem to help either since it most likely looks like so:

[![](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/08/Screen-Shot-2012-08-22-at-2.02.11-PM.png)](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/08/Screen-Shot-2012-08-22-at-2.02.11-PM.png)



At this point you start the hunt through your code looking to see who sent the -doThatThingYouDo message. You may find the answer right away and things may be all hunky dory, or you may spend the next hour trying to figure out where the hell that call is coming from.

The good news is there is a better way. All you need to do is pop over to the Breakpoint Navigator, click the + button at the bottom and choose **Add Symbolic Breakpoint...**

In the **Symbol** field enter this symbol:

    
    -[NSObject(NSObject) doesNotRecognizeSelector:]


Now when any instance of any object within your program is sent a message to which it does not respond you will be presented with a backtrace that takes you right to the point where that message was sent.

[![](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/08/Screen-Shot-2012-08-22-at-2.09.04-PM.png)](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/08/Screen-Shot-2012-08-22-at-2.09.04-PM.png)



Cheers and happy debugging.
