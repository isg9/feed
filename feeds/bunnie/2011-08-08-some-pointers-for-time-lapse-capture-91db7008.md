---
title: Some Pointers for Time Lapse Capture
url: https://www.bunniestudios.com/blog/2011/some-pointers-for-time-lapse-capture/
published: "2011-08-08T14:24:25Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1794
---

# Some Pointers for Time Lapse Capture

A couple of folks had requested a how-to on modifying the chumby One for video capture.

Unfortunately, I did this hack almost a year ago and took few notes on it, but I’ll post my fuzzy recollection here, and hopefully we can figure out any issues in the comment thread.

First thing to do is to pick a USB camera that’s compatible. That’s a little bit tricky because I don’t actually know why some cameras work and some don’t. The USB camera I used is one that is salvageable from a laptop — the camera board has a connector onto which I soldered the USB cable. I opted to use this because it’s a small, rectangular and flat board that’s easy to tape to a window (the ball cameras used for video conferencing are not as easy to tape in place). And it was free. I’d take a photo of the assembly except it’s taped to the window inside a cardboard baffle to reduce glare at night time from the indoor lights, and if I move it the video capture will shift. But, the video drivers compiled into the chumby One kernel are just the stock drivers taken straight out of the Linux source tree, so if it’s a camera known to work with Linux circa 2008 you’ll have a decent chance of it just working.

Next, you’ll need to grab mplayer and install it. A pre-compiled and statically linked version that just works with the chumby One can be downloaded [here](http://bunniestudios.com/chumcap/mplayer.gz) (the file is gzipped, you must unzip it before running it). mplayer is tricky and tedious to cross-compile, and the config files are long lost as xobs did the cross-compile for me. This particularly annoying barrier is being fixed for the future by migrating chumby’s new platform (which I hope to announce next month) to Open Embedded and providing developers with a pre-configured EC2 cloud image that will hopefully allow you to build and install packages with deep dependency trees with much less effort than previously required.

Once you have mplayer installed, try this script:

`mplayer tv:// -tv driver=v4l2:width=1280:height=1024 -vo jpeg -frames 10`

This will create 10 jpeg files in the directory that mplayer is located.

If this works for you, then you’re almost there.

The rest of the tweaks I’ll share are for getting around aperture-setting weirdnesses unique to my camera and the automatic photo taking. This particular camera has a problem where when you turn it on, it always starts with the aperture wide open, which means the first image is way over exposed. The following script represents close to the final arrangement for image taking:

` #!/bin/sh cd /mnt/storage echo "running first pass" mplayer tv:// -tv driver=v4l2:width=320:height=240 -frames 10`

`echo "running second pass" mplayer tv:// -tv driver=v4l2:width=1280:height=1024 -vo jpeg -frames 10 echo "Resize a preview thumbnail so you can monitor image quality from the screen" chumbthumb -x 320 -y 240 -i /mnt/storage/00000010.jpg -o /mnt/storage/resize.jpg echo "show the image on the screen" imgtool /mnt/storage/resize.jpg`

`echo "Give the JPEG a unique name and move to storage" NOW=$(date +'%s') mv /mnt/storage/00000010.jpg /mnt/storage/stills/${NOW}.jpg `

The first pass exists to get around a bug where about 5% of the time, the camera would grab just a plain green screen. The second pass captures 10 frames and I only use the 10th frame captured because that’s empirically about how long it takes for the camera to adjust its aperture. A thumbnail is made, so that another script can toss an image on the LCD so you can monitor the quality of the camera. And finally, the image is given a unique name which is equal to the current time since epoch and moved to storage.

In order to set the timing for the image capture, the following crontab was used to call the above script once every 15 minutes:

`chumby:/psp/crontabs# cat root 8 3 * * * /usr/chumby/scripts/sync_time.sh 30 * * * * /mnt/storage/take-image.sh 0 * * * * /mnt/storage/take-image.sh 15 * * * * /mnt/storage/take-image.sh 45 * * * * /mnt/storage/take-image.sh`

In order to “guarantee” long term stability of the device, the actual implementation I used has the device rebooting itself after taking the image. There are a few quirks in the camera driver that are always solved by a reboot, and I didn’t want to have to worry about a quirk of the camera driver ruining frames. It’s been reliable enough for a year, most of the missing images are due to times when we knocked the power supply out of the wall while cleaning house and the battery ran out before we noticed.

The other thing is that the control panel that normally runs on a chumby gets in the way of showing your resized thumbnail (the chumby will show widgets that bash the image on the screen), so to disable it I created a /psp/rfs1/userhook2 file (userhooks are run during boot automatically in the chumby OS implementation) (don’t forget to give the script a+x permissions) with the following contents:

` #!/bin/sh`

`start_network imgtool --fb=0 --fill=0,0,0 imgtool --fb=1 --fill=0,0,0 imgtool /mnt/storage/resize.jpg`

`while true do sleep 1700 stop_control_panel start_network done `

This ensures that the network is started (which is important to set/keep network time), and the script is designed to continuously call stop\_control\_panel and start\_network just in case there is a connectivity issue that comes up (which would normally be fixed by the control panel, but since you’ve killed it you need to manage it yourself).

That’s about it. I get the files off by mounting a NAS over SMB and copying them, and I had a couple cgi-scripts that also let me preview the thumbnail via the web server built into the chumby, but these are really just embellishments, you can get quite fancy on the network copying part depending upon what you have or don’t have in your home LAN.

Oh, and finally — creating the video. With the files on the SMB share, I encode the video using a “real” PC with the requisite horsepower. I used mencoder, with these arguments:

` opt="vbitrate=6400000:mbd=2:keyint=132:vqblur=1.0:cmp=2:subcmp=2:dia=2:mv0:last_pred=3"`

` `

`mencoder mf://*.jpg -mf w=1280:h=1024:fps=12:type=jpg -ovc lavc -lavcopts vcodec=mpeg4:vpass=1:$opt -nosound -o /dev/null mencoder mf://*.jpg -mf w=1280:h=1024:fps=12:type=jpg -ovc lavc -lavcopts vcodec=mpeg4:vpass=2:$opt -nosound -o output.avi `

It’s a two-pass encode that creates a decently good looking stream with no sound.

Happy hacking!
