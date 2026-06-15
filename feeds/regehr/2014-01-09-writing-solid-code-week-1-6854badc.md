---
title: Writing Solid Code Week 1
url: https://blog.regehr.org/archives/1080
published: "2014-01-09T22:22:03Z"
feed: regehr
guid: http://blog.regehr.org/?p=1080
---

# Writing Solid Code Week 1

This semester I’m teaching a new undergraduate course called *Writing Solid Code*. The idea is to take a lot of lessons I’ve learned on this subject — testing, debugging, defensive coding, code reviews, etc. — and try to teach it. Since I don’t have any slides or even any lecture notes that are good enough to put on the web, I thought it would be useful to summarize each week in a blog post.

One thing that I wanted to do is create an environment where cheating is impossible. To accomplish this, all student code is going into a [public Github repo](https://github.com/regehr/solid_code_class), and everyone is free to look at everyone else’s code if they wish to. In other words, cheating is impossible because the actions that would normally be considered cheating aren’t. Of course, I am encouraging students to write their own code. In a self-selected group of students I don’t expect problems. If people want to exchange ideas or test cases or even subroutines, then great.

The other thing I wanted to do was get away from the kind of grading where a failing test case detracts from a student’s grade. So I am not going to do that. Rather, everyone starts with a default grade of “A” and they only lose this grade if they fail to attend class, fail to participate in activities such as code reviews, or fail to submit assignments. I am counting on people’s competitive spirit to drive them towards creating code that is at least as solid as their peers’ code. This has worked well in the past and I expect it to work again.

The class is going to be structured around a set of small coding exercises, each designed to reinforce some aspects of writing solid code. Each exercise will be iterated until all (or as many as possible) students’ codes pass all test cases that they — and I — can think of. If this takes too long I’ll start making the projects easier; if (as I expect) we can converge quickly, I’ll make the projects more complicated. This structure is designed to go against the grain of the typical fire-and-forget coding style that CS students are trained to adopt; I’m convinced that it builds very poor habits. The idea that I want the students to get out of these exercises is that in most cases creating a piece of code is the easy part; the hard part is figuring out what the code is supposed to do in the first place and also making sure that the code actually works as intended.

The first exercise is the venerable triangle classifier. The students, with a bit of guidance from me — their program manager — developed [the spec](https://github.com/regehr/solid_code_class/blob/master/triangle/spec.txt) in class on Tuesday. Their code is due next Tuesday, along with a collection of test cases that achieves 100% partition coverage as well as 100% coverage of feasible branches. This exercise is in C. The language isn’t very relevant, but C does enjoy good tool support. We may switch languages later on.

Since we don’t yet have any code to look at, I lectured on testing today. Basically I focused on the three hard problems in testing:

- Generating test inputs

- Determining if the result of a test is acceptable

- Determining what a collection of passing test cases means

This was more or less an abbreviated version of the material I tried to cover in the early parts of my [Udacity course](https://www.udacity.com/course/cs258). I wish that I had a nice collection of notes about this but I don’t (yet).
