---
title: 'IC Design Tools: From Verilog to Masks'
url: https://sam.zeloof.xyz/ic-design-tools-from-verilog-to-mask/
published: "2018-02-10T03:51:33Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1527
---

# IC Design Tools: From Verilog to Masks

There are a number of open source layout and design tools for ICs however here I will just focus on [Magic VLSI](http://opencircuitdesign.com/magic/index.html). Below are the steps to take a design from Verilog through synthesis and layout to the physical mask that is used for fabrication, via the [Qflow](http://opencircuitdesign.com/qflow/) digital synthesis flow.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.55.27-PM-1024x520.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.55.27-PM.png)

Design flow

The Verilog digital design for this example is a UART interface from [www.asic-world.com](http://www.asic-world.com/code/hdl_models/uart.v) and is synthesized/routed with the standard SCMOS rules, however a custom .tech file can be specified containing process details and design rules for DRC.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.05-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.05-PM.png)

Execute command

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.17-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.17-PM.png)

Place/route

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.28-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.09.28-PM.png)

Place/route

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.11.25-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.11.25-PM.png)

Layout

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.11.51-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.11.51-PM.png)

Layout

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.13.22-PM-1024x640.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-02-09-at-10.13.22-PM.png)

GDS Mask

Once Qflow executes successfully, the design can be viewed in Magic after loading the appropriate cells and a GDS file is generated. This is opened in [OwlVision GDSII Viewer](http://www.owlvision.org/) or similar which can be used to generate the individual mask image files for each layer (active, poly, metal, etc.)

Using the same tools mentioned above, I also designed a simple PMOS chip to test my process. It is scalable to any reasonable size and contains 2 differential amplifier circuits (seen on right and middle) and a number of diodes/resistors and other test features on the left.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-03-02-at-9.58.40-PM-300x188.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-03-02-at-9.58.40-PM.png)

Magic layout

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-03-02-at-9.56.27-PM-300x188.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/Screen-Shot-2018-03-02-at-9.56.27-PM.png)

Mask generation

The design requires 4 masks for fabrication: active/diffusion, gate oxide, contact, and metal.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/1active-300x169.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/1active.png)

Active

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/2gate-300x169.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/2gate.png)

Gate

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/3contact-300x169.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/3contact.png)

Contact

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/4metal-300x169.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/4metal.png)

Metal

Fiducials should be added for subsequent layer alignment. Note: Grateful Dead bears are necessary for the circuit to function correctly.
