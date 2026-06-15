---
title: Inexpensive CPU Monster
url: https://blog.regehr.org/archives/1225
published: "2015-04-16T03:07:09Z"
feed: regehr
guid: http://blog.regehr.org/?p=1225
---

# Inexpensive CPU Monster

Rather than using the commercial cloud, my group tends to run day-to-day jobs on a tiny cluster of machines in my office and then to use [Emulab](https://www.emulab.net/) when a serious amount of compute power is required. Recently I upgraded some nodes and thought I’d share the specs for the new machines on the off chance this will save others some time:

processor[Intel Core i7-5820K](http://www.amazon.com/Intel-i7-5820K-Haswell-E-Processor-BX80648I75820K/dp/B00MMLXIKY/)$ 380CPU cooler[Cooler Master Hyper 212 EVO](http://www.amazon.com/Cooler-Master-Hyper-212-RR-212E-20PK-R2/dp/B005O65JXI/)$ 35mobo[MSI X99S SLI Plus](http://www.newegg.com/Product/Product.aspx?Item=N82E16813130796)$ 180RAM[CORSAIR Vengeance LPX 16GB (4 x 4GB) DDR4 SDRAM 2400](http://www.newegg.com/Product/Product.aspx?Item=N82E16820233720)$ 195SSD[SAMSUNG 850 Pro Series MZ-7KE256BW](http://www.newegg.com/Product/Product.aspx?Item=N82E16820147360)$ 140video[PNY Commercial Series VCG84DMS1D3SXPB-CG GeForce 8400](http://www.newegg.com/Product/Product.aspx?Item=N82E16814133454)$ 49case[Corsair Carbide Series 200R](http://www.newegg.com/Product/Product.aspx?Item=N82E16811139018)$ 60power supply[Antec BP550 Plus 550W](http://www.newegg.com/Product/Product.aspx?Item=N82E16817371016)$ 60

These machines are well-balanced for the kind of work I do, obviously YMMV. The total cost is about $1100. These have been working very well with Ubuntu 14.04. They do a full build of LLVM in about 18 minutes, as opposed to 28 minutes for my previously-fastest machines that were based on the i7-4770. I’d be interested to hear where other research groups get their compute power — everything in the cloud? A mix of local and cloud resources? This is an area where I always feel like there’s plenty of room for improvement.
