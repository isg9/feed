---
title: Emacs, Thanksgiving, and Hanukkah
url: https://nullprogram.com/blog/2013/11/28/
published: "2013-11-28T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2013/11/28/
---

# Emacs, Thanksgiving, and Hanukkah

## [Emacs, Thanksgiving, and Hanukkah](/blog/2013/11/28/)

November 28, 2013

nullprogram.com/blog/2013/11/28/

Today is Thanksgiving in the United States. It also happens to be Hanukkah. There’s been news going around that Thanksgiving and Hanukkah [will not coincide again for about 80,000 years](http://www.leancrew.com/all-this/2013/01/hanukkah-and-thanksgiving/). This sounded somewhat unbelievable to me because [the Gregorian repeats every 400 years](http://blog.plover.com/calendar/july-weekends.html). I decided to compute it for myself to double-check this figure.

I’m not Jewish and I know very little about Hanukkah, so I had to look it up. After learning that Hanukkah is based on the Hebrew calendar, the rumors were sounding more believable. The Hebrew calendar repeats every 689,472 Hebrew years. This means the correspondence between Gregorian and Hebrew calendars [is about 14 billion years](http://hebrewcalendar.tripod.com/). That 80,000 seems lowball.

Since I decided to use Emacs Lisp for the computation, I fortunately was able to ignore all the unfamiliar, complicated rules for the Hebrew calendar: Emacs knows how to compute Hebrew dates. It can be accessed through the function `calendar-hebrew-date-string`.

```
;; Thanksgiving 2013
(calendar-hebrew-date-string '(11 28 2013))
;; => "Kislev 25, 5774"

```

Hanukkah begins on the 25th of Kislev, so I can write a quick-and-dirty function to detect if a date is the first day of Hanukkah.

```
(defun hanukkah-p (date)
  "Return non-nil if DATE is Hanukkah."
  (string-match-p "^Kislev 25" (calendar-hebrew-date-string date)))

```

Next I need a function to compute Thanksgiving, which is really simple. Thanksgiving falls on the fourth Thursday of November.

```
(defun thanksgiving (year)
  "Return the date of Thanksgiving for YEAR."
  (loop for day from 1 upto 7
        when (= 4 (calendar-day-of-week `(11 ,day ,year)))
        return `(11 ,(+ day 21) ,year)))

```

If there was no `calendar-day-of-week` I could compute it using [Zeller’s algorithm](http://en.wikipedia.org/wiki/Determination_of_the_day_of_the_week#Gauss.27s_algorithm), which I already happen to have implemented,

```
(defun cal/day-of-week (year month day)
  "Return day of week number (0-7)."
  (let* ((Y (if (< month 3) (1- year) year))
         (m (1+ (mod (+ month 9) 12)))
         (y (mod Y 100))
         (c (/ Y 100)))
    (mod (+ day (floor (- (* 26 m) 2) 10) y (/ y 4) (/ c 4) (* -2 c)) 7)))

```

Now for each year find Thanksgiving and test it for Hanukkah. I started with 1942 because that’s when the fourth-Thursday-of-November rule was established. Presumably due to the regexp part, this expression takes a moment to compute.

```
(loop for year from 1942 to 80000
      when (hanukkah-p (thanksgiving year))
      collect year)
;; => (2013 79043 79290 79537 79564 79635 79784 79811 79882)

```

My result exactly matches what I’m seeing elsewhere. The rumors are correct! The next coincidence occurs on November 23rd, 79043. Thanks, Emacs!

- [emacs](/tags/emacs/)
- [elisp](/tags/elisp/)
- [meatspace](/tags/meatspace/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Emacs,%20Thanksgiving,%20and%20Hanukkah) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Emacs%2C+Thanksgiving%2C+and+Hanukkah).

« [Elfeed Tips and Tricks](/blog/2013/11/26/)

» [Emacs Lisp Reddit API Wrapper](/blog/2013/12/16/)
