---
title: 'research!rsc: Backups, heal thyself'
url: https://research.swtch.com/backups
published: "2008-04-13T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/backups
---

# research!rsc: Backups, heal thyself

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Backups, heal thyself Russ Cox April 13, 2008 *research.swtch.com/backups* Posted on Sunday, April 13, 2008.

![MIT venti server](https://research.swtch.com/venti.jpg)

Around 2000, Sean Quinlan and Sean Dorward built a content-addressed storage system called Venti. Their paper, “ [**Venti: a new approach to archival storage**](https://9p.io/sys/doc/venti/venti.pdf)” (PDF; also [HTML](https://9p.io/sys/doc/venti/venti.html)), won best paper at the storage conference where it was presented. Today's post is about how Venti can heal itself.

Venti names stored objects by their SHA1 hashes: you write a block, you get back the SHA1 hash. You show up with a SHA1 hash, you can get the block. Objects larger than a block are typically broken up into blocks and then stored in a tree. Each pointer in the tree is the SHA-1 hash of the block it points to:

![Hash trees](https://research.swtch.com/tree1.png)

Thus, if you've stored a big file (say, a disk image) and you change one block, only that one block and the (probably five or so) blocks that point at it actually change: the rest of the storage can be shared with the previous copy of the file.

The primary benefit of using content-addressing is that duplicates get removed for free. If two different people write the same data, Venti only stores the data once. Or if one person archives the same data twice, Venti only stores the data once. My research group at MIT runs a Venti server to store FFS disk images of its main file server. Writing the same images over and over again only stores changed blocks, and if multiple people have the same giant files, they take up extra space on the file server itself, but not in Venti. Our live file system usage is about 600GB, but the incremental backup cost in Venti is about 2 GB per night. All our nightly backups going back to 2005 take up about 3 TB, or 1.7 TB compressed.

Another benefit of using content-addressing is that if the data goes bad, you notice. You can check the SHA-1 hash of the data returned by a read to make sure it is the block you expected. Clients can verify the data returned by the server, and the server can verify the data returned by its disks. In fact the latter has been the bigger problem in Venti installations: disks go bad, and if you tend to notice more if you have checks like these.

A few years ago, I wanted to download all the [6 GB of file system traces](https://pdos.csail.mit.edu/archive/p9trace/) used in the Venti paper. Not all of the files downloaded correctly: a few were truncated too early. It turned out that the Venti server now storing the traces at Bell Labs had some disk corruption, and the Venti server noticed because the SHA-1 hashes didn't match expectations. No problem, just use the earlier Venti server that the data was copied from. That server has had disk corruption too, some of it overlapping the bad blocks. So there are still traces that can't be read. Okay, well the traces were saved on backup tapes too. Dig those up, and the backup tapes aren't complete either: still two trace files with missing blocks. There are also backup tapes of the original file system blocks that were used to create the traces. Those are actually readable, so we can regenerate the bad trace files and have a complete set again.

Here's the fun part. The original trace files were stored as a single Venti archive, a giant hash tree like the one pictured above that contains pointers to all the trace files. Not all the files can be read out of the archive, since we have these pointers to missing blocks. Creating a new archive holding just the two missing trace files *makes the old archive start working*. Now that Venti has blocks with the desired SHA1 hashes, the pointers to missing blocks become pointers to existing blocks. Given the right raw materials, the broken archive healed automatically.

With a little engineering, you could create a typical incremental backup system that didn't use hashes to identify duplicate blocks, and you'd probably end up being within a factor of two as space-efficient as Venti. For a long time I had this lingering fear that perhaps indexing by SHA-1 hash, which is pretty expensive to implement efficiently, was overkill. Watching that previously broken archive start working again was the moment when I fully believed that Venti's was the right approach.

*Further reading*. The use of SHA1 specifically isn't really important: any strong collision-resistant hash function will do, though [people](http://www.usenix.org/events/hotos03/tech/henson.html) [disagree](http://www.usenix.org/event/usenix06/tech/full_papers/black/black_html/index.html) on the wisdom of this approach. Remember that Venti is being used in situations where there are no adversaries: the main concern is an accidental collision. The second paper just linked to summarizes:

> We conclude that it is certainly fine to use a 160-bit hash function like SHA1 or RIPEMD-160 with compare-by-hash. The chances of an accidental collision is about 2^-160. This is unimaginably small. You are roughly 2^90 times more likely to win a U.S. state lottery and be struck by lightning simultaneously than you are to encounter this type of error in your file system.

(Comments originally posted via Blogger.)

- [Craig](http://www.blogger.com/profile/02430615130349705538)(April 14, 2008 11:16 AM) That's also how the StarTeam Native-II file repository works. Stores full images of each version, compressed and indexed with MD5 hashes. So no reverse-delta penalty for diffing back versions of files.

- [francois.beausoleil](http://www.blogger.com/profile/17140800379422429193)(April 14, 2008 12:34 PM) The Git version control system also stores content as blobs, indexed by SHA-1 hashes.

- [andrew cooke](http://www.blogger.com/profile/11760508644619954982)(April 14, 2008 2:50 PM) the 2^-160 is a bit misleading. as the first paper points out (search for "birthday") you're going to start getting collisions when you have around 2^80 blocks. that's still a lot of blocks, but \*significantly\* less than the 2^160 you might infer from what you wrote.

- [Karthik](http://www.blogger.com/profile/17888392276064618168)(April 14, 2008 6:51 PM) EMC's Centera uses the same concept, though it used MD5 hashes.

- [jricher](http://www.blogger.com/profile/04108383538978020010)(April 14, 2008 9:20 PM) I understand the birthday problem, but it could be easily solved by moving from sha1 to sha256. This would be less space efficient when you were storing the trees, but that's not going to be the majority of your storage cost anyway. Doing this would make the chance of a collision insignificant for even the largest datasets.

As far as the overhead of using cryptographically strong hashes is concerned - I'm a lot more concerned about data integrity than a slightly slow backup, and it's always possible to move the hash calculation into hardware if it becomes advantageous to do so.

- [stevedekorte](http://www.blogger.com/profile/16242875319857103661)(October 9, 2009 2:07 PM) While there's a lot of overhead, it seems the data integrity problem be solved by verifying the bits on writes just as we do for normal hash tables. Perhaps using larger blocks that are losslessly compressed would speed this up.

I'm curious about using this system for general purpose file systems where we often can't afford to keep around all changes. Would reference counts be the best way to go for tracking when to delete blocks?
