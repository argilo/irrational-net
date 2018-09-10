---
layout: post
title: Compiz on my Inspiron 5100
date: 2007-11-17 04:02:24 -0500
permalink: /:year/:month/:day/:title/
---
My laptop is an Inspiron 5100, and for the longest time I couldn't figure out how to get "Desktop Effects" (compiz) going in Ubuntu.  I didn't think my video card (Radeon Mobility 7500) was compatible, but it turns out it is.  I just needed to change a couple settings.

First, I put the following in `/etc/drirc`:

```
<option name="allow_large_textures" value="2" />
```

Then, in `/etc/X11/xorg.conf` I changed my `DefaultDepth` from `24` to `16`.  Finally, I restarted X with Ctrl-Alt-Backspace, and enabled compiz in System → Preferences → Appearance → Visual Effects.  Voila!

I found these instructions [here](https://lists.ubuntu.com/archives/ubuntu-users/2007-October/124611.html).
