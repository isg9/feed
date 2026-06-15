---
title: Silicon Drift Detector Energy Dispersive Spectroscopy (SDD EDS/EDX) Setup
url: https://sam.zeloof.xyz/eds/
published: "2018-01-02T23:13:40Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1430
---

# Silicon Drift Detector Energy Dispersive Spectroscopy (SDD EDS/EDX) Setup

Instillation and setup of a Silicon Drift Detector EDS system. A SDD is a modern detector for [Energy-dispersive X-ray spectroscopy](https://en.wikipedia.org/wiki/Energy-dispersive_X-ray_spectroscopy) that mounts onto an Electron Microscope and allows the user to analyze the composition of a sample. When electrons from the SEM hit the sample, it emits secondary electrons (along with many other particles and energies) which are used by the SEM to produce the image you see on screen and X-rays which are detected by the SDD to produce EDS information. The X-rays have a specific energy which is characteristic of the material and equal to the energy difference between excited and ground state electron orbitals in the atom.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/1.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/1.jpg)

SDD

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/2-e1515116317498.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/2-e1515116317498.jpg)

SDD

The SDD (30mm2 in my case) quantizes the energies of impinging X-rays and amplifies them to be read by a computer. As compared to traditional Si(Li) detectors, SDDs are faster, have higher resolution, and do not require Liquid Nitrogen cooling.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2389-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2389.jpg)

EDS flange

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2390-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2390.jpg)

Detector prep

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2392-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2392.jpg)

Detector prep

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/D7V7582-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/D7V7582.jpg)

Mounted

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2397-e1516977413211-300x202.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2397-e1516977413211.jpg)

Needs a chamberscope…

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2410-e1516977433505-300x204.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/IMG_2410-e1516977433505.jpg)

Peltier controller/PSU

The detector is mounted to the chamber at a 35 **°** angle which makes it most convenient for use at a WD of 25mm. When using the SEM at lower working distances, the SDD probe is retracted out of the chamber as to avoid collision with the sample stage. The detector is chilled to -30 **°** C with Peltier modules.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/20171230T24440-PM-768x1024.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/20171230T24440-PM.jpg)

IR

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/20171230T24451-PM-768x1024.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/20171230T24451-PM.jpg)

IR

The detector has a built in preamplifier. The output is fed to a pulse processor containing an FPGA which connects to a laptop via ethernet.With the SDD, EDS spectra can be acquired in less than a few minutes at high beam currents.

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/SS304.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/SS304.jpg)

304 SS

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/18-8-nut.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/18-8-nut.jpg)

18/8 SS

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/TI-GRADE-4-6.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/TI-GRADE-4-6.jpg)

Ti grade 6-4

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/LESKER-AL.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/LESKER-AL.jpg)

Lesker Al evaporation source

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/LESKER-CU.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/LESKER-CU.jpg)

Lesker Cu evaporation source

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/CU-ON-WAFER.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/CU-ON-WAFER.jpg)

Cu sputter/implantation on Si

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/WireMishMash.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/WireMishMash.jpg)

Unknown alloy

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/MountingScrew.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/MountingScrew.jpg)

SS (304?) Screw

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/01/FINGERPTINT.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/01/FINGERPTINT.jpg)

Fingerprint (S/Cl) residue

(Click on image to enlarge)

Thanks to Pulsetor LLC, Rick Mott, Jeff Thompson, David Bono, and John Guerard for their extreme generosity and help with the detector.
