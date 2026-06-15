---
title: All Aboard — wingolog
url: https://wingolog.org/archives/2004/09/11/all-aboard
published: "2004-09-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/09/11/all-aboard
---

# All Aboard — wingolog

## [All Aboard](/archives/2004/09/11/all-aboard)

11 September 2004 11:06 PM

- [scheme](/tags/scheme)
- [gnome](/tags/gnome)

### On D Bus

Trying to get releases together for guile-gnome was getting me down. Distcheck on a slow system sucks. Anyway, I needed a project to work the brain a bit, so I started on some DBus bindings for Guile.

The difficulty of wrapping a library is directly proportional to the number of types that it defines. Functions are easy, you just have to generate .defs and tell the wrapper generator to go. guile-glib already wraps a lot of types, so I was hoping to avoid a lot of the low-level stuff.

That plan was confounded a bit by the existence of array types in DBus. Sure, you could wrap them as GValueArrays, but that's really inefficient for the "blob" data case, where Guile's uniform vectors should be used. Other than that, there were a few types (e.g. custom, object path) that weren't wrapped already as GBoxed. Instead I had to implement generic procedures to read and write arguments for messages.

DBus has implemented a really nice set of container types. Arrays are uniform; elements have to be all of the same type. This is a particularly big win for numeric or byte data, and there are wrappers in libdbus for accessing these as blocks of memory. On the other hand, the elements can be of any type known to DBus, including other arrays, dicts, customs, etc. Dicts are the same way. The type system has closure. Containers can contain any type. Havoc Pennington for president!

I got this to work with scheme as well. Numeric or byte data is returned and passed as uniform vectors, and other types as lists. Dicts are implemented as association lists. It all works out pretty well, although I wonder why boolean arrays are not bit vectors.

From the messages, it's a simple task to invoke methods on "objects". First, you get a connection to the bus. Following the lead of the python bindings, there are some helper classes to keep track of the bookkeeping: a "service" is really a service name + dbus connection, and a "remote object" is a service + an object name + an interface. Thus you have:

```
guile> (dbus-bus-get 'system)
$1 = # 405c0368>
guile> (dbus-connection-get-service $1 "org.freedesktop.DBus")
$2 = #< 808bdc0>
guile> (get-object $2 "/org/freedesktop/DBus" "org.freedesktop.DBus")
$3 = #< 808aa30>
guile> (invoke $3 'ListServices)
$4 = (":1.97" ":1.0" ":1.72" "org.freedesktop.Hal")

```

`invoke` calls a "method" on the remote object. In this case, the reply message was an array of strings, which was properly converted to a lisp list. The only problem I have now is that there are no nice services to play with!

### other

Sitting here on a Saturday evening with a couple of buddies. One went to the U of Michigan, and the other to Notre Dame. Seems their teams are playing tonight. We're following it as close as possible from 10,000 kilometers away from the game. Excitement in the village!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)
- [happy birthday gnome!](/archives/2007/08/14/happy-birthday-gnome)
- [documenting language bindings](/archives/2007/07/18/documenting-language-bindings)
- [foreign interfaces](/archives/2007/05/24/foreign-interfaces)

Comments are closed.
