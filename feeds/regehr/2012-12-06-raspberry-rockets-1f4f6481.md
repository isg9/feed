---
title: Raspberry Rockets
url: https://blog.regehr.org/archives/851
published: "2012-12-06T21:27:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=851
---

# Raspberry Rockets

One of the things I most enjoy about teaching embedded systems is that the students show up with a very diverse set of skills. Some are straight-up CS, meaning they can hack but probably are intimidated by a breadboard, logic analyzer, or UART. Others are EE, meaning that they can design a noise-free circuit or lay out a PCB, but probably can’t write an advanced data structure. Even the computer engineering students (at Utah this isn’t a department but rather a degree program that spans departments), who know both hardware and software, have plenty to learn.

The projects in my class are always done on some embedded development board; I’ve used four or five different ones. This semester we used the Raspberry Pi, which has a very different feel than all previous boards we’ve used because you can just shell into the thing and run emacs and gcc.

This fall I did something a bit differently than usual, which was to structure all of the programming projects around a single specification taken from [Knight and Leveson’s 1986 paper](http://sunnyday.mit.edu/papers/nver-tse.pdf) about n-version programming. The specification is for a (fake) launch interceptor system that takes a collection of parameters and a collection of points from a radar, applies a bunch of rules, and outputs a launch / no-launch decision for a missile interceptor. It’s a good fit for class projects because it’s generally pretty straightforward code but has a few wrinkles that most people (including me) aren’t going to get right on the first try. The students divided into groups and implemented my [slightly adapted version of the specification](http://www.eng.utah.edu/~cs5785/lab/decide.pdf), then we did several rounds of testing and fixing. This all ended up eating up a lot of the semester and we didn’t end up having time to use the nice little fuzzer that I built. For another assignment I asked each group to compute the worst-case stack memory usage of their code.

For the final project, with just a few weeks left in the semester, we put the whole class together on a project where multiple boards cooperated to make launch decisions. First, a master node, running Linux, loaded launch interceptor conditions and sent them to a couple of subordinate nodes using [SPI](http://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus). Although good SPI master drivers are available for the RPi, we were unable to use the BCM2835’s SPI device in slave mode since the appropriate pins are not exposed to the external connectors (also the SoC is a BGA so we can’t “cheat” and get access to the pins a different way). Therefore, a group implemented a bit-banged SPI slave for the subordinate nodes. For fun, and to avoid problems with the tight timing requirements for bit-banged SPI, these nodes dropped Linux and ran on bare metal. When a subordinate node received a collection of launch interceptor data it computed the launch or no-launch decision and then provided the answer to the master node when queried. The master, then, would only make a launch decision when both subordinate nodes said that a launch should happen, at which point it sent a signal to a high-power relay that sent a generous amount of current to the rocket igniter.

Since the students did an awesome job getting all of this working, we spent the last day of classes launching rockets in a field at the edge of campus; pictures follow. The system shown in the pictures actually includes two different implementations of the subordinate node: one on RPi and the other on a Maple board, which uses an ARM Cortex-M3 chip. I like this modification since it’s more in the spirit of the original n-version paper anyway.

\[nggallery id=65\]
