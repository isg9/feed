---
title: 'Make: Article on Novena'
url: https://www.bunniestudios.com/blog/2014/make-article-on-novena/
published: "2014-01-09T03:23:48Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=3597
---

# Make: Article on Novena

Recently, the Make: blog [ran an article](http://makezine.com/magazine/building-an-open-source-laptop/) on our laptop project, Novena. You can now follow [@novenakosagi](https://twitter.com/intent/user?screen_name=novenakosagi) for updates on the project. I’d also like to reiterate here that the photos shown in the article are just an early prototype, and the final forms of the machine are going to be different — quite different — from what’s shown.

Below is a copy of the article text for your convenient reading. And, as a reminder, specs and source files can be [downloaded at our wiki](http://www.kosagi.com/w/index.php?title=Novena_Main_Page).

**Building an Open Source Laptop**

[![](http://bunniefoo.com/novena/pvt_shoot/novena_full_sm.jpg)](http://bunniefoo.com/novena/pvt_shoot/novena_full.jpg)

About a year and a half ago, I engaged on an admittedly quixotic project to build my own laptop. By I, I mean we, namely [Sean “xobs” Cross](http://xoblo.gs) and me, [bunnie](http://bunniestudios.com). Building your own laptop makes about as much sense as retrofitting a Honda Civic with a 1000hp motor, but the lack of practicality never stopped the latter activity, nor ours.

My primary goal in building a laptop was to build something I would use every day. I had previously spent several years at chumby building hardware platforms that I’m ashamed to admit I rarely used. My parents and siblings loved those little boxes, but they weren’t powerful enough for a geek like me. I try to allocate my discretionary funds towards things based on how often I use them. Hence, I have a nice bed, as I spend a third of my life in it. The other two thirds of my life is spent tapping at a laptop (I refuse to downgrade to a phone or tablet as my primary platform), and so when picking a thing to build that I can use every day, a laptop is a good candidate.

[![SONY DSC](http://bunniefoo.com/novena/pvt_shoot/novena_atwork_sm.jpg)](http://makezineblog.files.wordpress.com/2014/01/novena_atwork.jpg)*I’m always behind a keyboard!*

The project was also motivated by my desire to learn all things hardware. Before this project, I had never designed with Gigabit Ethernet (RGMII), SATA, PCI-express, DDR3, gas gauges, eDP, or even a power converter capable of handling 35 watts – my typical power envelope is under 10 watts, so I was always able to get away with converters that had integrated switches. Building my own laptop would be a great way for me to stretch my legs a bit without the cost and schedule constraints normally associated with commercial projects.

The final bit of motivation is my passion for Open hardware. I’m a big fan of opening up the blueprints for the hardware you run – if you can’t Hack it, you don’t Own it.

Back when I started the project, it was me and a few hard core Open ecosystem enthusiasts pushing this point, but Edward Snowden changed the world with revelations that the NSA has in fact taken advantage of the black-box nature of the closed hardware ecosystem to implement spying measures ( [“good news, we weren’t crazy paranoids after all”](http://www.spiegel.de/international/world/catalog-reveals-nsa-has-back-doors-for-numerous-devices-a-940994.html)).

Our Novena Project is of course still vulnerable to techniques such as [silicon poisoning](http://people.umass.edu/gbecker/BeckerChes13.pdf), but at least it pushes openness and disclosure down a layer, which is tangible progress in the right direction.

While these heady principles are great for motivating the journey, actual execution needs a set of focused requirements. And so, the above principles boiled down to the following requirements for the design:

- All the components should have a reasonably complete set of NDA-free documentation. This single requirement alone culled many choices. For example, Freescale is the only SoC vendor in this performance class where you can simply go to their website, click a link, and download a mostly complete [6,000-page programming manual](http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q&fpsp=1&tab=Documentation_Tab). It’s a ballsy move on their part and I commend them for the effort.
- Low cost is not an objective. I’m not looking to build a crippled platform based on some entry-level single-core SoC just so I can compete price-wise with the likes of Broadcom’s non-profit Raspberry Pi platform.
- On the other hand, I can’t spec in unicorn hair, although I come close to that by making the outer case from genuine leather (I love that my laptop smells of leather when it runs). All the chips are ideally available off the shelf from distributors like Digi-Key and have at least a five year production lifetime.
- Batteries are based off of cheap and commonly available packs used in RC hobby circles, enabling users to make the choice between battery pack size, runtime, and mass. This makes answering the question of “what’s the battery life” a bit hard to answer – it’s really up to you – although one planned scenario is the trans-Siberian railroad trek, which is a week-long trip with no power outlets.
- The display should also be user-configurable. The US supply chain is weak when it comes to raw high-end LCD panels, and also to address the aforementioned trans-Siberian scenario, we’d need the ability to drive a low-power display like a Pixel Qi, but not make it a permanent choice. So, I designed the main board to work with a cheap LCD adapter board for maximum flexibility.
- No binary blobs should be required to boot and operate the system for the scenarios I care about. This one is a bit tricky, as it heavily limits the wifi card selection, I don’t use the GPU, and I rely on software-only decoders for video. But overall, the bet paid off; the laptop is still very usable in a binary-blob free state. We prepared and [gave a talk recently at 30C3](http://www.youtube.com/watch?v=r3GDPwIuRKI) using only the laptops.
- The physical design should be accessible – no need to remove a dozen screws just to pull off the keyboard. This design requires removing just two screws.
- The design doesn’t have to be particularly thin or light; I’d be happy if it was on par with the 3cm-thick Thinkpads or Inspirons I would use back in the mid 2000’s.
- The machine must be useful as a hardware hacking platform. This drives the rather unique inclusion of an FPGA into the mainboard.
- The machine must be useful as a security hacking platform. This drives the other unusual inclusion of two Ethernet interfaces, a USB OTG port, and the addition of 256 MiB DDR3 RAM and a high-speed expansion connector off of the FPGA.
- The machine must be able to build its own firmware from source. This drives certain minimum performance specs and mandates the inclusion of a SATA interface for running off of an SSD.

After over a year and a half of hard work, I’m happy to say our machines are in a usable form. The motherboards are very reliable, the display is a 13” 2560×1700 (239ppi) LED-backlit panel, and the cases have an endoskeleton made of 5052 and 7075 aluminum alloys, an exterior wrapping of genuine leather, an interior laminate of paper (I also love books and [papercraft](http://chibitronics.com)), and cosmetic panels 3D printed on a [Form 1](http://formlabs.com/). The design is no Thinkpad Carbon X1, but they’ve held together through a couple of rough international trips, and we use our machines almost every day.

[![](http://bunniefoo.com/novena/pvt_shoot/novena_lab_sm.jpg)](http://makezineblog.files.wordpress.com/2014/01/novena_lab.jpg)*Laptop parked in front of the Form1 3D printer used to make its body panels.*

I was surprised to find the laptop was well-received by hackers, given its homebrew appearance, relatively meager specs and high price. The positive response has encouraged us to plan a crowd funding campaign around a substantially simplified (think “all in one PC” with a battery) case design. We think it may be reasonable to kick off the campaign shortly after Chinese New Year, maybe late February or March. Follow [@novenakosagi](https://twitter.com/intent/user?screen_name=novenakosagi) for updates on our progress!

[![](http://bunniefoo.com/novena/pvt_shoot/novena_edge_sm.jpg)](http://makezineblog.files.wordpress.com/2014/01/novena_edge.jpg)

*The first two prototypes are wrapped in red sheepskin leather, and green pig suede leather.*

[![](http://bunniefoo.com/novena/pvt_shoot/novena_quarterview_sm.jpg)](http://makezineblog.files.wordpress.com/2014/01/novena_quarterview.jpg)

*Detail view of the business half of the laptop.*
