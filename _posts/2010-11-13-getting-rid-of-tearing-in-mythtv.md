---
layout: post
title: Getting rid of tearing in MythTV
date: 2010-11-13 12:46:07 -0500
permalink: /:year/:month/:day/:title/
---
I just upgraded to MythTV 0.24, and so far it's looking great!  Closed captions look much nicer and the revamped on-screen display looks very sharp.  But one problem I'd seen occasionally in 0.23 seemed to be worse: tearing in video playback.  I used to see tearing only occasionally in HD playback, but after the upgrade I started to see it more frequently, and in SD playback as well.  After a bit of quick googling, I found a [solution](https://www.mythtv.org/wiki/VDPAU#Tearing.2FStuttering) in the MythTV wiki.  I added the following to my frontend's /etc/X11/xorg.conf file:

```
Section "Extensions"
    Option "Composite" "Disable"
EndSection
```

After a quick restart, the tearing was completely gone.  Yay!
