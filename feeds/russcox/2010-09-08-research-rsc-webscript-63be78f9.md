---
title: 'research!rsc: Webscript'
url: https://research.swtch.com/webscript
published: "2010-09-08T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/webscript
---

# research!rsc: Webscript

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Webscript Russ Cox September 8, 2010 *research.swtch.com/webscript* Posted on Wednesday, September 8, 2010.

Back in 2005, before JavaScript was required for everything on the web, I created a toy language called webscript, to let you script automatic web interactions. I wanted to use webscript to replace some clunky wget-based shell scripts we were using to do things like track packages and reboot computers. Two real webscript programs illustrate the language.

This used to work for tracking UPS packages:

```
#!./webscript

load "http://www.ups.com/WebTracking/track?loc=en_US"
find textbox "InquiryNumber1"
input "1z30557w0340175623"
find next checkbox
input "yes"
find prev form
submit
if(find "Delivery Information"){
 find outer table
 print
}else if(find "One or more"){
 print
}else{
 print "Unexpected results."
 find page
 print
}

```

And this used to work to connect to a APC power strip and reboot the computer named `yoshimi`:

```
#!./webscript

load "http://apc-reset/outlets.htm"
print
print "\n=============\n"
find "yoshimi"
find outer row
find next select
input "Immediate Reboot"
submit
print

```

I say that the scripts used to work because I haven't run them in a few years. One of the goals of webscript was to let the programmer describe the interactions with the web page instead of working with the raw form parameters, so that scripts would even as the hidden plumbing behind the web page changed. Even so the UPS script mentions `InquiryNumber1`.

Every few years I go looking for a replacement for webscript, but I never find anything. As more web sites require JavaScript to do anything useful, the job of implementing something like webscript gets harder. I think you'd essentially have to link against a browser to make it work.

Do you know of a tool like webscript that works with today's web pages? Do you know an easy way to make one? (I know about AppleScript and [Chickenfoot](http://groups.csail.mit.edu/uid/chickenfoot/index.php), but I'd prefer something that can run and produce output as a command line program, without even displaying a web browser on my screen, so that it can be used in shell scripts and cron jobs.)

(Comments originally posted via Blogger.)

- [Fedor Soreks](http://www.blogger.com/profile/11628417257537497590)(September 8, 2010 7:06 AM) Perl (via cpan) has http://search.cpan.org/dist/WWW-Mechanize/ which seems to do the thing.

Other langs may have something similar.

- [Marshall](http://www.blogger.com/profile/01283557892488990606)(September 8, 2010 8:02 AM) Maybe [HtmlUnit](http://htmlunit.sourceforge.net/)? I have it used it, but I ran across it a while ago, and the elevator pitch looks like what you want.

- [jnwhiteh](http://www.blogger.com/profile/10740626841132661339)(September 8, 2010 8:03 AM) There was an interesting article on TUAW a few days ago that talks about bringing Automator-level scripts to Mac OS X with an application called 'Fake.app'. It's no scripting language, but it's an interesting take on the idea:

[http://www.tuaw.com/2010/09/06/fake-app-makes-powerful-web-automation-easy/](http://www.tuaw.com/2010/09/06/fake-app-makes-powerful-web-automation-easy/)

- [radar](http://www.blogger.com/profile/05307602198223780471)(September 8, 2010 8:11 AM) Ruby's Mechanize tool:

http://mechanize.rubyforge.org/mechanize/GUIDE\_rdoc.html

For example:

require 'rubygems'

require 'mechanize'

agent = Mechanize.new

page = agent.get('http://google.com/')

google\_form = page.form('f')

google\_form.q = 'ruby mechanize'

page = agent.submit(google\_form)

pp page

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(September 8, 2010 9:16 AM) i'm glad you mentioned this. i had a vague memory of reading a paper about this (i though i remembered it in a usenix proceedings) but totally failed to find it when i looked for it.

i'd love something like this. maybe there's a way of twisting webkit towards this purpose.

none of the solutions listed above seem to quite fit the bill: either they don't support javascript, or they require a connection to a real graphical browser.

- [Michael](http://www.blogger.com/profile/14618340371367353616)(September 8, 2010 12:27 PM) Michael Bolin, creator of Chickenfoot here. In the Related Work chapter of my thesis (the main topic was Chickenfoot, but the title was "End-User Programming for the Web" http://bolinfest.com/Michael\_Bolin\_Thesis\_Chickenfoot.pdf), I discuss a lot of alternatives of Chickenfoot and their shortcomings. The main problem with browserless approaches such as Mech and HtmlUnit is that so many pages are dynamically generated with the browser these days that you really need the rendering engine and the JavaScript interpreter to properly produce them. That being said, it would be reasonable to run a browser off-screen in VNC or something (which I have done when using Chickenfoot for automated testing) and then communicate with Chickenfoot through that via a command-line interface. If I had more cycles, I would certainly like to invest the time in making that happen. Even better would be to have the Chickenfoot API working on top of WebDriver so it could be used across browsers.

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(September 9, 2010 6:49 AM) @Michael: is there a reason that the rendering engine needs to actually render the grapics? couldn't it just remember where the various objects are without drawing the pixels?

- [Michael](http://www.blogger.com/profile/14618340371367353616)(September 9, 2010 6:58 AM) @rog Probably, but I imagine that requires messing with the Firefox source code. Offhand, I would bet that if you tried to launch Firefox on a Linux machine that did not have XWindows installed, it would crash. Though maybe someone has added a mode to support headless Firefox already? In the absence of having an extension installed (or having some programmatic API to use once it were running), it is not clear as to what you could do with headless mode. So yes, I absolutely believe it should be possible, but requires some hacking on somebody's part.

- [Maht](http://www.blogger.com/profile/01863908675256558774)(September 9, 2010 10:51 AM)This post has been removed by the author.

- [Maht](http://www.blogger.com/profile/01863908675256558774)(September 9, 2010 10:55 AM) node.js / Jsdom / jQuery is what you want

http://blog.nodejitsu.com/jsdom-jquery-in-5-lines-on-nodejs

Which runs server side, no rendering.

- [Kevin Krouse](http://www.blogger.com/profile/13206024250829484576)(September 9, 2010 1:41 PM) Envjs (http://env-js.appspot.com/) runs a headless simulated browser in javascript. You can write scripts for it using Rhino and jQuery.

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(September 10, 2010 12:16 AM) jQuery still doesn't scratch the original webscript itch, because it requires knowing how a page is implemented internally rather than just how it looks. (which is not to say that this approach couldn't be useful)

- [charmless](http://www.blogger.com/profile/08540566813510508054)(September 10, 2010 2:58 PM) Have a look at Selenium and it's Selenium IDE. The IDE is a visual tool that lets you record (pretty much straight-line) interactions with a website. (it's primary purpose is as a testing tool.)

But you can output the selenium test to any number of languages, and have as much structured control flow as you like. This is accomplished by running a server program that actually launches a browser and directs it's actions step-by-step, so it's certainly more authentic than most of the browser-simulating toolkits.

- [Marshall](http://www.blogger.com/profile/01283557892488990606)(September 12, 2010 10:55 AM) @Michael I'm not sure what the problem is w/ HtmlUnit regarding pages which dynamically generate themselves with JavaScript -- the project claims it has "fairly good JavaScript support (which is constantly improving) and is able to work even with quite complex AJAX libraries."

If that isn't good enough, there's always WebKit. The Qt bindings to WebKit don't require actually rendering the page and provide various DOM manipulation functions up to and including executing JavaScript in the context of a page element. Due to Qt randomness, it does require successful communication w/ an X server to run, even if you never actually create a window or paint to the screen.

- [Marshall](http://www.blogger.com/profile/01283557892488990606)(September 12, 2010 10:55 AM)This post has been removed by the author.

- [gertrud](http://www.blogger.com/profile/05121721102795724874)(September 30, 2010 3:55 AM) Russ, do you have your original implementation lying around?

I would like to play with it. My pc is too slow for all these java scripts anyway, so I've been using mostly awk and sed for my web scripts.

Next we want flash support, and captcha recognition...

- [dhobsd](http://www.blogger.com/profile/12275921831209319945)(December 10, 2010 9:40 AM) Hey Russ,

You might like to take a look at Selenium, which is designed to be a browser testing framework, but I imagine could similarly be instrumented to do general browser automation as well. It works with many browsers:

http://seleniumhq.org/

--dho

- [zsbana](http://www.blogger.com/profile/16556953218810348871)(January 28, 2011 7:58 AM) There's also the new perl module WWW::Mechanize::Firefox which is one of those tools that run a full-blown browser.
