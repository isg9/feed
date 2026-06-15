---
title: Use of Goto in Systems Code
url: https://blog.regehr.org/archives/894
published: "2013-02-05T04:28:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=894
---

# Use of Goto in Systems Code

The [goto wars](http://c2.com/cgi/wiki?GotoConsideredHarmful) of the 1960s and 1970s are long over, and goto is dead—except that it isn’t. It has a number of legitimate uses in well-structured code including an idiom seen in systems code where a failure partway through a sequence of stateful operations necessitates unwinding the operations that have already completed. For example:

```
int init_device (void) {
  if (allocate_memory() != SUCCESS) goto out1;
  if (setup_interrupts() != SUCCESS) goto out2;
  if (setup_registers() != SUCCESS) goto out3;
  .. more logic ...
  return SUCCESS;
 out3:
  teardown_interrupts();
 out2:
  free_memory();
 out1:
  return ERROR;
}

```

Is goto necessary to make this code work? Certainly not. We can write this instead:

```
int init_device (void) {
  if (allocate_memory() == SUCCESS) {
    if (setup_interrupts() == SUCCESS) {
      if (setup_registers() == SUCCESS) {
        ... more logic ...
        return SUCCESS;
      }
      teardown_interrupts();
    }
    free_memory();
  }
  return ERROR;
}

```

And in fact a decent compiler will turn both of these into the same object code, or close enough. Even so, many people, including me, prefer the goto version, perhaps because it doesn’t result in as much unsightly indentation of the central part of the function.

Tonight’s mainline Linux kernel contains about 100,000 instances of the keyword “goto”. Here’s a [nice clean goto chain of depth 10](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/fs/nfs/inode.c?id=44549e8f5eea4e0a41b487b63e616cb089922b99#n2046).

Here are the goto targets that appear more than 200 times:

out (23228 times)

error (4240 times)

err (4184 times)

fail (3250 times)

done (3179 times)

exit (1825 times)

bail (1539 times)

out\_unlock (1219 times)

err\_out (1165 times)

out\_free (1053 times)

nla\_put\_failure (929 times)

failed (849 times)

out\_err (841 times)

unlock (831 times)

cleanup (713 times)

drop (535 times)

retry (533 times)

again (486 times)

end (469 times)

bad (454 times)

errout (376 times)

err1 (362 times)

found (362 times)

error\_ret (331 times)

error\_out (276 times)

err2 (271 times)

fail1 (264 times)

err\_free (262 times)

next (260 times)

out1 (242 times)

leave (240 times)

abort (228 times)

restart (224 times)

badframe (221 times)

out2 (218 times)

error0 (208 times)

fail2 (208 times)

“goto out;” is indeed a classic. This kind of code has been on my mind lately since I’m trying to teach my operating systems class about how in kernel code, you can’t just bail out and expect someone else to clean up the mess.
