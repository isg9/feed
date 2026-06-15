---
title: 'different times in C: calendar&nbsp;times'
url: https://gustedt.wordpress.com/2013/10/28/different-times-in-c-calendar-times/
published: "2013-10-28T08:41:27Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=2011
---

# different times in C: calendar&nbsp;times

Let’s take the occasion of the change back from DST here in Europe, not in the US, yet, to look how times are handled in C.

The C standard proposes a large variety of types for representing times: `clock_t`, `time_t`, `struct timespec`, `struct tm`, `double` and textual representations as `char[]`. It is a bit complicated to find out what the proper type for a particular purpose is, so let me try to explain this.

The first class of “times” can be classified as *calendar times*, times with a granularity and range as it would typically appear in a human calendar, as for appointments, birthdays and so on. Some of the functions that manipulate these in C99 are a bit dangerous, they operate on global state. Let us have a look how these interact:

```
       char[]
        ^              o
strftime| \_ ctime _   |time
 asctime|           |  |
        |   mktime  |  v    difftime
struct tm --------> time_t ----------> double
        ^             |
        |  localtime  |
        |-------------|
           gmtime

```

`struct tm` structures a calendar time mainly as you would expect. It has hierarchical date fields such as `tm_year` for the year, `tm_mon` for the month and so on, down to the granularity of a second. They have one pitfall, though, how the different fields are counted. All but one start with `0`, e.g `tm_mon` set to `0` stands for January and `tm_wday` `0` stands for Sunday.

Unfortunately, there are exceptions:

- `tm_mday` starts counting days in the month at 1.
- `tm_year` must add 1900 to get the year in the Gregorian calendar. Years represent in that way should lie between Gregorian years 0 and 9999.
- `tm_sec` is in the range from `0` to `60`, including. The later is for the rare occasion of leap seconds.

There are three supplemental date fields that are used to supply additional information to a time value in a `struct tm`.

- `tm_wday` for the week day,
- `tm_yday` for the day in the year, and
- `tm_isdst` a flag that informs if a date is considered being in DST of the local time zone or not.

The consistency of all these fields can be enforced with the function `mktime`. It can be seen to operate in three steps

1. The hierarchical date fields are normalized to their respective ranges.
2. `tm_wday` and `tm_yday` are set to the corresponding values.
3. If `tm_isday` has a negative value, this value is modified to 1 if the date falls into DST for the local platform, and to 0 otherwise.

`mktime` also serves an extra purpose. It returns the time as a `time_t`. `time_t` is thought to represent the same calendar times as `struct tm`, but is defined to be an arithmetic type, more suited to compute with them. It operates on a linear time scale. A `time_t` value of `0`. the beginning of `time_t` is called *epoch* in the C jargon. Often this corresponds to the beginning of Jan 1, 1970,

The granularity of `time_t` is usually to the second, but nothing guarantees that. Sometimes processor hardware has special registers for clocks that obey a different granularity. `difftime` translates the difference between two `time_t` values into seconds that are represented as a `double` value.

Two functions for the inverse operation from `time_t` into `struct tm` come into view:

- `localtime` stores the broken down local time
- `gmtime` stores the broken time, expressed as UTC.

As indicated, they differ in the time zone that they assume for the conversion. Under normal circumstances `localtime` and `mktime` should be inverse to each other, `gmtime` has no direct counterpart for the inverse direction.

Textual representations of calendar times are also available. `asctime` stores the date in a fixed format, independent of any locale, language (it uses English abbreviations 🙂 or platform dependency. The format is a string of the form

```
"Www Mmm DD HH:MM:SS YYYY\n"

```

`strftime` is more flexible and allows to compose a textual representation with format specifiers similar to the `printf` functions.

One additional **warning** is in order. `localtime`, `gmtime`, `ctime`, and `asctime` use internal state and return pointers to static data. They are not thread safe, not reentrant, and calling one of them erases the return value of any previous call to one of them, be careful.

The new optional annexe K of C11, has functions that check for validity of their arguments and are reentrant: `localtime_s`, `gmtime_s`, `ctime_s`, and `asctime_s`. Use them when you may. P99 now implements them in its C11 emulation layer, for those of you that don’t have a complying C library, yet. Give it some testing, if you like.
