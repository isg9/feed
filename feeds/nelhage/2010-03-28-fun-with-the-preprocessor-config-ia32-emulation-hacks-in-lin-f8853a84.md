---
title: 'Fun with the preprocessor: CONFIG_IA32_EMULATION hacks in Linux'
url: https://blog.nelhage.com/2010/03/config_ia32_emulation_hacks/
published: "2010-03-28T20:07:43Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/03/config_ia32_emulation_hacks/
---

# Fun with the preprocessor: CONFIG_IA32_EMULATION hacks in Linux

About two months ago, Linux saw CVE-2010-0307, which was a trival denial-of-service attack that could crash essentially any 64-bit Linux machine with 32-bit compatibility enabled. LWN has an excellent writeup of the bug, which turns out to be a subtle error related to the details of the execve system call and with 32-bit compatibility mode. While dealing with this patch for Ksplice, I ended up reading an awful lot of the code in Linux that deals with handling 32-bit processes on 64-bit machines.
