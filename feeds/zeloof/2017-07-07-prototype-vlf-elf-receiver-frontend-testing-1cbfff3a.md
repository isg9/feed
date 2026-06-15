---
title: Prototype VLF/ELF Receiver Frontend Testing
url: https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/
published: "2017-07-07T00:31:48Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=940
---

# Prototype VLF/ELF Receiver Frontend Testing

[Live VLF Stream](http://sam.zeloof.xyz/live-vlf-stream/)

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/IMG_0802-300x225.jpg)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/img_0802/)

2 Stage Amplifier and Filter

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/IMG_0801-300x225.jpg)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/img_0801/)

60Hz Harmonics

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/IMG_0811-e1499466012246-300x181.jpg)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/img_0811/)

Testing Outside – No 60Hz Harmonics

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/Capture-300x181.png)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/capture/)

10.8KHz Reject filter (dog invisible fence)

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/FullSizeRender-300x225.jpg)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/fullsizerender-5/)

V2 Receiver

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/07/FullSizeRender-1-300x225.jpg)](https://sam.zeloof.xyz/prototype-vlfelf-receiver-frontend-testing-natural-radio-1hz-30khz/fullsizerender-1/)

V2 Receiver

[http://sam.zeloof.xyz/wp-content/uploads/2017/07/RecordedAudio\_0004.wav](http://sam.zeloof.xyz/wp-content/uploads/2017/07/RecordedAudio_0004.wav)

Quick prototype of a VLF/ELF receiver with two stage low noise amplifier and passive RC bandpass filtering. First stage has a gain of 180 then is fed to a lowpass and highpass filter then to another opamp with adjustable gain up to 500. Resulting signal is sent to a Focusrite Scarlet i2i 24bit USB audio interface and sampled at 192KHz.

This simple preamplifier can easily pick out the “fire crackling” of lightning strikes around the world form the noise floor and can easily resolve other natural low frequency radio phenomena such as “”whistlers”. Active bandpass filtering and 60Hz notch filter will be added as well as a shielding enclosure and low noise Analog Devices OP270 or similar opamps will be used in another iteration of the design.

[http://sam.zeloof.xyz/wp-content/uploads/2017/07/2.m4v](http://sam.zeloof.xyz/wp-content/uploads/2017/07/2.m4v)

Antenna used was two large screw drivers inserted into the ground soil separated by about 50 feet in the woods 1/4 mile from any house or power lines. 60Hz noise was exceptionally low considering only passive RC filtering was used which has high and low -3dB points far away from what you actually design the cutoff frequencies of the filter to be.

This frontend contained no input protection in the form of glow discharge neon lamp, clamping diodes, thermistor, MOV, fuse, etc. Next design will incorporate multiple such devices however it is not a great priority for testing because I am using the screwdriver ground probes for antenna which is less susceptible to high voltage buildup in storms as a whip or dipole antenna.

![](http://theproblemwithwindpower.com/info/sanguineGlengarryForestELF_files/elftxsubtxf.jpg)

[http://www.vlf.it/](http://www.vlf.it/)
