---
layout: post
title: Inverting the Raspberry Pi Debug Probe's UART pins
date: 2023-05-26 00:01:12 -0400
permalink: /:year/:month/:day/:title/
---
Last week I attended the [NorthSec](https://nsec.io/) conference & CTF in Montreal. It was a great event, and I had a lot of fun! This year's conference badge included a neat game where attendees could plug their badges together to earn points and unlock additional LED blinking patterns:

![NorthSec 2023 electronic badges](/images/northsec-2023-badge.jpg)
*[Photo](https://lightroom.adobe.com/shares/2cf9b72f6f514b85b9ecf0df02bbd985/albums/fd4d11962be74f7eaab836fafe3352c0/assets/e3f41ef1b8e64c4aa3372c474471af63) by [Simon Carpentier](https://spacebar.ca/) is licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)*

I wanted to reverse engineer the badge-to-badge communication protocol, and soon found out that it was asynchronous serial at 38400 baud, but with inverted polarity. While my Saleae logic analyzer happily decoded the inverted signal, my [Raspberry Pi Debug Probe](https://www.raspberrypi.com/products/debug-probe/) could not. I worked around the problem by building a couple inverters (one for transmit, one for receive) on a breadboard, and putting them between the badge and the Debug Probe. This worked, but it was cumbersome enough that I wanted to find a better solution.

After searching the web and digging through datasheets, I learned that the RP2040 chip's GPIO pins can be inverted by setting GPIO control registers (specifically, the `INOVER` and `OUTOVER` fields in the `GPIO*_CTRL` registers). The [`gpio_set_outover`](https://www.raspberrypi.com/documentation/pico-sdk/hardware.html#rpip822202dd157864f2ab6e) and [`gpio_set_inover`](https://www.raspberrypi.com/documentation/pico-sdk/hardware.html#rpip7b7f80d4ec51606be1f2) functions in the Raspberry Pi Pico SDK provide a convenient way to set these registers. It only took [a couple lines of code](https://github.com/argilo/picoprobe/commit/50affa6517e53f5299deb69cce0715d4c784c45c) to invert the UART pins in the Debug Probe firmware.

In case you'd like to try this out on your own Debug Probe, I've posted the [compiled firmware](https://github.com/argilo/picoprobe/releases/tag/invert-uart-v1) on GitHub.
