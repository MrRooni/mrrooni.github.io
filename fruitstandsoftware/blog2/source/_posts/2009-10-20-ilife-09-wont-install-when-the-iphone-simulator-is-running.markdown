---
author: Michael
comments: false
date: 2009-10-20 23:59:43+00:00
layout: post
slug: ilife-09-wont-install-when-the-iphone-simulator-is-running
title: iLife 09 Won't Install When the iPhone Simulator Is Running
wordpress_id: 362
---

This post is a bit of a deviation from my usual posts, but it falls into the **something-that-bit-me-that-might-bite-you-too-so-here's-how-to-fix-it** category.

I recently upgraded to Snow Leopard by doing an erase and install.  After getting everything back up and running I realized that I still needed to install iLife.  I popped in disc 2 of my MacBook Pro install DVDs and proceeded to open the Install Bundled Software installer.  After customizing my options I hit the install button and waited. And waited. And waited.  The installer never got past the Preparing Bundled Applications stage.

I checked out the console and saw a series of messages that ended with:
`10/17/09 4:23:10 PM	Installer[8135]	*** -[NSMachPort handlePortMessage:]: dropping incoming DO message because the connection or ports are invalid`

I decided to try a restart to see if that would fix my problem but shortly after restarting and before I could try my install again I cranked open Xcode and the iPhone simulator to check out something in [my latest project](http://www.getsportsbookapp.com/).  Remember this part of the story, because it's important.

A few days later I remembered that I still needed to install iLife and grabbed my MacBook Pro installation discs again. I kicked off the installation and instantly hit the same issue.  The installer never got past the Preparing Application Bundles stage.  I opened Console.app again and saw the same message:
`10/20/09 7:39:40 PM	Installer[9025]	*** -[NSMachPort handlePortMessage:]: dropping incoming DO message because the connection or ports are invalid`

I decided to turn to Google and happily came across this gem of a radar filing: [Installer hanging for any installer after uptime of X hours](http://www.openradar.appspot.com/7105069).  If you read through the comments on that bug you will find that the installer hang isn't based on uptime, but rather on the iPhone Simulator running.  I had been working on an iPhone project both times I tried to install iLife and it was causing the installer to hang (for some inexplicable reason).

I quit the simulator and Installer.app immediately stopped trying to prepare the bundled applications and displayed an error.  I quit the installer, tried again, and voila, everything worked as it should.
