---
id: 85
title: Developing Mac Homebrew with RubyMine
date: 2018-02-10T14:30:29+00:00
author: apjanke
layout: post
categories:
  - MacHomebrew
---
I&#8217;ve been working with [Mac Homebrew](https://brew.sh/) lately, and I prefer to work in an IDE for Ruby projects. You can set up JetBrains&#8217; [RubyMine](https://www.jetbrains.com/ruby/) to work with Homebrew, but it can be a little tricky. Here, I&#8217;ll explain how to do it in case anyone else is interested in working that way.

These instructions are written against RubyMine 2017.3.2 and Homebrew 1.5.3, but should work for other versions as well.

First, of course, install Homebrew and RubyMine.

Then, open RubyMine and create a new Empty project. You can call it whatever you want and store pretty much anywhere. I call mine `Homebrew-local` and save it in the default `~/RubymineProjects` directory.

![](/images/85707-rubyminenewprojectscreen.png)

Then you need to get Homebrew&#8217;s code on to your project load path. Type `⌘-,` to open Preferences, and select Project Structure from the list on the left. Click &#8220;+ Add Content Root&#8221; on the right side, and select `/usr/local/Homebrew/Library/Homebrew` in the file selection dialog that comes up. `/usr/local` is not visible in the file chooser by default, so you&#8217;ll need to hit `⌘-⇧-G` and type in `/usr/local/Homebrew/Library/Homebrew` to get there. Click on the newly added content root in the list on the right, and you should see a whole tree of files appear.

![](/images/903fc-rubymineprojectstructureforhomebrewproject.png)

Then click on the &#8220;Load Path&#8221; tab at the top, click the &#8220;+&#8221; button at the bottom of the (empty) list, and again select `/usr/local/Homebrew/Library/Homebrew` in the file selector. That should take care of the paths.

![](/images/1a0c5-loadpaths.png)

Next, you need to switch the project to use the &#8220;Portable Ruby&#8221; that Homebrew is running under, instead of the system Ruby. Click the triangle next to &#8220;Languages & Frameworks&#8221; in the list on the left, and click &#8220;Ruby SDK and Gems&#8221;. In the list in the middle, click the &#8220;+&#8221; button and choose &#8220;New local&#8230;&#8221;. Point it at `/usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/<version>/bin/ruby` (again, use `⌘-⇧-G` if needed) and select Open. The version will be something like &#8220;2.3.3&#8221;, as of this writing. Then click the radio button next to the newly-added &#8220;ruby-2.3.X-pXXX&#8221; in the list. It should look something like this.

![](/images/7b7cd-rubysdkselection.png)

_Note:_ You will need to repeat this Ruby SDK selection process each time Homebrew updates their Portable Ruby version!

That&#8217;s it. RubyMine will now be able to follow around all the references in the core Homebrew codebase.

(I also like to turn off spell checking. It&#8217;s in Preferences > Editor > Spelling, click &#8220;Configure &#8216;Spelling&#8217; Inspection&#8221;, uncheck &#8220;Typo&#8221;.)

Happy hacking!
