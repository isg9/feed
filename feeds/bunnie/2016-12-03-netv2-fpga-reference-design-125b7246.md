---
title: NeTV2 FPGA Reference Design
url: https://www.bunniestudios.com/blog/2016/netv2-fpga-reference-design/
published: "2016-12-03T07:17:17Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=4866
---

# NeTV2 FPGA Reference Design

A complex system like NeTV2 consists of several layers of design. About a month ago, we pushed out the [PCB design](https://alphamaxmedia.com/w/index.php?title=NeTV2#Source_Links). But a PCB design alone does not a product make: there’s an [FPGA design](https://github.com/bunnie/netv2-fpga-basic-overlay), [firmware for the on-board MCU](https://github.com/xobs/chibios-netvcr), [host drivers](https://github.com/xobs/netv2-kms), host application code, and ultimately layers in the cloud and beyond. We’re slowly working our way from the bottom up, assembling and validating the full system stack. In this post, we’ll talk briefly about [the FPGA design](https://github.com/bunnie/netv2-fpga-basic-overlay).

This design targets an Artix-7 XC7A50TCSG325-2 FPGA. As such, I opted to use Xilinx’s native Vivado design flow, which is [free to download and use](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2016-1.html), but not open source. One of Vivado’s more interesting features is a hybrid schematic/TCL design flow. The designs themselves are stored as an XML file, and dynamically rendered into a schematic. The schematic itself can then be updated and modified by using either the GUI or TCL commands. This hybrid flow strikes a unique balance between the simplicity and intuitiveness of designing with a schematic, and the power of text-based scripting.

[![](http://bunniefoo.com/netv2/netv2-dec16-sch_sm.png)](http://bunniefoo.com/netv2/netv2-dec16-sch.png)

*Above: top-level schematic diagram of the NeTV2 FPGA reference design as rendered by the Vivado tools*

However, the main motivation to use Vivado is not the design entry methodology per se. Rather, it is Vivado’s tight integration with the [AXI](https://en.wikipedia.org/wiki/Advanced_Microcontroller_Bus_Architecture#Advanced_eXtensible_Interface_.28AXI.29) IP bus standard. Vivado can infer AXI bus widths, address space mappings, and interconnect fabric topology based on the types of blocks that are being strung together. The GUI provides some mechanisms to tune parameters such as performance vs. area, but it’s largely automatic and does the right thing. Being able to mix and match IP blocks with such ease can save months of design effort. However, the main downside of using Vivado’s native IP blocks is they are area-inefficient; for example, the memory-mapped PCI express block includes an area-intensive slave interface which is synthesized, placed, and routed — even if the interface is totally unused. Fortunately many of the IP blocks compile into editable verilog or VHDL, and in the case of the PCI express block the slave interface can be manually excised after block generation, but prior to synthesis, reclaiming the logic area of that unused interface.

Using Vivado, I’m able to integrate a PCI-express interface, AXI memory crossbar, and DDR3 memory controller with just a few minutes of effort. With similar ease, I’ve added in some internal AXI-mapped GPIO pins to provide memory-mapped I/O within the FPGA, along with a video DMA master which can format data from the DDR3 memory and stream it out as raster-synchronous RGB pixel data. All told, after about fifteen minutes of schematic design effort I’m positioned to focus on coding my application, e.g. the HDMI decode/encode, HDCP encipher, key extraction, and chroma key blender.

Below is the “hierarchical” view of this NeTV2 FPGA design. About 75% of the resources are devoted to the Vivado IP blocks, and about 25% to the custom NeTV application logic; altogether, the design uses about 72% of the XC7A50T FPGA’s LUT resources. A full-custom implementation of the Vivado IP blocks would save a significant amount of area, as well as be more FOSS-friendly, but it would also take months to implement an equivalent level of functionality.

[![](http://bunniefoo.com/netv2/netv2-dec16-heir_sm.png)](http://bunniefoo.com/netv2/netv2-dec16-heir.png)

Significantly, the FPGA reference design shared here implements only the “basic” NeTV chroma-key based blending functionality, as [previously disclosed here](http://www.bunniestudios.com/blog/?p=2117). Although we would like to deploy more advanced features such as alpha blending, I’m unable to share any progress because this operation is generally prohibited under Section 1201 of the DMCA. With the help of the EFF, I’m suing the US government for the right to disclose and share these developments with the general public, but until then, my right to express these ideas is chilled by Section 1201.
