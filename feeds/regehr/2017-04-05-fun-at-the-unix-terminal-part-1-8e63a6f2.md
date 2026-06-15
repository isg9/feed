---
title: Fun at the UNIX Terminal Part 1
url: https://blog.regehr.org/archives/1483
published: "2017-04-05T20:04:48Z"
feed: regehr
guid: http://blog.regehr.org/?p=1483
---

# Fun at the UNIX Terminal Part 1

This post is aimed at kids, like the 6th graders who I was recently teaching about programming in Python. It is more about having fun than about learning, but I hope that if you enjoy playing around at the UNIX terminal, you’ll eventually learn to use this kind of system for real. Keep in mind this immortal [scene from Jurassic Park](https://www.youtube.com/watch?v=dFUlAQZB9Ng).

To run the commands in this post, you’ll need a UNIX machine: either Linux or Mac OS X will work. You’ll also need the ability to install software. There are two options:

- Install precompiled binaries using a package manager, I’ll give command lines for Homebrew on OS X and for Apt on Ubuntu Linux. You’ll need administrator access to run Apt or to install Homebrew, but you do not need administrator access to install packages after Homebrew has been installed. Other versions of Linux have their own package managers and they are all pretty easy to use.

- Build a program from source and install it in your home directory. This does not require administrator access but it’s more work and I’m not going to go into the details, though I hope to do this in a later post.

## ROT13 using tr

The tr utility should be installed by default on an OS X or Ubuntu machine. It translates the characters in a string into different characters according to rules that you provide. To learn more about tr (or any other command in this post) type this command (without typing the dollar sign):

```
$ man tr

```

This will show you the UNIX system’s built-in documentation for the command.

In this and subsequent examples, I’ll show text that you should type on a line starting with a dollar sign, which is the default UNIX prompt. Text printed by the system will be on lines not starting with a dollar sign.

We’re going to use tr to encrypt some text as ROT13, which simply moves each letter forward in the alphabet by 13 places, wrapping around from Z to A if necessary. Since there are 26 letters, encrypting twice using ROT13 gives back the original text. ROT13 is fun but you would not want to use it for actual secret information since it is trivial to decrypt. It is commonly used to make it hard for people to accidentally read spoilers when discussing things like movie plot twists.

Type this:

```
$ echo 'Hello this is a test' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
Uryyb guvf vf n grfg

```

Now to decrypt:

```
$ echo 'Uryyb guvf vf n grfg' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
Hello this is a test

```

Just two more things before moving on to the next command.

First, the UNIX pipe operator (the “\|” character in the commands above, which looks a little bit like a piece of pipe) is plumbing for UNIX commands: it “pipes” the output of one command to the input of a second command. We’ll be using it quite a bit.

Second, how exactly did we tell tr to implement ROT13? Well, the first argument, ‘A-Za-z’, gives it a set of characters to work with. Here A-Z stands for A through Z and a-z stands for a through z (computers treat the capital and lowercase versions of letters as being separate characters). So we are telling tr that it is going to translate any letter of the alphabet and leave any other character (spaces, punctuation, numbers, etc.) alone. The second argument to tr, ‘N-ZA-Mn-za-m’, specifies a mapping for the characters in the first argument. So the first character in the first argument (A) will be translated to the first character of the second argument (N), and so on. We could just as easily use tr to put some text in all uppercase or all lowercase, you might try this as an exercise.

## fortune

Tragically, this command isn’t installed by default on a Mac or on an Ubuntu Linux machine. On a Mac you can install it like this:

```
brew install fortune

```

If this doesn’t work then you need to get Homebrew setup, [try this page](https://brew.sh/).

On Ubuntu try this:

```
sudo apt-get install fortune

```

The “sudo” command will ask you to enter your password before running the next command, apt-get, with elevated privileges, in order to install the fortune program in a system directory that you are normally not allowed to modify. This will only work if your machine has been specifically configured to allow you to run programs with elevated privileges.

In any case, if you can’t get fortune installed, don’t worry about it, just proceed to the next command.

Fortune randomly chooses from a large collection of mildly humorous quotes:

```
$ fortune
I have never let my schooling interfere with my education.
		-- Mark Twain

```

## say

This command is installed by default on a Mac; on Ubuntu you’ll need to type “sudo apt install gnustep-gui-runtime”.

Type this:

```
$ say "you just might be a genius"

```

Make sure you have sound turned up.

The Linux say command, for whatever reason, requires its input to be a command line argument, so we cannot use a pipe to send fortune’s output to say. So this command will not work on Linux (though it does work on OS X):

```
$ fortune | say

```

However, there’s another trick we can use: we can turn the output of one command into the command-line arguments for another command by putting the first command in parentheses and prefixing this with a dollar sign. So this will cause your computer to read a fortune aloud:

```
$ say $(fortune)

```

Another way to accomplish the same thing is to put fortune’s output into a file and then ask say to read the file aloud:

```
$ fortune > my_fortune.txt
$ say -f my_fortune.txt

```

Here the greater-than symbol “redirects” the output of fortune into a file. Redirection works like piping but the output goes to a file instead of into another program. It is super useful.

If you run “say” on both a Linux box and a Mac you will notice that the Mac’s speech synthesis routines are better.

## cowsay

The extremely important cowsay command uses ASCII art to show you a cow saying something. Use it like this:

```
$ fortune | cowsay
 __________________________________
/ What is mind? No matter. What is \
| matter? Never mind.              |
|                                  |
\ -- Thomas Hewitt Key, 1799-1875  /
 ----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```

Both Homebrew and Apt have a package called “cowsay” that you can install using the same kind of command line you’ve already been using.

Cowsay has some exciting options, such as “-d” which makes the cow appear to be dead:

```
$ fortune | cowsay -d
 _________________________________________
/ Laws are like sausages. It's better not \
| to see them being made.                 |
|                                         |
\ -- Otto von Bismarck                    /
 -----------------------------------------
        \   ^__^
         \  (xx)\_______
            (__)\       )\/\
             U  ||----w |
                ||     ||

```

Use “man cowsay” to learn more.

## ponysay

Don’t install ponysay unless you feel that cowsay is too restrictive. Also, it isn’t available as a precompiled package. You can build it from source code by first installing the “git” package using apt-get or brew and then running the following commands:

```
$ git clone https://github.com/erkin/ponysay.git
$ cd ponysay
$ ./setup.py --freedom=partial --private install

```

This procedure puts ponysay into an odd location, but whatever. Here (assuming Linux, on a Mac you’ll need to pipe a different command’s output to ponysay) a cute pony tells us the prime factorization of a number:

![](../extra_files/ponysay.png)

## figlet

Figlet (actually called FIGlet but that’s not what you type to run the command) prints text using large letters comprised of regular terminal characters. For example:

```
$ whoami | figlet
                     _
 _ __ ___  __ _  ___| |__  _ __
| '__/ _ \/ _` |/ _ \ '_ \| '__|
| | |  __/ (_| |  __/ | | | |
|_|  \___|\__, |\___|_| |_|_|
          |___/

```

Figlet has lots of options for controlling the appearance of its output. For example, you can change the font:

```
$ echo 'hello Eddie' | figlet -f script
 _          _   _          ___
| |        | | | |        / (_)   |     |  o
| |     _  | | | |  __    \__   __|   __|      _
|/ \   |/  |/  |/  /  \_  /    /  |  /  |  |  |/
|   |_/|__/|__/|__/\__/   \___/\_/|_/\_/|_/|_/|__/

```

Another command, toilet, is similar to figlet but has even more options. Install both of these programs using the same kinds of commands we’ve already been using to talk to package managers.

## lolcat

The UNIX “cat” program prints a file, or whatever is piped into it, to the terminal. Lolcat is similar but it prints the text in awesome colors:

![](../extra_files/lolcat.png)

## bb

The bb program doesn’t seem to be available from Homebrew, but on Ubuntu you can install it using “sudo apt-get install bb”. It is a seriously impressive ASCII art demo.

## rig

You know how lots of web sites want you to sign up using your name and address, but your parents hopefully have trained you not to reveal your identity online? Well, the rig utility can help, it creates a random identity:

```
$ rig
Juana Waters
647 Hamlet St
Austin, TX  78710
(512) xxx-xxxx

```

The zip codes and telephone area codes are even correct. For some reason rig will never generate an identity that lives in Utah.

## bc

The bc program is a calculator but unlike almost every other calculator you use, it can handle numbers of unlimited size (or, more precisely, numbers limited only by the amount of RAM in your computer) without losing precision. Try this:

```
$ echo '2 ^ 100' | bc
1267650600228229401496703205376

```

Unfortunately bc does not have a built-in factorial function but you can write one easily enough using bc’s built-in programming language. Start bc in interactive mode (this will happen by default if you don’t pipe any text into bc) by just typing “bc”. Then enter this code:

```
define fact(n) {
  if (n < 2) return 1;
  return n * fact(n - 1);
}

```

Now you can compute very large factorials:

```
fact(1000)
40238726007709377354370243392300398571937486421071463254379991042993\
85123986290205920442084869694048004799886101971960586316668729948085\
58901323829669944590997424504087073759918823627727188732519779505950\
99527612087497546249704360141827809464649629105639388743788648733711\
91810458257836478499770124766328898359557354325131853239584630755574\
09114262417474349347553428646576611667797396668820291207379143853719\
58824980812686783837455973174613608537953452422158659320192809087829\
73084313928444032812315586110369768013573042161687476096758713483120\
25478589320767169132448426236131412508780208000261683151027341827977\
70478463586817016436502415369139828126481021309276124489635992870511\
49649754199093422215668325720808213331861168115536158365469840467089\
75602900950537616475847728421889679646244945160765353408198901385442\
48798495995331910172335555660213945039973628075013783761530712776192\
68490343526252000158885351473316117021039681759215109077880193931781\
14194545257223865541461062892187960223838971476088506276862967146674\
69756291123408243920816015378088989396451826324367161676217916890977\
99119037540312746222899880051954444142820121873617459926429565817466\
28302955570299024324153181617210465832036786906117260158783520751516\
28422554026517048330422614397428693306169089796848259012545832716822\
64580665267699586526822728070757813918581788896522081643483448259932\
66043367660176999612831860788386150279465955131156552036093988180612\
13855860030143569452722420634463179746059468257310379008402443243846\
56572450144028218852524709351906209290231364932734975655139587205596\
54228749774011413346962715422845862377387538230483865688976461927383\
81490014076731044664025989949022222176590433990188601856652648506179\
97023561938970178600408118897299183110211712298459016419210688843871\
21855646124960798722908519296819372388642614839657382291123125024186\
64935314397013742853192664987533721894069428143411852015801412334482\
80150513996942901534830776445690990731524332782882698646027898643211\
39083506217095002597389863554277196742822248757586765752344220207573\
63056949882508796892816275384886339690995982628095612145099487170124\
45164612603790293091208890869420285106401821543994571568059418727489\
98094254742173582401063677404595741785160829230135358081840096996372\
52423056085590370062427124341690900415369010593398383577793941097002\
77534720000000000000000000000000000000000000000000000000000000000000\
00000000000000000000000000000000000000000000000000000000000000000000\
00000000000000000000000000000000000000000000000000000000000000000000\
0000000000000000000000000000000000000000000000000000

```

While we're at it, you should figure out why the factorial of any large number contains a lot of trailing zeroes.

## Conclusion

We've only scratched the surface, I'll share more entertaining UNIX commands in a followup post. Some of these programs I hadn't even heard of until recently, a bunch of people on Twitter gave me awesome suggestions that I've used to write this up. If you want to see a serious (but hilariously outdated) video about what you can do using the UNIX command line, check out [this video](https://www.youtube.com/watch?v=tc4ROCJYbm0) in which one of the original creators of UNIX shows how to build a spell checker by piping together some simple commands.
