---
layout: post
title: Building challenges for the GRCon22 CTF
date: 2022-10-04 20:53:00 -0400
permalink: /:year/:month/:day/:title/
---
Since 2016, the GNU Radio Conference has held a [Capture the Flag (CTF)](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) competition in parallel with its technical track. Secret messages ("flags") are hidden in radio signals, which players have to find and then submit to a scoring system to earn points. The CTF is my favourite part of GRCon, so when the conference organizers asked whether I'd be willing to organize this year's competition, I jumped at the opportunity. Luckily, I didn't have to do it alone: Muad'Dib, Aerospace Corporation, Daniel Estévez, and Yamakaja all stepped up to contribute challenges of their own.

The event was a phenomenal success. There were 52 challenges, and 71 teams submitted 686 valid flags. Even though the competition period has ended, most of the challenges are still up and playable at [https://ctf-2022.gnuradio.org/](https://ctf-2022.gnuradio.org/).

In this post, I'll describe how I created my challenges, and how I anticipated they might be solved. If you don't want any spoilers, stop reading here!

Signal identification
---------------------

![Screenshot of Gqrx receiving signals from the "Signal identification" challenge](/images/signal-identification.png)

This challenge track consisted of 13 flags embedded in 11 signals, all contained in a single [SigMF](https://github.com/gnuradio/SigMF) recording. I expected that players might use the [Signal Identification Wiki](https://www.sigidwiki.com/wiki/Signal_Identification_Guide) to help identify the signals and find decoding software.

From left to right, the signals were:

1. FM-modulated [Slow-Scan Television (SSTV)](https://www.sigidwiki.com/wiki/Slow-Scan_Television_(SSTV)). The flag appeared in a QR code within the image.
1. [Narrow-band FM voice](https://www.sigidwiki.com/wiki/NFM_Voice).
1. [Automatic Packet Reporting System (APRS)](https://www.sigidwiki.com/wiki/APRS).
1. [M17 digital voice](https://m17project.org/).
1. [Wide-band (broadcast) FM](https://www.sigidwiki.com/wiki/FM_Broadcast_Radio). Flags were in the mono audio (L+R) channel, stereo audio (L-R) channel, and [Radio Data System (RDS)](https://www.sigidwiki.com/wiki/Radio_Data_System_(RDS)) subcarrier.
1. [POCSAG](https://www.sigidwiki.com/wiki/POCSAG) 1200 bps.
1. [PSK 31](https://www.sigidwiki.com/wiki/Phase_Shift_Keying_(PSK)).
1. [Upper sideband voice](https://www.sigidwiki.com/wiki/Single_Sideband_Voice).
1. [AM voice](https://www.sigidwiki.com/wiki/Amplitude_Modulation_(AM)).
1. [Morse Code (CW)](https://www.sigidwiki.com/wiki/Morse_Code_(CW)).
1. [Lower sideband voice](https://www.sigidwiki.com/wiki/Single_Sideband_Voice).

This challenge has its origins in a tutorial session I created for the [Ottawa Amateur Radio Club](https://oarc.net/) in 2014. I later published the [source code](https://github.com/argilo/sdr-examples#multi_txgrc--multi_txpy), and in 2019 I adapted it for use as a CTF challenge at [BSides Ottawa](https://bsidesottawa.ca/). For GRCon22, I added [M17](https://m17project.org/), a promising open-source alterative to proprietary digital voice protocols.

Most of the signals can be received directly within [Gqrx](https://github.com/gqrx-sdr/gqrx), but a few require additional software. To test the challenges, I used the following:

* SSTV → qsstv
* M17 → [m17-cxx-demod](https://github.com/mobilinkd/m17-cxx-demod)
* POCSAG → multimon-ng
* PSK 31 → fldigi

If you solved the SSTV challenge, you can also receive the [SSTV images](http://ariss-sstv.blogspot.com/) that the International Space Station occasionally broadcasts on 145.800 MHz!

Fox hunting
-----------

This challenge was inspired by [amateur radio direction finding](https://en.wikipedia.org/wiki/Amateur_radio_direction_finding) (also known as radio fox hunting) where competitors run through the woods searching for hidden radio transmitters.

I built two hidden transmitters ("foxes") which transmitted Morse code. One ran at approximately 433 MHz, and was hidden at a fixed location in the conference area. The other transmitted at approximately 904 MHz, and was carried around by various conference organizers in their backpacks, making it a moving target. Players had to determine the exact frequency of each transmitter, copy the flag which was included in the Morse code message, and then physically locate the transmitter to read another flag which was printed on it. Here's a peek inside the hidden transmitters:

![Two hidden transmitters sitting on a table with covers removed](/images/grcon22-foxes.jpg)

I built them with Adafruit Feather M0 RFM69HCW Packet Radios, which are available in [433 MHz](https://www.adafruit.com/product/3177) and [915 MHz](https://www.adafruit.com/product/3176) versions. These boards are normally used to transmit and receive [FSK](https://en.wikipedia.org/wiki/Frequency-shift_keying) signals, but they also have an [on-off keying](https://en.wikipedia.org/wiki/On%E2%80%93off_keying) mode where the transmitter can be switched on and off using a GPIO pin. That makes it easy to tap out a message in Morse code. Each transmitter is powered by a 2000 mAh lithium cell, which is enough to provide four days of power.

Gqrx does a fine job of receiving the Morse code, and it's even possible to read it by eye from the waterfall:

![Gqrx receiving morse code from the 433 MHz hidden transmitter](/images/grcon22-fox-1.png)

Although directional antennas can be helpful to find hidden transmitters, most players got by with omnidirectional antennas. I recommend putting on a pair of headphones and disabling both hardware and software [AGC](https://en.wikipedia.org/wiki/Automatic_gain_control), thus making the audio amplitude directly proportional to the incoming RF signal power. Walking closer to the transmitter will then make the signal louder, and walking away will make it quieter. Once you're close enough to the signal that the receiver is saturated (and audio begins clipping), reduce the RF input gain and keep searching for a stronger signal. When you get very close and your RF gain is already at a minimum, it may be necessary to switch to a worse antenna or remove the antenna completely. Beware that some things other than distance from the transmitter also affect received signal power. For instance, obstacles may absorb or reflect radio signals.

Signal to noise
---------------

![The start of the flag for "Signal to noise", as viewed in inspectrum](/images/signal-to-noise-1.png)

In this challenge, the start of a spectrum-painted flag is apparent when the signal is viewed in [inspectrum](https://github.com/miek/inspectrum), but as time progresses the signal narrows in frequency, widens in time, and is engulfed by an ever-increasing amount of noise. By playing with the "FFT size", "Power max", and "Power min" settings it is possible to read off the first half of the flag: `flag{a4146a8247`. But the remainder is impossibly difficult to read.

Fortunately, Gqrx allows larger FFT sizes and finer control over the FFT rate, which is just barely enough to read off the rest of the letters:

![The middle of the flag for "Signal to noise", as viewed in Gqrx](/images/signal-to-noise-2.png)
![The end of the flag for "Signal to noise", as viewed in Gqrx](/images/signal-to-noise-3.png)

By stitching these parts together, we get the complete flag: `flag{a4146a8247fa439d6879}`.

To make this challenge, I used [gr-paint](https://github.com/drmpeg/gr-paint) to generate a rectangular spectrum-painted image, then passed this through a Polyphase Arbitrary Resampler block with progressively increasing resampling rate, and added noise from a Gaussian noise source with progressively increasing amplitude. A [Python script](https://github.com/argilo/grcon22/blob/f319bafc59dd6efaa8842ee90b9759026078ca9c/signal_to_noise/paint_tx_tri.py#L135-L146) dynamically adjusts the parameters of the resampler and noise source after each line from the rectangular image is processed.

Never the same color
--------------------

The name of this challenge hints at [NTSC](https://en.wikipedia.org/wiki/NTSC), an analog television standard. Engineers jokingly referred to it as "Never The Same Color" because its colour accuracy was sometimes poor.

The challenge consisted of an NTSC signal containing seven flags. One was in the video, four were in the audio (mono, stereo, [second audio program](https://en.wikipedia.org/wiki/Second_audio_program), and [PRO subcarrier](https://en.wikipedia.org/wiki/Second_audio_program#Frequencies)), and two were in [EIA-608 closed captions](https://en.wikipedia.org/wiki/EIA-608) (channels CC1 and CC3). One twist was that the video flag only appeared once every 176 frames. The other frames contained a decoy. (If you're curious, scan the QR code in the image below to see what it was!)

To produce the signal, I made some improvements to the NTSC signal generator from my [SDR examples](https://github.com/argilo/sdr-examples) repository, and tested with a television set. In fact, every flag except for the PRO (professional) subcarrier can be received by an ordinary television set:

![A television set receiving an NTSC signal and displaying a flag](/images/grcon22-ntsc-flags.jpg)

Many players found the flags this way, piping the signal into a nearby television. (Even the television sets in guest rooms at the conference hotel could be coaxed into receiving NTSC.) But I was very impressed by Daniel Estévez, who [built his own NTSC demodulator in a Jupyter notebook](https://destevez.net/2022/10/grcon22-capture-the-flag/)!

Shall we play a game?
---------------------

This was my favourite challenge, and I had a lot of fun building it.

The challenge description asked the player to transmit an APRS packet at 903.5 MHz, with their team name as the source address and "new" in the comment field. A response would then arrive somewhere in the 900 MHz ISM band. In fact, it arrived at 926 MHz in the form of a spectrum-painted rules page for a variation of Wordle called "SDRdle":

![A Gqrx waterfall showing spectrum-painted text: "Guess the SDRdle in 6 tries. Each guess must be a valid 5-letter word. Transmit an APPRS packet to submit. After each guess, the color of the tiles will change to show how close your guess was to the word."](/images/sdrdle-1.png)

All the player had to do at this point was follow the instructions and transmit further APRS packets containing their guesses:

![A Gqrx waterfall showing a spectrum-painted Wordle board, with "ATONE" as the guessed word](/images/sdrdle-2.png)
![A Gqrx waterfall showing a spectrum-painted Wordle board, with "ATONE" and "FLAIR" as the guessed words](/images/sdrdle-3.png)
![A Gqrx waterfall showing a spectrum-painted Wordle board, with "ATONE", "FLAIR", and "CAULK" as the guessed words. A message instructs the player to contact @argilo to get their flag.](/images/sdrdle-4.png)

To test the challenge, I used [Dire Wolf](https://github.com/wb2osz/direwolf) to generate APRS packets and place them in WAV files:

```
echo "ARGILO>WORLD:>new" | gen_packets -r 48000 -o aprs.wav -
```

I then made a simple flow graph to FM-modulate them and transmit them on a HackRF.

The challenge server received APRS packets using [rtl_fm](http://kmkeen.com/rtl-demod-guide/) piped into Dire Wolf, generated game board images using the Python Imaging Library, and transmitted them using gr-paint.

Conclusion
----------

Source code for all of the above challenges can be found in my [grcon22 GitHub repository](https://github.com/argilo/grcon22). I've released the code under the GPL so that they can be adapted and used for other purposes. If you use them for an event of your own, I'd love to hear about it!

I hope players had as much fun solving the challenges as I did creating them.
