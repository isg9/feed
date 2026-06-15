---
title: Teaching Python Informally to Kids
url: https://blog.regehr.org/archives/1470
published: "2017-02-24T19:29:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=1470
---

# Teaching Python Informally to Kids

For the last few months I’ve been running a “coding club” for my son’s sixth-grade class. Once a week, the interested students (about 2/3 of the class) stick around for an hour after school and I help them learn to program. The structure is basically like the lab part of a programming class, without the lecture. I originally had planned to do some lecturing but it soon became clear that at the end of a full day of school (these kids get on the bus around 7:00am, ugh) there was very little attention span remaining. So instead I decided to give them a sheet of problems to work on each week and I spend the hour walking around helping whoever needs help.

One of my main goals is to avoid making anyone hate programming. There are three parts to this. First, I’m not forcing them to do anything, but rather providing suggestions and guidance. Second, there’s no assessment going on. I’ve never been too comfortable with the grading part of teaching, actually. It can really get in the way of learning. Third, I’m encouraging them to work together. I’ve long noticed (starting when I was a CS student) that most learning occurs between students in the lab. Not as much learning happens in the lecture hall.

For a curriculum, we started out with turtle graphics, which I’ve always found to be wonderful. [Python’s turtle library](https://docs.python.org/3.5/library/turtle.html) is clunkier than Logo and also is abysmally slow, but the advantages (visual debugging, many kids enjoy graphics) outweigh the problems. Fractals (Sierpinski triangle, dragon curve) were a good way to introduce recursion. We spent three or four weeks doing turtle stuff before moving on to simple crypto, which didn’t work as well. On the other hand, the students did really well with mathy code, [here’s the handout](https://blog.regehr.org/extra_files/coding-club-jan-25.pdf) from one of the math weeks.

Some observations:

- One of my favorite moments so far has been when a couple of students implemented rot13 functions and used it to send rude messages to each other.

- It’s pretty important to take a snack break.

- Although the rockstar / 10x programmer idea is out of favor, it is absolutely clear that there is a huge amount of variation (much more than 10x) in how easily a group of twenty 10-11 year olds learn programming. Some kids are teaching themselves to open files and parse text while others are stuck on basic syntax issues.

- Syntax, which computer professionals more or less learn to overlook, is hugely important.

- I’ve always disliked Python’s significant whitespace and watching kids struggle with it has made me hate it even more.

Anyhow, this has been a fun teaching exercise and hopefully it is benefiting the kids.
