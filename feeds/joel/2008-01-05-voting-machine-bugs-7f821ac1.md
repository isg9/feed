---
title: Voting machine bugs
url: https://www.joelonsoftware.com/2008/01/05/voting-machine-bugs/
published: "2008-01-05T00:11:32Z"
feed: joel
guid: https://www.joelonsoftware.com/?p=691
---

# Voting machine bugs

[Clive Thompson](http://www.nytimes.com/2008/01/06/magazine/06Vote-t.html?_r=1&hp=&oref=slogin&pagewanted=all), writing in The New York Times: “In 2005, the state of California complained that the machines were crashing. In tests, Diebold determined that when voters tapped the final “cast vote” button, the machine would crash every few hundred ballots. They finally intuited the problem: their voting software runs on top of Windows CE, and if a voter accidentally dragged his finger downward while touching “cast vote” on the screen, Windows CE interpreted this as a “drag and drop” command. The programmers hadn’t anticipated that Windows CE would do this, so they hadn’t programmed a way for the machine to cope with it. The machine just crashed.”
