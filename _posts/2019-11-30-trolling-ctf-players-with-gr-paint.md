---
layout: post
title: Trolling CTF players with gr-paint
date: 2019-11-30 18:00:00 -0500
permalink: /:year/:month/:day/:title/
---
BSides Ottawa took place on November 28 & 29, and this year's CTF was the biggest ever! 200 players showed up, and organized themselves into 35 teams. I volunteered to create a radio track with 24 flags, as well as a crypto track with four flags. Thanks to a generous donation from [Xanthus Security](https://www.xanthus.io/), each team received an RTL-SDR dongle which could be used to solve the radio challenges.

My favourite radio challenge was called "Waterfall". When players tuned to 924.5 MHz, they saw the BSides Ottawa logo painted onto the waterfall, followed by the start of a flag:

![aoeu](/images/paint-begin.png)

At this point it seemed as though you just had to patiently wait for the rest of the flag to scroll by. But as time went on, the letters in the flag got smaller, and smaller, and smaller:

![aoeu](/images/paint-end.png)

By the end, the letters were much too small to read in Gqrx, even when the FFT rate was increased to the maximum of 60 fps!

Fortunately, there are other tools which are designed for offline signal analysis, and which allow the user to have a much closer look a short, bursty signals. My favourites are [Inspectrum](https://github.com/miek/inspectrum) and [Baudline](https://www.baudline.com/). After capturing the signal to a file with Gqrx's "Record and play I/Q data" button, it was easy to read off the tail end of the flag with Baudline:

![aoeu](/images/paint-baudline.png)

So how did I build this challenge? I needed two things: a way to distort an image so it would become thinner and thinner at one end, and a way to paint that image onto the radio spectrum.

To produce the distorted image, I used ImageMagick's [FX special effects image operator](https://imagemagick.org/script/fx.php), which allows the user to define an arbitrary mapping between input pixels and output pixels. After some experimentation, I found that transforming the *y* axis using an exponential function worked best:

```
convert \
  \( \
    -background White \
    -gravity Center \
    -pointsize 400 \
    label:flag\{look_closer_a4146a8247fa439d6879\} \
    -rotate 270 \
  \) \
  bsides-ottawa-logo.jpg \
  -append \
  -crop 50x100%-30+0 \
  -gravity North \
  -extent 100%x200% \
  -fx "xx = i; yy = ln(j/7978/2 * (exp(5)-1) + 1) / 5 * 7978; v.p{xx,yy}" \
  logo-flag.png
```

Painting the resulting image onto the spectrum was easy, thanks to [gr-paint](https://github.com/drmpeg/gr-paint), a GNU Radio module written by Ron Economos (a.k.a. [@drmpeg](https://twitter.com/drmpeg)). This module takes an image file as input, and uses an inverse FFT to map each row of pixels into a set of OFDM carriers with the corresponding amplitudes.

It was a joy to watch players copying down the start of the flag, only to realize they would have to work harder to get the rest. The first person to solve the problem was my friend and former colleague Serge Mister from Entrust Datacard's "Reverse Solidus" team. By the end of the competition, another 12 teams had solved it.
