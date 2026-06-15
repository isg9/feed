---
title: Guidelines for Teaching Assistants
url: https://blog.regehr.org/archives/489
published: "2011-03-11T18:45:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=489
---

# Guidelines for Teaching Assistants

I’ve been teaching university-level courses for the last nine years, usually with the support of teaching assistants (TAs): students who get paid to do things like grading, office hours, fielding email questions, making and debugging assignments, proctoring exams, and perhaps even giving a lecture when I’m sick or traveling.

At the start of each semester I sit down with my new TAs and go over some guidelines. Unfortunately, this is a chaotic time and I’m pretty sure I’ve never managed to give the same advice twice. This piece is an attempt to put all of my guidelines in one place. I’m certainly leaving things out — please fill in details in comments. This is all somewhat specific to computer science classes with programming assignments.

# The Golden Rule of the CS TA: Never Type at a Student’s Keyboard

When a student doesn’t get it, the temptation to just sit down and take over can be overwhelming. But you have to resist because as long as the student is driving, the student is thinking. When you take over, their brain switches into passive mode.

# Progressive Hints

When a student is struggling on an assignment, first give oblique hints: “Did you read Chpater 7?” “Did you search for that on Wikipedia?” If this fails to help, be more concrete: “Are you checking all return codes?” “Did you run it under Valgrind?” Finally, and only when you are convinced the student has made a genuine effort to solve the problem, give a more direct answer, but be sure that it only helps the student past the current roadblock: “Your foo() function has to return 0 on success, not 1.” “You’re dereferencing a NULL pointer.” With time, you will learn which students can run with a quick hint and which ones probably need stronger advice.

My model for progressive hints is based on the Invisiclues packets for old Infocom games — so awesome.

# Amend, But Communicate

Often, when answering a student’s question, it becomes clear that some part of the assignment or lab was underspecified and it should have been more clear. In cases like this, your answer to the student is effectively a supplemental part of the assignment. This is fine, but when this happens it is necessary to let the instructor and/or the rest of the class know the additional information.

# When Things Aren’t Working, Tell Me

Sometimes there are infrastructure problems: a bunch of machines in the lab are down, or some critical piece of software doesn’t work correctly. Other times, there is a systematic problem in an assignment or lab: a crucial ambiguity or error. When you notice anything like this happening, let me know — I can’t fix problems that I’m unaware of.

# Treat Students With Respect

I’ve only seen one problem out of probably 30 TAs, and it wasn’t that serious. But still this is worth saying.

# Don’t Play the 10,000 Questions Game

In a big class there are always one or two students who, instead of reading the assignment, just email the instructor or TAs. If you answer, they immediately have another question. This goes on forever — I do not exaggerate, they apparently never eat or sleep. The solution is to recognize these cycles and delay the next response for 12 hours or so. Sooner or later the student realizes that it’ll be more efficient to just read the assignment.

# Do the Assignment First

This sucks, but you have to do each assignment, or at least the ones for which you are the principal TA, before it is handed out. Otherwise, it is effectively impossible to help students get past roadblocks they encounter. This mainly applies to courses like operating systems where the assignments may be really hard. For freshmen and sophomore-level programming courses, any conceivable problem can by eyeballed in a few seconds.

# Don’t Miss Office Hours

At least give advance warning if it’ll be impossible to show up so students don’t waste time coming to the lab.

# Bounce Problem Cases Up To The Instructor

You’re not being paid enough to deal with difficult students. If anyone is pushy about a grade, rude about any issue, or anything else, simply refer the student to me. This almost never happens.

# Don’t Screw Up the File

There is one grade spreadsheet, and it has a specific file name. Do not put grades in any other file, do not create duplicate files, do not put unlabeled columns in the spreadsheet. Label each new column carefully. Every time you finish working with the file, make sure the permissions are such that the other TAs and I can read and write it, but nobody else can.

# Anonymize Grades Carefully

When posting grades, delete names, student ID numbers, and email addresses before exporting to HTML.

# Never Deal With Cheating

This is my job. If you suspect cheating, talk to me; but never bring it up with a student.

# Be Timely

Unless I say otherwise, labs should be graded, and grades entered, within a week of being handed in.

# Do Not Flake Out

TAs are themselves students and are subject to getting overwhelmed like anyone else. If this seems to be happening, come talk to me. If you just disappear in the middle of the semester and stop doing your job, I will seriously make sure you never work in this town again.
