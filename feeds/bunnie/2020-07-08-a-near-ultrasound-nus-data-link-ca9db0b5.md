---
title: A Near-Ultrasound (NUS) Data Link
url: https://www.bunniestudios.com/blog/2020/a-near-ultrasound-nus-data-link/
published: "2020-07-08T10:54:47Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5847
---

# A Near-Ultrasound (NUS) Data Link

We were requested to investigate “near ultrasound” (NUS) links as part of our research on developing the [Simmel reference design](https://simmel.betrusted.io/) for a privacy-preserving COVID-19 contact tracing device. After a month of poking at it, the TL;DR is that, as suspected, the physics of NUS is not conducive to reliable contact tracing. While BLE has the problem that you have too many false positive contacts, NUS has the problem of too many false negatives: pockets, purses, and your own body can effectively block the signal.

That being said, we did develop a pretty decent-performing NUS data link, so we’ve packed up what we did into an open source [reference design that you can clone and use](https://github.com/simmel-project/nus-link) in your own projects.

[![](https://raw.githubusercontent.com/simmel-project/nus-link/master/demodulator/examples/simmel_demod_data_1meter_with_50dB_background_music.png)](https://raw.githubusercontent.com/simmel-project/nus-link/master/demodulator/examples/simmel_demod_data_1meter_with_50dB_background_music.png)

*Top trace: demodulated data at 1 meter, 50dB background noise. Bottom trace: raw signal, normalized so it is visible. Without normalization the trace just looks like a flat line.*

I imagine one use for this would be a way to provision IoT devices: the “how do I get wifi credentials into an IoT device that lacks both screen and keyboard?” problem. With the addition of a ~$1 microphone to a Cortex-M4 class device, you get a short-range data link to a host device, such as a phone. You can use a web page (via Javascript) to generate the modulated audio directly ( [relevant example](https://github.com/chibitronics/ltc-npm-modulate)), thus bypassing a host of multi-platform issues, or you [can generate a file off-line](https://github.com/simmel-project/bpsk-modem) and send it to any standard music player.

The TL;DR on the link is it uses a 20,833Hz carrier modulated with BPSK. We use PSK31 coding, so our baud rate is ~651 symbols per second (this is the 1/0 symbol rate before Varicode encoding). This isn’t breaking any speed records, but it’s good enough to send a UUID and some keys over the air in a couple seconds. Tests show decent performance over a distance of 1 meter with about 60dB ambient noise (normal conversation or background music playing at the same time).

[![](https://raw.githubusercontent.com/simmel-project/nus-link/master/simmel_demodulator.png)](https://raw.githubusercontent.com/simmel-project/nus-link/master/simmel_demodulator.png)

The demodulator uses a [Costas loop](https://en.wikipedia.org/wiki/Costas_loop). We’ve [documented its details, including comments on porting to other chipsets](https://github.com/simmel-project/nus-link/tree/master/demodulator) than the NRF52.

We also have a [reference modulator using a non-linear transducer](https://github.com/simmel-project/nus-link/tree/master/modulator) (e.g. a piezo element), which uses some of the more advanced features of the NRF52 PWM block to eliminate audible sidebands. We also have a rough [C program to generate a .wav file](https://github.com/simmel-project/bpsk-modem), which needs to be run through a high-pass filter using e.g. Audacity to eliminate the low-frequency modulation sidebands; but the [resulting .wav file](https://github.com/simmel-project/bpsk-modem/blob/master/samples/modulated-20833-hpf2.wav) can be played directly on your smartphone and it will demodulate correctly.

*Acknowledgements: [Sean ‘xobs’ Cross](https://xobs.io/) is an equal contributor to this research. This research is funded through the [NGI0 PET Fund](https://nlnet.nl/PET), a fund established by NLnet with financial support from the European Commission’s [Next Generation Internet](https://ngi.eu/) programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 825310.*
