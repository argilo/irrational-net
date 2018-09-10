---
layout: post
title: Making the Raspberry Pi a little less British
date: 2012-04-18 19:24:48 -0400
permalink: /:year/:month/:day/:title/
---
![Raspberry Pi](/wp-content/uploads/2012/04/wpid-IMG_20120418_192251.jpg)

After a month and a half of obsessively checking for status updates, I finally received my [Raspberry Pi](https://www.raspberrypi.org/) yesterday.  I love it!

The first thing I noticed after booting up the stock Debian image was that things are set up for British users.  Most annoyingly, the symbols on the keyboard weren't where I expected them to be.  To arrive at a more Canadian configuration, I did the following.

First, I changed the system locale, turning off `en_GB.UTF-8` and turning on `en_CA.UTF-8`:

```
sudo dpkg-reconfigure locales
```

Next, I changed the keyboard layout:

```
sudo dpkg-reconfigure keyboard-configuration
```

Next, I changed the time zone:

```
sudo dpkg-reconfigure tzdata
```

And finally, I changed to a Canadian Debian mirror by editing the "sources.list" file:

```
sudo vi /etc/apt/sources.list
```

On the first line, I changed "ftp.uk.debian.org" to "ftp.ca.debian.org":

```
deb http://ftp.ca.debian.org/debian/ squeeze main

# Nokia Qt5 development
deb http://archive.qmh-project.org/rpi/debian/ unstable main
```

After rebooting (`sudo reboot`) and updating my package list (`sudo apt-get update`) I had a pleasantly Canadian Pi.  :-)
