---
layout: post
title: Using the Raspberry Pi's serial port
date: 2012-04-19 08:38:18 -0400
permalink: /:year/:month/:day/:title/
---
The stock Debian image for the Raspberry Pi uses the UART as a serial console.  I was able to connect to it from my Ubuntu laptop via my [3.3-volt USB FTDI TTL-232 cable](https://www.adafruit.com/products/70).  I connected Raspberry Pi's ground pin to the ground pin of the FTDI, the Rasberry Pi's TX pin to the FTDI's RX pin and vice versa.  (The Raspberry Pi's pinout is available [here](https://elinux.org/RPi_Low-level_peripherals).)  Then on my Ubuntu laptop I installed minicom (`sudo apt-get install minicom`) and fired it up with:

```
minicom -b 115200 -o -D /dev/ttyUSB0
```

After typing in a username, I got a password prompt and was able to log in.  Also, the serial console allowed me to see all the kernel output during boot, which could be handy someday.

But I wanted to use the Raspberry Pi's UART for my own purposes, not as a serial console.  To achieve that, I did the following.

First, I made a backup of the /boot/cmdline.txt file, which contains the kernel parameters:

```
sudo cp /boot/cmdline.txt /boot/cmdline_backup.txt
```

Then I edited it:

```
sudo vi /boot/cmdline.txt
```

Originally it contained:

```
dwc_otg.lpm_enable=0 rpitestmode=1 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 rootwait
```

I deleted the two parameters involving the serial port (`ttyAMA0`) to get the following:

```
dwc_otg.lpm_enable=0 rpitestmode=1 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 rootwait
```

I rebooted (`sudo reboot`) to confirm that kernel output was no longer going to the serial port.  But the serial console was still available.  So I edited /etc/inittab:

```
sudo vi /etc/inittab
```

I commented out the following line:

```
2:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100
```

Finally, I rebooted again and confirmed that nothing was touching the serial port anymore.  Then, to test it out I installed minicom on the Raspberry Pi:

```
sudo apt-get install minicom
```

And ran it:

```
minicom -b 115200 -o -D /dev/ttyAMA0
```

After firing up minicom on my Ubuntu laptop again, I was able to send data in both directions!

Now to get the Raspberry Pi talking to an Arduino...
