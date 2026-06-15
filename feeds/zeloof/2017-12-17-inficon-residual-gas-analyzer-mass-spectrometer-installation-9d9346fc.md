---
title: Inficon Residual Gas Analyzer (Mass Spectrometer) Installation
url: https://sam.zeloof.xyz/inficon-residual-gas-analyzer-mass-spectrometer-installation/
published: "2017-12-17T02:37:50Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1381
---

# Inficon Residual Gas Analyzer (Mass Spectrometer) Installation

An Inficon Transpector2 HPR RGA was purchased from eBay and refurbished. I will use it for leak checking and ion selection experiments. Thanks to Aota Vac and Inficon for software assistance!

Put simply, an RGA is a small mass spectrometer that allows the operator to see the composition of the residual gases left in the vacuum chamber after pumping. [Basic theory here.](https://en.wikipedia.org/wiki/Residual_gas_analyzer)

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7369-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7369.jpg)

RGA

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7370-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7370.jpg)

Ion source

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7379-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/09/D7V7379.jpg)

RGA

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/12/IMG_2298-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/12/IMG_2298.jpg)

Serial debug

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/12/RGA-spectra-300x185.png)](https://sam.zeloof.xyz/wp-content/uploads/2017/12/RGA-spectra.png)

Gas Spectra

[![](https://sam.zeloof.xyz/wp-content/uploads/2017/12/IMG_2603-e1516936238816-300x180.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2017/12/IMG_2603-e1516936238816.jpg)

Base pressure

This RGA is equipped with an electron multiplier so it operates at a maximum pressure of 1e-4Torr and is good down to about 1e-15Torr which is not a limit I will be reaching anytime soon. The spectra shown above indicates a large peak at an AMU of 18 which is H2O. This means that the chamber needs to be baked out to remove water embedded in chamber walls and surfaces until the highest peaks are N2 and O2.

A spectrum analyzer and near field probe was used to determine the main operating frequency of the RGA to approximately 3.02mhz. The RGA is also equipped with an Ar calibration source. Serial logs indicate that it was manufactured in 2007 and has been run thousands of hours past recommended service/filament replacement and the measured total pressure in the chamber was over an order of magnitude too high when compared to a hot-cathode ion gauge, but after calibration everything seems to be in spec.
