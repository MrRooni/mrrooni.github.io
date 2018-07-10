---
author: MrRooni
comments: false
date: 2012-06-05 23:23:21+00:00
layout: post
slug: what-to-do-if-your-sandbox-application-shows-up-as-not-sandboxed
title: What To Do If Your Sandboxed Application Shows Up As Not Sandboxed
wordpress_id: 570
categories:
- Cocoa
---

This afternoon I started working on turning [MoneyWell for Mac](http://nothirst.com/moneywell/) into a sandboxed application for our next major release. I watched the [intro videos](https://developer.apple.com/devcenter/mac/app-sandbox/), checked the appropriate checkboxes in Xcode, ran MoneyWell, checked Activity Monitor and saw...

[![](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/06/Activity-Monitor-1-1.jpg)](http://www.fruitstandsoftware.com/blog/wp-content/uploads/2012/06/Activity-Monitor-1-1.jpg)



Well crap. After a bit of unsuccessful searching on the [Apple Dev Forums](https://devforums.apple.com/community/mac?view=discussions) I did some testing with [Kevin Hoctor](http://twitter.com/kevinhoctor) and discovered that the **Release** configuration of MoneyWell was properly sandboxed. The only significant difference between the **Release** and **Debug** configurations was that one was code signed and one was not. Once we enabled code signing for the **Debug** configuration MoneyWell launched as a sandboxed app.

I asked on Twitter,


> [Is it common knowledge that an app that is not code signed will run in non-sandboxed mode even with sandboxing enabled?](http://twitter.com/MrRooni/status/210136419224649728)


Both Brian Webster and Jim Correia got back to me:


> [@bwebster](http://twitter.com/bwebster): [That does make sense, since it is the code sign tool that's used to encode the sandbox entitlements when building.](http://twitter.com/bwebster/status/210142826468618240)

[@jimcorreia](http://twitter.com/jimcorreia): [The app-sandbox is an entitlement. Entitlements are embedded in the code signature.](http://twitter.com/jimcorreia/status/210143389579096065)


Hopefully this helps you out if you find that your sandboxed app is showing up as not sandboxed.
