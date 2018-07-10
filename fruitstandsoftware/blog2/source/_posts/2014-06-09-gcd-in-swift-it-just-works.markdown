---
layout: post
title: "GCD in Swift: It Just Works"
date: 2014-06-09 17:02:07 -0400
comments: true
categories: [Swift,GCD]
author: Michael Fey
---

With the awesome announcement of [Swift](https://developer.apple.com/swift/) last week, alongside [all of the other incredible announcements,](https://developer.apple.com/technologies/) I'm sure that your reaction was similar to mine: Where do I begin?!

I've been reading [the book](https://itunes.apple.com/us/book/swift-programming-language/id881256329?mt=11), watching [the sessions](https://developer.apple.com/videos/wwdc/2014/), checking out [the sample code](https://developer.apple.com/library/prerelease/ios/navigation/), and basically trying to orient myself with the new syntax as much as possible.

One question I had today while listening to the latest episode of the [Debug](http://www.imore.com/debug) podcast was, "How will Grand Central Dispatch (GCD) work in Swift? Will GCD work in Swift?" With that question in mind I settled in, opened Xcode, and chose File > New > Project...

I chose a *Single View Application* since I wouldn't need much of app to answer my question. I named it SwiftlyGCD and chose Swift as the language. I opened ```ViewController.swift``` and started typing out ```dispatch``` to see what would code-complete. Happily all of the wonderful GCD methods that we know and love appeared in the code-completion list in their Swift-equivalent notation.

For instance, ```dispatch_async()``` in Objective-C looks like so:

```objective-c
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
```

Whereas in Swift it appears like this:

```objective-c
func dispatch_async(queue: dispatch_queue_t!, block: dispatch_block_t!)
```

As you can see we get a one for one translation from C to Swift thanks to Xcode 6. Additionally we get some extra niceties like named parameters. What does the code look like in usage?

```objective-c
		dispatch_async(dispatch_get_main_queue(), {
			println("Currently dispatched asynchronously")
			})
```

The call to ```dispatch_get_main_queue()``` is exactly the same as we're used to, while the block syntax is simplified to just a pair of ```{}```.

Bottom line: GCD is fully available in Swift and works exactly as we'd expect. Way to go Apple!

**UPDATES**

After this post went live on Twitter [Jacob Gorban](http://twitter.com/jacobgorban/) replied and said, ["It should even work with trailing block, no?"](http://twitter.com/jacobgorban/status/476353702857424897)

He's right! You can write the above dispatch like so:

```objective-c
		dispatch_async(dispatch_get_main_queue()) {
			println("Currently dispatched asynchronously")
			}
```
