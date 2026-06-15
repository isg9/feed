---
title: 'research!rsc: Bourne Shell Macros'
url: https://research.swtch.com/shmacro
published: "2008-02-15T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/shmacro
---

# research!rsc: Bourne Shell Macros

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Bourne Shell Macros Russ Cox February 15, 2008 *research.swtch.com/shmacro* Posted on Friday, February 15, 2008.

Steve Bourne\* worked on an [Algol 68](http://en.wikipedia.org/wiki/ALGOL_68) compiler at the [Cambridge University Computer Lab](http://www.cl.cam.ac.uk/) before moving to Bell Labs, where he worked on [Seventh Edition Unix](https://9p.io/7thEdMan/). He is best known for writing V7's `/bin/sh`, now known as the Bourne shell.

Bourne's preference for Algol syntax is evident in shell scripts, which use Algol constructs like `if ... then ... elif ... else ... fi` and `case ... esac` instead of C equivalents that might be more familiar to Unix users. (The [C shell](http://www.mkssoftware.com/docs/man1/csh.1.asp) and [*rc*](https://9p.io/sys/doc/rc.html) were both inspired, in part, by a desire for C-like syntax.)

Bourne's preference for Algol syntax is, surprisingly, also evident in the [**shell source**](http://www.google.com/codesearch?hl=en&q=show:AGRYxgHSzlc:L709QqsJUc8). More comfortable with Algol but stuck with C, Bourne found a way to cope:

```
/usr/src/cmd/sh/mac.h:
#define IF      if(
#define THEN    ){
#define ELSE    } else {
#define ELIF    } else if (
#define FI      ;}

#define BEGIN   {
#define END     }
#define SWITCH  switch(
#define IN      ){
#define ENDSW   }
#define FOR     for(
#define WHILE   while(
#define DO      ){
#define OD      ;}
#define REP     do{
#define PER     }while(
#define DONE    );
#define LOOP    for(;;){
#define POOL    }
```

Using these macros, one can write programs that would certainly feel familiar to an Algol programmer:

```
/usr/src/cmd/sh/service.c:297:
LOCAL VOID      gsort(from,to)
        STRING          from[], to[];
{
        INT             k, m, n;
        REG INT         i, j;

        IF (n=to-from)<=1 THEN return FI

        FOR j=1; j<=n; j*=2 DONE

        FOR m=2*j-1; m/=2;
        DO  k=n-m;
            FOR j=0; j=0; i-=m
                DO  REG STRING *fromi; fromi = &from[i];
                    IF cf(fromi[m],fromi[0])>0
                    THEN break;
                    ELSE STRING s; s=fromi[m]; fromi[m]=fromi[0]; fromi[0]=s;
                    FI
                OD
            OD
        OD
}
```

(It's not clear why Bourne used braces around function bodies instead of `BEGIN` and `END`.)

The Bourne shell code, along with the [4.2BSD finger implementation](http://swtch.com/42bsd-finger.c.txt), was the impetus for the [International Obfuscated C Code Contest](http://www.ioccc.org/all/README).

\\* Really, Stephen R. Bourne, but even then not quite specific enough. Years ago, I heard a story that at one point there were two Stephen R. Bournes working at SGI. Callers would ask the operator for Steve Bourne and would be asked, ‘which one?’ ‘Steven R. Bourne,’ they'd say. ‘Sure, which one?’ The ambiguity was apparently resolved by referring to them as ‘hardware Steve’ and ‘software Steve.’ I can find no justification for this story nor any trace of ‘hardware Steve’ other than the mention of two Steven R. Bournes in the [sendmail README](http://www.google.com/codesearch?hl=en&q=+two%5Cs%2Bstephen%5Cs%2Br.%3F%5Cs%2Bbourne+show:1JoR4XwCs08:l5mWgw3iF7E:YDKvvHBSpiI&sa=N&cd=1&ct=rc&cs_p=http://ftp.kfki.hu/Sun/sun-fixes/2.6_Recommended.tar.Z&cs_f=2.6_Recommended/105395-09/SUNWcsu/reloc/usr/lib/mail/README#first), which claims that they were both at Bell Labs, not SGI. If you can confirm or deny this story, please post a comment or mail me ( *rsc@swtch.com*).

(Comments originally posted via Blogger.)

- [pixelbeat](http://www.blogger.com/profile/09859979434686189342)(February 15, 2008 7:12 AM) [Justin Mason just blogged about this](http://taint.org/2008/02/14/170208a.html)

- [fanf](http://www.blogger.com/profile/17320082374768944309)(February 15, 2008 12:41 PM) Cambridge's influence on Unix programming languages is interesting and amusing.

Martin Richards created BCPL, which inspired dmr to create B then C. BCPL was a by-product of the CPL project, which aimed to create something like a PL/1 for the Atlas/Titan computers, but was an utter failure.

Steve Bourne collaborated with Chris Cheney on the Algol 68 compiler. Chris invented the Cheney copying garbage collector, which is still a standard technique. Chris recently retired, promising to resurrect his Algol 68 project.

When Bjarne Stroustrup did his PhD at the computer lab, BCPL was the standard system programming language. His experience of struggling with it inspired him to develop C++.

Not a Unix connection, but BCPL was the development language for the TRIPOS research operating system, which was another of Martin Richards' projects. It went on to be the core of the Amiga OS.

- [Stuart](http://www.blogger.com/profile/16907976357342906712)(September 8, 2009 2:02 AM) “The Bourne shell code, along with the 4.2BSD finger implementation, was the impetus for the International Obfuscated C Code Contest.''

Speaking of which, here's two familiar names for you ... you may have already seen this:

http://www.ioccc.org/1990/tbr.hint
