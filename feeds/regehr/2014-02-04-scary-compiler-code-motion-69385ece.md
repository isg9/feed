---
title: Scary Compiler Code Motion
url: https://blog.regehr.org/archives/1097
published: "2014-02-04T21:18:42Z"
feed: regehr
guid: http://blog.regehr.org/?p=1097
---

# Scary Compiler Code Motion

I ran across this [blog post about reader-writer locks](http://jfdube.wordpress.com/2014/01/03/implementing-a-recursive-read-write-spinlock/) and thought it would be fun to present them to the Advanced Operating Systems class that I’m running this spring. Here’s my version of the code, which needed to be adapted a bit to work with GCC:

```
struct RWSpinLock
{
  volatile uint32_t lock;
};

void LockReader(struct RWSpinLock *l)
{
  while (1) {
    uint32_t OldLock = l->lock & 0x7fffffff;
    uint32_t NewLock = OldLock + 1;
    if (__sync_val_compare_and_swap(&l->lock, OldLock, NewLock) == OldLock)
      return;
  }
}

void UnlockReader(struct RWSpinLock *l)
{
  __sync_sub_and_fetch(&l->lock, 1);
}

void LockWriter(struct RWSpinLock *l)
{
 again:
  {
    uint32_t OldLock = l->lock & 0x7fffffff;
    uint32_t NewLock = OldLock | 0x80000000;
    if (!(__sync_val_compare_and_swap(&l->lock, OldLock, NewLock) == OldLock))
      goto again;
    while (l->lock & 0x7fffffff) { }
  }
}

void UnlockWriter(struct RWSpinLock *l)
{
  l->lock = 0;
}

```

At first I was suspicious that some sort of fence is required in the UnlockWriter() function. However, I believe that that is not the case: TSO will naturally make sure that all cores see operations inside the locked region prior to seeing the lock release. On some non-TSO platforms this unlock function would be too weak.

But the hardware is only one source of reordering that can mess up locks — the compiler is another. The store to the volatile lock field in UnlockWriter() cannot be moved around another access to a volatile-qualified object, but (in my reading of the standard) stores to non-volatile objects can be moved around stores to volatiles.

Can we get a compiler to translate the (in principle) broken code into assembly that actually doesn’t work? Turns out this is easy:

```
int important_data;
struct RWSpinLock l;

void update (void)
{
  LockWriter (&l);
  important_data /= 20;
  UnlockWriter (&l);
}

```

Using the default compiler on an x86-64 Ubuntu 12.04 machine (GCC 4.6.3) at -O2, the resulting assembly is:

**```** **update:** **movl $l, %edi** **call LockWriter** **movl important_data(%rip), %ecx** **movl $1717986919, %edx** **movl $0, l(%rip)** **movl %ecx, %eax** **sarl $31, %ecx** **imull %edx** **sarl $3, %edx** **subl %ecx, %edx** **movl %edx, important_data(%rip)** **ret**

**```**

Notice that important\_data is read inside the lock and written well outside of it. Yikes! Here’s how to stop the compiler from doing that:

```
void UnlockWriter(struct RWSpinLock *l)
{
  // this is necessary or else GCC will move operations out of the locked region
  asm volatile("" ::: "memory");
  l->lock = 0;
}

```

Versions of Clang that I tried (3.3 and 3.4 for x86-64) didn’t want to move the store to important\_data outside of the lock, but it’s possible that I just wasn’t asking the right way.
