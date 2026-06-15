---
title: FizzBuzz Bugs
url: https://blog.regehr.org/archives/1031
published: "2013-09-06T02:58:06Z"
feed: regehr
guid: http://blog.regehr.org/?p=1031
---

# FizzBuzz Bugs

It is said that an overwhelming majority of applicants for some programming jobs fail [the FizzBuzz test](http://www.codinghorror.com/blog/2007/02/why-cant-programmers-program.html). As part of a simple warmup exercise, I asked my computer systems class (a required course for CS majors, usually taken in the 3rd year) to implement FizzBuzz in C. The problem statement was, I think, the standard one:

> Write a C program that prints the numbers from 1 to 100 except that for multiples of three print “Fizz” instead of the number and for the multiples of five print “Buzz” and for numbers which are multiples of both three and five print “FizzBuzz”. Each number or string should be on its own line.

Out of 152 submissions, one failed to compile since the student had for some reason commented out `main()`. Out of the remaining 151 submissions, 26 did not produce precisely the expected output. If we ignore whitespace-related issues (extra newline at the start, no newline at the end, etc.), 17 submissions were still incorrect. I looked at each of them:

- Six submissions had what I would characterize as string bugs instead of logic bugs: printing strings like “The number 51 is Fizz” instead of “Fizz”; “Fizzbuzz” or “FissBuzz” instead of “FizzBuzz”; failing to print lines with just the numbers; etc.

- Four submissions printed a “FizzBuzz” line corresponding to zero instead of starting at one. One of these also neglected to print a line corresponding to 100. An additional submission left off the line corresponding to 100.

- Four submissions never printed “FizzBuzz” at all due to logic errors, but did print “Fizz” and “Buzz” properly.

- One submission printed an extra “Fizz” line every time “FizzBuzz” was printed.

- One submission had a logic error that defies explanation.

While I feel strongly that all third-year students in my program should be able to write a correct FizzBuzz, I was still happy to see that nobody failed the basic task of determining multiples of 3 and 5 and that all of the incorrect solutions had more or less the correct shape. Of course, writing this code at home or in the lab, with plenty of time, is far easier than trying to put it up on a whiteboard in the first 15 minutes of a job interview.
