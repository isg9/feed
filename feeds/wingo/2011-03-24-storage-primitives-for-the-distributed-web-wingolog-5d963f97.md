---
title: storage primitives for the distributed web — wingolog
url: https://wingolog.org/archives/2011/03/24/storage-primitives-for-the-distributed-web
published: "2011-03-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/03/24/storage-primitives-for-the-distributed-web
---

# storage primitives for the distributed web — wingolog

## [storage primitives for the distributed web](/archives/2011/03/24/storage-primitives-for-the-distributed-web)

24 March 2011 2:28 PM

- [gnu](/tags/gnu)
- [cloud](/tags/cloud)
- [privacy](/tags/privacy)
- [guile](/tags/guile)
- [ocap](/tags/ocap)
- [scheme](/tags/scheme)

*An example-driven discussion of what seems to me to be the minimal set of persistent storage primitives needed in an [autonomous web](//wingolog.org/archives/2010/04/10/towards-a-gnu-autonomous-cloud).*

❧

Lisa would like to keep a private journal, and store the journal entries in [Marge's computer](//wingolog.org/archives/2011/03/19/bart-and-lisa-hacker-edition). Lisa is not worried about Marge reading her journal entries, but she wants to make sure that Bart, who also uses the computer, does not have access to her journal. Marge needs to give Lisa some storage primitives so that she can write her journal-entry program.

Marge calls up her operating system vendor, and gets the following procedure documentation in response:

make-cell init-stringCreate a new storage location, and store init-string in it. Returns a write-cap, which allows a user to change the string associated with this storage location.write-cap? objReturn `#t` if obj is a write-cap, and `#f` for any other kind of object.cell-set! write-capstringSet the string associated with a cell to a new value.write-cap->read-cap write-capTake a write-cap and return a read-cap, which allows a user to read the value associated with a cell.read-cap? objReturn `#t` if obj is a read-cap, and `#f` for any other kind of object.cell-ref read-capReturn the string associated with the cell.

Marge makes all of these procedures available to Lisa. Lisa then starts to program. She does not want to allow herself to edit her old entries, so she makes a helper:

```
(define (put-text string)
  (write-cap->read-cap (make-cell string)))

```

Since `put-text` throws away the write-cap for this string, nothing else will be able to change an entry, once it is written.

Read-caps and write-caps are capabilities. They are unforgeable. Since Lisa did not give any of these capabilities to Bart, she feels safe typing her innermost thoughts into `put-text`.

**persistent objects**

In Scheme, capabilities are objects. A piece of code has capabilities, in the normal English sense of the term, to all of the objects that are in its scope, and to objects that are in the current module.

But since all does not live in the warm world of Scheme, the storage primitives that the computer vendor provides allow read-caps and write-caps to be serialized to strings:

cap->string capReturn a string representation of the capability cap.string->cap stringReturn a capability corresponding to string. Read capabilities have a different representation from write capabilities, so this procedure may return a read-cap or a write-cap. It returns `#f` if this string does not look like a capability.

When Lisa started writing, she wrote down the cap-strings for all of her entries in a book. Then when she wants to read them, she types the capability strings into the terminal:

```
(cell-ref (string->cap "read-cap:a6785kjiyv8c0..."))
=> "I was really happy with my solo today at band, but..."

```

But this got tiring after a while, so she decided to store the list of capabilities instead:

```
;; Build a list data type.
;;
(define (make-list-cell)
  (make-mutable-cell ""))

(define (list-cell-ref read-cap)
  (let ((str (cell-ref read-cap)))
    (if (equal? str "")
        '()
        (map string->cap (string-split str #\,)))))

(define (list-cell-set! write-cap caps)
  (cell-set! write-cap
             (string-join (map cap->string caps) ",")))

;; Helper.
;;
(define (->readable cap)
  (if (write-cap? cap)
      (write-cap->read-cap cap)
      cap))

;; Make a new cell, and print out its cap-string.
;; Note to self: write down this string!
;;
(display (cap->string (make-list-cell)))

(define (add-entry! entries cap)
  (list-cell-set!
   entries
   (cons cap (list-cell-ref (->readable entries)))))

```

Now she just has to write down the cap-string of the new list cell that she made, and she has a reference to all of her entries. Whenever she writes a new entry, she uses `add-entry!` to update the cell's value, adding on the new cap-string.

**distribution**

Colin's father Bono has a computer just like Marge's, and Lisa would like for Colin to be able to be able to read some specific entries. So she asks Marge how to give access to the cells that she is using for data storage to other machines.

Marge asks her vendor, and the vendor says that actually, the cells implementation that was provided to her stores its data *in the cloud*. So Lisa can just give Colin a cap-string -- read-only, presumably -- for the essays that she would like to share, and all is good.

Marge doesn't know what this means, but she tells Lisa, and Lisa freaks out. "You mean that I've been practicing [Careless Computing](http://www.guardian.co.uk/technology/blog/2010/dec/14/chrome-os-richard-stallman-warning) this whole time, and you didn't tell me??? You mean that Chief Wiggum could have called up the cloud storage provider and gotten access to all of my data?"

**careful computing**

Lisa's concern is a good one. Marge puts her in contact with the vendor directly, who explains that actually, the cells implementation is designed to preserve users' autonomy.

Creating a cell creates an [RSA key-pair](http://en.wikipedia.org/wiki/Public-key_cryptography). The write-cap contains the signing key (SK) and the public key (PK), and the read-cap contains just the PK. Before sending data to the cloud provider, the data is signed and encrypted using the SK, so only people with access to the read-cap can actually decrypt the strings.

The cells are stored in a standard key-value store. The key is computed as the secure hash (H) of the PK, so that even the cloud storage provider cannot decrypt the data. Furthermore, the `cell-ref` user does not rely on the provider to vouch for the data's integrity, as she can verify it directly against the PK. The only drawback is that Lisa cannot be sure that `cell-ref` is returning the latest value of a cell, whatever that means.

The vendor continues by noting that it doesn't actually matter much, from Lisa's perspective, what the key-value store is. It could be Amazon S3-backed web service, a [Tahoe-LAFS friendnet](http://lwn.net/Articles/280483/), home-grown [CouchDB](http://couchdb.apache.org/) things, or [GNUnet](http://gnunet.org/). This is an important property in a time in which the peer-to-peer key-value stores have not yet stabilized. The vendor also says that they don't have accounting figured out yet, so they don't know how to charge people for storage, but that they trust that the commercial folks will work that out.

**context**

These primitives are sufficient to build proper data structures on top of k-v stores -- tries, queues, and such -- all with fine-grained access controls, and without having to trust the store itself. Lisa can, if she ever grows up, publish all (or a set) of her diaries to the world, which could then form part of larger data structures, like the "wall" of whatever comes after Facebook.

It seems to me that this set of primitives is a minimal set. You could add in better support for immutable data, but since you can implement it in terms of mutable data, it seemed unnecessary.

This scheme was mostly inspired by [Tahoe-LAFS](http://tahoe-lafs.org/). You can read a short and very interesting technical paper about it [here](http://tahoe-lafs.org/~zooko/lafs.pdf).

**future**

Next up would be seeing if these primitives interact well with a capabilities-based [security kernel](http://mumble.net/~jar/pubs/secureos/) for mobile code. Cross your fingers!

## related articles

- [bart and lisa, hacker edition](/archives/2011/03/19/bart-and-lisa-hacker-edition)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 2 responses

1. [Keith Rarick](http://xph.us/) says:[25 March 2011 0:59 AM](#9d145600ff73e1ec6234085ab248e10f3cc0bd67)

   I might add to this list a write-only cap (containing

   just the SK), so that Lisa can allow others to send

   her secret anonymous messages.

   kr

2. [Stu Card](http://www.critical.com) says:[21 July 2011 3:57 PM](#adfce38c1f38fa4bb3193e1d9f1b8a0ca26f3e45)

   For the security kernel, check out the L4 microkernel and the several hypervisors (NOVA, OKL4, CodeZero) and one OS (Genode) based upon it: they offer native object capability support.

   For securing network routing and including IP multicast w/capabilities, see the DIPLOMA work at Columbia University.

   For combining Tahoe-LAFS storage, DIPLOMA secured NACK Oriented Reliable Multicast (NORM) transport, L4 microvisor, etc. in a capability/credential based active networking (mobile agent) environment, see the ARGOS work ongoing at my firm, critical.com.

Comments are closed.
