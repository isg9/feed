---
title: Multi-Version Execution Defeats a Compiler-Bug-Based Backdoor
url: https://blog.regehr.org/archives/1282
published: "2015-11-23T14:51:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=1282
---

# Multi-Version Execution Defeats a Compiler-Bug-Based Backdoor

\[This piece is jointly authored by [Cristian Cadar](https://www.doc.ic.ac.uk/~cristic/), [Luí­s Pina](http://www.luispina.me/), and John Regehr\]

What should you do if you’re worried that someone might have [exploited a compiler bug to introduce a backdoor](https://www.alchemistowl.org/pocorgtfo/pocorgtfo08.pdf#page=7) into code that you are running? One option is to find a bug-free compiler. Another is to run versions of the code produced by multiple compilers and to compare the results (of course, under the additional assumption that the same bug does not affect all the compilers). For some programs, such as those whose only effect is to produce a text file, comparing the output is easy. For others, such as servers, this is more difficult and specialized system support is required.

Today we’ll look at using [Varan the Unbelievable](http://srg.doc.ic.ac.uk/projects/varan/) to defeat the sudo backdoor from the PoC\|\|GTFO article. Varan is a multi-version execution system that exploits the fact that if you have some unused cores, running additional copies of a program can be cheap. Varan designates a leader process whose system call activity is recorded in a shared ring buffer, and one or more follower processes that read results out of the ring buffer instead of actually issuing system calls.

Compilers have a lot of freedom while generating code, but the sequence of system calls executed by a program represents its external behaviour and in most cases the compiler is not free to change it at all. There might be slight variations e.g., due to different compilers using different libraries, but these can be easily handled by Varan. Since all correctly compiled variants of a program should have the same external behaviour, any divergence in the sequence of system calls across versions flags a potential security attack, in which case Varan stops the program before any harm is done.

Typically, Varan runs the leader process at full speed while also recording the results of its system calls into the ring buffer. However, when used in a security-sensitive setting, Varan can designate some system calls as blocking, meaning that the leader cannot execute those syscalls until all followers have reached that same program point without diverging. For sudo, we designate `execve` as blocking, since that is a point at which sudo might perform an irrevocably bad action.

So here’s the setup:

1. We have a patched version of sudo 1.8.13 from the PoC\|\|GTFO article. It runs correctly and securely when compiled by a correct C compiler, but improperly gives away root privileges when compiled by Clang 3.3 because the patch was designed to trigger a wrong-code bug in that compiler.

2. We are going to pretend that we don’t know about the Clang bug and the backdoor. We compile two versions of the patched sudo: one with Clang 3.3, the other with the default system compiler, GCC 4.8.4.

3. We run these executables under Varan. Since the critical system call `execve` is blocking, it doesn’t much matter which version is the leader and which is the follower.

Now let’s visit an Ubuntu 14.04 VM where both versions of sudo (setuid root, of course) and Varan are installed. We’re using a user account that is not in the sudoers file — it should not be allowed to get root privileges under any circumstances. First let’s make sure that a sudo that was properly compiled (using GCC) works as expected:

**```** **$ /home/varan/sudo-1.8.13/install/bin/sudo-gcc cat /etc/shadow** **Password:** **test is not in the sudoers file. This incident will be reported.**

**```**

Next, we make sure that the backdoor is functioning as intended:

**```** **$ /home/varan/sudo-1.8.13/install/bin/sudo-clang cat /etc/shadow** **Password:** **root:!:16693:0:99999:7:::** **...** **test:$6$nL9rosCD$A.prR28WWZYgDHy.m6ovWYaRhP/OhOX7V9nDDirm9ZffjTWhLdJ7C171wTfLBpQNC2EoeEKAJpcEz9gV9ravL/:16693:0:99999:7:::** **messagebus:*:16757:0:99999:7:::** **colord:*:16757:0:99999:7:::**

**```**

So far so good. Next let’s try the gcc-compiled sudo as the leader with the backdoored sudo as the follower:

**```** **$ vx-suid /home/varan/sudo-1.8.13/install/bin/sudo-gcc \** **/home/varan/sudo-1.8.13/install/bin/sudo-clang -- cat /etc/shadow** **Password:** **$** **test is not in the sudoers file. This incident will be reported.**

**```**

What happened here is that the gcc-compiled leader runs as before, since it doesn’t ever try to execute an `execve` call. When the backdoored follower tries to execute the malicious `execve` call, Varan detects the divergence and terminates both processes safely.

Now let’s try switching around the leader and follower, i.e., run the backdoored sudo as the leader with the gcc-compiled sudo as the follower:

**```** **$ vx-suid /home/varan/sudo-1.8.13/install/bin/sudo-clang \** **/home/varan/sudo-1.8.13/install/bin/sudo-gcc -- cat /etc/shadow** **Password:** **$**

**```**

This time the leader tries to execute the malicious `execve` call, and Varan blocks its execution until the follower reaches the same system call or diverges. In this case, the follower tries to execute a `write` system call (to print “ `test is not in the sudoers file...`“) and thus Varan detects divergence and again terminates execution safely.

In this example, we only ran two versions in parallel, but Varan can run more than two versions. In terms of performance and resource utilization, security applications like sudo are a great match for multi-version execution: they are not CPU-bound, so any performance degradation is imperceptible to the user, and the extra cores are needed only briefly, during the critical security validation checks. We are looking into applying this approach to other critical security applications (e.g. ssh-agent and password managers), and are investigating a way of hardening executables by generating a single binary with Varan and a bunch of versions, each version generated by a different compiler. We can then deploy this hardened executable instead of the original program.

Of course, Varan can detect misbehavior other than compiler-bug-based backdoors. Divergence could be caused by a memory or CPU glitch, by a plain old compiler bug that is triggered unintentionally instead of being triggered by an adversarial patch, or by a situation where an application-level undefined behavior bug has been exploited by only one of the compilers, or even where both compilers exploited the bug but not in precisely the same way. A nice thing about N-version programming at the system call level is that it won’t bother us about transient divergences that do not manifest as externally visible behaviour through a system call.

We’ll end by pointing out a piece of previous work along these lines: the Boeing 777 uses compiler-based and also hardware-based N-version diversity: there is a [single version of the Ada avionics software that is compiled by three different compilers and then it runs on three different processors](http://www.citemaster.net/get/db3a81c6-548e-11e5-9d2e-00163e009cc7/R8.pdf): a 486, a 68040, and an AMD 29050.
