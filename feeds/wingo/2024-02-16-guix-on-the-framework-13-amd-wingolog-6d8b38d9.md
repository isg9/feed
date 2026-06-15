---
title: guix on the framework 13 amd — wingolog
url: https://wingolog.org/archives/2024/02/16/guix-on-the-framework-13-amd
published: "2024-02-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/02/16/guix-on-the-framework-13-amd
---

# guix on the framework 13 amd — wingolog

## [guix on the framework 13 amd](/archives/2024/02/16/guix-on-the-framework-13-amd)

16 February 2024 10:23 AM

- [guix](/tags/guix)
- [laptop](/tags/laptop)
- [framework](/tags/framework)
- [amd](/tags/amd)
- [firmware](/tags/firmware)
- [fsf](/tags/fsf)
- [praxis](/tags/praxis)

I got a new laptop! It’s a [Framework 13 AMD](https://frame.work/fr/en/products/laptop-diy-13-gen-amd): 8 cores, 2 threads per core, 64 GB RAM, 3:2 2256×1504 matte screen. It kicks my 5-year-old Dell XPS 13 in the pants, and I am so relieved to be back to a matte screen. I just got it up and running with [Guix](https://guix.gnu.org/), which though easier than past installation experiences was not without some wrinkles, so here I wanted to share a recipe for what worked for me.

(I swear this isn’t going to become a product review blog, but when I went to post something like this on the [Framework forum](https://community.frame.work/) I got an error saying that new users could only post 2 links. I understand how we got here but hoo, that is a garbage experience!)

### The basic deal

Upstream Guix works on the Framework 13 AMD, but only with software rendering and no wifi, and I wasn’t able to install from upstream media. This is mainly because Guix uses a modified kernel and doesn’t include necessary firmware. There is a third-party [nonguix](https://gitlab.com/nonguix/nonguix) repository that defines packages for the vanilla Linux kernel and the linux-firmware collection; we have to use that repo if we want all functionality.

Of course having the firmware be user-hackable would be better, and it would be better if the framework laptop used parts with free firmware. Something for a next revision, hopefully.

### On firmware

As an aside, I think the official Free Software Foundation position on firmware is bad praxis. To recall, the idea is that if a device has embedded software (firmware) that can be updated, but that software is in a form that users can’t modify, then the system as a whole is not free software. This is technically correct but doesn’t logically imply that the right strategy for advancing free software is to forbid firmware blobs; you have a number of potential policy choices and you have to look at their expected results to evaluate which one is most in line with your goals.

Bright lines are useful, of course; I just think that with respect to free software, drawing that line around firmware is not interesting. To illustrate this point, I believe the current FSF position is that if you can run e.g. a USB ethernet adapter without installing non-free firmware, then it is kosher, otherwise it is haram. However many of these devices *have* firmware; it’s just that you aren’t updating it. So for example the the USB Ethernet adapter I got with my Dell system many years ago has firmware, therefore it has bugs, but I have never updated that firmware because that’s not how we roll. Or, on my old laptop, I never updated the CPU microcode, despite spectre and meltdown and all the rest.

“Firmware, but never updated” reminds me of the [wires around some New York neighborhoods](https://en.wikipedia.org/wiki/Eruv) that allow orthodox people to leave the house on Sabbath; useful if you are of a given community and enjoy the feeling of belonging, but I think even the faithful would see it as a hack. It is like how Richard Stallman wouldn’t use travel booking web sites because they had non-free JavaScript, but would happily call someone on the telephone to perform the booking for him, using those same sites. In that case, the net effect on the world of this particular bright line is negative: it does not advance free software in the least and only adds overhead. Privileging principle over praxis is generally a losing strategy.

### Installation

Firstly I had to turn off secure boot in the bios settings; it’s in “security”.

I wasn’t expecting wifi to work out of the box, but for some reason the upstream Guix install media was not able to configure the network via the Ethernet expansion card nor an external USB-C ethernet adapter that I had; stuck at the DHCP phase. So my initial installation attempt failed.

Then I realized that the [nonguix](https://gitlab.com/nonguix/nonguix) repository has installation media, which is the same as upstream but with the vanilla kernel and linux-firmware. So on another machine where I had Guix installed, I added the nonguix channel and built the installation media, via `guix system image -t iso9660 nongnu/system/install.scm`. That gave me a file that I could write to a USB stick.

Using that installation media, installing was a breeze.

However upon reboot, I found that I had no wifi and I was using software rendering; clearly, installation produced an OS config with the Guix kernel instead of upstream Linux. Happily, at this point the ethernet expansion card was able to work, so connect to wired ethernet, open `/etc/config.scm`, add the needed lines as described in the `operating-system` part of [the nonguix README](https://gitlab.com/nonguix/nonguix), reconfigure, and reboot. Building Linux takes a little less than an hour on this machine.

### Fractional scaling

At that point you have wifi and graphics drivers. I use GNOME, and things seem to work. However the screen defaults to 200% resolution, which makes everything really big. Crisp, pretty, but big. Really you would like something in between? Or that the Framework ships a higher-resolution screen so that 200% would be a good scaling factor; this was the case with my old Dell XPS 13, and it worked well. Anyway with the Framework laptop, I wanted 150% scaling, and it seems these days that the way you have to do this is to use Wayland, which Guix does not yet enable by default.

So you go into `config.scm` again, and change where it says `%desktop-services` to be:

```
(modify-services %desktop-services
  (gdm-service-type config =>
    (gdm-configuration (inherit config) (wayland? #t))))

```

Then when you reboot you are in Wayland. Works fine, it seems. But then you have to go and enable an experimental mutter setting; install `dconf-editor`, run it, search for keys with “mutter” in the name, find the “experimental settings” key, tell it to not use the default setting, then click the box for “scale-monitor-framebuffer”.

Then! You can go into GNOME settings and get 125%, 150%, and so on. Great.

HOWEVER, and I hope this is a transient situation, there is a problem: in GNOME, applications that aren’t native Wayland apps don’t scale nicely. It’s like the app gets rendered to a texture at the original resolution, which then gets scaled up in a blurry way. There aren’t so many of these apps these days as most things have been ported to be Wayland-capable, Firefox included, but Emacs is one of them :( However however! If you install the `emacs-pgtk` package instead of `emacs`, it looks better. Not perfect, but good enough. So that’s where I am.

### Bugs

The laptop hangs on reboot due to [this bug](https://community.frame.work/t/tracking-gnu-guix-system-on-the-framework/22459/24), but that seems a minor issue at this point. There is an ongoing [tracker discussion](https://community.frame.work/t/tracking-gnu-guix-system-on-the-framework/22459) on the community forum; like other problems in that thread, I hope that this one resolves itself upstream in Linux over time.

### Other things?

I didn’t mention the funniest thing about this laptop: it comes in pieces that you have to put together :) I am not so great with hardware, but I had no problem. The build quality seems pretty good; not a MacBook Air, but then it’s also user-repairable, which is a big strong point. It has these funny extension cards that slot into the chassis, which I have found to be quite amusing.

I haven’t had the machine for long enough but it seems to work fine up to now: suspend, good battery use, not noisy (unless it’s compiling on all 16 threads), graphics, wifi, ethernet, good compilation speed. (I should give compiling LLVM a go; that’s a useful workload.) I don’t have bluetooth or the fingerprint reader working yet; I give it 25% odds that I get around to this during the lifetime of this laptop :)

Until next time, happy hacking!

## related articles

- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [on gnu and on hackers](/archives/2014/08/18/on-gnu-and-on-hackers)
- [hacking v8 with guix, bis](/archives/2024/03/26/hacking-v8-with-guix-bis)
- [here we go again](/archives/2021/03/25/here-we-go-again)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [state of the gnunion 2020](/archives/2020/02/09/state-of-the-gnunion-2020)

### 5 responses

1. Guillermo says:[16 February 2024 1:14 PM](#8b011d0962ce8dff17c45c48a154c1d377c6db3a)

   Oh my god, the scaling on Wayland is so painful. I went back to X11. I hear the latest versions of KDE fix the scaling of X11 apps in Wayland so they are not blurred.

   Congrats on making the wifi work. I was getting low speeds, and random disconnections (with about 20 seconds to reconnect), so my patience ran out and I got a AX210. Now I need to replace the antenna cables to get faster wifi speeds (from what I read on the forums).

   Thank you very much for sharing your experience.

2. Elia says:[16 February 2024 2:18 PM](#22628dec40a178278ffb499b69f3f439fe12fc51)

   Thanks for sharing this great writeup! I don’t think I’ll be installing Guix on a Framework anytime soon, but it was interesting to hear about your experience. Your build (hardware wise) is precisely the one I’m intending to buy though, so maybe I’ll give it a shot.

3. panuh says:[16 February 2024 2:37 PM](#4f585a1350467f40ef25b12d8eeca5505169aa0d)

   I have the same device, I’m very happy with it. But the BIOS in this particular case is terrible.

   It would be great to have the BIOS source code, given that the Indsyde approach to password security is beyond ridiculous:

   When setting up a password for BIOS access, you are asked to choose one that contains capital and lowercase letters, a number and a special character. The password may not consist of more than 10 character - so no passphrases.

   As if that weren’t enough, there is some obscure password expiry date that can’t be seen or set anywhere. The bios will simply decide at some point that it’s time for you to change the password immediately.

   When you do that, you will notice that not only aren’t you allowed to use the same password again, the BIOS also remembers a number of past passwords that won’t be allowed either. So no memorizing two alternating ‘strong’ 10 digit passwords either.

   It’s a mess... There is a community thread that doesn’t give me much hope: https://community.frame.work/t/complexity-rules-for-bios-password-why-moved/40497

4. Juhani Viheräkoski says:[17 February 2024 5:56 PM](#46d65aebb21157e298e4ae7bc4cef64d1111572f)

   Speaking of Stallman, the telecom network he uses contain proprietary software.

5. Aliaksei says:[18 February 2024 6:59 PM](#ffdee25ab35690e439d130755f253d9ee1293a39)

   I have experience with first generation of Framework laptop. And I also had concerns about hidpi thingy. It was my first hidpi machine.

   My finding is that just letting X do the work, and having it set DPI to it’s native value (I think it ends up at around 140 dpi) just works. In your case since we’re talking smaller 13 inch diagonal, you’ll get higher dpi, but it’ll probably still just work.

   Wayland for some reason chose to “fix” DPI to 75 and instead forces us into tweakable scaling factor. Which for native wayland frameworks (I checked GTK) ends up working the same (it gets 75 and multiplies by that factor). But xwayland gets broken. Maybe it is better now, but I found that it gets fixed and wrong 75 dpi and then scaling thing ends up doing raster image scaling which is wrong. So I am sticking to X for now.

   IMHO this is one of those cases where “new and shiny” also ends up being super wrong (imagine, say 10 years from now and DPIs into high hundreds, we’ll be all running wayland with scale factor 10x ?!?)

   Interestingly, Chrome OS thingy does wayland and offers “linux in the VM” with the facilities to expose outer wayland-based input and display to that VM and X stuff just works there too. xdpyinfo from inside that VM shows me plausibly-looking 153 dpi. So it can be made to work.

Comments are closed.
