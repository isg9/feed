---
title: knocking on private back doors with the web browser — wingolog
url: https://wingolog.org/archives/2013/02/04/knocking-on-private-back-doors-with-the-web-browser
published: "2013-02-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/02/04/knocking-on-private-back-doors-with-the-web-browser
---

# knocking on private back doors with the web browser — wingolog

## [knocking on private back doors with the web browser](/archives/2013/02/04/knocking-on-private-back-doors-with-the-web-browser)

4 February 2013 8:47 AM

- [security](/tags/security)
- [javascript](/tags/javascript)
- [guile](/tags/guile)
- [port scanning](/tags/port%20scanning)

I woke up at five this morning with two things in my head.

One was, most unfortunately, Rebecca Black's [Friday](http://www.youtube.com/watch?v=kfVsfOSbJY0), except with "Monday" -- you know, "Monday, Monday, gettin' up on Monday, hack, hack, hack, hack, which thing should I hack?", et cetera.

The other was a faint echo of Patrick McKenzie's great [article on the practical implications of the recent Rails vulnerabilities](http://www.kalzumeus.com/2013/01/31/what-the-rails-security-issue-means-for-your-startup/). In particular, he pointed out that development web servers running only on your local laptop could be accessed by malicious web pages.

Of course this is not new, strictly speaking, but it was surprising to me in a way that it shouldn't have been. For example, Guile has a [command-line argument to run a REPL server on a local port](http://www.gnu.org/software/guile/manual/html_node/Command_002dline-Options.html). This is fantastic for interactive development and debugging, but it is also a vulnerability if you run a web server on the same machine. A malicious web page could request data from the default Guile listener port, which is a short hop away from rooting your machine.

I made a little test case this morning: a [local port scanner](//wingolog.org/pub/localhost-portscan.html). It tries to fetch `http://localhost:port/favicon.ico` for all ports between 1 and 10000 on your machine. (You can click that link; it doesn't run the scan until you click a button.)

If it finds a favicon, that indicates that there is a running local web application, which may or may not be vulnerable to CSRF attacks or other attacks like the Rails yaml attack.

If the request for a favicon times out, probably that port is vulnerable to some kind of attack, like the old [form protocol attack](http://www.remote.org/jochen/sec/hfpa/index.html).

If the request fails, the port may be closed, or the process listening at the port may have detected invalid input and closed the connection. We could run a timing loop to determine if a connection was made or not, but for now the port scanner doesn't output any information for this case.

In my case, the page finds that I have a probably vulnerable port 2628, which appears to be [dictd](http://packages.debian.org/sid/dictd).

Anyway, I hope that page is useful to someone. And if you are adding backdoors into your applications for development purposes, be careful!

## related articles

- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [there are no good constant-time data structures](/archives/2014/12/02/there-are-no-good-constant-time-data-structures)
- [scheme workshop 2014](/archives/2014/11/27/scheme-workshop-2014)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)

### 2 responses

1. [J Altfas](http://webserv.bmedctr.com) says:[8 February 2013 5:48 AM](#c58afdd697863a5d0b1e61fa083d6cf3c557ba75)

   Naturally anyone using the Internet is interested in security, or certainly ought to be.

   So I was inspired to try out the "local port scanner" you described. Sure enough, it printed out a very long list of "potentially vulnerable" ports, which left me curious about what was going on. That is to say, I didn't seem likely that all those ports were actuallly being listened on.

   Investigating further, using netstat (on Linux) showed just a few ports having listeners (beyond the expected IANA services ports), and these didn't correspond to ports in the web-generated list.

   Testing from anothe angle, I hacked up a simple tcl program that attempted connecting to each port < 65536, and if connected, sending an http request and reading the response. Here's a typical result:

   % source port-vul.tcl

   PORT..22 READ => data:

   SSH-2.0-OpenSSH\_6.1p1-hpn13v11

   PORT..25 READ => data:

   554 SMTP synchronization error

   PORT..465 READ => data:

   554 SMTP synchronization error

   PORT..587 READ => data:

   554 SMTP synchronization error

   PORT..631 READ => data:

   HTTP/1.1 200 OK

   Date: Thu, 07 Feb 2013 20:52:48 GMT

   Server: CUPS/1.6

   Connection: Keep-Alive

   Keep-Alive: timeout=30

   Content-Language: en\_US

   Content-Type: text/html; charset=utf-8

   Last-Modified: Sun, 12 Aug 2012 21:34:33 GMT

   Content-Length: 3792

   PORT..5432 READ => timed out (9156 reads)

   PORT..16385 READ => data:

   PORT..51324 READ => data:

   GET / HTTP/1.1

   %

   In other words, the resuts were consistent with the operations of the usual daemons, with exceptions of the "mystery" port 16385, whose associated process wasn't identified. However, the port's minimal, two-character response to the http header might suggest vulnerability is not problematic there.

   I admit I may be missing important lessons you were trying to teach, but it is not at all clear what your web-based scan was showing in reference to my running system and potential vulnerabilities. If there's more I should know, I'd appreciate the information.

   BTW, this is the tcl script:

   #port-vul.tcl

   set h 1

   for {set i 1} {$i $lt; 65536} {incr i} {

   if {!\[catch {socket localhost $i} r\]} {

   chan configure $r -blocking 0

   chan puts $r "GET / HTTP/1.1\\n"

   chan flush $r

   set a \[after 500 {

   set h 0

   }\]

   set h 1

   set n 0

   while {$h} {

   if {\[catch {read $r 256} err\]} {

   break

   } else {

   if {>> \[string length $err\] 0\]} {

   break

   }

   update

   incr n

   }

   }

   after cancel $a

   puts -nonewline \[format "PORT..%-10d" $i\]

   if {\[== $h 0\]} {

   puts "READ => timed out ($n reads)"

   } else {

   puts "READ => data:"

   foreach ln \[split $err "\\n"\] {

   puts " $ln"

   }

   }

   chan close $r

   }

   }

2. J McFarell says:[17 June 2016 3:24 PM](#58a31a2f9385baf2c66ba9bf066bb4bbf317e575)

   I realize this is a late reply. He's getting at this: Running a web server locally that can DO things on the computer means there is a potentially exploitable vulnerability. All a remote website that you visit has to do is send a few requests (e.g. in a hidden iframe) to your running localhost web server, which, for the sake of argument, grants root/Administrator access to the system if a certain sequence of requests are made, tell the localhost web server and the web app behind it to deploy malware (e.g. ransomware), and BOOM, compromised system.

   Then, deploy that to an advertising network and you're golden.

   Which is why everyone should run AdBlock Plus and Ghostery as this person does: http://cubicspot.blogspot.com/2014/03/why-i-run-adblock-plus-and-ghostery.html

Comments are closed.
