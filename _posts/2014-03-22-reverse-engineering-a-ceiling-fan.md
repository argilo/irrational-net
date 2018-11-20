---
layout: post
title: Reverse engineering a ceiling fan
date: 2014-03-22 23:31:32 -0400
permalink: /:year/:month/:day/:title/
---
Tonight I was visiting a friend of mine, and noticed a strange looking switch on the wall. My friend explained that it was a wireless controller for his ceiling fan.  Since we're both radio geeks, and I happened to have my [BladeRF](https://www.nuand.com/) with me, I got the idea to reverse engineer it.

The first step was to figure out what frequency the controller was transmitting on.  The BladeRF makes that a fairly easy task, since it has a bandwidth of 28 MHz.  I fired up [gqrx](http://gqrx.dk/) to get a nice waterfall view of all that bandwidth.  My first guess was that the signal might be on the 902-928 MHz band, and sure enough, I spotted a signal popping up at 911.24 MHz whenever I pressed a button on the controller.  But it was quite weak, which led me to suspect it might be a harmonic.  Indeed, when I tuned lower I found a very strong signal at 303.747 MHz, and I could easily detect it from across the room.

The next step was to check what modulation scheme the controller used. Most simple devices like this are using either [on-off keying](https://en.wikipedia.org/wiki/On-off_keying) or [frequency-shift keying](https://en.wikipedia.org/wiki/Frequency-shift_keying).  Zooming in on the signal in gqrx, I saw only a single peak, which suggested on-off keying.

I knew my trusty RTL-SDR dongle would be more than capable of receiving and demodulating the signal, so I threw together a very simple GNU Radio flow graph to show me the amplitude of the 303.747 MHz signal over time:

![ceiling-fan-rx-flowgraph](/images/ceiling-fan-rx-flowgraph.png)

Here's what I saw on the scope, once I set it to trigger on a rising edge and pressed the "light" button on the ceiling fan controller:

![ceiling-fan-ask](/images/ceiling-fan-ask.png)

The transmission was short enough that I could just read the bits off visually: 1011011001011001001001001001001001011. And by measuring the time from the start to the end of those bits, I worked out that the symbol rate was about 3211 baud.

In fact, all the buttons generated very similar 37-bit patterns:

```
off:   1011011001011001001001001001001011001
low:   1011011001011001001001001011001001001
med:   1011011001011001001001011001001001001
high:  1011011001011001001011001001001001001
light: 1011011001011001001001001001001001011
```

The bits were repeated for as long as a button was held, with about another 37 bits worth of zeroes between each repetition.

Given this information, it was trivial to build a flow graph to transmit an on-off keying signal using the BladeRF:

![ceiling-fan-tx-flowgraph](/images/ceiling-fan-tx-flowgraph.png)

My first attempt was unsuccessful, but it turned out the problem was just that the output gain wasn't set high enough.  Bringing it up to about 15 dB was sufficient to reliably control the ceiling fan!

The whole reverse engineering project took only about a half an hour, which really demonstrates the power of software-defined radio.

I've already added the receiver and transmitter to my [sdr-examples](https://github.com/argilo/sdr-examples) repository on Github:

Receiver: [ceiling_fan_rx.grc](https://github.com/argilo/sdr-examples/blob/master/ceiling_fan_rx.grc)  
Transmitter: [ceiling_fan_tx.grc](https://github.com/argilo/sdr-examples/blob/master/ceiling_fan_tx.grc)

**Update:** Looking at the bit patterns above, it is apparent that the bits come in groups of three: either 001 or 011.  Presumably, 001 represents a baseband 0, and 011 represents a baseband 1.  That is, a narrow pulse represents a zero and a wide pulse represents a one.  That would make the baseband bit patterns as follows:

```
off:   0110100000010
low:   0110100001000
med:   0110100010000
high:  0110100100000
light: 0110100000001
```
