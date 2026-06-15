---
title: Python Exercises for Kids
url: https://blog.regehr.org/archives/1300
published: "2016-03-02T21:14:24Z"
feed: regehr
guid: http://blog.regehr.org/?p=1300
---

# Python Exercises for Kids

For the last year or so I’ve been giving Python exercises to my 11 year old. I thought I’d share some of them. If any of you have been doing similar things, I’d love to hear what worked for you. I think it is helpful that I’m not much of a Python programmer, this forces him to read the documentation. The other day he said “Wow– Stack Overflow is great!”

## Fibonacci

Print the Fibonacci series, useful for teaching basics of looping.

## Number Guessing Game

The user thinks of a number between 1 and 100. The computer tries to guess it based on feedback about whether the previous guess was too high or low. This one was great for learning about the kinds of off-by-one errors that one customarily runs into while implementing a binary search.

## Binary Printer

Print the non-negative binary integers in increasing order. This one forces some representation choices.

## Palindrome Recognizer

Recognizing “A man, a plan, a canal — Panama!” as a palindrome requires some string manipulation.

## Door Code Recognizer

Our Paris apartment building will let you in if you enter the correct five-digit sequence regardless of how many incorrect digits you previously entered. I thought replicating this behavior in Python would be slightly tricky for the kid but he saw that a FIFO could easily be created from a list.

## Monte Carlo Pi Estimator

If you generate random points in the square between -1,-1 and 1,1, the fraction of points that lie inside the unit circle will approach pi/4. The kid got a kick out of seeing this happen. Of course I had to explain the derivation of pi/4 and the formula for a circle since he hasn’t seen this stuff in school yet. Also of course we could have simply worked in quadrant one but that seemed to make the explanations more complicated.

I should perhaps add that I explained this exercise to my kid very differently than I’m explaining it here. We started by drawing a box containing a circle and then pretended to throw darts into the box. The concepts are not hard: all we need to understand is random sampling and the area of a circle. Math gets made to seem a lot harder than it is, in many cases, and also we tend to conflate the difficulty with the order in which the concepts are presented in the school system.

## Turtle Graphics Interpreter

The current project is implementing a turtle that takes four commands:

- forward n

- right-turn n

- left-turn n

- color c

So far we’ve been restricting angles to 0, 90, 180, 270 (and no radians yet…) but I don’t think sine and cosine will be hard concepts to explain in this context. Also, so far the turtle commands are just a series of function calls, the parser will come later. I wonder what would be the simplest loop construct for this turtle language? Perhaps “repeat n” followed by some sort of block construct.
