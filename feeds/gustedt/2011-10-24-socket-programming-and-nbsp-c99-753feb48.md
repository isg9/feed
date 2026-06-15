---
title: socket programming and&nbsp;C99
url: https://gustedt.wordpress.com/2011/10/24/socket-programming-and-c99/
published: "2011-10-24T23:06:59Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1084
---

# socket programming and&nbsp;C99

Network socket programming (BSD sockets) is one of the dark arts, the point where legacy interfaces can become a real burden to the programmer. In this post I will propose some hints and techniques that might this a bit easier, provided your compiler is complying to C99. Since it seems that there has been some misunderstanding about this post: **this is not a tutorial** or introduction **to socket programming**. This is to show how C99 can help to deal with a badly designed legacy interface.

If you are interested in socket programming as such, you’d probably better look elsewhere. There is a lot of information available on the web. On the other hand if you don’t know much about sockets (of even nothing at all) you can just take this as an example of a *type of interface* with which you might be confronted when programming in C.

Socket programming traditionally uses a handcrafted polymorphism to capture the features of different socket protocols. It does that by re-interpreting objects of different types, one for each protocol. Here we will use

```
struct sockaddr;       /* generic, used by the socket interfaces */
struct sockaddr_in;    /* used for the IPv4 protocol. weird! */
struct sockaddr_in6;   /* used for the IPv6 protocol */

```

but there are many others. The polymorphism works that these `struct` have a member of type `sa_family_t` that indicates the real type. This member has a different name for each of the `struct`, but it is guaranteed to sit at exactly the same position in each of the `struct`. For the types above they are `sa_family`, `sin_family` and `sin6_family`.

A convenient way to deal with socket calls would be to use a `union` of these types

```
P99_DECLARE_UNION(orwl_addr);
union orwl_addr {
  struct sockaddr sa;        /* generic, used by the socket interfaces */
  struct sockaddr_in sin;    /* used for the IPv4 protocol. weird! */
  struct sockaddr_in6 sin6;  /* used for the IPv6 protocol */
};

```

and then to pass the sa member to the socket interfaces.

## Designated initializers for IP address initialization

Hopefully one day when somebody will be reading this he will be only concerned with IPv6 or later so let us start with that.

```
int fd = socket(AF_INET6, SOCK_DGRAM, 0); /* prepare for an IPv6 connection with udp */
orwl_addr addr = /* initialize somehow */;
if (bind(fd, &amp;addr.sa, sizeof addr)) {
  /* error handling */
}

```

The difficulty lies in the initialization part. For IPv6 this is still relatively straight. We are provided with two macros that can be used for the most common cases, namely `IN6ADDR_ANY_INIT` and `IN6ADDR_LOOPBACK_INIT`. If we want to initialize a variable with a fixed IP address, designated initializers come to our rescue:

```
orwl_addr addr = {
 .sin6 = {
   .sin6_family = AF_INET6,
   .sin6_port = htons(53),
   .sin6_addr = {
      .s6_addr = { 0x26, 0x20, 0x00, 0x0c, 0xcc, [15] = 0x02 },
   },
 },
};

```

this ensures the correct initialization of the `struct` (all other members such as `sin6_scope_id` are automatically set to `0`) with the IPv6 address `2620:c:cc00::2`. Without designated initializers (so pre-C99) such an initialization would already be quite tedious. You’d have to give the initializers in declaration order. But that order may differ from platform to platform, the POSIX standard only specifies the names of the members, not their order.

Also note how the designator `[15]` allows to skip the zeros that correspond to the `::` part of an IPv6 address.

The initializer for the port number also shows a feature of C99. `Htons` translates the port number into its network byte order representation. Because this is a function, a C89 initializer could not use that and would have to know the representation of 53 (usually used for a DNS service) on the particular target machine. So portability would be a real headache.

Here the disadvantage of `htons` is that it is a function. Such an initializer would not be allowed for a static variable.

## Ensure complete initialization of `union` s

This was simple because IPv6 addresses always are specified and used in so-called network order, just byte by byte as they come. The picture darkens a bit already for IPv6 if we want to compare such addresses. A simple idea would be that we’d use `memcmp` for that, but this would not work reliably. `Memcmp` compares two objects byte by byte and sees if they differ.

In particular it would also compare padding bytes, and for C99 compilers are not obliged to zero out these. Some compilers (e.g gcc) do, and some others (e.g clang) don’t so you can’t rely on anything here. As a result you could have two objects representing the same IPv6 address compare equal for one compiler and unequal for another. (Fortunately C1x will change this, but unfortunately at the point of the writing we are not yet there.)

To make sure the complete initialization of the structure we add an artificial first member to the union:

```
union orwl_addr {
  uint8_t all[some_magic];   /* the naked bytes of the structure */
  struct sockaddr sa;        /* generic, used by the socket interfaces */
  struct sockaddr_in sin;    /* used for the IPv4 protocol. weird! */
  struct sockaddr_in6 sin6;  /* used for the IPv6 protocol */
};

```

and change the initializer

```
orwl_addr addr = {
 .all  = { 0 },            /* &lt;&lt;-- zero out all padding bytes */
 .sin6 = {
   .sin6_port = htons(53),
   .sin6_family = AF_INET6,
   .sin6_addr = {
      .s6_addr = { 0x26, 0x20, 0x00, 0x0c, 0xcc, [15] = 0x02 },
   },
 },
};

```

The C99 standard guarantees that the effect of the initializers for members are as if they are executed in the order they are given. So here first all bytes would be zeroed by the initialization of `.all`, and then the relevant parts would be overwritten by the one for `.sin6`. Some compilers (clang) will complain that initializers are overwritten, but this is simply how it is currently foreseen by the standard.

Having such a dummy first member of the `union` also ensures that the default initializer `{ 0 }` will really zero out the whole structure.

## IPv4: trick out endianess with macros

As we have already seen endianess is a real headache for networking programs. POSIX defines some functions to deal with that, but as functions they are not appropriate in all contexts. The becomes weird when it goes to IPv4.

For IPv4 the `struct sockaddr_in` contains a member `sin_addr.s_addr` for the address, but this is of type `uint32_t`. So to initialize a variable with an IPv4 address that is given in network format (say `127.0.0.1`) we’d have to use the function `ntoh` to store it in there. And as it is a function the same restriction applies as it did for the port number above: such an initializer would not be suitable for a `static` initializer.

Let us try to get this done by adding yet another member to orwl\_addr:

```
struct orwl_sockaddr_raw {
  sa_family_t raw_family;
  char raw_data[some_magic - magic_offset];
};
union orwl_addr {
  uint8_t all[some_magic];   /* the naked bytes of the structure */
  struct orwl_sockaddr_raw raw; /* access to the raw data of the different socket types */
  struct sockaddr sa;        /* generic, used by the socket interfaces */
  struct sockaddr_in sin;    /* used for the IPv4 protocol. weird! */
  struct sockaddr_in6 sin6;  /* used for the IPv6 protocol */
};

```

Here we assume that the `sa_family_t` member of all socket `struct` is actually located at the beginning. The following code can then place the bytes of an IPv4 address exactly at the place they belong:

```
orwl_addr const ipv4_localnet = {
  .raw = {
    .raw_family = AF_INET,
    .raw_data = {
      [magic_in4_addr] = 127, 0, 0, 1,
    },
  }
}

```

Actually you probably don’t want to repeat such a code in several places. This is what macros are made for. Here are macros that can be used to initialize with any IPv4 address:

```
#define ORWL_IN4_ADDR(_0, _1, _2, _3) [magic_in4_addr] = (_0), (_1), (_2), (_3)
#define ORWL_IN4_PORT(PORT) [magic_in4_port] = ((PORT)>>8), (PORT)
#define ORWL_IN4_INITIALIZER(...)                       \
{                                                       \
  .raw = {                                              \
    .raw_family = AF_INET,                              \
    .raw_data = {                                       \
       __VA_ARGS__                                      \
    },                                                  \
  }                                                     \
}

```

These allow declarations that are portable and still somewhat readable:

```
orwl_addr const nameserver[] = {
  ORWL_IN4_INITIALIZER(ORWL_IN4_ADDR(193, 51, 208, 13), ORWL_IN4_PORT(53)),
  ORWL_IN4_INITIALIZER(ORWL_IN4_ADDR(58, 6, 115, 43), ORWL_IN4_PORT(53)),
  ORWL_IN4_INITIALIZER(ORWL_IN4_ADDR(208, 67, 222, 222), ORWL_IN4_PORT(53)),
};

```

What I didn’t show above is how to have all the magic constants determined at compile time. This can be done by declaring \`enum\` constants.

```
  magic_in4_addr =
  /* Start of the address inside a IPv4 structure */
  offsetof(struct sockaddr_in, sin_addr.s_addr)
  /* offset to skip in the sockaddr structure that corresponds to
     the sa_family_t field plus possibly padding bytes. */
  - offsetof(orwl_addr, raw.raw_data),

```

Hopefully you can easily figure out how to do that for the other ones, too.
