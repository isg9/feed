---
title: Name that Ware December 2005
url: https://www.bunniestudios.com/blog/2005/name-that-ware-december-2005/
published: "2005-12-31T13:50:47Z"
feed: bunnie
guid: http://www.bunniestudios.com/wordpress/?p=74
---

# Name that Ware December 2005

The Ware for December, 2005, is shown below. Click on the image for a much larger view.

[![](http://bunniestudios.com/blog/images/ware_12_2005_sm.jpg)](http://bunniestudios.com/blog/images/ware_12_2005.jpg)

A friend of mine gave this to me to try out, and I had to open it up to see what was inside. I was a bit surprised when I looked in there, I thought there would be a lot more, but I guess simplicity is elegance. The board was well-marked, so I had to pixelate portions of the silkscreen and chip markings to make this contest non-trivial. However, based on past performance, I’m guessing people will figure this one out in no time flat.

Again, sorry for this month’s ware coming so late! I’m posting at the last possible moment to still claim a ware for December, 2005. It’s been an exciting month though; a lot of very interesting projects I’m working on have passed pivotal stages and I’m looking forward to seeing what January will bring. I’ve also been observing the progress on the Xbox360 hacking, and I’m impressed. The hacking scene is more or less an organized anarchy that is frightfully productive. Now that I’ve had a little brush with being a manager in my day job, I can see that clarity of purpose obviates the need for management; people just self-organize and things happen. I could ponder on this for many parargaphs, but I’ll spare you my treatise on human social behavior.

At any rate, some very interesting things are afoot. Much of it stems from the discovery of an [all-media bootable kiosk demo disk](http://www.theinquirer.net/?article=28574). Many hackers will instantly recognize the value of this, but it’s still interesting to reflect on the significance of this find.

Like the original Xbox, the Xbox360 uses a media flag on its executables. The media flag tells the OS what type of media it should be on; typically, games are released with the flag set to Microsoft’s proprietary secure Xbox DVD format (which is in itself not that secure…). Significantly, only the executable is signed for a game; the data sections typically are not signed (presumably for performance reasons). Thus, one has the ability to fuzz the executable by corrupting the data sections, potentially invoking a buffer overrun or some other unintentional behavior–if one could effectively modify the data sections. Remember that this is normally not possible, since modifying the data segment requires making a copy to a writeable media, and this contradicts the signed media flag.

Thus, the run-anywhere demo disk now enables software hackers to create and test the interaction of signed executables with modified game data using no tool other than a DVD-RW drive (and an Xbox360 console, still considerably rare and difficult to obtain in the US). Some of the more interesting modifiable data regions include Shockwave Flash movies, and the pixel shaders executed by the GPU (more info can be found on the [xboxhacker.net](http://www.xboxhacker.net/) website). Of particular interest is the [MEMEXPORT](http://www.beyond3d.com/articles/xenos/index.php?p=10) shader command in the 360, which could enable people to dump physical memory to the screen (where it can be digitized or extracted with a sniffer upstream of the ANA chip), or to some other peripheral function. Presuming plaintext kernel code can be extracted this way, it bootstraps further efforts in vulnerability analysis of the code running in the Xbox…and so forth. Of course, its quite possible that this hole is plugged, since Microsoft’s [NGSCB](http://en.wikipedia.org/wiki/Next-Generation_Secure_Computing_Base) spec calls for the Northbridge to limit DMA access from the graphics card to main memory. Furthermore, buffer overrun exploits have questionable applicability since each process runs as its own virtual machine and rumors has it that the no-execute bit is used on heap space. Still, I’m very surprised that such a media was even released into the wild by Microsoft…their own worst enemy is their own haste to get to the market and carelessness; security is for naught without consideration of human factors. Very exciting! Perhaps the Xbox360 will be opened without the need for significant hardware hacking.

Oh, and happy new year to everyone! I’ll be at [Spundae’s New Year event](http://www.spundae.com/fls/la19.html) in LA with some friends. If you happen to be going, give me a shout if you see me. Should be great fun!
