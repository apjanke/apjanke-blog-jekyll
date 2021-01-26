---
title: "My Weird FLOSS Hobby: Matlab"
date: 2021-01-18T14:50:00-05:00
author: apjanke
layout: post
categories:
  - Matlab
  - Computers
---

Matlab ranked #16, with a 1% rating, in the [January 2021 TIOBE Index](http://www.tiobe.com/tiobe-index?20210102). (Higher than I thought, actually. It's beating out Perl!)

[Matlab](https://www.mathworks.com/products/matlab.html) is a proprietary numeric programming language that started out life as an overgrown interactive graphing calculator, and has gradually grown towards a more general data-science and engineering oriented language, and is making efforts to be suitable for use in server-side automation and "production" workloads.

One of the reasons writing FLOSS Matlab code appeals to me is that, because Matlab lags mainstream languages in many ways, there's just _so much_ to be done on it, and that I can do! When I look at Python or Java or Javascript, it sometimes seems like everything has already been built for them, and I have a hard time coming up with something new to actually work on.

When you're a Matlab hacker, you're a big fish in a small pond. I'm one of the top worldwide users of MathWorks Tech Support, have a direct line to the head Product Manager for all of Matlab, correspond with senior MathWorks engineers, and am in the [top 20 users for the Stack Overflow "matlab" tag](https://stackoverflow.com/tags/matlab/topusers). Feels good!

Being a Matlab programmer and also a serious software developer also puts you in kind of a weird no man's land. When you tell other devs you work with Matlab, if they've even heard of the language, they'll generally shit on it and look down on you for using it. Meanwhile, your fellow Matlab users will sometimes resent you for trying to bring "pedantic", "software-engineeringy" tools and processes in to their freewheeling, business-oriented workflows.

There's also [GNU Octave](https://www.gnu.org/software/octave/index), a Free Software clone of Matlab. I like Octave, and have been a minor contributor to it, in the form of being co-maintainer of the [Octave.app](https://octave-app.org/) native Mac app distribution, and writing a [couple](https://github.com/apjanke/octave-tablicious) [Octave](https://github.com/apjanke/octave-testify) [packages](https://github.com/apjanke/octave-packajoozle). But Matlab has added some really nice language features in the last few years, and Octave has fallen far enough behind that I don't really enjoy writing Octave code any more.

I'm kinda pessimistic that Open Source for Matlab will ever really take off. There's a [$150 Matlab Home license](https://www.mathworks.com/products/matlab-home.html) now, but that's still a lot more than the $0 that most programming languages cost. Paying for a programming language is a hurdle I doubt most hobbyist programmers will jump. I don't mind: I pay for plenty of other stuff on my FLOSS development stack – my Mac (which is hardware and an OS), PyCharm, iTerm2 (through [gnachman's Patreon](https://www.patreon.com/gnachman)), Dropbox, SourceHut. But I also write Matlab professionally, so I'm already paying for, and making money off of, it. I suspect that FLOSS Matlab development will remain the domain of people who already have to use Matlab for work or school.

Maybe I haven't done a great job in selling it, but if you're interested in doing open source work that can help you land a stable, well-paying career, consider joining me and hacking on some Matlab!
