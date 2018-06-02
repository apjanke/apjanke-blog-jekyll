---
id: 278
title: Go away, mdbstatus!
date: 2018-02-24T01:16:18+00:00
author: apjanke
layout: post
guid: https://apjanketestblog.wordpress.com/?p=278
permalink: /index.php/2018/02/24/go-away-mdbstatus/
timeline_notification:
  - "1519434983"
categories:
  - Matlab
---
Matlab&#8217;s self-hosting nature can sometimes be an inconvenience. Case in point: if you have `dbstop if all error` enabled, and you save a file in the editor, Matlab will usually pull you up short at a breakpoint in `mdbstatus`.

If you find your Matlab editor randomly pulling you up in `mdbstatus` for no good reason, this might be what&#8217;s happening.

This happens because `mdsbstatus` is called automatically by Matlab, and in `mdbstatus`, in its local function `localGetFileBreakpoints`, it uses a `try`/`catch` for detecting the debugger status of a given file. And its `catch` regularly gets hit during normal usage. Unfortunate design.

Luckily, `mdbstatus` is implemented in M-code, and not as a builtin, so we can hack around it. Here&#8217;s a hack to fix it:

Locate Matlab&#8217;s `mdbstatus.m` file using `which mdbstatus`. Copy that to a user-local file in your Matlab documents directory. (This is at `~/Documents/MATLAB` on macOS and &#8220;My Documents/MATLAB&#8221; on Windows (`%USERPROFILE/Documents/MATLAB` by default on recent Windows versions.))

In your new user-local `mdbstatus.m`, add this code at the beginning of of the `localGetFileBreakpoints` function:

<pre>% apjanke's custom modification
% Avoid triggering "dbstop if all error" breakpoints here. Otherwise you'll
% pick up a breakpoint most times you do a file save!
origDebuggerState = dbstatus(<span class="s1">'-completenames'</span>);
RAII.dbstop = onCleanup(@() dbstop(origDebuggerState));
dbclear <span class="s1">if</span> <span class="s1">all</span> <span class="s1">error
</span>% end apjanke's custom modification</pre>

That will locally disable `dbstop if all error` while `mdbstatus` is doing its `try/catch` work.

There&#8217;s nothing special about the `RAII` variable. It&#8217;s just a local `struct` to hold cleanup functions that will be called when the stack unwinds; <a href="https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization" target="_blank" rel="noopener">RAII</a> is my conventional variable name for that. (Though it&#8217;s probably really &#8220;SBRM&#8221;.)

Since you&#8217;re hacking an internal Matlab function, I&#8217;d recommend making a prominent note of it. Stick this big old comment banner at the top of your new `mdbstatus.m` so it&#8217;s apparent what&#8217;s going on.

<pre class="p2">%<span class="Apple-converted-space">  </span>=========================================================
%<span class="Apple-converted-space">    </span>NOTE: This is apjanke's custom modification of mdbstatus.m, not the
%<span class="Apple-converted-space">    </span>original Matlab file. It has been modified to avoid triggering the
%<span class="Apple-converted-space">    </span>"dbstop if all error" breakpoint when files are saved in the editor.
%<span class="Apple-converted-space">  </span>=========================================================</pre>

(And use your own name if you want, of course.)

This was tested on Matlab R2016b and R2017b. YMMV on other versions. I&#8217;ve reported this behavior as a but to MathWorks, but who knows if and when they&#8217;ll get around to fixing it.