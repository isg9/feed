---
title: Mapping Mountains on Exoplanets
url: https://blog.regehr.org/archives/1479
published: "2017-03-04T05:13:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=1479
---

# Mapping Mountains on Exoplanets

Snow in the mountains evolves in predictable ways that are strongly influenced by the terrain. For example, warmish snowstorms leave a characteristic snow line that looks very much like a contour on a topographic map:

![](../extra_files/mapping_exoplanets/snowline1.jpg) ( [source](https://www.flickr.com/photos/38063599@N00/2821080868/in/photostream/))

![](../extra_files/mapping_exoplanets/snowline2.jpg) ( [source](https://www.flickr.com/photos/68112440@N07/8640849282/))

On the other hand, when snow melts the effect is very different, and is often dominated by aspect. There pictures taken near my house a few days apart show more melting on south-facing slopes that receive more solar energy:

![](../extra_files/mapping_exoplanets/melt1.jpg)

![](../extra_files/mapping_exoplanets/melt2.jpg)

Effects like these are taken into account by a [snowpack model](https://www.fs.fed.us/rm/boise/publications/watershed/rmrs_1996_tarbotond001.pdf): a computer simulation designed to predict snowpack evolution in order to, for example, make predictions about avalanches or about spring runoff. Runoff is important not only because it can cause [flooding](https://www.youtube.com/watch?v=oxYSEvZsDAA) but also because areas near the mountains depend on the runoff for drinking and agricultural water. A snowpack model would typically take topographic information and snow depth information as inputs, but people have also done research into [driving snowpack models using weather information](http://www.the-cryosphere.net/5/1115/2011/tc-5-1115-2011.pdf); this is useful for areas where there are not a lot of snow depth measurement stations, such as the [Snotel](https://www.wcc.nrcs.usda.gov/snow/) network in the western USA.

This post asks the question: could we use [images](https://www.nytimes.com/interactive/2015/03/05/us/one-giant-picture-of-all-the-snow-across-the-us.html) of [snowpacks](https://www.flickr.com/photos/nasamarshall/9599342069/) from above to create topographic maps of exoplanets? Obviously this requires telescopes with vastly improved angular resolution, these will likely be [optical interferometers](http://dept.astro.lsa.umich.edu/~monnier/Publications/ROP2003_final.pdf) in space, or perhaps on the moon. Resolution can be increased by making the telescope’s baseline longer and light-gathering can be improved using larger mirrors or by integrating over longer time periods (planets don’t evolve rapidly, I don’t see why early exoplanet images wouldn’t be aggregated from weeks or months of observations). Space telescopes involving multiple satellites are going to be expensive and there have already been some [canceled missions](http://www.space.com/11877-alien-planets-search-canceled-missions-marcy.html) of this type. I haven’t checked their math but [this article](http://www.citizensinspace.org/2012/03/rethinking-the-webb-space-telescope/) describes the collecting area and baseline required to get images of different resolutions; unfortunately it doesn’t mention what assumptions were made about exposure times to get the images.

Assuming we can image a snowy exoplanet in sufficient detail, how can we determine its topography? We do it by making a snowpack model for the planet, probably involving a lot of educated guesses, and then invert it: instead of parameterizing the model with the topography, we instead search for the topography that is most likely to explain the observations. We could easily prototype this mapping software right now; it can be validated in two ways. First, we can take archived [satellite photos of Earth](http://www.ospo.noaa.gov/Products/imagery/archive.html), degrade them using reasonable assumptions about telescope performance and exoplanet distance, and then use them to infer the topography. Of course we have good ground truth for Earth so validation is nearly trivial. The second way to validate this mapping method is using simulated mountainous, snowy exoplanets. This would be useful to check the robustness of the methods for less-Earthlike bodies. A prototype of these ideas seems very much in reach for an ambitious PhD student.

I tried to find previous work on this topic and didn’t turn up much, I would appreciate links either in a comment here or in private email if you know of something. Let’s take a quick look at why most of the ways that we get topographic data won’t work for exoplanets:

- The original way to make maps, getting out there and surveying the land, obviously doesn’t apply.

- [Photogrammetry](https://en.wikipedia.org/wiki/Photogrammetry) requires parallax.

- Active sensing using radar or a laser is infeasible.

On the other hand, inferring mountains’ shape from changing light and shadows over the course of a day and from day to day (if the planet’s axis is tilted) will work, and no doubt this is how we’ll first know that an exoplanet is mountainous. Snow-based techniques could be used to refine these results.
