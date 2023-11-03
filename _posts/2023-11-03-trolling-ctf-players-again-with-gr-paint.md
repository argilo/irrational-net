---
layout: post
title: Trolling CTF players (again) with gr-paint
date: 2023-11-03 15:42:00 -0400
permalink: /:year/:month/:day/:title/
---
Last week I headed back to Ottawa for the [BSides Ottawa](https://bsidesottawa.ca/) conference & CTF. As usual, I put together a series of radio challenges, and [Xanthus Security](https://www.xanthus.io/) generously provided each team with an [RTL-SDR dongle](https://www.nooelec.com/store/sdr/sdr-receivers/nesdr/nesdr-mini-2-plus.html) that they could use to receive the mysterious signals floating around the conference area.

CTF players were asked to imagine they were employees of "Hackers 4 Cash," a consulting firm hired by pickle farm "Ye Olde Pickles" to hack their competitor, "CyberPickle." Not surprisingly, CyberPickle relied heavily on radio to automate their operations.

The first radio challenge was called "Wide Load," and players were given the following challenge:

> To keep their communications private, CyberPickle is testing out a new wideband communication system on the 902-928 MHz ISM band. Is it possible to intercept the message?

When tuning to the low end of the band, around 905 MHz, the following signal appeared in the waterfall once every 45 seconds:

![Screenshot of Gqrx, showing "flag{this_is_" in the waterfall](/images/wide-load-rtl-1.png)

Clearly that's the start of a flag, but we need to tune higher to see more. After tuning to 907 MHz and patiently waiting, the next piece of the flag could be seen:

![Screenshot of Gqrx, showing "is_a_very_wi" in the waterfall](/images/wide-load-rtl-2.png)

After repeating this process another nine times or so, players would finally reach the end of the flag:

![Screenshot of Gqrx, showing "15a166ff9}" in the waterfall](/images/wide-load-rtl-3.png)

Many players asked me whether it's possible to "zoom out" and see more of the radio spectrum at once. Unfortunately, it's not possible to do that directly because the RTL-SDR is limited to a sample rate of approximately 2.4 million samples per second, which means that it's only possible to see a frequency span of 2.4 MHz at a time. (In practice, the usable bandwidth is a bit lower than that due to [aliasing](https://en.wikipedia.org/wiki/Aliasing); if you look closely at the screenshots above, you'll see that signals near the edges of the display begin fading out and then "wrap around" to the opposite edge due to the imperfect anti-aliasing filter in the receiver.)

More expensive receivers can operate at much higher sample rates. For instance, the USRP B200 can receive up to 56 MHz at once, allowing the entire 902-928 MHz ISM band to be recorded:

![Screenshot of Gqrx, showing "flag{this_is_a_very_wide_flag_indeed_c928359da12a4a00e32fa43063e5037256c571a0e75ff2019a6741215a166ff9}" in the waterfall](/images/wide-load-usrp.png)

To create this challenge, I used ImageMagick to create a large PNG file containing the flag, and then painted that onto the radio spectrum using [gr-paint](https://github.com/drmpeg/gr-paint). I've published the [source code for all my challenges](https://github.com/argilo/bso2023), and the pieces used to generate the "Wide Load" challenge can be found [here](https://github.com/argilo/bso2023/blob/847fbb99876cd832d7a4e1c52e742ecb3f6cf501/gen_all.sh#L21-L31) and [here](https://github.com/argilo/bso2023/blob/main/paint/paint_wide.grc).
