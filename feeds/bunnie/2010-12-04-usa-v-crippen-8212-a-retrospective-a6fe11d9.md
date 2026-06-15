---
title: USA v. Crippen &#8212; A Retrospective
url: https://www.bunniestudios.com/blog/2010/usa-v-crippen-a-retrospective/
published: "2010-12-04T09:39:04Z"
feed: bunnie
guid: http://www.bunniestudios.com/blog/?p=1472
---

# USA v. Crippen &#8212; A Retrospective

Some readers may be aware that [I was called upon](http://www.wired.com/threatlevel/2010/10/xbox-modder-tria/) to perform as an expert witness in a [landmark case](http://www.wired.com/threatlevel/2010/12/no-deal-in-xbox-modding-case-trial-begins/), USA v. Crippen, where for the first time an individual, Mr. Crippen, was charged with an alleged violation of the criminal portion of the DMCA statute. There have been numerous civil cases over the same statute, but this is the first time that a felony conviction could result from a court case.

As reported by [numerous sources](http://www.wired.com/threatlevel/2010/12/crippen-dismissed/), the case was dismissed after the first witness’ testimony. This would be as if two armies brought all their artillery and troops to a border, fired a single shot, and then one side surrendered, realizing that there is no point incurring casualties for a war they cannot win. And thanks to [double-jeopardy](http://www.lectlaw.com/def/d075.htm) provision of the US constitution, Mr. Crippen cannot be tried again, since a jury was assembled for his trial. It is a remarkable victory for Mr. Crippen’s defense: as [Sun Tzu said in The Art of War](http://en.wikiquote.org/wiki/The_Art_of_War), “The best victory is when the opponent surrenders of its own accord before there are any actual hostilities”.

On the surface, it’s hard to appreciate how unique this case is. Not only is it the first of its kind, it’s rare for a US prosecutor to dismiss their case. I’m told that typically, the US government does not go to trial unless they are sure to win the case — they win 90+ % of their cases, with a typical outcome resulting in a plea bargain because of the strong evidence they prepare prior to filing the case. I’m also told that despite the [prosecutor’s alleged misbehavior](http://www.wired.com/threatlevel/2010/12/xbox-judge-riled/) [in the case](http://www.crimeandfederalism.com/2010/12/-allen-chiu-unethical-prosecutor-forced-to-dismiss-xbox-modding-case.html), his pedigree is prestigious ( [UCLA is a top-15 law school](http://en.wikipedia.org/wiki/UCLA_School_of_Law)) and his career trajectory is toward a top spot as a judge or politician. And, as I’m learning, neither the prosecution nor the defense leave much to chance in the court of law — so kudos to the defense for educating the judge on terms such as “fair use” and “homebrew”, and applying overwhelming pressure to “crack” the prosecution: a job well done. To be fair, the case was without precedent, so the [prosecutor was unaware](http://www.wired.com/threatlevel/2010/12/no-deal-in-xbox-modding-case-trial-begins/) of basic things, such as the US government’s own guidelines for evidence in prosecuting crimes related to the DMCA. In this case, the US government had to demonstrate that Crippen knew he was violating the DMCA, an element missing from the original evidence but introduced in a surprise statement by the first witness.

However, in a broader legal sense, the trial is a cliffhanger. In some respects, it’s a setup for prosecutors to prepare a stronger, more informed case in the future. Before a case goes to trial, each side must disclose all their evidence and facts to the opposition (and, in fact, part of the reason the prosecution had to dismiss was because they had failed to do just that — it is improper to withhold both exculpatory, and in this instance, impeaching evidence [(Giglio v United states)](http://www.conservapedia.com/Giglio_v._United_States)).

As a corollary, the prosecution has a full copy of my prepared testimony. My role as an expert witness is to testify, as an unbiased expert, upon the facts of the case. By dismissing the case before a public hearing of all testimony, the prosecution gets to see the entire roadmap (of which my testimony is a small part) for a defense without its disclosure to the public.

A problem with technology-related cases is that they are never as simple as they seem. The evidence presented by the US government included 150 non-original games in Crippen’s possession, along with two Xboxes that prior to Crippen’s modification, did not play copied games; but, after such modification, they did. As I mentioned earlier in this post, the US government does not go to trial unprepared.

While the true facts are not as simple, raw facts are essentially useless to a jury. The real challenge for me personally was to take a world of technical jargon full of one-way hashes, modular exponentiation, prime numbers, finite fields of characteristic two, stealth sectors, lead-ins, lead-outs, and reflectivity measurements using a laser and a four-quadrant photodetector and boil it down into a set of factually correct statements that any lay jury could not only understand, but feel confident enough to use to decide upon two felony counts.

So, for the purpose of encouraging discussion, criticism, and education, here are some of the key concepts I was to present in the case.

First, it’s important to clarify some basic cryptography terms (click on all images for larger, more readable versions).

[![](http://bunniestudios.com/blog/images/usvcr_datacontrols_sm.jpg)](http://bunniestudios.com/blog/images/usvcr_datacontrols.jpg)

The common use of “encryption” or “scambling” is tantamount to an “access control” insofar as a work is scrambled, using the authority imbued via a key, so that any attempt to read the work after the scrambling reveals gibberish. Only through the authority granted by that key, either legitimately or illegitimately obtained, can one again access the original work.

However, in the case of the Xbox360, two technically different systems are required to secure the authenticity of the content, without hampering access to the content: digital signatures, and watermarks (to be complete, the game developer may still apply traditional encryption but this is not a requirement by Microsoft: remember, Microsoft is in the business of typically selling you someone else’s copyrighted material printed on authentic pieces of plastic; in other words, they incur no loss if you can read the material on the disk; instead, they incur a loss if you can fake the disk or modify the disk contents to cheat or further exploit the system).

Simply put:

Digital signatures leave a document’s body completely readable, but attach an unforgeable signature that is irrevocably tied to an unmodifiable version of the document.

Watermarks leave a document’s body completely readable, but attach an unforgeable physical mark that is irrevocably tied to the physical disk itself.

Relating this back to [the DMCA statute](http://copyright.gov/title17/92chap12.html):

> 1201(a)(1)(A) No person shall circumvent a technological measure that effectively controls access to a work protected under this title.
>
> 1201(a)(3)(B) a technological measure “effectively controls access to a work” if the measure, in the ordinary course of its operation, requires the application of information, or a process or a treatment, with the authority of the copyright owner, to gain access to the work.

So the first question upon which a jury must deliberate is: given that the document is entirely readable despite anti-counterfeit measures, do these anti-counterfeit measures constitute an effective access control that requires the application of information, or a process or a treatment, with the authority of the copyright owner, to gain access to the work?

To further educate upon that question, it’s important to demonstrate an example of a system where data cannot be accessed, and contrast it to one where it is. The image below compares and contrasts a [CSS-protected](http://en.wikipedia.org/wiki/Content_Scramble_System) DVD to the systems used in the Xbox360.

[![](http://bunniestudios.com/blog/images/usvcr_gamevdvd_sm.jpg)](http://bunniestudios.com/blog/images/usvcr_gamevdvd.jpg)

As one can see, on the left, I could access all kinds of images, text, etc. on an Xbox360 DVD. On the right, on the other hand, an authentic DVD secured with a fairly established access control, such as CSS, reads back as gibberish until I can circumvent the scrambling with either a legitimate or illegitimate key.

Now, per the DMCA statute:

> 1201(a)(1)(A) No person shall circumvent a technological measure that effectively controls access to a work protected under this title.
>
> 1201(a)(3)(A) to “circumvent a technological measure” means to descramble a scrambled work, to decrypt an encrypted work, or otherwise to avoid, bypass, remove, deactivate, or impair a technological measure…

So the next question the jury must deliberate upon is, does an Xbox360 optical disk drive (ODD) modification descramble a scrambled work, decrypt an encrypted work, or otherwise avoid, bypass, remove, deactivate, or impair a technological measure?

To further education upon that question, it’s important to understand what an Xbox360 ODD modification does; the requisite background to this is “how does an Xbox360 ODD work in the first place?”. Below is a diagram that outlines, in simplified terms, the flow of authenticating an Xbox360 game disk.

[![](http://bunniestudios.com/blog/images/usvcr_gameplay_sm.jpg)](http://bunniestudios.com/blog/images/usvcr_gameplay.jpg)

As you can see, the ODD is responsible for returning measurements of watermark features (such as reflectivity) that are not burnable by a regular DVD burner.

What the ODD modification does is redirect these requests to verify the watermark to an “answer table” contained in what amounts to a few files on the copied disk:

[![](http://bunniestudios.com/blog/images/usvcr_copydisk_sm.jpg)](http://bunniestudios.com/blog/images/usvcr_copydisk.jpg)

The most important fact to be cognizant of in this system is that the “answer table” is not contained anywhere within the Xbox360 ODD mod applied by Mr. Crippen. Without the user of the modification *also* contributing the “answer table”, the mod is entirely incapable of performing any function. This is demonstrated by what happens if, for example, the “answer table” is missing or damaged:

[![](http://bunniestudios.com/blog/images/usvcr_gamenotplay_sm.jpg)](http://bunniestudios.com/blog/images/usvcr_gamenotplay.jpg)

In the case that the “answer table” is lacking from the disk inserted into the ODD, the disk will not play. Thus, the question is: given that the user of the modified Xbox360 (in this case, the private investigators and agents that the government hired) must also materially participate in the “process” by providing an “answer table”, is the mod alone sufficient to justify felonious conduct?

Unfortunately, the answer is: “we don’t know”. Since the case was dismissed, the answer to this question is a cliffhanger; and the prosecution, now educated, should have a clearer roadmap for future actions under the criminal provision of the DMCA; I wouldn’t count on them making the same mistakes twice. Technical facts, such as the ones outlined in this post, and disclosed to the prosecution, don’t change from case to case … but the individuals, specific evidence, and overall angle of the case *can* change.
