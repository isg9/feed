---
title: steal our code — wingolog
url: https://wingolog.org/archives/2007/05/03/steal-our-code
published: "2007-05-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/05/03/steal-our-code
---

# steal our code — wingolog

## [steal our code](/archives/2007/05/03/steal-our-code)

4 May 2007 2:52 AM

- [hacks](/tags/hacks)
- [flumotion](/tags/flumotion)
- [work](/tags/work)

I was ruminating on [Flumotion](http://flumotion.net/), the free-software streaming server that I hack as a day job. We don't really do contributors. That's not because of a lack of public SVN, not because of a lack of a public bugtracking system. We're not assholes either. So what gives?

Well, for whatever reason, Flumotion doesn't have developers outside of [Fluendo](http://fluendo.com/), the company I work for. But those of us that do hack it are a prideful bunch. Thomas [writes frequently](http://thomas.apestaart.org/log/?s=flumotion) about it, Mike takes every critical bug as his own, and Zaheer spends weekends expanding its feature set. So we have code we care about, code that is free software, but code that nobody sees, as far as I can tell. This is in spite of our [trac](https://core.fluendo.com/flumotion/trac/)'s absurdly high google juice.

So if hackers won't come to Flumotion, Flumotion should come to hackers. Take our code! It's all licensed under the GPL, so you can probably use it in your free software project. Of course, using python helps, but is not always necessary; using [Twisted](http://twistedmatrix.org/) helps even more. But some things can be used by any project, for example today's example.

**integration test suite code**

An integration test is like a unit test, but larger. It merits a different name because it involves multiple processes. Flumotion has code to write integration tests as normal [trial](http://twistedmatrix.com/trac/wiki/TwistedTrial) unit tests.

`flumotion.twisted.integration` is here: the [code (steal this!)](https://core.fluendo.com/flumotion/trac/browser/flumotion/trunk/flumotion/twisted/integration.py), its [unit tests](https://core.fluendo.com/flumotion/trac/browser/flumotion/trunk/flumotion/test/test_twisted_integration.py), and an [example usage](https://core.fluendo.com/flumotion/trac/browser/flumotion/trunk/tests/integration/test_torture.py).

Concretely, an example usage might be this:

```
from twisted.trial import unittest
from flumotion.twisted import integration
class TestEcho(unittest.TestCase):
    @integration.test
    def testHelloWorld(self, plan):
        p = plan.spawn('echo', 'Hello World')
        plan.wait(p, 0)

```

This test is marked as an integration test by its decorator, `integration.test`. The test gets an extra argument, `plan`, which it is expected to populate. In this case the plan has two steps: one to spawn the echo process, and one to wait for its successful termination.

In the event that something unexpected happens, the test will fail. Precisely, "something unexpected" is an unexpected process exit (an exit without a `wait` step in the plan), processes hanging around when the plan has finished, or an exit with an unexpected error code or signal. Furthermore there is a default timeout, which can be adjusted for specific tests. In the event of a failure, stdout and stderr of all spawned processes will be left on disk in separate files for the developer to analyze.

The only other operation available on plans, besides `spawn` and `wait`, is `kill`, to kill a process with some signal. Pretty simple, but powerful enough to test large systems. We use it to test our streaming platform software, in which each test spawns dozens of processes.

And with that, we wrap up this first of N articles on Flumotion internals that people should use. Happy hacking!

## related articles

- [when the mike is in my hand, i'm never hesitant](/archives/2007/11/27/when-the-mike-is-in-my-hand-im-never-hesitant)
- [reducing the footprint of python applications](/archives/2007/11/27/reducing-the-footprint-of-python-applications)
- [roundup](/archives/2007/08/06/roundup-2)
- [songs about pottery](/archives/2006/12/21/songs-about-pottery)
- [new beginnings](/archives/2011/04/05/new-beginnings)
- [shoptalk](/archives/2010/06/21/shoptalk)

### 6 responses

1. HiHo says:[4 May 2007 5:03 AM](#448)

   Maybe more users would bring more contributors?

   I was just looking at the flumotion website, and it's kinda not obvious what it does. I mean, I have a vague idea of what a 'streaming server' is -- serves streams, right? Uhh.. but what can I do with it?

   As features, the website lists: twisted, python, and then linux. And then distributed. I guess that means features of Gnome would be: C, modular, released regularly, ..

   What cool things can I do with it? Send tv round my house? Live podcasts? Backend for Elisa?

   More direct, hands on experience using the code would probably lead to more people hacking on it.

   Thankyou!

2. [Kalle Vahlman](http://iki.fi/zuh) says:[4 May 2007 10:32 AM](#450)

   I have to agree with HiHo on this. It took me a good while to set up Flumotion on my home machine, not because it was all that difficult to do but because it took time to grok how it all was mixed together.

   I'm not sure why this is really, the documentation is there though maybe a bit sparse. One thing to note is that the manual at least strictly talks concepts and roles in the overview section, leaving the hands-on-this-is-the-binary experience a bit thin.

   Given this complexity, and the fact that all the developers ar fluendians, people might easily look upon it as "your thing" and don't realize that they can hack and contribute too.

   This is by no means a unique situation though, Scratchbox is another project that is \*perceived\* to be a "Movial" thing, although it's open source and contributions are more than welcome. It's just a big lump of black magic, hard to wrap your head around and most people hacking it are on Movial payroll...

   Hopefully Scratchbox2 will have better luck, being a much smaller lump (though it still is black magic ;) and not started by a Movial employee.

   To attract developers, I guess you could produce some "how to hack flumotion" documentation and provide easyish-to-test examples like HiHo suggested. Also the trac has the phrase "Internal documentation" which immediately will translate as "only for employees of Fluendo" ;)

   Maybe some sort of hacking session at GUADEC? Or has there already been those?

3. Michael Scherer says:[4 May 2007 11:56 AM](#451)

   Developers are users before being hackers. So if they don't use your product, they will not develop and help you on this point.

   I recently had to stream and reencode a rtsp stream from my home server to be served on another one using http, it took me 2/3h to do it with vlc from scratch, but I do not even know where I should start to do it with flumotion ( if we do not take in account the fact rtsp is not finished in gstreamer and such problem ).

   The same could be said of a simple private webradio, with a list of file. Using icecast and ices, it is quite simple to setup, provided file are all in the same format.

   I think this could be easy to in flumotion ( seems there is a jukebox component, and there is some code in gstreamer for doing this if i am not wrong ), but there is no real doc on the config file format, and there is only a wizard to administrate with the gui.

   What about a doc on setting a server without the gui, some simple howto, and do some buzz around it ?

4. [Tiffany Carter](http://www.catatoniatoday.com) says:[4 May 2007 3:02 PM](#452)

   Oh my god. I think I am going to get hacking on that right away. I mean, after I wax.

5. Johan Dahlin says:[4 May 2007 4:15 PM](#453)

   I have a similar problem with code I write. I even put it in a separate library which has good documentation (api reference, howto), unit testing and everything.

   But even so people are not using it as much as I wished, it's really difficult to get people to start using your own code.

   Not even you guys are using any of it, even though it makes perfect sense for you to use. Perhaps I just need to blog a little bit more about it, to increase the awareness of all goodies.

6. [the phoneless -- andy wingo](http://wingolog.org/archives/2007/05/05/the-phoneless) says:[5 May 2007 2:21 AM](#454)

   \[...\] My last entry on Flumotion internals began with a rhetorical question, something like “why aren’t more people hacking Flumotion?” Unexpectedly, I got some real responses in the comments, all quite good. The general theme is that “developers are users first”. If your project doesn’t present itself well and offer a good initial experience, you’re shutting off potential contributors as well as users. So say HiHo, Kalle, and Michael, more eloquently than I. \[...\]

Comments are closed.
