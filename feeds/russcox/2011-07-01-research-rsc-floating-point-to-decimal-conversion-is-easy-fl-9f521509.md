---
title: 'research!rsc: Floating Point to Decimal Conversion is Easy (Floating Point Formatting, Part 1)'
url: https://research.swtch.com/ftoa
published: "2011-07-01T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/ftoa
---

# research!rsc: Floating Point to Decimal Conversion is Easy (Floating Point Formatting, Part 1)

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Floating Point to Decimal Conversion is Easy ( *[Floating Point Formatting](fp-all), Part 1*) Russ Cox July 1, 2011 *research.swtch.com/ftoa* Posted on Friday, July 1, 2011.

Floating point to decimal conversions have a reputation for being difficult. At heart, they're really very simple and straightforward. To prove it, I'll explain a working implementation. It only formats positive numbers, but expanding it to negative numbers, zero, infinities and NaNs would be very easy.

An IEEE 64-bit binary floating point number is an integer *v* in the range \[252, 253) times a power of two: *f* = *v* × 2_e_. Constraining the fractional part of the unpacked `float64` to the range \[252, 253) makes the representation unique. We could have used any range that spans a multiplicative factor of two, but that range is the first one in which all the values are integers.

In Go, `math.Frexp` unpacks a `float64` into *f* = *fr* × 2_exp_ where *fr* is in the range \[½, 1). (C's `frexp` does too.) Converting to our integer representation is easy:

```
fr, exp := math.Frexp(f)
v := int64(fr * (1<<53))
e := exp - 53

```

To convert the integer to decimal, we'll use `strconv.Itoa64` out of laziness; you know how to write the direct code.

```
buf := make([]byte, 1000)
n := copy(buf, strconv.Itoa64(v))

```

The allocation of `buf` reserves space for 1000 digits. In general 1/2_e_ requires about 0.7 *e* non-zero decimal digits to write in full. For a `float64`, the smallest positive number is 1/21074, so 1000 digits is plenty. The second line sets `n` to the number of bytes copied in from the string representation of the (integer) `v`. Throughout, `n` will be the number of digits in `buf`. Note that we're working with ASCII decimal digits `'0'` to `'9'`, not bytes 0-9.

Now we've got the decimal for *v* stored in `buf`. Since *f* = *v* × 2_e_, all that remains is to multiply or divide `buf` by 2 the appropriate number of times ( *e* or *-e* times).

Here's the loop to handle positive *e*:

```
for ; e > 0; e-- {
    δ := 0
    if buf[0] >= '5' {
        δ = 1
    }
    x := byte(0)
    for i := n-1; i >= 0; i-- {
        x += 2*(buf[i] - '0')
        x, buf[i+δ] = x/10, x%10 + '0'
    }
    if δ == 1 {
        buf[0] = '1'
        n++
    }
}
dp := n

```

Each iteration of the inner loop overwrites `buf` with twice `buf`. To start, the code determines whether there will be a new digit ( `δ` = 1), which happens when the leading digit is at least 5. Then it runs up the number from right to left, just as you learned in grade school, multiplying each digit by two and using `x` to carry the result. The digit `buf[i]` moves into `buf[i+δ]`. At the end, if the code needs to insert an extra digit, it does. Run `e` times.

After the loop finishes, we record in `dp` the current location of the decimal point. Since `buf` is still an integer (it started as an integer and we've only doubled things), the decimal point is just past all the digits.

Of course, `e` might have started out negative, in which case we've done nothing and still need to halve `buf` `e` times:

```
for ; e < 0; e++ {
    if buf[n-1]%2 != 0 {
        buf[n] = '0'
        n++
    }
    δ, x := 0, byte(0)
    if buf[0] < '2' {
        δ, x = 1, buf[0] - '0'
        n--
        dp--
    }
    for i := 0; i < n; i++ {
        x = x*10 + buf[i+δ] - '0'
        buf[i], x = x/2 + '0', x%2
    }
}

```

Dividing by two needs an extra digit if the last digit is odd, to store the final half; in that case we add a `0` to `buf` so that we'll have room to store a completely precise answer.

After adding the `0`, we can set up for the division itself. If the first digit is less than 2, it's going to become a zero; in the interest of avoiding leading zeros we use an initial partial value `x` and move digits up ( `δ` = 1) during the division. The multiplication ran from right to left copying from `buf[i]` to `buf[i+δ]`. The division runs left to right copying from `buf[i+δ]` to `buf[i]`.

Now `buf[0:n]` has all the non-zero digits in the exact decimal representation of our `f`, and `dp` records where to put the decimal point. We could stop now, but we might as well implement correct rounding while we're here.

The interesting case is when we have more digits than requested ( `n` \> `ndigit`). To make the decision, just like in grade school, we look at the first digit being removed. If it's less than 5, we round down by truncating. If it's greater than 5, we round up by incrementing what's left after truncation. If it's equal to 5, we have to look at the rest of the digits being dropped. If any of the rest of the digits are nonzero, then rounding up is more accurate. Otherwise we're right on the line. In grade school I was taught to handle this case by rounding up: 0.5 rounds to 1. That's a simple rule to teach, but breaking this tie by rounding to the nearest even digit has better numerical properties, because it rounds up and down equally often, and it is the usual rule employed in real calculations.

This all results in the thunderclap rounding condition:

```
buf[prec] > '5' ||
buf[prec] == '5' && (nonzero(buf[prec+1:n]) || buf[prec-1]%2 == 1)

```

You can see why children are taught `buf[prec] >= '5'` instead.

Anyway, if we have to increment after the truncation, we're still working in decimal, so we have to handle carrying incremented 9s ourselves:

```
i := prec-1
for i >= 0 && buf[i] == '9' {
    buf[i] = '0'
    i--
}
if i >= 0 {
    buf[i]++
} else {
    buf[0] = '1'
    dp++
}

```

The loop handles the 9s. The `if` increments what's left, or, if we turned the whole string into zeros, it simulates inserting a 1 at the beginning by changing the first digit to a 1 and moving the decimal place.

Having done that and set `n` `=` `prec`, we can piece together the actual number:

```
return fmt.Sprintf("%c.%se%+d", buf[0], buf[1:n], dp-1)

```

That prints the first digit, then a decimal point, then the rest of the digits, and finally the `e±N` suffix. The exponent must adjust the number by `dp` −1 because we are printing all but the first digit after the decimal point.

That's all there is to it. To print  *f* = *v* × 2_e_  you write *v* in decimal, multiply or divide the decimal by 2 the right number of times, and print what you're left holding.

The Plan 9 C library and the Go library both use variants of the approach above. On my laptop, the code above does 100,000 conversions per second, which is plenty for most uses, and the code is easy to understand.

Why are some converters more complicated than this? Because you can make them a little faster. On my laptop, glibc's `sprintf(buf, "%.50e",` `M_PI)` is about 15 times faster than an equivalent print using the code above, because the implementation of `sprintf` uses much more sophisticated mathematics to speed the conversion.

If you have a stomach for pages of equations, there are many interesting papers that discuss how to do this conversion and its inverse more quickly:

- William Clinger, “ [How to Read Floating Point Numbers Accurately](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.45.4152)”, PLDI 1990.

- Guy L. Steele Jr. and Jon L. White, “ [How to Print Floating Point Numbers Accurately](http://portal.acm.org/citation.cfm?id=93559)”, PLDI 1990.

- David M. Gay, “ [Correctly Rounded Binary-Decimal and Decimal-Binary Conversions](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.31.4049)”, AT&T Bell Laboratories, 1990.

- Vern Paxson, “ [A Program for Testing IEEE Decimal-Binary Conversion](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.144.5889)”, May 1991.

- Robert Burger and R. Kent Dybvig, “ [Printing Floating-Point Numbers Quickly and Accurately](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.52.2247)”, SIGPLAN 1996.

- Florian Loitsch, “ [Printing Floating-Numbers Quickly and Accurately With Integers](http://florian.loitsch.com/publications/dtoa-pldi2010.pdf)”, PLDI 2010.

Just remember: The conversion is easy. The optimizations are hard.

## Code

The full program is available in [this Gist](https://gist.github.com/1057873) or you can try it [using the Go playground](javascript:playground();).

```
package main

import (
    "fmt"
    "math"
    "strconv"
)

func ftoa(f float64, prec int) string {
    fr, exp := math.Frexp(f)
    v := int64(fr * (1<<53))
    e := exp - 53

    buf := make([]byte, 1000)
    n := copy(buf, strconv.Itoa64(v))

    for ; e > 0; e-- {
        δ := 0
        if buf[0] >= '5' {
            δ = 1
        }
        x := byte(0)
        for i := n-1; i >= 0; i-- {
            x += 2*(buf[i] - '0')
            x, buf[i+δ] = x/10, x%10 + '0'
        }
        if δ == 1 {
            buf[0] = '1'
            n++
        }
    }
    dp := n

    for ; e < 0; e++ {
        if buf[n-1]%2 != 0 {
            buf[n] = '0'
            n++
        }
        δ, x := 0, byte(0)
        if buf[0] < '2' {
            δ, x = 1, buf[0] - '0'
            n--
            dp--
        }
        for i := 0; i < n; i++ {
            x = x*10 + buf[i+δ] - '0'
            buf[i], x = x/2 + '0', x%2
        }
    }

    if prec > 0 {
        if n > prec {
            if buf[prec] > '5' || buf[prec] == '5' && (nonzero(buf[prec+1:n]) || buf[prec-1]%2 == 1) {
                    i := prec-1
                    for i >= 0 && buf[i] == '9' {
                        buf[i] = '0'
                        i--
                    }
                    if i >= 0 {
                        buf[i]++
                    } else {
                        buf[0] = '1'
                        dp++
                    }
            }
            n = prec
        }
        for n < prec {
            buf[n] = '0'
            n++
        }
    }
    return fmt.Sprintf("%c.%se%+d", buf[0], buf[1:n], dp-1)
}

func nonzero(buf []byte) bool {
    for _, c := range buf {
        if c != '0' {
            return true
        }
    }
    return false
}

// Difficult boundary cases, derived from tables given in
//    Vern Paxson, A Program for Testing IEEE Decimal-Binary Conversion
//    ftp://ftp.ee.lbl.gov/testbase-report.ps.Z
//
var ftoaTests = []struct{
    N int
    F float64
    A string
}{
    // Table 3: Stress Inputs for Converting 53-bit Binary to Decimal, < 1/2 ULP
    {0, math.Ldexp(8511030020275656, -342), "9.e-88"},
    {1, math.Ldexp(5201988407066741, -824), "4.6e-233"},
    {2, math.Ldexp(6406892948269899, +237), "1.41e+87"},
    {3, math.Ldexp(8431154198732492, +72), "3.981e+37"},
    {4, math.Ldexp(6475049196144587, +99), "4.1040e+45"},
    {5, math.Ldexp(8274307542972842, +726), "2.92084e+234"},
    {6, math.Ldexp(5381065484265332, -456), "2.891946e-122"},
    {7, math.Ldexp(6761728585499734, -1057), "4.3787718e-303"},
    {8, math.Ldexp(7976538478610756, +376), "1.22770163e+129"},
    {9, math.Ldexp(5982403858958067, +377), "1.841552452e+129"},
    {10, math.Ldexp(5536995190630837, +93), "5.4835744350e+43"},
    {11, math.Ldexp(7225450889282194, +710), "3.89190181146e+229"},
    {12, math.Ldexp(7225450889282194, +709), "1.945950905732e+229"},
    {13, math.Ldexp(8703372741147379, +117), "1.4460958381605e+51"},
    {14, math.Ldexp(8944262675275217, -1001), "4.17367747458531e-286"},
    {15, math.Ldexp(7459803696087692, -707), "1.107950772878888e-197"},
    {16, math.Ldexp(6080469016670379, -381), "1.2345501366327440e-99"},
    {17, math.Ldexp(8385515147034757, +721), "9.25031711960365024e+232"},
    {18, math.Ldexp(7514216811389786, -828), "4.198047150284889840e-234"},
    {19, math.Ldexp(8397297803260511, -345), "1.1716315319786511046e-88"},
    {20, math.Ldexp(6733459239310543, +202), "4.32810072844612493629e+76"},
    {21, math.Ldexp(8091450587292794, -473), "3.317710118160031081518e-127"},

    // Table 4: Stress Inputs for Converting 53-bit Binary to Decimal, > 1/2 ULP
    {0, math.Ldexp(6567258882077402, +952), "3.e+302"},
    {1, math.Ldexp(6712731423444934, +535), "7.6e+176"},
    {2, math.Ldexp(6712731423444934, +534), "3.78e+176"},
    {3, math.Ldexp(5298405411573037, -957), "4.350e-273"},
    {4, math.Ldexp(5137311167659507, -144), "2.3037e-28"},
    {5, math.Ldexp(6722280709661868, +363), "1.26301e+125"},
    {6, math.Ldexp(5344436398034927, -169), "7.142211e-36"},
    {7, math.Ldexp(8369123604277281, -853), "1.3934574e-241"},
    {8, math.Ldexp(8995822108487663, -780), "1.41463449e-219"},
    {9, math.Ldexp(8942832835564782, -383), "4.539277920e-100"},
    {10, math.Ldexp(8942832835564782, -384), "2.2696389598e-100"},
    {11, math.Ldexp(8942832835564782, -385), "1.13481947988e-100"},
    {12, math.Ldexp(6965949469487146, -249), "7.700366561890e-60"},
    {13, math.Ldexp(6965949469487146, -250), "3.8501832809448e-60"},
    {14, math.Ldexp(6965949469487146, -251), "1.92509164047238e-60"},
    {15, math.Ldexp(7487252720986826, +548), "6.898586531774201e+180"},
    {16, math.Ldexp(5592117679628511, +164), "1.3076622631878654e+65"},
    {17, math.Ldexp(8887055249355788, +665), "1.36052020756121240e+216"},
    {18, math.Ldexp(6994187472632449, +690), "3.592810217475959676e+223"},
    {19, math.Ldexp(8797576579012143, +588), "8.9125197712484551899e+192"},
    {20, math.Ldexp(7363326733505337, +272), "5.58769757362301140950e+97"},
    {21, math.Ldexp(8549497411294502, -448), "1.176257830728540379990e-119"},

    {3, 12345000, "1.234e+7"},
}

func main() {
    fmt.Printf("%s\n", ftoa(math.Pi, 50))  // 50 digits of math.Pi (not 50 digits of π)
    //fmt.Printf("%s\n", ftoa(math.Nextafter(0, 1), 0))
    for _, tt := range ftoaTests {
        if a := ftoa(tt.F, tt.N+1); a != tt.A {
            fmt.Printf("ftoa(%g, %d) = %q, want %q\n", tt.F, tt.N+1, a, tt.A)
        }
    }
}

```

(Comments originally posted via Blogger.)

- [nicolas cellier](http://www.blogger.com/profile/13769333498003444630)(July 1, 2011 10:44 AM) In general 1/2e requires about 0.7e non-zero decimal digits to write in full:

It requires exactly e digits after the decimal point, but removing the leading zeros is interesting...

2^10 > 10^3, thus 2^-10e has at least 3e leading zeroes.

Thus 2^-e has at least trunc(0.3e) leading zeroes, thus the 0.7e digits. Very good to know :)

- [Jan-willem](http://www.blogger.com/profile/12548850641640358518)(July 1, 2011 10:47 AM) Do you simply cap prec to avoid extra decimal places? I'm worried about the 1.99999999999 problem that Steele & White discuss, where you really want the shortest decimal that reads as the number you've got.

- [Russ Cox](http://swtch.com/~rsc/)(July 1, 2011 10:55 AM) http://golang.org/src/pkg/strconv/ftoa.go's roundShortest has the logic for the '1.9999999999' problem.

Basically, it's easy to figure out the (v', e') for the minimum (least) number you could print to get the right answer when converted back, and also the maximum number. Then you walk all three at the same time until they start to differ. Then you can round. If a short form is a valid answer, it you'll get a digit difference very quickly.

- [GlacJAY](http://www.blogger.com/profile/03215784713435892961)(July 27, 2011 2:08 AM) Lost minus sign in the power's range.
