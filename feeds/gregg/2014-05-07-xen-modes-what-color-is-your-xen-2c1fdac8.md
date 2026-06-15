---
title: 'Xen Modes: What Color Is Your Xen?'
url: https://www.brendangregg.com/blog/2014-05-07/what-color-is-your-xen.html
published: "2014-05-07T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-05-07/what-color-is-your-xen.html
---

# Xen Modes: What Color Is Your Xen?

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

# [Brendan Gregg's Blog](/blog/index.html)

[home](/blog/index.html)

## Xen Modes: What Color Is Your Xen?

07 May 2014

What's faster: **PV**, **HVM**, **HVM with PV drivers**, **PVHVM**, or **PVH**? Cloud computing providers using [Xen](http://www.xenproject.org) can offer different virtualization "modes", based on paravirtualization (PV), hardware virtual machine (HVM), or a hybrid of them. As a customer, you may be required to choose one of these. So, which one?

Rackspace offers [PV or PVHVM](http://www.rackspace.com/knowledge_center/article/choosing-a-virtualization-mode-pv-versus-pvhvm), and Amazon EC2 offers [PV or HVM](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html). I'd summarize the choice today as:

> The best performance is the latest PV/HVM hybrid mode that Xen supports.

For most places right now, PVHVM should deliver the best performance. On AWS EC2, if you create a "HVM" instance, and Linux supports it, you get PVHVM. In the future (Xen 4.4+) the fastest mode will be [PVH](http://blog.xen.org/index.php/2014/01/31/linux-3-14-and-pvh/).

Not picking the latest hybrid mode should only make sense for some corner cases. You can test this yourself, as your mileage and requirements may vary.

Now, if you search for [PV vs HVM](http://lmgtfy.com/?q=PV+vs+HVM), the more you read the more confused you may become. My first hit was a post from 2012 discussing EC2 choices, concluding with:

> "If you want a good performance for Linux/Unix then use PV"

You might still be thinking the same, and a while ago this was right, before many changes in the world of Xen (before PV drivers on HVM, PVHVM, and PVH). At least the post from 2012 has a date on it (like this one).

What makes this even more confusing is that newer versions of the hybrid model have been named as separate "modes". In this post, I'll summarize the current virtualization spectrum model, and suggest a slightly different explanation: instead of focusing on the choice of modes, I'll show them as an evolution of versions.

Note that I'm not a Xen developer, so I'm not an official source of Truth for this: I'm a performance architect for a large EC2 user, and I'm hoping to contribute to our collective understanding of Xen choices.

## The Virtualization Spectrum

[![](/blog/images/2014/xen-virt-spectrum-crop-800.png)](/blog/images/2014/xen-virt-spectrum-crop-800.png)

In the articles [The Ends of the Spectrum](http://blog.xen.org/index.php/2012/10/23/the-paravirtualization-spectrum-part-1-the-ends-of-the-spectrum/) and [From Poles to a Spectrum](http://blog.xen.org/index.php/2012/10/31/the-paravirtualization-spectrum-part-2-from-poles-to-a-spectrum/), George Dunlap introduced the idea that there are no longer two modes of Xen virtualization – PV or HVM – but, rather, a spectrum of modes, where PV and HVM are the poles.

This is pictured in the right diagram [from Lars Kurth](http://www.slideshare.net/xen_com_mgr/linuxcon-eu-virtualization-in-the-cloud-featuring-xen-and-xcp/16), which lists different Xen virtualization modes as rows, and features as columns. Full HVM mode is the top row, full PV mode is the bottom, and hybrid modes are in-between. (This shows PVH as a Xen 4.3 feature; it's now Xen 4.4.)

The HVM and PV technologies provide their own performance benefits:

- **HVM**: A processor technology for accelerating CPU virtualization (privileged instructions, syscalls) and the MMU (page tables). This is supported by Intel (VT-x) and AMD (AMD-V).
- **PV**: A software technology where the guest kernel can use an accelerated interface for virtualized components, including disks and network interfaces, rather than emulating hardware.

Xen has been developing additional hybrid modes for the best of both words, where a HVM guest can use faster PV kernel parts, or vice versa. At first, PV disk and network drivers on HVM were possible. Then, interrupts and timers. Each of these moves reduced the need for slower emulation, such as via QEMU, improving performance. This is all summarized pretty well by the spectrum diagram.

You can find more detailed explanations of the modes in those [two](http://blog.xen.org/index.php/2012/10/23/the-paravirtualization-spectrum-part-1-the-ends-of-the-spectrum/) [articles](http://blog.xen.org/index.php/2012/10/31/the-paravirtualization-spectrum-part-2-from-poles-to-a-spectrum/). As George suggested, PVH is expected to be the best hybrid mode, and that other modes could eventually be dropped from the Linux kernel, simplifying it. And, if HVM existed (the Intel VT-x or AMD-V processor features) when Xen was written, PVH would probably have been the chosen mode.

## The Colors of Xen

So Xen today is something of a mess – the result of evolving software and hardware features, aiming to provide the best performance possible. And, as George noted in his first article, the terminology has been a little unclear, and has evolved over time.

The spectrum of modes is great way to make sense of this, but there are some loose ends. One is that a spectrum of modes can imply a choice when doesn't need to be one: if my instance supports PVHVM, should I study its pros and cons versus PV drivers over HVM? No, PVHVM is just the latest version of the same thing. So, instead of a spectrum of modes, is it more a spectrum of versions?

[![](/blog/images/2014/cosby-xen-modes.jpg)](/blog/images/2014/cosby-xen-modes.jpg)

There are other sources of confusion: George's [article](http://blog.xen.org/index.php/2012/10/31/the-paravirtualization-spectrum-part-2-from-poles-to-a-spectrum/) describes "PVHVM mode" and "PVH mode", then depicts them inside "HVM mode" and "PV mode" groups, which are labeled as "mode/domain" in the above figure. So "PVHVM *mode*" is a *mode*, but "PV *mode*" is a *domain* or group of modes?

His article also warns about PVHVM mode confusion:

> ("PVHVM" mode should not be confused with "PV-on-HVM" mode, which is a term sometimes used in the past for "fully virtualized with PV drivers".)

I understand this statement, however, many sources seem to still misuse the terms: the Xen wiki page on [PV on HVM](http://wiki.xen.org/wiki/PV_on_HVM) describes this as "also called PVHVM or PV-on-HVM drivers", which seems to contradict the warning. Also, what I think are the developer's slides on PVHVM are actually titled " [Linux PV on HVM](http://www.slideshare.net/xen_com_mgr/6-stefano-spvhvm)". Maybe I'm wrong and these are consistent, but I struggle to see how.

## Back to Poles

Here's a different way to think about the modes, if it helps. Lets go back to the poles model (PV and HVM), before the spectrum, and explain the current state of Xen in that context.

[![](/blog/images/2014/xen-mode-flow.png)](/blog/images/2014/xen-mode-flow.png)

On AWS EC2, you choose between PV and HVM, which is referring to the AMI virtualization and boot process. Once booted, each of these modes then may run other features, like PVHVM drivers. But focusing on just the AMI virtualization choice, there are only two modes to worry about, PV and HVM: the poles.

So, choosing between PV vs HVM:

- **PV**: Use this if the processor doesn't do HVM: eg, following the EC2 [instance type matrix](http://aws.amazon.com/amazon-linux-ami/instance-type-matrix/), where you will end up with a fully PV mode instance. If it does do HVM, and PVH is supported, this mode boots the best type of hybrid.
- **HVM**: Until PVH, choose this. Xen, and the guest kernel, are likely to support a number of enhancements which use PV code paths, so the booted instance will be a hybrid. The number of enhancements depends on the Xen and guest kernel versions: newer for more.

This selection of instances is pictured in the flowchart on the right (which I've updated since posting). It shows what can be selected to achieve the highest performance, in general. You may have a corner case that differs, due to different requirements.

PVHVM and PVH can be thought of as features and not modes. The choices are PV or HVM, which describes how the instance boots, and your choice may be influenced by the features available. Currently, on EC2, PVH is not available, but PVHVM is: so the best choice for Linux (in general) would be booting a "HVM" instance with PVHVM enabled (eg, setting CONFIG\_XEN\_PVHVM).

[![](/blog/images/2014/xen-colors.png)](/blog/images/2014/xen-colors.png)

I've redrawn the spectrum picture on the right to emphasize the two modes, and to group all mixed types as "Hybrid", showing that they are a progression of Xen versions, rather than a group of modes to choose from. Is this helpful?

If you don't find this an improvement, I'll try a different way to clarify what your Xen guest is doing in my next post, where I'll cover tools for feature detection.

## Enhanced Networking

While my spectrum diagram may be an improvement, I've left a loose end untied: [SR-IOV for enhanced networking](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html). If the instance type supports it, this uses hardware virtualization enhancements to improve performance and provide an interface better suited for virtualization. In some ways, this is an evolution of I/O controllers that is similar to what processors have done, with their own virtualization extensions.

SR-IOV, if you can enable it, should provide the best I/O performance. So perhaps the first feature column in the spectrum diagram should really show two options: P (paravirt) as yellow, and VH (SR-IOV) as green. Here's how it [looks](/blog/images/2014/xen-colors-sriov.png) (although I'm not sure all the hybrid types support SR-IOV).

## Conclusion

Nowadays, there are various hybrid Xen modes (or versions), which obsolete older advice that "PV is fastest". On AWS EC2 today, "HVM" mode (which supports PVHVM) should be fastest. In the future, it will be PVH. If you can, you can also enable SR-IOV to improve I/O performance further.

In this post I summarized the spectrum model of the modes, and described them a little differently beyond just PV and HVM. The number of Xen modes is a little confusing, and is a result of an evolution of software and hardware, and Xen development work to deliver the fastest guests they can.

A while ago I wrote a post on [the performance of Zones vs KVM vs Xen](/blog/2013-01-11/virtualization-performance-zones-kvm-xen.html), and for Xen I pictured a fully HVM instance with the QEMU I/O proxy, mentioning that paravirtualization drivers could improve I/O performance. That how I'd draw the digram today: as I/O should be using the faster PV drivers, even for [Windows](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/Upgrading_PV_drivers.html).

Despite the Xen improvements, Solaris Zones, or Linux containers, should still be the fastest possible virtualization option, allowing guests to run bare-metal everything. But there is still a road ahead for their widespread adoption: a topic for another post.

In my [next post](/blog/2014-05-09/xen-feature-detection.html), I show various tools you can use from your Linux Xen guest to detect which HVM and PV features are enabled.

*Update 2017: PVH never eventuated in AWS EC2, however, a whole new hypervisor did called Nitro. See my post: [AWS EC2 Virtualization 2017: Introducing Nitro](http://www.brendangregg.com/blog/2017-11-29/aws-ec2-virtualization-2017.html).*

## References and Resources

- George Dunlap's [The Ends of the Spectrum](http://blog.xen.org/index.php/2012/10/23/the-paravirtualization-spectrum-part-1-the-ends-of-the-spectrum/) and [From Poles to a Spectrum](http://blog.xen.org/index.php/2012/10/31/the-paravirtualization-spectrum-part-2-from-poles-to-a-spectrum/) are the best summary of all the modes, and how these have been developed (evolved) little by little.
- Lars Kurth's talk [Virtualization in the Cloud Featuring Xen](http://www.slideshare.net/xen_com_mgr/linuxcon-eu-virtualization-in-the-cloud-featuring-xen-and-xcp/16).
- Stefano Stabellini's [Linux PV on HVM](http://www.slideshare.net/xen_com_mgr/6-stefano-spvhvm), and especially the [video](http://vimeo.com/33056481), give the best insight into why the PVHVM features were developed, and the value they bring.
- The [Xen Linux PV on HVM drivers](http://wiki.xen.org/wiki/Xen_Linux_PV_on_HVM_drivers) shows more commands for studying Xen configuration.
- Also see the Xen wiki [PV on HVM](http://wiki.xen.org/wiki/PV_on_HVM) page.
- [Linux 3.14 and PVH](http://blog.xen.org/index.php/2014/01/31/linux-3-14-and-pvh/), describing how to get the development version working.
- See the [Xen Project 4.4](http://www.slideshare.net/xen_com_mgr/xen-project-4-4-features-and-futures) slides by Russell Pavlicek, Mar 27, 2014, for a recent summary.
- The AWS [EC2 instance types](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) does explain that "HVM" isn't fully "HVM".
- The AWS [glossary](http://docs.aws.amazon.com/general/latest/gr/glos-chap.html#H) also defines "HVM".
- Rackspace [choosing PV vs PVHVM](http://www.rackspace.com/knowledge_center/article/choosing-a-virtualization-mode-pv-versus-pvhvm).
- EC2 [enhanced nerworking (SR-IOV)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html).
- The first hit from google for [PV vs HVM](http://palakonda.org/2012/10/30/aws-virtualization-hvm-vs-paravirtualization/) (out of date).
- Moolah ecommerce [benchmarked](http://moolah-ecommerce.com/blog/36-2014/219-amazon-vs-rackspace-performance) the difference between PV and PVHVM, using *[UnixBench](http://www.brendangregg.com/blog/2014-05-02/compilers-love-messing-with-benchmarks.html)*?
- Xen Orchestra has also benchmarked [Debian PVHVM vs PV](https://xen-orchestra.com/debian-pvhvm-vs-pv/) using ... *UnixBench? again?? (flips desk.)*

Thanks to the AWS performance engineers and Boris Ostrovsky for helping me understand this better.

---

Click here for Disqus comments (ad supported).

*You are welcome to comment here, but I've been meaning to switch comment systems one day and I don't know yet if I can preserve existing comments (I'll try to find a way).*

[comments powered by Disqus](http://disqus.com)

---

Site Navigation

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

---

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

Copyright 2025 Brendan Gregg.

[About this blog](/blog/about.html)
