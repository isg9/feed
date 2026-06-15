---
title: So you want to stream a conference? — wingolog
url: https://wingolog.org/archives/2006/07/07/so-you-want-to-stream-a-conference
published: "2006-07-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/07/07/so-you-want-to-stream-a-conference
---

# So you want to stream a conference? — wingolog

## [So you want to stream a conference?](/archives/2006/07/07/so-you-want-to-stream-a-conference)

7 July 2006 9:52 PM

- [guadec](/tags/guadec)
- [flumotion](/tags/flumotion)
- [conferences](/tags/conferences)

Ah, conferences. Those ephemeral events that make virtual communities into sweaty reality, if only for a week.

I've been involved with the streaming of three conferences now, and have a few things to say about it. A lot of things have to go right for the first image ever see the internet. Here's what I've learned.

### A note on cooking

Picture in your mind the perfect stream: hundreds or thousands of happy viewers and listeners; clear, readable slides; clear sound; good view of the presenter. All of these are reflections of quality inputs to the streaming process. A good pie is made of good ingredients. Everyone likes good pie.

Good ingredients take you most of the way, and free your time to focus on polish and details. Make sure that you have the support you need before taking on this responsibility.

### Pre-conference preparation

Each room that will need streaming will need the following things:

1. A computer to encode the video, save it, and send it over the network.

2. A video camera with a tripod, line-level audio input, and firewire video output.

3. An ethernet network connection for the encoding machine, with a fixed address and dedicated bandwidth.

4. Power (two plugs, one for the camera and one for the computer).

5. A sound cable coming from the mixing desk, at line level.

All of these things need to be in the same place in the room, at an appropriate location to record the speaker and the screen. This is very tricky; typically 3, 4, and 5 depend on other people, and separate people at that. These people need to cooperate with each other before you get to the site, and with you once you are there. Tricky!

So, well before the conference starts, it must be absolutely clear both to the organization and to the respective people who is responsible for each of these items. Furthermore, it is your responsibility to communicate to these people exactly what it is you need from them, and to ensure that they are providing this to you.

Note that streaming that involves moving the camera is too hard. Don't commit yourself to do it. No one that is responsible for streaming talks should have a technical burden like that -- moving sound, moving power, moving video, moving network, moving computers. Too tough.

### The computers

The actual type of the computer is not terribly important, other than the fact that it should be based on a more recent X86 processor, to take advantage of optimizations in the theora encoder. It needs an ethernet interface, a moderate amount of memory, and a moderate amount of disk.

For example, these are the specs of the machines I bought for GUADEC 2006: AMD Sempron 3000+, 512 MB ram, 80 GB hard drive, integrated NIC and USB. USB is useful to connect to an external hard drive for copying off the archives. I then had to buy firewire cards (any brand should work). The total cost for four machines was about 1150 euros. Hyperthreading or SMP will offer a significant performance boost.

The machines worked pretty well. The one problem with the process is that theora encoding is extremely expensive in terms of CPU, and its cost depends on the amount of noise and motion in the video. We were recording 360x288 at 12.5 frames a second, which was about the maximum the machines could do, and probably a bit too high -- there were a few times in which the encoder couldn't do it in real time and so we missed some frames. You *really* want to avoid this. Once encoding starts lagging behind it's difficult to catch up to steady-state again.

Note that for best optimization on the theora encoder, you should install the 32-bit x86 version of your favorite up-to-date linux distro. While I like Ubuntu on the desktop, Fedora Core 5 has better packaging of the software I use to stream (flumotion), so I use it.

Also note that preparing a machine without an up-to-date suitable operating system will take a fair amount of time. If you can get this done before arriving at the conference, your life will be much easier.

### Cameras and sound

For best quality, you want a good camera. A good lens means less noise in the raw video, leading to more efficient encoding and a higher quality result. Decent zoom allows you to place the camera in the back of the room, which both gives a better angle and is out of the participant's way. Of course the camera needs to sit on a tripod; there's not much to say there, other than the fact that better cameras are heavier and need sturdier tripods.

All cameras that I have seen, from handicams to professional cameras, have DV output over Firewire (IEEE 1394). DV Firewire cameras are all accessible via Linux, via the same interfaces, so you should not have a problem there.

Firewire cameras are very nice because they guarantee proper synchronization of audio and video. The audio plugs directly into the camera, and the camera hardware ensures that the audio and video have proper timestamps. While recording audio from a sound card is also a possibility, it's much preferable to get the audio into the camera directly.

For the audio to enter the camera, you have a number of options. I will enumerate them from worst to best. Worst is to use the internal microphone of the camera. You will catch all of the background noise, very little of the presenter (unless the camera is close to a speaker), and lots of chit-chat and noise from people around the camera. Also you will get the fan noise from the computer. Very bad, this.

Hooking into the room's sound system is the other, better option. However the cheapest handicams only have an unbalanced mic-level minijack (1/8" [jack plug](http://en.wikipedia.org/wiki/Jack_plug)) input, intended to be used with a simple microphone. This sucks. Normally from the room's sound system you can only get a balanced line-level input, which is too much for the camera. You end up having to string a bunch of plug converters together, precariously sticking out of the side of the camera, and the quality is still terrible. Very bad. This is also to be avoided.

If you are in this situation, but you want good sound, the best option is to get an attenuator, something like the [BeachTek DXA-2s](http://beachtek.com/dxa2s.html), about 150 euros or so. You could have the mixing desk lower the level and get a converter cable (e.g. female XLR to male minijack), but it could be noisy. Depends on cost. Planning is key, though.

Line-level minijack inputs to handicams exist, but are less common. Your dynamic range will be better, but you still have to get a plug converter.

The best option by far is to get a camera that can hook directly to what the mixing desk gives you, which is likely to be a line-level [XLR](http://en.wikipedia.org/wiki/XLR) male cable. XLR connectors, in addition to being structurally much stronger than a minijack, are [balanced](http://en.wikipedia.org/wiki/Balanced_audio_connector), which reduces the noise in the signal. Better cameras allow direct connections from XLR cables, both for use with microphones and with inputs from mixing desks.

The cameras we used at GUADEC 2006 were [Panasonic AG-DVX100AE](http://salvadorserra.com/producto.aspx?idcodigo=59139) s, and one handicam. Renting the three cameras and four tripods cost 1600 euros for a week. The Panasonics were pretty good, although we had some problems obtaining proper sound in a number of rooms, due to cable problems and people playing with the mixing desks. Also, the camera would take the input and record it to one channel only, which is a bit of a pain. The handicam was missing some kind of breakout cable for the audio input, so we had to use the handicam internal mic.

As you can see, the issues involved in getting sound into the camera are tricky. It is best to think about it all beforehand; analyze each room, determine what you will need, and make sure that the event organizers provide it to you, or obtain it yourself.

If you do not know what will be there, you have to be prepared. If the room is wired and has mic jacks outlets on the walls, assume there is a female XLR jack you can connect to. If you will be connecting directly to the mixing desk, you will probably need a male 1/4" jack connector. In both cases, make sure you have the proper length of cable.

Note that there is a second kind of digital video camera that uses something called IIDC instead of DV for storing and transmitting the video. The difference is that DV is compressed (slightly; it's similar to JPEG), and IIDC transfers raw video, which can offer higher quality. You don't have to worry about it too much, though -- IIDC is mainly found in industrial and scientific cameras only. I mention it because of the one exception, the iSight from Apple. You will not be able to use the iSight as a DV camera.

### Network and power

In the conferences I've worked with, it's always been like this: some facility is rented out, the event organizers work with that facility and with a sound company to do the sound, and then run the network and the power themselves. So my comments are colored by those experiences, and that you normally have to talk to the same people for network and power.

On the topic of power, there's not much to say. You need one plug for the camera and one for the computer. It's useful to have a couple extra, for example for your laptop if you need to plug in, and an external hard drive. But you can get by with two outlets.

Actually, there is something worth saying: many facilities completely power off at night. As in, the security guard comes in and throws the breaker. Whine and complain, try to make sure the power will be there for you, but in the end it might happen during the conference. Cameras handle this gracefully, turning back on and functioning when the power comes back. Computers do not, in my experience. If you can figure out a way to make computers turn back on when the power returns you will be a happier person.

The network requirements of live streaming are the most obvious ones. You need an ethernet (cable) connection, of about the bandwith of your stream (400-700 kilobit), with a fixed IP address, dedicated bandwidth, and no blocked ports. The encoder computers should normally connect to another machine, internal to the conference, that will aggregate the streams for internal streaming and for relaying to some external server with more bandwidth. Of course in the case that the conference has lots of bandwidth itself, you can stream from that machine instead.

The reason you want that extra machine is to make the network peoples' job easier. Normally you're on a private network with NAT access to the internet, and you want to limit the holes in the firewall to one machine, located physically close to the firewall. If your server can have two IPs, one public with no ports filtered and one internal, that is the ideal solution.

As GUADEC 2006 showed, it is possible to do streaming over a wireless connection. However it has to be your wireless connection, not shared by anyone else, and with sufficient bandwidth. Even then, the routing issues can get tricky, and GUADEC 2006 was not perfect in that regard.

In GUADEC 2006 we also ran into a strange issue, where the NICs on the machine would not link over a long cable, whereas it would link on our laptop NICs and with the switches. The solution was to install a switch right next to the machine. That was an odd problem.

Note that in the end, once you get to the conference, you will probably have to run around to each of the machines, setting the network addresses and verifying that things are working correctly. You can do this with a laptop if the machine has two NICs, plugging into the alternate interface with a crossover cable (or a normal cable if your laptop detects and switches, as in some modern laptops). Otherwise you will be dragging a monitor and keyboard around everywhere for the first few hours.

### At-conference preparation

You arrive at the conference with just your laptop. All of the computers are there, installed with Fedora Core 5, updated, and hooked to ethernet cables. You have a keyboard and a mouse. There is also a wifi network available so that you can access these machines. You boot them all, check their IPs with the keyboard and mouse, write them down, then go and sit in a nice comfortable chair with a desk.

Of course, if any of this is missing, you will spend a lot of time getting it together. Carrying computers, cameras, tripods, cables. Marshalling the network and working space that the organizers forgot about in their last-minute rush. Et cetera.

Anyway, back to fantasy land! From your working space, you ssh into all of the servers as root, adding your ssh key to `~root/.ssh/authorized_keys`. The servers need to be updated to the latest errata and to have the streaming software installed. We will use FC5 with [Flumotion](http://www.flumotion.net/). Flumotion is distributed in GStreamer's repository, so add it to your repos list and upgrade:

```
wget http://thomas.apestaart.org/pkg/thomas.pubkey
rpm --import thomas.pubkey cd /etc/yum.repos.d
wget http://gstreamer.freedesktop.org/download/gstreamer-0.10-deps.repo
wget http://gstreamer.freedesktop.org/download/gstreamer-0.10-gst.repo
yum -y upgrade && yum -y install flumotion
# make sure all users have access to firewire
echo 'KERNEL=="raw1394", MODE="0666"' > /etc/udev/rules.d/60-flumotion.rules

```

Do this on all your machines. It will take quite a while to complete depending on your network speed; an alternate option is to somehow mirror the updates you need to DVD or portable hard drive and take that with you. I'm not that good with Fedora, however, and so I just rely on the network being there. The consequent is that you *really* need the network to work.

While this is installing, go find your network people and get them to give you fixed IPs for the encoding machines, and check to see that you have network, power, and sound at an appropriate place in each room. Set up the cameras, and do sound checks. A decent camera will show you the sound levels on the video screen. Adjust the levels (either on the camera or from the mixing desk) so that you hit red occasionally in normal speech, with plosives like p and b.

Back at your computer, everything has updated. Now you just need to get Flumotion running on the system. You might find the [official manual](http://www.flumotion.net/doc/flumotion/manual/manual.pdf) useful. I find it a bit hard to read, so I'm not going to refer to it.

### Flumotion setup

I will describe use of Flumotion to stream the conference. It's a pretty good streaming server, with several desirable characteristics.

First, Flumotion is completely Free Software. It emphasizes free formats, like Ogg Theora and Vorbis. When people fly halfway across the world to talk about freedom, it is important that the tools they use reflect and support the freedom they seek.

Secondly, Flumotion is complete. It handles the entire streaming process from capture device access to serving http to clients. It has a graphical administration tool with a wizard to help you configure the streaming process, with good error handling. Flumotion is integrated with the standard Unix services architecture as well.

Finally, Flumotion is distributed. The streaming process can span multiple machines, which is good as the resource constraints are topologically different as well; typically you have CPU power at the video source, and bandwidth in one or more other locations. Flumotion allows many "workers" to run Flumotion processes on behalf of a manager.

I don't know of any other end-to-end solutions like Flumotion. You can probably get `dvgrab | ffmpeg2theora` to feed an Icecast server if you are sufficiently wizardly. But as far as something that's easy to install and run, Flumotion's probably the only game worth playing.

Architecturally, a Flumotion system is defined by one manager running somewhere, and one or more workers running on the machines of interest. All workers and admin clients will have to log into the manager, so if you are relaying streams to an external server, your manager should be placed on a public machine, like the server machine. I'll assume it's there.

What we did at GUADEC was to have one manager for every room we streamed, which meant that there were four managers. Since we were streaming through the server machine, there were four workers running on the manager, one on each of the encoding machines, and four workers running on the [streaming platform](http://stream.fluendo.com/). (A worker can only connect to one manager.) It's a bit complicated, which is mostly due to the number of machines in the system (6), and the need to authenticate on the various connections. We could have pushed everything through one manager, but the admin client is better optimized for one "flow" (the streaming process, from capture to encoding to streaming). Also one manager is one point of failure for the whole system, which is bad considering the various networking problems you might run into.

To configure one room, for instance the Sala de Juntes, you have to (1) make a worker on the worker in the Sala de Juntes; (2) make a worker on the server; (3) make a manager; and (4) start a worker on the streaming platform. This is actually pretty easy, given Flumotion's service script integration.

Given that you will need four managers, first decide what ports they will listen on on the server. The default port is 7531, so I chose 7531-7534. The Sala de Juntes was the last one I configured, so it was 7534.

For (1), drop the following XML snippet in /etc/flumotion/workers as juntes.xml:

```

  4

    10.0.0.120
    7534


    flumotion
    Pb75qla


```

The password will need to be the same as the one configured in the manager. Delete `/etc/flumotion/workers/default.xml` and `/etc/flumotion/managers/default/`, because we are not running the default setup, and say `service flumotion status` on the command line. It should give you something like the following:

```
worker juntes not running

```

That's sweet. Go ahead and `service flumotion start`, and `chkconfig flumotion start'` to ensure flumotion will be started on boot.

On the manager, remove `/etc/flumotion/workers/default.xml` and `/etc/flumotion/managers/default/` as before. Drop the same XML snippet as before into `/etc/flumotion/workers/juntes-server.xml`. Note that the name of the file by default determines the name of the worker, and two workers with the same name can't be logged in at the same time. It's important to make sure the files are named differently.

Now, you need to set up the manager. Make a directory for it, `/etc/flumotion/managers/juntes/`, and a directory for your flows, `/etc/flumotion/managers/juntes/flows/`. Drop the following snippet in `/etc/flumotion/managers/juntes/planet.xml`:

```



    7534
    5





```

The bit about the CDATA is the user that's allowed to log in, in htpasswd format. You can generate this via `htpasswd -nb flumotion Pb75qla`.

Now, when on the server you do `service flumotion status`, you should get:

```
manager juntes not running
worker juntes-server not running.

```

At that point you're good to go. `service flumotion start` and `chkconfig flumotion on`. Sweet. Your Flumotion is ready to go. You'll have to wait until the computers are connected to the cameras and have their proper IPs to set up the flows though, because the setup wizard actually checks that all of the hardware is working, including the DV connection to the camera.

### Final setup

Having obtained the IP addresses for your boxen, configure them (on fedora, that involves mucking about in `/etc/sysconfig/networking/` somewhere), and add their IP addresses to your own `/etc/hosts`. Shut them down and move them to their locations. Verify with your laptop that the network cables that you have actually have the right IP addresses, and that they can reach the gateway and the internet. While they don't have to have internet, it is good for last-minute package installs. Turn on the machines at their locations and plug in the Firewire cables that you brought (2 meter minimum) to the camera.

On your local machine, run flumotion-admin. Choose "Connect to running manager", and enter the host and port of the flumotion manager. Enter your authentication information. At this point flumotion-admin will pop up a wizard to help you configure the flow. Choose Firewire as the audio and video input and choose 360x288 as the video size with 12.5 frames a second. When you go on to the overlay, note that turning off all overlay options is a big performance improvement, so you might want to do that if you are seeing loads that are too high. You can choose to stream or save any combination of audio and video. Remember to choose the right worker on each page, so that you are running the encoding on the capture machine, for example.

After finishing the wizard, you should see lots of smiling faces in your flumotion-admin window. At that point, check to see if you can watch the stream. Dump the configuration to a file via File->Export configuration, and drop that file into `/etc/flumotion/managers/juntes/flows/default.xml` on the server machine. Once you've set up on stream you can just copy/paste for the rest.

For the record, the flow that we used for the GUADEC carpa is available [carpa.xml](//wingolog.org/pub/carpa.xml).

### In-conference activities

Ideally everything is ready to go the day before the conference. If you haven't done anything, and have to install from barebones machines, you'll need three days to prepare, provided that things go well. If you have arranged for the machines to be installed already with up-to-date FC5, and the sound, network, and power, and cameras are installed in a decent spot in all rooms, you might be able to do it in a few hours. I haven't seen that happen before though, so do give yourself some time.

Potential problems during the conference include power outages, network outages, bad sound connections, and poor framing of the speaker and slides. The actual circumstance of getting the speaker and slides to come out nicely is tricky, and probably merits another discussion. People working the cameras like to zoom in on the presenter, while remote viewers place more importance on seeing the slides, especially if the slides are not posted for public download. Decide beforehand which you want to see, and consider printing out guidelines and taping them to the encoding computers.

By this time, you will need some autonomous help. You'll be a bit tired, and other people will be more able to help out. The quality of the streams will need to be monitored, and the inputs corrected if they go bad (bad sound, etc). Other people can do this for you, and fix problems if possible.

### Archives

In our next installment, hopefully coming tomorrow, I'll talk about how to get decent archives. Until then, corrections, questions and comments are most welcome, preferably via comments on this weblog entry.

## related articles

- [to guadec!](/archives/2011/08/03/to-guadec)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [guadec ho!](/archives/2009/07/02/guadec-ho)
- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [regarding decadence](/archives/2008/06/10/regarding-decadence)
- [gnome in the age of decadence](/archives/2008/06/07/gnome-in-the-age-of-decadence)

### 14 responses

1. Christian Hudon says:[7 July 2006 10:35 PM](#128)

   Great article. One small tidbit of interest. When you say that "Cameras handle this \[power loss\] gracefully, turning back on and functioning when the power comes back. Computers do not, in my experience."... in my experience, most PCs have a setting in the BIOS to automatically power on when the power returns. (Very useful when said computer is connected to a UPS, for instance.) I guess that would be what you are looking for.

   Also, given the rather low ($50) price for a low-end UPS, if you're expecting power loss at night, it might be a good idea to buy one for each computer, just to guarantee you a clean shutdown.

2. [Ben Hutchings](http://womble2.livejournal.com) says:[7 July 2006 10:43 PM](#129)

   If you only managed 12.5 fps, it sounds like you weren't using the MMX version of libtheora. At DebConf we were encoding at 25/30 fps and initially this became CPU-bound at times on an Opteron (I forget the exact model). Installing the MMX version of libtheora cut processor usage down to a comfortable 40-70% (depending on the scene). I wrote a program called dvtail that reads DV frames starting near the end of a file, so that we could record to disc and drop frames from the stream but not the recording if necessary. We used the pipeline dvtail \| ffmpeg2theora \| oggfwd.

3. Jose Antonio Verrire says:[7 July 2006 11:02 PM](#130)

   Helpful guidelines Wingo. Thank you.

   Here is the network diagram of Guadec 2006.

   http://guifi.net/ca/node/4964?size=\_original

4. Juanjo Marín says:[7 July 2006 11:17 PM](#131)

   Congratulations Wingo for your excelent work !!!. By the way, a good recipe for streaming video.

5. Ben says:[7 July 2006 11:52 PM](#132)

   Neat info. How feasible would it be to create live-CDs for the streaming and encoding servers? (I'm assuming you could still use the hard drives for scratch space). I'd think that would save a lot of work. Does fluendo work w/ Avahi? That might help even more.

6. nzjrs says:[8 July 2006 4:12 AM](#133)

   Great article!

   Re: Computers turing off; most new computers have an option in the bios which will re-power on the machine if there is a powercut.

   Also I noticed considerable out-of-sync-audio-and-video when watching the streams (from outside the conference). I think 12.5 fps was a little too much for the computers

   Speaking of archives - how are the Guadec archives coming along? Is there an ETA of when they will be available at http://stream.fluendo.com/archive/guadec/2006/

7. [J.B. Nicholson-Owens](http://www.digitalcitizen.info/) says:[8 July 2006 7:56 AM](#134)

   Very interesting and informative article, thanks so much for blogging this.

   One correction (quite minor for anyone who is skilled enough to do this): you need a new line for the cd command after you get import Thomas' public key.

8. Matt says:[8 July 2006 8:40 AM](#135)

   Are you going to be releasing the archives for the 2006 guadec anytime soon? I wasn't able to tune into the live streams, and I enjoyed watching the 2005 videos last year. Great post btw!

9. [Too Far Afield » Blog Archive » So you want to stream a conference?](http://blog.nachtarbeiter.net/?p=371) says:[8 July 2006 6:40 PM](#136)

   \[...\] So you want to stream a conference? Read that article. \[...\]

10. [Zaheer Merali](http://zaheer.merali.org) says:[8 July 2006 11:19 PM](#137)

    Ben,

    We also had 2 other inefficient pieces in the cpu load of the machines. One is we use libdv for dv decoding which is a lot less performant than ffmpeg's dv decoder. Second is that we overlayed images onto the video, for which the code we have written is not the most performant.

11. [Zaheer Merali](http://zaheer.merali.org) says:[8 July 2006 11:22 PM](#138)

    njrs, the issues had with videos being poor on the stream URLs are more likely the contention on the traffic going out of the Guadec network. From what I have seen of the archives, they are pretty good and in-sync.

12. Baris Cicek says:[9 July 2006 0:00 AM](#139)

    What a great article. I like people keeping their promises. :) But a missing section about tools, which does not need to reencode streams, to cut archives.

13. [passing through unconscious states -- andy wingo](http://wingolog.org/archives/2006/07/10/passing-through-unconscious) says:[11 July 2006 1:10 AM](#140)

    \[...\] Um, speaking of which, your last writing product was rather long. \[...\]

14. [cypherpunks writecode -- andy wingo](http://wingolog.org/archives/2006/07/21/cypherpunks-writecode) says:[22 July 2006 1:42 AM](#149)

    \[...\] Anyway, so this is a prelude to the sequel to my last post on conference streaming. What I wanted to say was something about ensuring that you get archives on disk, and then to edit them later. \[...\]

Comments are closed.
