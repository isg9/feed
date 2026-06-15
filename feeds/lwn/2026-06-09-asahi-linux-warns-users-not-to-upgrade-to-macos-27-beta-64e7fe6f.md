---
title: Asahi Linux warns users not to upgrade to macOS 27 beta
url: https://lwn.net/Articles/1077209/
published: "2026-06-09T14:30:25Z"
feed: lwn
guid: https://lwn.net/Articles/1077209/
---

# Asahi Linux warns users not to upgrade to macOS 27 beta

The [Asahi Linux](https://asahilinux.org/) project, which brings Linux support to Apple Arm-based Macs, has [warned
its users](https://social.treehouse.systems/@AsahiLinux/116719749555082847) not to upgrade to the macOS 27 "Golden Gate" beta.

> Apple has changed how the boot picker and Startup Disk applications detect valid OS boot volumes. When using either from macOS 27, your Asahi partition will not be visible! We believe this to be a bug, and have filed a report (FB22994760).
>
> If you have already upgraded to the beta and noticed that your Asahi partition has disappeared, do not stress. Your Asahi partition is still there, and you have not lost any data.

The Asahi Linux installer has been patched to prevent use with macOS 27 for now, but any users already bitten by the change will need to use macOS 26 to restore access to Asahi Linux.
