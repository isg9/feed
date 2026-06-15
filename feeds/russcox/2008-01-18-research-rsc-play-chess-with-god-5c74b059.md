---
title: 'research!rsc: Play Chess with God'
url: https://research.swtch.com/chess
published: "2008-01-18T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/chess
---

# research!rsc: Play Chess with God

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Play Chess with God Russ Cox January 18, 2008 *research.swtch.com/chess* Posted on Friday, January 18, 2008.

Starting in the late 1970s, Ken Thompson proved by example that computers could completely analyze chess endgames, which involve only a small number of pieces. Thompson's key insight was that when there are only a few pieces on the board, there might be very many possible move sequences, but there are relatively few distinct board positions. An exhaustive search over board positions can run efficiently even though an exhaustive search over move sequences would not. (The same idea underlies Thompson's efficient [parallel NFA simulation](http://swtch.com/~rsc/regexp/) for regular expression matching.)

Thompson's first endgame analyses ran on boards with only four pieces, but over time he took advantage of improvements in processor speeds and storage sizes to analyze five- and six-piece endgames as well. Thompson described the programs in the ICCA Journal: his 1986 article “ [**Retrograde Analysis of Certain Endgames**](http://pdos.csail.mit.edu/~rsc/thompson86endgame.pdf)” (PDF; 1MB) introduces the technique and covers five-piece endgames, and his 1996 article “ [6-Piece Endgames](http://pdos.csail.mit.edu/~rsc/thompson96endgame.pdf)” (PDF; 1MB) describes refinements and optimizations for six pieces.

Writing about a 262-move optimal ‘rook and knight versus two knights’ game Thompson found in the 6-piece endgames, former championship chess player [Tim Krabbé explained](http://www.xs4all.nl/~timkr/chess2/diary_3.htm):

> Playing over these moves is an eerie experience. They are not human; a grandmaster does not understand them any better than someone who has learned chess yesterday. The knights jump, the kings orbit, the sun goes down, and every move is the truth. It's like being revealed the Meaning of Life, but it's in Estonian.

On his web page, Thompson links to the online 6-piece endgame database as “ [Play Chess with God](http://cm.bell-labs.com/who/ken/chesseg.html).” The [CD art](http://www.spinroot.com/gerard/img/holzmann-chess.gif) for the five-piece databases used a similar theme.

*Further reading*: With Joe Condon, Thompson built the chess playing computer Belle, which won the ACM North American Computer Chess Championship five times. Belle was later [confiscated by U. S. Customs](http://swtch.com/moscow-belle.txt) on its way to a contest in Moscow, on suspicion of being of military use. Thompson was quoted as saying that Belle's only military use would be “to drop it out of an airplane. You might kill somebody that way.”

Thompson's work in computer chess was featured in a [Computer History Museum exhibit](http://www.computerhistory.org/chess/), which included a [video of Thompson telling his story](http://www.computerhistory.org/chess/viewAll.php?sec=thm-42eeabf470432&sel=thm-42f15c52333a3&table=item_oral_history). (Beware that in the transcript, ‘endgames’ is usually written as ‘n games.’)

(Comments originally posted via Blogger.)

- [Elver](http://www.blogger.com/profile/10883756259775426088)(January 18, 2008 10:00 AM) *"The knights jump, the kings orbit, the sun goes down, and every move is the truth. It's like being revealed the Meaning of Life, but it's in Estonian."*

Heh. I'm actually Estonian :P

- [Josh](http://www.blogger.com/profile/13766251583136140236)(January 18, 2008 1:43 PM) \> Heh. I'm actually Estonian :P

I thought you were gonna slip us The Answer, then. ;)

This post was absolutely fantastic, Russ.

- [AKS](http://www.blogger.com/profile/16120300424669884662)(January 20, 2008 7:39 AM) Hi,

I once tried to implement this.

I wanted to make a full-fldeged compiler , which would take a reg-ex as input an generate an executable to scan just that input.

I wanted to convert input regex into an AST. My aim was to traverse the AST and generate code for each node , sequentially building the program. However the POSIX reg-ex rules , are arcane and I lost interest in the parser itself.

Your website is superb , I hope you continue the good work..

Thanks,
