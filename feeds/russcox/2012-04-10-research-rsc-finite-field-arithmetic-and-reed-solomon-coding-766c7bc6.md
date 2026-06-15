---
title: 'research!rsc: Finite Field Arithmetic and Reed-Solomon Coding'
url: https://research.swtch.com/field
published: "2012-04-10T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/field
---

# research!rsc: Finite Field Arithmetic and Reed-Solomon Coding

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Finite Field Arithmetic and Reed-Solomon Coding Russ Cox April 10, 2012 *research.swtch.com/field* Posted on Tuesday, April 10, 2012.

Finite fields are a branch of algebra formally defined in the 1820s, but interest in the topic can be traced back to public sixteenth-century polynomial-solving contests. For the next few centuries, finite fields had little practical value, but all changed in the last fifty years. It turns out that they are useful for many applications in modern computing, such as encryption, data compression, and error correction.

In particular, Reed-Solomon codes are an error-correcting code based on finite fields and used everywhere today. One early significant use was in the [Voyager spacecraft](http://en.wikipedia.org/wiki/Error_detection_and_correction#Deep-space_telecommunications): the messages it still sends back today, from the edge of the solar system, are heavily Reed-Solomon encoded so that even if only a small fragment makes it back to Earth, we can still reconstruct the message. Reed-Solomon coding is also used on CDs to withstand scratches, in wireless communications to withstand transmission problems, in QR codes to withstand scanning errors or smudges, in disks to withstand loss of fragments of the media, in high-level storage systems like Google's GFS and BigTable to withstand data loss and also to reduce read latency (the read can complete without waiting for all the responses to arrive).

This post shows how to implement finite field arithmetic efficiently on a computer, and then how to use that to implement Reed-Solomon encoding.

## What is a Finite Field?

One way mathematicians study numbers is to abstract away the numbers themselves and focus on the operations. (This is kind of an object-oriented approach to math.) A *field* is defined as a set F and operators \+ and · on elements of F that satisfy the following properties:

1. (Closure) For all x, y in F, x+y and x·y are in F.

2. (Associative) For all x, y, z in F, (x+y)+z = x+(y+z) and (x·y)·z = x·(y·z).

3. (Commutative) For all x, y in F, x+y = y+x and x·y = y·x.

4. (Distributive) For all x, y, z in F, x·(y+z) = (x·y)+(x·z).

5. (Identity) There is some element we'll call 0 in F such that for all x in F, x+0 = x. Similarly, there is some element we'll call 1 in F such that for all x in F, x·1 = x.

6. (Inverse) For all x in F, there is some element y in F such that x+y = 0. We write y = −x. Similarly, for all x in F except 0, there is some element y in F such that x·y = 1. We write y = 1/x.

You probably recognize those properties from high school algebra class: the most well-known example of a field is the real numbers, where + is addition and · is multiplication. Other examples are complex numbers and fractions.

A mathematician doesn't have to prove the same results over and over for the real numbers ℝ, the complex numbers ℂ, the fractions ℚ, and so on. Instead, she can prove that a particular result holds for all fields—by assuming only the above properties, called the field axioms. Then she can apply the result by substituting a specific instance like the real numbers for the general idea of a field, the same way that a programmer can implement just one `vector(T)` and then instantiate it as `vector(int)`, `vector(string)` and so on.

The integers ℤ are not a field: they lack multiplicative inverses. For example, there is no number that you can multiply by 2 to get 1, no 1/2. Surprisingly, though, the integers modulo any prime p do form a field. For example, the integers modulo 5 are 0, 1, 2, 3, 4. 1+4 = 0 (mod 5), so we say that 4 = −1. Similarly, 2·3 = 1 (mod 5), so we say that 3 = 1/2. After we've proved that ℤ/p is in fact a field, all the results about fields can be applied to ℤ/p. This is very useful: it lets us apply our intuition about the very familiar real numbers to these less familiar numbers. This field is written ℤ/p to emphasize that we're dealing with what's left after subtracting out all the p's. That is, we're dealing with what's left if you assume that p = 0. When you make that assumption, you get math that wraps around at p. These fields are called finite fields because, in contrast to fields like the real numbers, they have a finite number of elements.

For a programmer, the most interesting finite field is ℤ/2, which contains just the integers 0, 1. Addition is the same as XOR, and multiplication is the same as AND. Note that ℤ/p is only a field when p is prime: arithmetic on `uint8` variables corresponds to ℤ/256, but it is not a field: there is no 1/2.

## What can you do with a field?

The only problem with fields is that there's not a ton you can do with just the field axioms. One thing you *can* do is build polynomials, which were the original motivation for the mathematicians who pioneered the use of fields in the early 1800s. If we introduce a symbolic variable x, then we can build polynomials whose coefficients are field values. We'll write F\[x\] to denote the polynomials over x using coefficients from F. For example, if we use the real numbers ℝ as our field, then the polynomials ℝ\[x\] include x2+1, x+2, and 3.14x2 − 2.72x + 1.41. Like integers, these polynomials can be added and multiplied, but not always divided—what is (x2+1)/(x+2)?—so they are not a field. However, remember how the integers are not a field but the integers modulo a prime are a field? The same happens here: polynomials are not a field but polynomials modulo some prime polynomial *are*.

What does “polynomials modulo some prime polynomial” mean anyway? A prime polynomial is one that cannot be factored, like x2+1 cannot be factored using real numbers. The field ℤ/5 is what you get by doing math under the assumption that 5 = 0; similarly, ℝ\[x\]/(x2+1) is what you get by doing math under the assumption that x2+1 = 0. Just as ℤ/5 math never deals with numbers as big as 5, ℝ\[x\]/(x2+1) math never deals with polynomials as big as x2: anything bigger can have some multiple of x2+1 subtracted out again. That is, the polynomials in ℝ\[x\]/(x2+1) are bounded in size: they have only x1 and x0 (constant) terms. To add polynomials, we just add the coefficients using the addition rule from the coefficient's field, independently, like a vector addition. To multiply polynomials, we have to do the multiplication and then subtract out any x2+1 we can. If we have (ax+b)·(cx+d), we can expand this to (a·c)x2 \+ (b·c+a·d)x + (b·d), and then subtract (a·c)(x2+1) = (a·c)x2 \+ (a·c), producing the final result: (b·c+a·d)x + (b·d−a·c). That might seem like a funny definition of multiplication, but it does in fact obey the field axioms. In fact, this particular field is more familiar than it looks: it is the complex numbers ℂ, but we've written x instead of the usual i. Assuming that x2+1 = 0 is, except for a renaming, the same as defining i2 = −1.

Doing all our math modulo a prime polynomial let us take the field of real numbers and produce a field whose elements are pairs of real numbers. We can apply the same trick to take a finite field like ℤ/p and product a field whose elements are fixed-length vectors of elements of ℤ/p. The original ℤ/p has p elements. If we construct (ℤ/p)\[x\]/f(x), where f(x) is a prime polynomial of degree n (f's maximum x exponent is n), the resulting field has pn elements: all the vectors made up of n elements from ℤ/p. Incredibly, the choice of prime polynomial doesn't matter very much: any two finite fields of size pn have identical structure, even if they give the individual elements different names. Because of this, it makes sense to refer to all the finite fields of size pn as one concept: GF(pn). The GF stands for Galois Field, in honor of Évariste Galois, who was the first to study these. The exact polynomial chosen to produce a particular GF(pn) is an implementation detail.

For a programmer, the most interesting finite fields constructed this way are GF(2n)—the polynomial extensions of ℤ/2—because the elements of GF(2n) are bit vectors of length n. As a concrete example, consider (ℤ/2)\[x\]/(x8+x4+x3+x+1) . The field has 28 elements: each can be represented by a single byte. The byte with binary digits b7b6b5b4b3b2b1b0 represents the polynomial b7·x7 \+ b6·x6 \+ b5·x5 \+ b4·x4 \+ b3·x3 \+ b2·x2 \+ b1·x1 \+ b0. To add polynomials, we add coefficients. Since the coefficients are from ℤ/2, adding coefficients means XOR'ing each bit position separately, which is something computer hardware can do easily. Multiplying the polynomials is more difficult, because standard multiplication hardware is based on adding, but we need a multiplication based on XOR'ing. Because the coefficient math wraps at 2, (x2+x)·(x+1) = x3+2x2+x = x3+x, while computer multiplication would choose 1102\\* 0112 = 6 \* 3 = 18 = 100102. However, it turns out that we can implement this field multiplication with a simple lookup table. In a finite field, there is always at least one element α that can serve as a generator. All the other non-zero elements are powers of α: α, α2, α3, and so on. This α is not symbolic like x: it's a specific element. For example, in ℤ/5, we can use α=2: {α, α2, α3, α4} = {2, 4, 8, 16} = {2, 4, 3, 1}. In GF(2n) the math is more complex but still works. If we know the generator, then we can, by repeated multiplication, create a lookup table exp\[i\] = αⁱ and an inverse table log\[αⁱ\] = i. Multiplication is then just a few table lookups: assuming a and b are non-zero, a·b = exp\[log\[a\]+log\[b\]\]. (That's a normal integer +, to add the exponents, not an XOR.)

## Why do we care?

The fact that GF(2n) can be implemented efficiently on a computer means that we can implement systems based on mathematical theorems without worrying about the usual overflow problems you get when modeling integers or real numbers. To be sure, GF(2n) behaves quite differently from the integers in many ways, but if all you need is the field axioms, it's good enough, and it eliminates any need to worry about overflow or arbitrary precision calculations. Because of the lookup table, GF(28) is by far the most common choice of field in a computer algorithm. For example, the Advanced Encryption Standard (AES, formerly Rijndael) is built around GF(28) arithmetic, as are nearly all implementations of Reed-Solomon coding.

## Code

Let's begin by defining a `Field` type that will represent the specific instance of GF(28) defined by a given polynomial. The polynomial must be of degree 8, meaning that its binary representation has the 0x100 bit set and no higher bits set.

```
type Field struct {
    ...
}

```

Addition is just XOR, no matter what the polynomial is:

```
// Add returns the sum of x and y in the field.
func (f *Field) Add(x, y byte) byte {
    return x ^ y
}

```

Multiplication is where things get interesting. If you'd used binary (and Go) in grade school, you might have learned this algorithm for multiplying two numbers (this is *not* finite field arithmetic):

```
// Grade-school multiplication in binary: mul returns the product x×y.
func mul(x, y int) int {
    z := 0
    for x > 0 {
        if x&1 != 0 {
            z += y
        }
        x >>= 1
        y <<= 1
    }
    return z
}

```

The running total `z` accumulates the product of `x` and `y`. The first iteration of this loop adds `y` to `z` if the low bit (the 1s digit) of `x` is 1. The next iteration adds `y*2` if the next bit (the 2s digit, now shifted down) of `x` is 1. The next iteration adds `y*4` if the 4s digit is 1, and so on. Each iteration shifts x to the right to chop off the processed digit and shifts y to the left to multiply by two.

To adapt this to multiply in a finite field, we need to make two changes. First, addition is XOR, so we use `^=` instead of `+=` to add to `z`. Second, we need to make the multiply reduce modulo the polynomial. Assuming that the inputs have already been reduced, the only chance of exceeding the polynomial comes from the shift of `y`. After the shift, then, we can check to see if we've overflowed, and if so, subtract (XOR) out one copy of the polynomial. The finite field version, then, is:

```
// GF(256) mutiplication: mul returns the product x×y mod poly.
func mul(x, y, poly int) int {
    z := 0
    for x > 0 {
        if x&1 != 0 {
            z ^= y
        }
        x >>= 1
        y <<= 1
        if y&0x100 != 0 {
            y ^= poly
        }
    }
    return z
}

```

We might want to do a lot of multiplication, though, and this loop is too slow. There aren't that many inputs—only 28×28 of them—so one option is to build a 64kB lookup table. With some cleverness, we can build a smaller lookup table. In the `NewField` constructor, we can compute α0, α1, α2, ..., record the sequence in an `exp` array, and record the inverse in a `log` array. Then we can reduce multiplication to addition of logarithms, like a [slide rule](http://en.wikipedia.org/wiki/Slide_rule) does.

```
// A Field represents an instance of GF(256) defined by a specific polynomial.
type Field struct {
    log [256]byte // log[0] is unused
    exp [510]byte
}

// NewField returns a new field corresponding to
// the given polynomial and generator.
func NewField(poly, α int) *Field {
    var f Field
    x := 1
    for i := 0; i < 255; i++ {
        f.exp[i] = byte(x)
        f.exp[i+255] = byte(x)
        f.log[x] = byte(i)
        x = mul(x, α, poly)
    }
    f.log[0] = 255
    return &f
}

```

The values of the `exp` function cycle with period 255 (not 256, because 0 is impossible): α255 = 1. The straightforward way to implement `Exp`, then, is to look up the entry given by the exponent modulo 255.

```
// Exp returns the base 2 exponential of e in the field.
// If e < 0, Exp returns 0.
func (f *Field) Exp(e int) byte {
    if e < 0 {
        return 0
    }
    return f.exp[e%255]
}

```

`Log` is an even simpler table lookup, because the input is only a byte:

```
// Log returns the base 2 logarithm of x in the field.
// If x == 0, Log returns -1.
func (f *Field) Log(x byte) int {
    if x == 0 {
        return -1
    }
    return int(f.log[x])
}

```

`Mul` is where things get interesting. The obvious implementation of `Mul` is `exp[(log[x]+log[y])%255]`, but if we double the `exp` array, so that it is 510 elements long, we can drop the relatively expensive `%255`:

```
// Mul returns the product of x and y in the field.
func (f *Field) Mul(x, y byte) byte {
    if x == 0 || y == 0 {
        return 0
    }
    return f.exp[int(f.log[x])+int(f.log[y])]
}

```

`Inv` returns the multiplicative inverse, 1/x. We don't implement divide: instead of x/y, we can use x · 1/y.

```
// Inv returns the multiplicative inverse of x in the field.
// If x == 0, Inv returns 0.
func (f *Field) Inv(x byte) byte {
    if x == 0 {
        return 0
    }
    return f.exp[255-f.log[x]]
}

```

## Reed-Solomon Coding

In 1960, Irving Reed and Gustave Solomon proposed a way to build an error-correcting code using GF(2n). The method interpreted the m message bits as coefficients of a polynomial f of degree m−1 over GF(2n) and then sent f(0), f(α), f(α2), f(α3), ..., f(1). Any m of these, if received correctly, suffice to reconstruct f, and then the message can be read off the coefficients. To find a correct set, Reed and Solomon's algorithm constructed the f corresponding to every possible subset of m received values and then chose the most common one in a majority vote. As long as no more than (2n−m)/2 values were corrupted in transit, the majority will agree on the correct value of f. This decoding algorithm is very expensive, too expensive for long messages. As a result, the Reed-Solomon approach sat unused for almost a decade. In 1969, however, Elwyn Berlekamp and James Massey proposed a variant with an efficient decoding algorithm. In the 1980s, Berlekamp and Lloyd Welch developed an even more efficient decoding algorithm that is the one typically used today. These decoding algorithms are based on systems of equations far too complex to explain here; in this post, we will only deal with encoding. (I can't keep the decoding algorithms straight in my head for more than an hour or two at a time, much less explain them in finite space.)

In Reed-Solomon encoding as it is practiced today, the choice of finite field F and generator α defines a generator polynomial g(x) = (x−1)(x−α)(x−α2)...(x−αn−m). To encode a message m, the message is taken as the top coefficients of a degree n polynomial f(x) = m0xn−1+m1xn−2+...+mmxn−m−1. Then that polynomial can be divided by g to produce the remainder polynomial r(x), the unique polynomial of degree less than n−m such that f(x) − r(x) is a multiple of g(x). Since r(x) is of degree less than n−m, subtracting r(x) does not affect any of the message coefficients, just the lower terms, so the polynomial f(x) − r(x) (= f(x) + r(x)) is taken as the encoded message. All encoded messages, then, are multiples of g(x). On the receiving end, the decoder does some magic to figure out the simplest changes needed to make the received polynomial a multiple of g(x) and then reads the message out of the top coefficients.

While decoding is difficult, encoding is easy: the first m bytes are the message itself, followed by the c bytes defining the remainder of m·xc/g(x). We can also check whether we received an error-free message by checking whether the concatenation defines a polynomial that is a multiple of g(x).

## Code

The Reed-Solomon encoding problem is this: given a message m interpreted as a polynomial m(x), compute the error correction bytes, m(x)·xc mod g(x).

The grade-school division algorithm works well here. If we fill in `p` with m(x)·xc (m followed by c zero bytes), then we can replace `p` by the remainder by iteratively subtracting out multiples of the generator polynomial g.

```
for i := 0; i < len(m); i++ {
    k := f.Mul(p[i], f.Inv(gen[0]))  // k = pi / g0
    // p -= k·g
    for j, g := range gen {
        p[i+j] = f.Add(p[i+j], f.Mul(k, g))
    }
}

```

This implementation is correct but can be made more efficient. If you want to try, run:

```
go get code.google.com/p/rsc/gf256
go test code.google.com/p/rsc/gf256 -bench Blog

```

That benchmark measures the speed of the implementation in `blog_test.go`, which looks like the above. Optimize away, or follow along.

There's definitely room for improvement:

```
$ go test code.google.com/p/rsc/gf256 -bench ECC
PASS
BenchmarkBlogECC   500000   7031 ns/op   4.55 MB/s
BenchmarkECC      1000000   1332 ns/op  24.02 MB/s

```

To start, we can expand the definitions of `Add` and `Mul`. The Go compiler's inliner would do this for us; the win here is not the inlining but the simplifications it will enable us to make.

```
for i := 0; i < len(m); i++ {
    if p[i] == 0 {
        continue
    }
    k := f.exp[f.log[p[i]] + 255 - f.log[gen[0]]]  // k = pi / g0
    // p -= k·g
    for j, g := range gen {
        p[i+j] ^= f.exp[f.log[k] + f.log[g]]
    }
}

```

(The implementation handles `p[i] == 0` specially because 0 has no log.)

The first thing to note is that we compute `k` but then use `f.log[k]` repeatedly. Computing the log will avoid that memory access, and it is cheaper: we just take out the `f.exp[...]` lookup on the line that computes `k`. This is safe because `p[i]` is non-zero, so `k` must be non-zero.

```
for i := 0; i < len(m); i++ {
    if p[i] == 0 {
        continue
    }
    lk := f.log[p[i]] + 255 - f.log[gen[0]]  // k = pi / g0
    // p -= k·g
    for j, g := range gen {
        p[i+j] ^= f.exp[lk + f.log[g]]
    }
}

```

Next, note that we repeatedly compute `f.log[g]`. Instead of doing that, we can iterate `lgen`—an array holding the logs of the coefficients—instead of `gen`. We'll have to handle zero somehow: let's say that the array has an entry set to 255 when the corresponding `gen` value is zero.

```
for i := 0; i < len(m); i++ {
    if p[i] == 0 {
        continue
    }
    lk := f.log[p[i]] + 255 - f.log[gen[0]]  // k = pi / g0
    // p -= k·g
    for j, lg := range lgen {
        if lg != 255 {
            p[i+j] ^= f.exp[lk + lg]
        }
    }
}

```

Next, we can notice that since the generator is defined as

g(x) = (x−1)(x−α)(x−α2)...(x−αn−m)

the first coefficient, g0, is always 1! That means we can simplify the k = pi / g calculation to just k = pi. Also, we can drop the first element of lgen and its subtraction, as long as we ignore the high bytes in the result (we know they're supposed to be zero anyway).

```
for i := 0; i < len(m); i++ {
    if p[i] == 0 {
        continue
    }
    lk := f.log[p[i]]
    // p -= k·g
    for j, lg := range lgen {
        if lg != 255 {
            p[i+1+j] ^= f.exp[lk + lg]
        }
    }
}

```

The inner loop, which is where we spend all our time, has two additions by loop-invariant constants: `i+1+j` and `lk+lg`. The `i+1` and `lk` do not change on each iteration. We can avoid those additions by reslicing the arrays outside the loop:

```
for i := 0; i < len(m); i++ {
    if p[i] == 0 {
        continue
    }
    lk := f.log[p[i]]
    // p -= k·g
    q := p[i+1:]
    exp := f.exp[lk:]
    for j, lg := range lgen {
        if lg != 255 {
            q[j] ^= exp[lg]
        }
    }
}

```

As one final trick, we can replace `p[i]` by a range variable. The Go compiler does not yet use loop invariants to eliminate bounds checks, but it does eliminate bounds checks in the implicit indexing done by a range loop.

```
for i, pi := range p {
    if i == len(m) {
        break
    }
    if pi == 0 {
        continue
    }
    // p -= k·g
    q := p[i+1:]
    exp := f.exp[f.log[pi]:]
    for j, lg := range lgen {
        if lg != 255 {
            q[j] ^= exp[lg]
        }
    }
}

```

The code is in context in [gf256.go](http://code.google.com/p/rsc/source/browse/gf256/gf256.go#120).

## Summary

We started with single bits 0 and 1. From those we constructed 8-bit polynomials—the elements of GF(28)—with overflow-free, easy-to-implement mathematical operations. From there we moved on to Reed-Solomon coding, which constructs its own polynomials built using elements of GF(28) as coefficients. That is, each Reed-Solomon message is interpreted as a polynomial, and each coefficient in that polynomial is itself a smaller polynomial.

Now that we know how to create Reed-Solomon encodings, the next post will look at some fun we can have with them.
