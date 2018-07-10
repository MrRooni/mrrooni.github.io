---
author: Michael
comments: true
date: 2008-08-05 02:03:14+00:00
layout: post
slug: finding-memory-leaks-with-the-llvmclang-static-analyzer
title: Finding Memory Leaks With The LLVM/Clang Static Analyzer
wordpress_id: 15
categories:
- Cocoa
- Debugging
---

 




While you may be familiar with [using tools like Instruments to find and fix memory leaks in your application](http://www.cimgf.com/2008/04/02/cocoa-tutorial-fixing-memory-leaks-with-instruments/), the Clang Static Analyzer takes a different approach to memory leak detection by compiling your Xcode project and scanning each method, class, loop, and logic block for potential leaks. You may have heard of the Clang Static Analyzer referred to by the name of the command line tool used to run the analyzer: scan-build. That is how I will be referring to it for the remainder of this post.


### Requirements & Where To Get It


scan-build is currently only available in binary form for OS X 10.5.
If you haven't yet downloaded scan-build head on over to [the LLVM/Clang Static Analyzer homepage](http://clang.llvm.org/StaticAnalysis.html) and look for the Download section at the bottom of the page. Click the link for checker-NN.tar.gz (where NN is some build number). At the time of this writing the link reads checker-72.tar.gz. The developers of scan-build are very active so I have no doubt that the build number is already different.


### Installation


Since scan-build is a command line tool it makes sense to install it into one of OS X's pre-defined command line tool locations. We'll put it in /usr/local/bin.



	
  1. Make sure that you've expanded checker-NN.tar.gz to your Downloads folder

	
  2. We're going to be installing the checker binaries into your /usr/local/bin directory.  Run the following command to ensure that this directory exists:


    
     sudo mkdir -p /usr/local/bin


	
  3. Open Terminal.app and move the contents of checker-NN to the /usr/local/bin directory (remember to replace NN with the build number of your download):

    
    sudo mv ~/Downloads/checker-NN/* /usr/local/bin/







### Basic Usage


scan-build tests your code by compiling your Xcode project and studying it for defects during the build process. To check your code, you just invoke scan-build from the command line at the top level of any one of your project directories.



	
  1. Still in Terminal.app, change into one of your Xcode project directories

	
  2. Run scan-build on your Xcode project:


    
    scan-build xcodebuild


There's quite a bit of output when scan-build runs, but once it finishes running you will either see

    
    ** BUILD SUCCEEDED **
    scan-build: No bugs found.


or something similar to

    
    ** BUILD SUCCEEDED **
    
    scan-build: 7 bugs found.
    
    scan-build: Open '/tmp/scan-build-fw1RAD/2008-07-31-1/index.html' to examine bug reports.


	
  3. Copy the section similar to Open '/tmp/scan-build-fw1RAD/2008-07-31-1/index.html', paste it back onto the command line and hit return.

	
  4. You'll be presented with the Summary screen. Click on the View link next to each bug to see your code with an inset bubble describing the bug that scan-build found




### iPhone Usage


Using scan-build with the iPhone requires a little extra tweaking in your Xcode project settings to make sure that you are compiling your project using an SDK that is compatible with scan-build's compiler.  The rest of these instructions assume a project who's configuration has not been modified beyond what is provided when you create a new project.  I will be working with a project titled _WhatsMyIP_.



	
  1. Open the iPhone Xcode project that you want to run scan-build on and bring up the project settings panel.
![Open Project Settings](http://fruitstandsoftware.com/blog/wp-content/uploads/2008/08/projectsettings.png)

	
  2. On the **General** tab change the **Base SDK for All Configurations** to one of the Simulator SDKs.
![Project Settings - General](http://fruitstandsoftware.com/blog/wp-content/uploads/2008/08/projectsettings-general.png)

	
  3. Switch over to the **Build** tab and scroll down to the **Code Signing** section.  Change both the **Code Signing Identity **and the entry under it to **Don't Code Sign**.
![projectsettings-build](http://fruitstandsoftware.com/blog/wp-content/uploads/2008/08/projectsettings-build.png)

	
  4. After making these changes you should be able to run scan-build on your iPhone project successfully.




### Customizing Your Output


After running scan-build a few times the first thing that you might want to do is tell scan-build to put its reports in a different directory. To do that, simply specify the output folder on the command line like so:
`scan-build -o /path/to/the/directory/where/you/want/your/report xcodebuild`

There are a few other flags that can be passed to scan-build, but for now the reports that are generated should be the same regardless of the flags you set. Check out the Other Options section on the [Static Analyzer usage page](http://clang.llvm.org/StaticAnalysisUsage.html) for the full (but still pretty short) list of available options.


### Wrap Up


One thing to note is that scan-build is still in pre-1.0 and has some rough edges. You may notice some false-positives or other undesirable behavior. As with any pre-release software use it at your own risk and always have a backup of your work. That being said, I have not had nor heard of any disastrous problems with it, so your risk is probably pretty low.

A lot of folks in the OS X development community have gotten a lot of use out of scan-build in the past few months. One of the larger scale uses of it can be found on the Adium project, you can view the results of their static analysis [here](http://trac.adiumx.com/wiki/StaticAnalysis).

Good luck!
