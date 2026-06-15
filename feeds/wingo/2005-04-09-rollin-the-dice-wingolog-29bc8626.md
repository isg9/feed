---
title: Rollin the dice — wingolog
url: https://wingolog.org/archives/2005/04/09/101
published: "2005-04-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/04/09/101
---

# Rollin the dice — wingolog

## [Rollin the dice](/archives/2005/04/09/101)

9 April 2005 7:45 PM

- [computers](/tags/computers)

In a past life I had the, um, experience of working with [Monte Carlo N-Particle (MCNP)](http://laws.lanl.gov/x5/MCNP/index.html), Los Alamos' nuclear particle interaction simulator. It's used to model the absorption, refraction, and reflection of particles in materials, and also has a component for those interactions to produce new particles. So you can predict the distribution and kind of scattered radiation resulting from bombarding a surface with electrons, for instance. Or you can model a nuclear reactor core. Or a nuclear bomb, a purpose for which it continues to be used.

It's a bit peculiar from a software engineering perspective:

> MCNP is written in the style of Dr. Thomas N. K. Godfrey, the principal MCNP programmer from 1975-1989 ... All variables local to a routine are no more than two characters in length, and all COMMON variables are between three and six characters in length ... The principal characteristic of Tom Godfrey's style is its terseness. Everything is accomplished in as few lines of code as possible. Thus MCNP does more than some other codes that are more than ten times larger. It was Godfrey's philosophy that anyone can understand code at the highest level by making a flow chart and anyone can understand code at the lowest level (one FORTRAN line); it is the intermediate level that is most difficult. Consequently, by using a terse programming style, subroutines could fit within a few pages and be most easily understood. Tom Godfrey's style is clearly counter to modern computer science programming philosophies, but it has served MCNP well and is preserved to provide stylistic consistency throughout.
>
> [MCNP4c chapter 2, section B](http://www.ftj.agh.edu.pl/~domanska/mcnp4c/chapter2.pdf)

It's from the [X division](http://laws.lanl.gov/). As in, "Where do you work? *The X division.* So, what do you do? *Sorry, can't tell you that. Please don't ask anything else.*"

## related articles

- [Roundup](/archives/2005/10/27/roundup)
- [ppc ruminations](/archives/2005/06/09/ppc-ruminations)
- [computerisation](/archives/2005/05/26/computerization)
- [pretending we're bunny rabbits](/archives/2005/04/25/pretending-we)
- [vaquero](/archives/2005/04/20/vaquero)
- [nine months with arch](/archives/2004/10/01/nine-months-with-arch)

Comments are closed.
