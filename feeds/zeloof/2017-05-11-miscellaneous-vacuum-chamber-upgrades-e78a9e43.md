---
title: Miscellaneous Vacuum Chamber Upgrades
url: https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/
published: "2017-05-11T02:48:48Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=711
---

# Miscellaneous Vacuum Chamber Upgrades

Miscellaneous Upgrades – Vent Valve/TC Gauge CF Tee Construction, Gas Cylinder Holder Fabrication, and Ion Gauge Controller Analog Output Hack

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0272-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0272/)

ConFlat Components

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0275-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0275/)

Completed Tee Assembly

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0276-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0276/)

Vent Valve and TC Gauge

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0218-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0218/)

Temporary MFC Hookup

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0229-e1494470765612-300x256.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0229/)

Gas Cylinder Holder

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0252-e1494470781870-300x279.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0252/)

Gas Cylinder Holder

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0295-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0295/)

Gas Cylinder Holder

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0288-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0288/)

Output From Quad Opamp

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0290-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0290/)

3.5mm Connector

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/05/IMG_0289-300x225.jpg)](https://sam.zeloof.xyz/miscellaneous-upgrades-vent-valvetc-gauge-cf-manifold-construction-gas-bottle-holder-fabrication-and-ion-gauge-controller-analog-output-hack/img_0289/)

3.5mm Connector

My HP (Granville Phillips)  hot cathode ionization gauge controller (59822b) had provisions for an analog output however the optional connector was not installed on the back panel and the connection points on the main PCB of the controller did not show any voltage deflection proportional to pressure so I took the pressure output from one of the stages in the electrometer which happened to be one of the outputs of a quad opamp chip. This output is fed to a NI DAQ USB6009 and displayed in a LabView VI so that the pressure of the chamber while pumping down can be held constant when requested by PID feedback loop with one or more of the mass flow controllers.

A pressurized gas cylinder holder was designed and fabricated by Diversatech Manufacturing, INC ( [www.diversatech.com](http://www.diversatech.com)) for safer operation of my vacuum chamber when using gasses such as Oxygen, Nitrogen, or Argon for processes such as sputtering or plasma cleaning.
