---
layout: post
title: Tracking planes for $20 or less
date: 2012-08-06 19:44:28 -0400
permalink: /:year/:month/:day/:title/
---
My new favourite toy is a cheap $20 USB TV tuner.  It's made to receive DVB-T signals, which aren't even used in North America.  So what use could I possibly have for it?

Back in February, Linux kernel developer Antti Palosaari [discovered](https://article.gmane.org/gmane.linux.drivers.video-input-infrastructure/44461) that certain USB TV tuners can be configured to send the raw, unprocessed radio signal straight to the computer for decoding in software.  (They use this mode when tuning FM or DAB radio signals.  Think of it as the [Winmodem](https://en.wikipedia.org/wiki/Softmodem) approach to radio.)  Palosaari realized that by running the right software, almost any radio signal could be received by these tuners.  Not long thereafter, the [RTL-SDR](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr) project was born, allowing these tuners to be used in Linux.

I should note that receiving (and transmitting) radio signals in software is nothing new.  [Software-defined radio](https://en.wikipedia.org/wiki/Software-defined_radio) has been around for years, but the hardware required (such as the [Ettus Research USRP](https://www.ettus.com/) has generally been expensive.  The availability of a $20 software-defined radio receiver has truly opened up the world of radio to anyone who takes the time to learn.

Since getting my hands on a compatible TV tuner, I've been able to listen to police radio, pager networks, garage door openers, air traffic control, and lots more.  I recently tweeted that I had succeeded in tracking the aircraft in my area by using my TV tuner as an [ADS-B](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance-broadcast) receiver and feeding the output into Google Earth.  This caught the interest of a pilot friend of mine, so I thought I'd put together a tutorial for anyone interested in capturing these signals.  Although the tutorial is specific to ADS-B, keep in mind that the software tools (and in particular [GNU Radio](https://wiki.gnuradio.org/index.php/Main_Page)) can be reconfigured to tune in virtually any radio signal.

So let's get started!

1. Purchase a USB TV tuner based on the Realtek RTL2832U chip.  For best results, choose one that uses the Elonics E4000 tuner, which will let you tune in the widest range of frequencies, from 64 to 1700 MHz.  The OsmoSDR site has a [list of supported hardware](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr#Supported-Hardware) to get you started.  I chose the Newsky TV28T tuner, which I purchased from Aliexpress.  (I paid 40 USD for two tuners, shipping included.)
1. If you're not running it already, download and install [Ubuntu Desktop 12.04 LTS](https://www.ubuntu.com/download/desktop).  I would recommend using the 64-bit version.
1. Install all the available Ubuntu software updates using Update Manager and restart.
1. Download, build and install GNU Radio [using the build-gnuradio script](https://wiki.gnuradio.org/index.php/InstallingGRFromSource#Using_the_build-gnuradio_script).  This can be done in a terminal window by running the following commands:
```
cd ~
mkdir build-gnuradio
cd build-gnuradio/
wget http://www.sbrac.org/files/build-gnuradio
chmod a+x ./build-gnuradio
./build-gnuradio
```
Note that GNU Radio is quite a large piece of software and has a lot of dependencies, so the install process can take a long time.
1. Download, build and install [gr-air-modes](https://github.com/bistromath/gr-air-modes).  This is the piece of software that knows how to decode the ADS-B signals that many planes transmit.  In a terminal window, run the following commands:
```
cd ~
git clone https://github.com/bistromath/gr-air-modes.git
cd gr-air-modes
cmake .
make
sudo make install
sudo ldconfig
```
1. Plug in your TV tuner, and check whether you can receive ADS-B traffic by running the following in a terminal window:
```
uhd_modes.py --rtlsdr
```
If it works, you should see output like the following:
```
(-42 0.0000000000) Type 11 (all call reply) from c0636c in reply to interrogator 0 with capability level 6
(-41 0.0000000000) Type 17 BDS0,5 (position report) from c078b2 at (45.199942, -75.541590) at 30050ft
(-39 0.0000000000) Type 11 (all call reply) from c078b2 in reply to interrogator 0 with capability level 6
(-39 0.0000000000) Type 17 BDS0,9-1 (track report) from c078b2 with velocity 443kt heading 259 VS 1664
(-40 0.0000000000) Type 17 BDS0,5 (position report) from c078b2 at (45.199616, -75.544069) at 30075ft
(-42 0.0000000000) Type 17 BDS0,5 (position report) from c078b2 at (45.199265, -75.546504) at 30100ft
```
We're already seeing some GPS coordinates and altitudes!  Press CTRL-C to stop it for now.
If you don't see any traffic, try going outside for better reception.
1. To see the output in a more convenient form, we'll use [Google Earth](https://www.google.com/earth/).  Download the 64-bit .deb version from the [download page](https://www.google.com/earth/download/gep/agree.html), and open the file to run the installer.
1. For nicer-looking fonts in Google Earth, install the `xfonts-75dpi` and `xfonts-100dpi` packages by running the following in a terminal window:
```
sudo apt-get install xfonts-75dpi xfonts-100dpi
```
Then log out and log back in so the new fonts will get loaded.
1. Launch Google Earth.
1. If Google Earth fails to launch, it may be because it can't find libGL.  (This happened on one of my two laptops.)  To fix it, run the following command in a terminal window:
```
sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1.2 /usr/lib/libGL.so.1
```
1. Run `uhd_modes.py` again, this time telling it to write its output to a KML file, the format used by Google Earth.  In a terminal window, run the following:
```
uhd_modes.py --rtlsdr --kml=planes.kml
```
1. In Google Earth, select "Network Link" from the "Add" menu.  Enter "Planes" in the "Name" field, then click the Browse button next to the "Link" field and choose the "planes.kml" file in the file chooser.  Click on the "Refresh" tab and set a time-based refresh to occur periodically with a frequency of 5 seconds.  Click "OK", then zoom in to your location.  With any luck, you should see some planes start to appear and move around!
1. To see more details about a plane, click the "X" that appears on the map.  Or go to the "Places" section in the left sidebar and expand "Planes" and "Aircraft locations".
Here's a screenshot of Air Canada flight 839 coming in for a landing at YOW, with several more planes at cruising altitude in the background:
![Screenshot of gr-air-modes in Google Earth](/images/adsb-google-earth.png)

I hope you find this tutorial useful, and that you'll do more exploring with software-defined radio once you've succeeded in watching planes!
