---
title: Followup to "A Very Subtle Bug"
url: https://blog.nelhage.com/2010/03/followup-to-a-very-subtle-bug/
published: "2010-03-03T13:45:11Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/03/followup-to-a-very-subtle-bug/
---

# Followup to "A Very Subtle Bug"

After my previous post got posted to reddit, there was a bunch of interesting discussion there about some details I’d handwaved over. This is a quick followup on some the investigation that various people carried out, and the conclusions they reached. In the reddit thread, lacos/lbzip2 objected that in his experiments, he didn’t see tar closing the input pipe before it was done reading the file, and so questioned where the SIGPIPE/EPIPE was coming from in the first place.
