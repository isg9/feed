---
title: 'research!rsc: Lessons from the Debian/OpenSSL Fiasco'
url: https://research.swtch.com/openssl
published: "2008-05-21T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/openssl
---

# research!rsc: Lessons from the Debian/OpenSSL Fiasco

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Lessons from the Debian/OpenSSL Fiasco Russ Cox May 21, 2008 *research.swtch.com/openssl* Posted on Wednesday, May 21, 2008.

Last week, Debian announced that in September 2006 they accidentally broke the OpenSSL pseudo-random number generator while trying to silence a Valgrind warning. One effect this had is that the `ssh-keygen` program installed on recent Debian systems (and Debian-derived systems like Ubuntu) could only generate 32,767 different possible SSH keys of a given type and size, so there are a lot of people walking around with [the same keys](http://github.com/blog/63-ssh-keys-generated-on-debian-ubuntu-compromised).

Many people have had fingers pointed at them, but it is not really interesting who made the mistake: everyone makes mistakes. What's interesting is the situation that encouraged making the mistake and that made it possible not to notice it for almost two years.

To do that, you have to understand the code involved and the details of the bug; those require understanding a little bit about entropy and random number generators.

**Entropy**

Entropy is a measure of how surprising a piece of data is. Jon Lester didn't pitch in last night's Red Sox game, but that's not surprising--he only pitches in every fifth game, so most nights he doesn't pitch. On Monday night, though, he did pitch and (surprise!) threw a no-hitter. No one expected that before the game: it was a high-entropy event.

Entropy can be measured in bits: a perfect coin flip produces 1 bit of entropy. A biased coin produces less: flipping a coin that comes up heads only 25% of the time produces only about 0.4 bits for tails and 2 bits for heads, or 0.8 bits of entropy on average. The exact details aren't too important. What is important is that entropy is a quantitative measure of the amount of unpredictability in a piece of data.

An ideal random byte sequence has entropy equal to its length, but most of the time we have to make do with lower-entropy sequences. For example, the dates of Major League no-hitters this year would be a very unpredictable sequence, but also a very short one. On the other hand, collecting the dates that Lester pitches for the Red Sox this year is fairly predictable—it's basically every fifth day—but there are still unexpected events like injuries and rain-outs that make it not completely predictable. The no-hitter sequence is very unpredictable but is very short. The Lester starts sequence is only a little unpredictable but so much longer that it likely has more total entropy.

Computers are faced with the same problem: they have access to long low-entropy (mostly predictable) sources, like the interarrival times between device interrupts or static sampled from a sound or video card, but they need high-entropy (completely unpredictable) byte sequences where every bit is completely unpredictable. To convert the former into the latter, a secure pseudo-random number generator uses a deterministic function that acts as an “entropy juicer.” It takes a large amount of low-entropy data, called the entropy pool, and then squeezes it to produce a smaller amount of high-entropy random byte sequences. Deterministic programs can't create entropy, so the amount of entropy coming out can't be greater than the amount of entropy going in. The generator has just condensed the entropy pool into a more concentrated form. Cryptographic hash functions like MD5 and SHA1 turn out to be good entropy juicers. They let every input bit have some effect on each output bit, so that no matter which input bits were the unpredictable ones, the output has a uniformly high entropy. (In fact this is the very essence of a hash function, which is why cryptographers just say hash function instead of made-up terms like entropy juicer.)

The bad part about entropy is that you can't actually measure it except in context: if you read a 32-bit number from `/dev/random`, as I just did, you're likely to be happy with 4016139919, but if you read ten more, you won't be happy if you keep getting 4016139919. (That last sentence is, technically, completely flawed, but you get the point. See also [this comic](http://xkcd.com/221/) and [this comic](http://web.archive.org/web/20011027002011/http://dilbert.com/comics/dilbert/archive/images/dilbert2001182781025.gif).) The nice thing about entropy juicers is that as long as there's some entropy somewhere in the pool, they'll extract it. It never hurts to add some more data to the pool: either it will add to the collective entropy, or it will have no effect.

**OpenSSL**

OpenSSL implements such a hash-based entropy juicer. It provides a function `RAND_add(buf, n, e)` that adds a buffer of length `n` and entropy `e` to the entropy pool. Internally, the entry pool is just an MD5 or SHA1 hash state: `RAND_add` calls `MD_update` to add the bytes to a running hash computation. The parameter `e` is an assertion made by the caller about the entropy contained in `buf`. OpenSSL uses it to keep a running estimate of the total amount of entropy in the buffer. OpenSSL also provides a `RAND_bytes(buf, n)` that returns a high-entropy random byte sequence. If the running estimate indicates that there isn't enough entropy in the pool, `RAND_bytes` returns an error. This is entirely reasonable.

The Unix-specific code in OpenSSL seeds the entropy juicer with some dodgy code. The essence of `RAND_poll` in `rand_unix.c` is (my words):

```
    char buf[100];
    fd = open("/dev/random", O_RDONLY);
    n = read(fd, buf, sizeof buf);
    close(fd);
    RAND_add(buf, sizeof buf, n);

```

Notice that the parameters to `RAND_add` say to use the entire buffer but only expect `n` bytes of entropy. `RAND_load_file` does a similar thing when reading from a file, and there the uninitialized reference is explicitly marked as intentional (actual code):

```
    i=fread(buf,1,n,in);
    if (i <= 0) break;
    /* even if n != i, use the full array */
    RAND_add(buf,n,i);

```

The rationale here is that including the uninitialized fragment at the end of the buffer might actually increase the actual amount of entropy, and in any event being honest about the amount of entropy being claimed won't break the entropy pool estimates.

Similarly, the function `RAND_bytes(buf, n)`, whose main purpose is to fetch `n` high-entropy bytes from the juicer, adds the contents of `buf` to the entropy pool (it behaves like `RAND_add(buf, n, 0)`) before it fills in `buf`.

In at least three different places, then, the OpenSSL developers explicitly chose to use uninitialized buffers as possible entropy sources. While this is theoretically defensible (it can't hurt), it's mostly a combination of voodoo and wishful thinking, and it makes the code difficult to understand and analyze.

In particular, the `RAND_bytes` convention causes problems at every call site that looks like:

```
    char buf[100];
    n = RAND_bytes(buf, sizeof buf);

```

This idiom causes so many warnings with Valgrind (and its commercial cousin, Purify) that there is an `#ifdef` to turn the behavior off:

```
    #ifndef PURIFY
                MD_Update(&m,buf,j); /* purify complains */
    #endif

```

The other two cases occur much more rarely: in `RAND_poll`, one would have to be reading from `/dev/random` when the kernel had very little randomness to spare, or calling `RAND_load_file` on a file that reported one size in `stat` but returned fewer bytes in `read`. When they do occur, a different instance of `MD_update`, the one in `RAND_add`, is on the stack trace reported by Valgrind.

**Debian**

A Debian maintainer was faced with Valgrind reports of uses of uninitialized data at a call to `MD_update`, which is used to mix a buffer into the entropy pool inside both `RAND_add` and `RAND_bytes`. Normally you might look farther up the call stack, but there were multiple instances, with different callers. The only common piece was `MD_update`. Since the second `MD_update` was already known to be non-essential—it could be safely disabled to run under Purify—it stood to reason that perhaps the first one was non-essential as well. The context of each call looks basically identical (the source code is not particularly well factored).

The Debian maintainer knew he didn't understand the code, so he [asked for help](http://marc.info/?t=114651088900003&r=1&w=2) on the *openssl-dev* mailing list:

```
    Subject:    Random number generator, uninitialised data and valgrind.
    Date:       2006-05-01 19:14:00

    When debbuging applications that make use of openssl using
    valgrind, it can show alot of warnings about doing a conditional
    jump based on an unitialised value.  Those unitialised values are
    generated in the random number generator.  It's adding an
    unintialiased buffer to the pool.

    The code in question that has the problem are the following 2
    pieces of code in crypto/rand/md_rand.c:

    247:
                    MD_Update(&m,buf,j);

    467:
    #ifndef PURIFY
                    MD_Update(&m,buf,j); /* purify complains */
    #endif

    Because of the way valgrind works (and has to work), the place
    where the unitialised value is first used, and the place were the
    error is reported can be totaly different and it can be rather
    hard to find what the problem is.

    ...

    What I currently see as best option is to actually comment out
    those 2 lines of code.  But I have no idea what effect this
    really has on the RNG.  The only effect I see is that the pool
    might receive less entropy.  But on the other hand, I'm not even
    sure how much entropy some unitialised data has.

    What do you people think about removing those 2 lines of code?

```

He got two substantive replies. The first reply was from someone with an email address at *openssl.org* who appears to be one of the developers:

```
    List:       openssl-dev
    Subject:    Re: Random number generator, uninitialised data and valgrind.
    Date:       2006-05-01 22:34:12

    > What I currently see as best option is to actually comment out
    > those 2 lines of code.  But I have no idea what effect this
    > really has on the RNG.  The only effect I see is that the pool
    > might receive less entropy.  But on the other hand, I'm not even
    > sure how much entropy some unitialised data has.
    >
    Not much. If it helps with debugging, I'm in favor of removing them.
    (However the last time I checked, valgrind reported thousands of bogus
    error messages. Has that situation gotten better?)

```

The second is from someone not at *openssl.org* who answers the developer's question:

```
    List:       openssl-dev
    Subject:    Re: Random number generator, uninitialised data and valgrind.
    Date:       2006-05-02 6:08:10

    > Not much. If it helps with debugging, I'm in favor of removing them.
    > (However the last time I checked, valgrind reported thousands of bogus
    > error messages. Has that situation gotten better?)

    I recently compiled vanilla OpenSSL 0.9.8a with -DPURIFY=1 and on Debian
    GNU/Linux 'sid' with valgrind version 3.1.1 was able to debug some
    application using both TLS/SSL as S/MIME without any warning or error
    about the OpenSSL code. Without -DPURIFY you're indeed flooded with
    warnings.

    So yes I think not using the uninitialized memory (it's only a single
    line, the other occurrence is already commented out) helps valgrind.

```

Both essentially said, “go ahead, remove the `MD_update` line.” The Debian maintainer did, causing `RAND_add` not to add anything to the entropy pool but still update the entropy estimate. There were other `MD_update` calls in the code that didn't use `buf`, and those remained. The only one that was a little unpredictable was one in `RAND_bytes` that added the current process ID to the entropy pool on each call. That's why OpenSSH could still generate 32,767 possible SSH keys of a given type and size (one for each pid) instead of just one.

**Cascade of Failures**

Like any true fiasco, a cascade of bad decisions and mistakes all lined up to make this happen:

- The OpenSSL code was too clever by half. The intentional uninitialized memory references obscured the code and made it hard to test for real bugs.

- The OpenSSL code isn't very well organized. There was code in both `RAND_add` and `RAND_bytes` to update entropy to the pool instead of having that functionality factored into one place. The fact that one instance wasn't essential made it look like the other instance wasn't essential.

- The OpenSSL code's paranoia hid the Debian-introduced bug. Throwing the pid into the entropy pool on each call to `RAND_bytes` isn't actually helping create entropy, but it does keep the buggy Debian version from being completely deterministic. If it had been completely deterministic, the bug would likely have been noticed much sooner, probably long before it got into a stable release.

- The Debian maintainer asked for help with code he didn't understand, but the snippets in his post to the OpenSSL list don't include enough context to understand where the `MD_update` calls really are in the code. Showing a few lines around each call wouldn't have made the situation clearer, since the two code sections look pretty similar. Mentioning the names of the functions containing each call might have helped.

- The OpenSSL developers responded incorrectly, probably without actually looking at the code to see which calls were being talked about. The presence of the #ifdef PURIFY on the one call likely made it very easy to assume the other call was similarly unimportant.

Reading various blogs you find various involved party's intelligence being insulted, but this is really a failure of process, not of intelligence. The maintainer knew he wasn't an expert on the code in question, asked the experts for help, and was given bad information.

**Lessons**

I've spent a lot of the past decade maintaining both [Plan 9](https://9p.io/) and a [port of the Plan 9 software to Unix](http://swtch.com/plan9port/). I've edited a lot of code I didn't fully understand, and I've answered questions about code that I did understand when other people came asking. I've made my share of embarrassing mistakes editing unfamiliar code, and I've probably given my share of unintentionally wrong advice. Even the best programmers are always going to make mistakes. The lessons of the fiasco, for me, are the steps that can be taken to make the mistakes less frequent and easier to find.

For me, the lessons I take away are:

- Try not to write clever code. Try to write well-organized code.

- Inevitably, you will write clever, poorly-organized code. If someone comes along asking questions about it, use that as a sign that perhaps the code is probably too clever or not well enough organized. Rewrite it to be simpler and easier to understand.

- Avoid voodoo code. Zeroing a variable multiple times, for example, doesn't affect correctness now, but it does make the code harder to understand and easier to break without noticing.

- Mailing list discussions aren't a substitute for real code review. People respond to email when they're tired or on their way out the door. Code reviews are supposed to be thorough and considered. Showing a side-by-side file diff of the before and after versions of `md_rand.c` to an OpenSSL developer as a real code review would likely have turned up the mistake.

- Distributions like Debian have to maintain their own copies of some programs at least temporarily. That's inevitable, because not all projects will run on Debian's time constraints. But I'm surprised there was no followup with the OpenSSL developers once the patch was created, trying to get them to accept it into the main tree. That could have provoked a code review too. Failing that, I'm surprised Debian doesn't have an engineer whose job it is to understand OpenSSL and other security-critical bits of code and vet local changes in a formal process.

In my own software maintenance, I think I'm doing a pretty good job on the first three, and not such a great job on the last two. It is tempting to say that for a low-profile project like Plan 9 they're not really needed, but that's more likely to be laziness than the truth.

**Links and Notes**

Here's the [Debian announcement](http://www.debian.org/security/2008/dsa-1571). I have not seen any account of how Luciano Bello discovered the bug. If you do, please post a comment.

Here's one of the original [Debian bug reports](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=363516) about Valgrind complaints. The original poster actually has a correct fix; the maintainer incorrectly (but reasonably) argues that since there are other contexts that also cause uninitialized references, the bug must be in `RAND_add` itself.

I've been talking about `RAND_add` and `RAND_bytes`, but in the code the actual functions are `ssleay_rand_add` and `ssleay_rand_bytes`, in `md_rand.c`. The latter are used as the implementation of the former. `RAND_poll` is in `rand_unix.c`, and `RAND_load_file` is in `randfile.c`.

(Comments originally posted via Blogger.)

- [Jake Nelson](http://www.blogger.com/profile/04489474946274002576)(May 21, 2008 6:32 AM) "If it helps with debugging" is not even close to the same as "Go ahead and remove it for production deployments" and portraying it as such is being disingenuous.

- [Lester](http://www.blogger.com/profile/10908210050410667635)(May 21, 2008 6:46 AM) Another tactic might have been to XOR the remaining 'untouched' bytes in the buffer. That might have served everyones purposes.

- [Russ Cox](http://swtch.com/~rsc/)(May 21, 2008 6:52 AM) That's interesting. I've read the discussion as being about cleaning up the production version of the code so that it is easier to debug. I hadn't noticed until your comment, but it is certainly possible to read the discussion as being about temporary changes to make just for debugging.

This just reinforces that mailing list discussion is not a substitute for code reviews.

- [Tonetheman](http://www.blogger.com/profile/10974821214602254645)(May 21, 2008 7:27 AM) First off I DO NOT see why Debian needs to change openssl in the first place. Perhaps I do not understand that part.

Then second, valgrid/purify is a tool. Blindly following the suggestions of this or any other source analyzer tool is NOT the business of anyone who is upstream of the real package developers. Sorry. If you do not understand something do not change it.

I have worked at places before with zealots that use Java source analyzers like a weapon. It is a joke and they are a joke.

Use you head. Think think think. Just because valgrind/purify says it does not make it right. Lint was the same way and so will every other source analysis tool.

- [Maht](http://www.blogger.com/profile/01863908675256558774)(May 21, 2008 7:50 AM) Relying on magic is bad form and not documenting magic is suicidal.

A simple

// use uninitialized data as entropy

would have helped until someone came along and said "let's zero the heap" for extra security.

- [josh reich](http://www.blogger.com/profile/10303114242550248619)(May 21, 2008 8:03 AM) Thanks for the post. It was interesting & well written. Now I don't feel so bad for blindly updating my debian machine without doing further digging.

- [Justin Mason](http://www.blogger.com/profile/16955170493368020909)(May 21, 2008 8:46 AM) Very good points about the dangers of "clever code", well put.

Regarding mailing lists being a poor substitute for proper code review -- that's true, but open source projects simply \_have\_ to use lists for this, there is no alternative. Speaking from experience, a correctly-operating, well-established open source project may have developers spanning multiple continents, and an opportunity for such side-by-side code review arises once every couple of years at best, given that.

Maybe with the new trend of open source code review tools based on Google's Mondrian, we may come up soon with a mechanism for distributed code review better than patches and a bug tracker or mailing list. Here's hoping.

By the way, I'd like to point out, since I've seen comments elsewhere indicating that this kind of issue would never have arisen in proprietary software -- this is sadly not the case. I've worked in companies where old software passes into the "maintainance phase" of its lifecycle, and is immediately passed to a team of fresh-from-university junior developers who have had little or no contact with the product in advance, little understanding of the code, little understanding of how to fix bugs systematically, and no contact with the original developers for reviews. As you might imagine, the results are occasionally disastrous... but since it's proprietary software, it's nowhere near as visible as the occasional disaster in open-source land.

- [Gábor](http://www.blogger.com/profile/14780006777195037772)(May 21, 2008 9:29 AM) Also let's not forget the reason for this patch: to silence some valgrind warning.

i think debian should not patch a package (especially such an important, central package) just because some code-checker tool showed some warnings.

if openssl would crash, or be unusable in some other way, then ok, do something about it. but just for a valgrind-warning? simply submitting the patch upstream would have been a much better solution.

- [Jeff Walden](http://www.blogger.com/profile/11969143527962555101)(May 21, 2008 9:58 AM) Don't write over-clever code is a good point, but even better would be to make sure every piece of code you write has tests. If you know what the right behavior is, guarantee that behavior is reproducible and won't make its way into a release. (Note you can do this even for randomness algorithms, albeit non-deterministically, as in [regress-211590.js](http://mxr.mozilla.org/mozilla/source/js/tests/js1_5/Regress/regress-211590.js). That potentially could have been effective here, although it's probably unlikely.) This isn't really a substitute for clean code (and more importantly, clean APIs), but at least you don't backslide now or when you eventually rewrite the code.

- [We Are Dave](http://www.blogger.com/profile/03444621708860354327)(May 21, 2008 10:07 AM) tonetheman, maht and Gábor make excellent points.

Two more points that haven't yet been given enough emphasis --

\\* The OpenSSL guys' "cathedral" attitude - straight out of the BSD charm school - has to shoulder some of the blame. Each project needs a one-stop feedback mechanism that is followed by multiple core maintainers. Power users like the Debian maintainers are the alpha and omega of open source; treating them as an inconvenient distraction is not just rude, it's self-destructive.

\\* Debian's attitude to patching remains an accident waiting to happen. Many other vendors have similarly flawed attitudes. Patching should be a distributor's act of ultimate desperation, not a market differentiator or a policy thing. If you need changes, submit them upstream, and if upstream turns them down, find out why not.

- [Russ Cox](http://swtch.com/~rsc/)(May 21, 2008 10:39 AM) @Jeff Walden: One of the most interesting and subtle things about this bug is that the PRNG would have passed any single-process randomness test I can imagine. (Definitely [regress-211590.js](http://www.koders.com/javascript/fid9ACE373CC97944615ECD74938B8A900938FD42E5.aspx).)

The PRNG was still generating a byte stream that, in isolation, was cryptographically random. The problem is that if you ran it and I ran it, we had a good chance of getting the same byte stream.

I'd be very interested to hear about a plausible test that would have found this problem. I can't think of one.

- [sam](http://www.blogger.com/profile/16751485442056650593)(May 21, 2008 4:29 PM) Luciano discovered the bug by accident: he needed to generate many prime numbers for one of his projects and was surprised that he was getting many duplicates.

- [Russ Cox](http://swtch.com/~rsc/)(May 21, 2008 4:40 PM) @sam: Do you have a link to a description of exactly what Luciano was doing? I believe you, but the obvious way to generate a lot of primes (using a single process) wouldn't have turned this up.

- [Aaron Denney](http://www.blogger.com/profile/15613957348593645695)(May 21, 2008 10:42 PM) The complaints about how you shouldn't listen to source-code analysis tools are correct in general, but wrong in this specific case. In C, reading from uninitialized memory invokes undefined behaviour. Valgrind properly warned that something hinky was going on.

- [Sam Hocevar](http://www.blogger.com/profile/16751485442056650593)(May 22, 2008 1:43 AM) @rsc: I'm afraid you will have to ask Luciano for details. All I have is this conversation on #debian-devel from a few days ago:

how was this found? inspection, or that someone got the same key generated twice?

has really an accident. I was needing many primes numbers... 0:-)

and you got the same numbers every time?

Sesse, not every time :P

- [kragen](http://kragen.livejournal.com/)(May 22, 2008 3:01 AM) I see a lot of comments castigating Kurt for wanting to shut Valgrind

up. For example:

\> Then second, valgrid \[sic\]/purify is a tool. Blindly following the

\> suggestions of this or any other source analyzer tool is NOT the

\> business of anyone who is upstream \[sic\] of the real package

\> developers.

Leaving aside that neither Valgrind nor Purify analyzes source code,

there's a very good reason to modify the code to shut it up: if it

emits a lot of warnings that don't indicate actual errors in the

program, then you can't use it to find actual errors. Valgrind is a

very useful tool, and it's worth going to some effort to get it to

work, and work on the production version of the software, especially

security-critical software like OpenSSL, where a buffer overflow or a

double free is likely to result in an unauthenticated-remote-root hole.

I think Luciano was generating a bunch of SSL certificates. The

obvious way to do this would be with a shell script; I don't know if

that's what he did. He said he got the same certificate five times in

24 hours of generating certificates!

- [kragen](http://kragen.livejournal.com/)(May 22, 2008 3:01 AM) Wow, Blogger comments sure do suck. Sorry about that.

- [drew](http://www.blogger.com/profile/03185140656482868193)(May 22, 2008 5:53 AM) It was also recently noticed by the folk at github.com, where they noticed seemingly-unrelated users generating the same public SSH keys.

[(Brief mention here.)](http://github.com/blog/63-ssh-keys-generated-on-debian-ubuntu-compromised)

- [matt](http://matt.ucc.asn.au/)(May 22, 2008 7:14 AM) Putting getpid() into the mix is (most likely) a sensible design decision - otherwise if a program fork()s and keeps using the same pool, the child and parent will generate the same "random" bytes.

- [Resuna](http://www.blogger.com/profile/11926139083455275005)(May 22, 2008 7:31 AM) The biggest failure, to me, is not documenting when you're being clever. Sometimes you have to be clever, and use clever code, but you should explain why you're doing it. I would much rather read a piece of code that only had an average of 0.8 comments per file, if all those comments were places where the author thought he was being clever and explained why.

If you write clever code, brag about it in the comments. It will help some poor fellow who doesn't understand it, and even if it turns out you weren't really being clever they'll like you better for having explained why it was simply obscure so they know whether it's safe to simplify it. And that "poor fellow" may well be you, five years from now.

Comments about comments:

\\* Medieval cathedrals were actually built using the bazaar system. Many of them fell down as a result. This terminology is really horrible.

\\* I agree with whoever suggested that if a debugging tool is telling you you're doing something stupid, maybe you are.

\\* Bug trackers are much better places to do this sort of thing. When googling for a bug dumps me in a bug tracker somewhere, I always take advantage of that to do a bit of a read about similar bugs, since I'm thinking about the problem at the time, and drop in comments and patches as appropriate.

\\* Putting getpid() in the mix is probably useful if (a) you're sure that's the last place you're forking, or (b) you're doing it after you fork().

- [Mark](http://www.blogger.com/profile/13732301290186175840)(May 22, 2008 12:13 PM) "Also let's not forget the reason for this patch: to silence some valgrind warning.

i think debian should not patch a package (especially such an important, central package) just because some code-checker tool showed some warnings."

The issue here wasn't warnings on OpenSSL, but that you got the warnings whenever you ran valgrind on software that used OpenSSL. This made it very hard to see any potential problems with your software because of the huge numbers of OpenSSL-related warnings.

That's also presumably why "If it helps with debugging" was interpreted as it being OK to use in production deployments - it's a production deployment of OpenSSL, possibly used to debug some client code.

- [viric](http://viric.livejournal.com/)(May 23, 2008 10:00 AM) Here you (all) speak a bit on how to comment code... I want to add some pence... once I learnt a good lesson:

\- The code is the solution to a problem. In comments, it helps if you write what's the problem the code solves. It doesn't help that much, if the comment explains \*again\* only the solution.

For me, the /\* even if n != i, use the full array \*/ is another example of redundant comment, instead of explaining the problem the next line solves: Increase the entropy using the uninitialized part of the buffer, as we already counted it with RAND\_add.

(if I understood correctly)

- [m](http://www.blogger.com/profile/02944591945765091680)(May 24, 2008 1:14 AM) 2 Justin:

side-by-side review is not only "sit side by side at the table", but also "see side by side old version and new version".

basically, I don't see any problem establishing proper software reviewing process in open source teams spanning many continents.

1\. Patch is sent for review.

2\. Reviewer checks the change thoroughly, with diffs and so on, and comments on those.

3\. Sender addresses the comments.

2 and 3 repeat until reviewer gives his clear approval.

4\. Sender submits the code to the trunk.

And approval from reviewer means that he shares the responsibility with the sender for whatever bugs were introduced.

I wonder, why such big thing like Debian still doesn't get the idea of reviewing.

- [m](http://www.blogger.com/profile/02944591945765091680)(May 24, 2008 1:18 AM) continuing..

and this is not a question of tools, the process could be set up with a simple diff. I really don't understand what were the obstacles.

btw, there is one Mondrian-clone by Guido van Rossum, called Rietveld.

- [Owen](http://www.blogger.com/profile/12326115153674468849)(April 19, 2011 7:26 AM) Why has no-one mentioned testing as being one of the problems here?

If there were a test that generated say 100000 keys and did some statistical analysis, the problem would have had > 95% of being caught.
