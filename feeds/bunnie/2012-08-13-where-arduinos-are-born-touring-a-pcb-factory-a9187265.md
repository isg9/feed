---
title: 'Where Arduinos are Born: Touring a PCB Factory'
url: https://www.bunniestudios.com/blog/2012/where-arduinos-are-born-touring-a-pcb-factory/
published: "2012-08-13T20:20:40Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=2407
---

# Where Arduinos are Born: Touring a PCB Factory

I recently had the pleasure and privilege of touring the factories that make Arduinos.

![](http://bunniestudios.com/blog/images/arduino_location.JPG)

Arduinos are made in Scarmagno, Italy, a small town near the Olivetti factories on the outskirts of Torino. All of the circuit board fabrication, board stuffing and distribution is handled out of that small town. I was really excited to see the factories, and I’d like to share some photos of them with you.

The highlight of my tour was “System Electronica”, the PCB factory which makes the Arduino PCBs.

![](http://bunniestudios.com/blog/images/arduino_pcbfab_factory.JPG)

One charming aspect of System Electronica is the owner has the factory painted to match the colors of the Italian flag. In this wide view of the factory floor, you can see some of that in the red and white posts down the length of the facility.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_wideview_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_wideview.jpg)

Arduinos start as huge sheets of virgin copper-on-FR4.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_blank_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_blank.jpg)

The sheets above are 1.6mm thick (the most common thickness for a PCB, which corresponds to 1/16th of an inch), and probably a meter on one side by about a meter and a half.

The first step in processing is to drill the holes: hole drilling is done even before patterning. This is because once the holes are drilled, you can use the holes to align the masks used to pattern the traces later on in the process. Hole drilling is also a dirty and messy process, which can damage circuit patterns.

[![](http://bunniestudios.com/blog/images/arduino_pcb_drill_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcb_drill.jpg)

Above is the CNC drilling head used to drill the boards. The blank copper sheets are actually stacked three-high so a single pass of the drill can produce three substrates.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_drillrack_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_drillrack.jpg)

Above is the drill rack used by the CNC drilling machine. If you’ve ever had to muck around with creating NC-drill files, you’ll probably have seen the term “drill rack” used somewhere. This is what it looks like.

Above is the drill in action. It’s encoded in Ogg Theora; so, sorry if you’re using IE or Safari and you don’t have the codec installed. An HD version of the video is also available [here](http://bunniestudios.com/blog/images/arduino_pcb_drill_hires.ogv). Flash is dead; long live HTML5! \[ *note: I turned off video pre-loading to save bandwidth by just editing the attribute tags. That would have been a pain in Flash. Yet another benefit of HTML5!*\]

Every hole in the board is mechanically drilled, which is why via count is such an important parameter in calculating the cost of a PCB. This particular drill is a relatively small one. I’ve seen massive drill decks in China that slave four or six drill heads together in a truck-sized machine, processing dozens of panels at the same time; they drill so fast and hard that the ground shakes, even from several meters away, with every via drilled.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_drilled_plated_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_drilled_plated.jpg)

Above is a stack of finished, drilled boards that have been cleaned and deburred, ready for the next step in the process.

The next step is to apply a photoresist to the board and expose a pattern. At System Electronica, this is done using a light box and a high-contrast film. Other technologies I’ve seen include direct laser imaging, where the pattern is put on using a raster-scanning laser. Direct laser scanners are more common in quick-turn prototype houses, and film imaging is more common in mass production houses.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_exposure_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_exposure.jpg)

Above is a PCB blank being mounted into a light box for exposure. Note that most photos in this post can be clicked to reveal a much higher resolution version.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_exposed_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_exposed.jpg)

Above is what the board looks like before and after exposure.

![](http://bunniestudios.com/blog/images/arduino_pcbfab_stripper.JPG)

The boards are then sent into a machine to be developed. In this case, the same machine is capable of developing both the photoresist and the soldermask.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_post_resist_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_post_resist.jpg)

Above is an image of a developed board. It’s one of my favorite photos out of the factory. Also, there is just something cool sounding about “Codice: Leonardo”.

After photo-processing and development, the boards go through a series of chemical baths that etch and plate the copper.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_plating_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_plating.jpg)

The movement of the boards through the chemical baths is fully automated; this is necessary because between certain steps oxygen can spoil a board in a matter of seconds, so the transfer between the baths needs to be fast. Also, the baths contain caustic and harmful chemicals to humans, so it’s much safer for a robot to do this work.

\[ [HD verison](http://bunniestudios.com/blog/images/arduino_pcb_bath_hires.ogv)\] As you can see, the boards swish gently back and forth in the baths as they are processed. Each bath has a different solution in it.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_preplate_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_preplate.jpg)

Once the boards are processed in this series of solutions, a dull, white plating (which I’m guessing is nickel) has developed on all the non-resist covered surfaces of the board, including the previously unplated vias and pads.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_postplate_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_postplate.jpg)

At this point the resist and unplated copper is stripped off, leaving just the raw FR-4 and the plated copper.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_brightcopper_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_brightcopper.jpg)

A final step of processing results in a bright copper finish. Note that the photo above isn’t of an Arduino board, as their boards weren’t going through the machine at the time when I was taking photos of it.

The boards are now ready to have soldermask and silkscreen applied. These are applied using a very similar process to the trace patterns, using a photomask and developer/stripper machine.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_post_soldermask_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_post_soldermask.jpg)

Above is photo showing a board with both soldermask and “silkscreen” developed. The silkscreen in this case is actually a second layer of soldermask. A very specific formulation of dry-film white soldermask was procured for team Arduino to create a sharp and good-looking layer that can resolve the intricate artwork found on Arduino boards. Other techniques I’ve seen for producing silkscreen layers include high resolution inkjet printing, which is better suited for quick-turn board houses, and of course the namesake squeegee-and-paint silkscreen process.

Once the boards have finished chemical processing, they receive a protective plating of solder through a hot-air solder leveling machine, and then move on to testing.

\[ [HD version](http://bunniestudios.com/blog/images/arduino_pcb_test_hires.ogv)\] Every board is 100% tested. This means that every trace has its continuity and resistance measured using a pair of flying probes. That’s a lot of probing, and the video above gives you an idea of how fast the probing is done. An alternative to flying head testing is to use clamshell testers, where a set of pogo pins are put into a fixture that can test the entire board with a single mechanical operation. However, clamshell fixtures are very labor-intensive to assemble and maintain, and require physical rewiring every time the gerbers are updated. So, in many cases flying probe testing can be cost-favorable and more flexible when compared to fixtured testing.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_stack1_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_stack1.jpg)

Above is a stack of near-finished PCB panels, ready for a final step of routing.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_routing_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_routing.jpg)

Before the boards can be shipped, the individual PCBs panels need to be routed so they can fit inside SMT machines. The boards are once again stacked up and batch-processed through a machine that uses a router bit to cut and release the board panels.

[![](http://bunniestudios.com/blog/images/arduino_pcbfab_result_sm.jpg)](http://bunniestudios.com/blog/images/arduino_pcbfab_result.jpg)

Finally, the boards are ready to ship on to the SMT facility. As you can see, the boards are panelized in a 2×6 format to make SMT processing more efficient.

![](http://bunniestudios.com/blog/images/wholelotta_arduinos.JPG)

Here’s a veritable stack of about 25,000 bare Arduino PCBs ready to leave the PCB factory, moving on to be stuffed, shipped and sold to Makers the world around!

Many thanks to the Officine Arduino team and Davide Gomba in particular for taking time out of their busy schedules, showing me around, and patiently waiting as I expressed my inner shutterbug and general love for all things hardware…
