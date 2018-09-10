---
layout: post
title: Moving from Tomboy to Gnote in Ubuntu 12.04
date: 2012-04-27 21:50:04 -0400
permalink: /:year/:month/:day/:title/
---
![Gnote in the tray](/wp-content/uploads/2012/04/gnote-tray.png)

Yesterday I updated from Ubuntu 10.04 to 12.04.  As usual, I did a completely fresh install, only copying over the things I still need.  Tomboy is at the top of that list, so I copied over my `~/.local/share/tomboy/` directory.  I read that Gnote is now the preferred replacement for Tomboy, so I installed it and was pleased to see that it read in my Tomboy notes automatically.  But it didn't put an icon in my tray, and that was my favourite Tomboy feature.  To get the tray icon back, I did the following:

Fisrt, I added gnote to the systray whitelist:

```
gsettings set com.canonical.Unity.Panel systray-whitelist "['JavaEmbeddedFrame', 'Wine', 'Update-notifier', 'gnote']"
```

Next, I set Gnote to auto-start:

```
mkdir ~/.config/autostart
cp /usr/share/applications/gnote.desktop ~/.config/autostart/
chmod u+x ~/.config/autostart/gnote.desktop
```

Finally, I turned on the tray icon in Gnote itself: Edit → Preferences → Use Status Icon.  After logging out and back in, Gnote started up in the tray just the way Tomboy used to.
