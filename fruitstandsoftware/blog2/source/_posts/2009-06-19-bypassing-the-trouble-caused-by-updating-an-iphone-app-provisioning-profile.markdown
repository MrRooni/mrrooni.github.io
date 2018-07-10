---
author: Michael
comments: true
date: 2009-06-19 17:45:33+00:00
layout: post
slug: bypassing-the-trouble-caused-by-updating-an-iphone-app-provisioning-profile
title: Bypassing the Trouble Caused by Updating an iPhone App Provisioning Profile
wordpress_id: 244
categories:
- Cocoa
- Debugging
---

**[UPDATE]** After many discussions on Twitter and many recommendations by different folks, I think that we have determined that the method outlined below is not necessary.  [Mike Taylor](http://twitter.com/heymac) has hit upon what appears to be a foolproof method for getting around the trouble caused by updating an iPhone app provisioning profile, and best of all, he did it in 140 characters:


> @[kalperin](http://twitter.com/kalperin) @[MrRooni](http://twitter.com/MrRooni) Delete old profile from Organizer. Download new profile 'n drag to organizer. Restart Xcode. Choose new profile in target


Many thanks to Mike for helping me out here, I owe you a beer at WWDC10.  For those of you interested in the more masochistic way to get around the issues, feel free to continue reading:

A note before I begin: Everything below was done with the final build of the iPhone OS 3.0 SDK and I was building an app using the 2.2.1 frameworks.

I recently acquired a new iPod touch to use a development device.  One of the first things that I wanted to do was get my existing project up and running on the device so I headed over to the iPhone Development Program Portal to update my provisioning profile.

After adding the new device to my app's provisioning profile I downloaded the updated profile and installed it into Xcode and onto the device via the Organizer.  So far so good, everything up until this point worked exactly as I expected it to.

Switching back to my project I changed my build settings to be a Release build under 2.2.1 on the device.  The app built fine, but I got an error when it tried to install the app onto the device, something error akin to "This device doesn't contain the provisioning profile with which this app was built".  Thinking that maybe Xcode just hadn't seen the new profile yet I cleaned all targets in my project, restarted Xcode and tried again.  No dice, same error as before.

I then proceeded to delete all versions of the profile from the organizer and re-installed the new one.  My assumption was that once Xcode saw that the old profile didn't exists anymore it would switch over and use the new one.  I cleaned all targets and built again.  This time I was treated to a different error: "Code Sign error: Provisioning profile '3E6AA725-6534-46F8-B9CE-D19AC9FD854B' can't be found"

After a bit of Googling I discovered that Xcode stores the ID of the provisioning profile in its project.pbxproj file.  This discovery led me to the fix:



	
  1. Close your Xcode project

	
  2. Navigate to your project folder in the Finder

	
  3. Right click on your .xcodeproj file and 'Show Package Contents'

	
  4. Drag the project.pbxproj file to Xcode (or any plain text editor)

	
  5. Perform a search for the term 'provision' to find the PROVISIONING_PROFILE entry.

	
  6. Copy the existing profile ID and paste it into the find field of a find and a replace dialog.

	
  7. Open up the Organizer window (Window menu > Organizer) and navigate to your new profile under IPHONE DEVELOPMENT > Provisioning Profiles

	
  8. Click on your provisioning profile and copy its Profile Identifier

	
  9. Paste the string into the replace field in your open find and replace dialog.

	
  10. Replace all instances of the identifier, save the file, close it, and reopen your Xcode project.

	
  11. That should do it, build and go to run your app on your new device.


Now there is a great possibility that I am going WAY overboard here and missing a very obvious way to accomplish the same solution.  If that's the case please let me know in the comments.
