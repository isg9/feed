---
title: Android Projects Retrospective
url: https://blog.regehr.org/archives/647
published: "2011-12-11T19:12:23Z"
feed: regehr
guid: http://blog.regehr.org/?p=647
---

# Android Projects Retrospective

The Android Projects class I ran this semester has finished up, with students demoing their projects last week. It was a lot of fun because:

- I find mobile stuff to be interesting
- the students were excited about mobile
- the students were good (and taught me a ton of stuff I didn’t know)
- the course had 13 people enrolled (I seldom get to teach a class that small)

The structure of the class was simple: I handed each student a Samsung Galaxy 10.1 tab and told them that the goal was for each of them to do something cool by the end of the semester. I didn’t do any real lectures, but rather we used the class periods for discussions, code reviews, and a short whole-class software project (a sudoku app). Over the last several years I’ve become convinced that large lecture classes are a poor way for students to learn, and teaching a not-large, not-lecture class only reinforced that impression. If we believe all the stuff about online education, it could easily be the case that big lecture courses are going to die soon — and that wouldn’t be a bad thing (though I hope I get to keep my job during the transition period).

The course projects were:

- A sheet music reader, using [OpenCV](http://opencv.willowgarage.com/wiki/) — for the demo it played Yankee Doodle and Jingle Bells for us
- A tool for creating animated GIFs or sprites
- A “where’s my car” app (for the mall or airport) using Google maps
- Finishing and polishing the sudoku app that the whole class did (we had to drop it before completion since I wanted to get going with individual projects)
- An automated dice roller for Risk battles
- A generic board game engine
- A time card app
- A change counting app, using OpenCV
- A bluetooth-based joystick; the demo was using it to control Quake 2
- A cross-platform infrastructure (across Android and iOS) for games written in C++

The total number of projects is less than the number of students since there were a few students who worked in pairs. I can’t easily count lines of code since there was a lot of reuse, but from browsing our SVN repo it’s clear that the students wrote a ton of code, on average.

Overall, my impression that desktops and laptops and dying was reinforced by this course. Not only can tablets do most of what most people want from a computer, but also the synergy between APIs like Google maps and the GPS, camera, and other sensors is obvious and there are a lot of possibilities. Given sufficient compute power, our tabs and phones will end up looking at and listening to everything, all the time. [OpenCV’s Android port](http://opencv.willowgarage.com/wiki/Android) is pretty rough and it needs a lot of optimization before it will work well in real-time on tabs and phones, but the potential is big.

Android has a lot of neat aspects, but it’s an awful lot of work to create a decent app and it’s depressingly hard to make an app that works across very many different devices. This is expected, I guess — portable code is never easy. The most unexpected result of the class for me was to discover how much I dislike Java — the boilerplate to functional code ratio is huge. This hadn’t seemed like that much of a problem when I used to write standalone Java, and I had a great time hacking up extensions for the [Avrora simulator](http://compilers.cs.ucla.edu/avrora/), but somehow interacting with the Android framework (which seems pretty well done) invariably resulted in a lot of painful glue code. The students didn’t seem to mind it that much. It’s possible that I’m exaggerating the problem but I prefer to believe they’re suffering from the Stockholm syndrome.
