---
title: on the many roads to perdition — wingolog
url: https://wingolog.org/archives/2008/08/02/on-the-many-roads-to-perdition
published: "2008-08-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/08/02/on-the-many-roads-to-perdition
---

# on the many roads to perdition — wingolog

## [on the many roads to perdition](/archives/2008/08/02/on-the-many-roads-to-perdition)

2 August 2008 1:45 PM

- [code](/tags/code)
- [entropy](/tags/entropy)
- [bitrot](/tags/bitrot)
- [summertime](/tags/summertime)

**software failure modes**

Reflecting earlier today on some crappiness in a [software project I'm involved in](http://home.gna.org/guile-lib/), my brain started to enumerate the ways that free software projects can fail.

The maintainer can go [AWOL](http://www.urbandictionary.com/define.php?term=awol). The major culprits for this are babies, university thesis advisors, and Google.

This is pernicious in two ways: one, the person with the most knowledge about a project is no longer commenting on efforts by others to contribute. Patches go unreviewed, questions go unanswered.

Secondly, the maintainer is in a privileged position -- they typically have sole administrative access to the mailing list, the revision control system, and sometimes the web page. No one else can intervene to fix e.g. a spam problem on the list. But maintainers' privileged position extends to a kind of moral authority as well: "OK, she's been away for three months now... can I make a release without her permission?" The result is that in the meantime, the project's momentum decays, contributors spend their time elsewhere where they are more appreciated.

Even in the lack of a strange interstitial period of limbo, a maintainer's departure usually leads to the badness, as other less familiar, possibly less skilled contributors take control. As prudent as they might be, some bad patches probably slip in. The project enters a time of increasing entropy, of bitrot.

In addition, you have failure modes that have nothing to do with people involved in the project. API changes in a core system library can bitrot reams of dependent code in one commit. It's not enough for the old libraries to be available on the intertubes, they have to be installable on a contemporary computer. Your language specification or runtime could change. And of course those core libraries have dependencies as well: glibc versions, kernel syscalls, not to mention the ever-changing soup of hardware interfaces.

Ultimately behind those changes you will find human desires, "market forces", and even such things as global warming.

Any other failure modes come to mind? Follow on in yon comment bucket!

(To be clear, I'm not saying that guile-lib failed, but the mailing list situation is pretty terrible.)

**summertime**

Beaches, bars, and outdoor cinema. Quiet afternoons on the terrace, poking around the house, the neighborhood, a bit hobbitlike.

I'm about to head off travelling in August -- in Seattle for the week of the 11th, down to Portland OR for a few days, overnight train to San Fran, for a few days, then down to LA for the 27th, and back home to BCN on the 5th or so. Amtrak ho!

### One response

1. [Stu](http://stuartaxon.com) says:[2 August 2008 6:21 PM](#69b30ef34ddfdcad69babde5487c1d3e526a0590)

   Two promising projects I'm aware of where maintainer went awol:

   Nat Friedmans Dashboard - Went awol when a search framework was needed and he went off to make beagle... come back !

   PearPC - OS/X emulator.. Maintainer awol for reasons announced on mailing list

Comments are closed.
