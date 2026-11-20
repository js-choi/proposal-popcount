# Bitwise counting

A proposal for ECMAScript bitwise counting operations beyond [`Math.clz32`](https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-math.clz32) (uint32 count leading zeros).

## Status

[The TC39 Process](https://tc39.es/process-document/)

**Stage**: 0  \
**Champions:** Richard Gibson ([@gibson042](https://github.com/gibson042))  \
**Authors:** J. S. Choi ([@js-choi](https://github.com/js-choi))

## Rationale

### Popcount

Bitwise population count (aka “popcount”, “popcnt”, and “bit count”) counts the
number of 1-bits in an integer’s binary representation.
This is a useful and versatile operation in numerics, scientific applications,
binary parsing, and many other contexts—such that it is included as a
built-in instruction in many CPUs and in WebAssembly. It is also present in many
programming languages’ standard libraries.

Some known use cases are detailed in an [article by Vaibhav Sagar][]. These
include:

* [Succinct data structures][] such as [Roaring Bitmaps][] and other
  [rank–select bitmaps][] for [suffix trees][], [binary trees, and multisets][RRR].
* [Hash-array-mapped tries (HAMTs)][HAMTs].
* [Error-correcting codes for strings using their Hamming distance][Hamming].
* [Molecular fingerprinting][] in chemistry applications.
* [Calculating piece mobility][] of [bitboards][] in [chess programming][].
* Graph analysis when represented by bitmaps, e.g., [calculating adjacency matrices][].

[article by Vaibhav Sagar]: https://vaibhavsagar.com/blog/2019/09/08/popcount/
[succinct data structures]: https://en.wikipedia.org/wiki/Succinct_data_structure
[rank–select bitmaps]: http://www.cs.cmu.edu/~./dga/papers/zhou-sea2013.pdf
[Roaring bitmaps]: https://roaringbitmap.org
[RRR]: https://archive.org/details/proceedingsofthi2002acms/page/233
[suffix trees]: https://web.archive.org/web/20110929230740/http://www.dmi.unisa.it/people/cerulli/www/WSPages/WSFiles/Abs/S3/S33_abs_Grossi.pdf
[HAMTs]: https://vaibhavsagar.com/blog/2018/07/29/hamts-from-scratch/
[Hamming]: https://en.wikipedia.org/wiki/Hamming_distance#Error_detection_and_error_correction
[molecular fingerprinting]: http://www.dalkescientific.com/writings/diary/archive/2008/06/26/fingerprint_background.html
[chess programming]: https://www.chessprogramming.org/Population_Count
[bitboards]: https://www.chessprogramming.org/Bitboards
[calculating piece mobility]: https://www.chessprogramming.org/Mobility#Mobility_with_Bitboards
[calculating adjacency matrices]: https://news.ycombinator.com/item?id=20915187

Popcount is so pervasive in programs that both [GCC][] and [Clang][] will try
to detect popcount implementations and replace them with the built-in CPU
instruction. See also [LLVM’s detection algorithm][]. (Note that [SIMD-using
approaches may often be faster than using dedicated CPU instructions][SIMD];
LLVM/Clang has adopted the former for this reason.)

[GCC]: https://godbolt.org/z/JUzmD8
[Clang]: https://godbolt.org/z/AVqMGl
[LLVM’s detection algorithm]: https://github.com/llvm-mirror/llvm/blob/f36485f7ac2a8d72ad0e0f2134c17fd365272285/lib/Transforms/Scalar/LoopIdiomRecognize.cpp#L960
[SIMD]: https://arxiv.org/pdf/1611.07612.pdf

Popcount is annoying and inefficient to write in JavaScript. We therefore
propose adding some form of it to the JavaScript language.

#### Popcount in the broader ecosystem

| Precedent             | Form                                        | Size                      | Signed?  | Negative-int behavior                     |
| ----------------------| ------------------------------------------- | --------------------------| -------- | ----------------------------------------- |
|**[Python][]**         |`i.bit_count()`                              | Bignum                    | Signed   | Input treated as absolute value           |
|**[Wolfram][]**        |`DigitCount[i, 2, 1]`                        | Bignum                    | Signed   | Input treated as absolute value           |
|**[Common Lisp][]**    |`(logcount i)`                               | Bignum†                   | Signed   | Two’s complement; counts zeroes†          |
|**[Scheme (R7RS)][]**\*|`(bit-count i)`                              | Bignum†                   | Signed   | Two’s complement; counts zeroes†          |
|**[Scheme (R6RS)][]**  |`(bitwise-bit-count i)`                      | Bignum†                   | Signed   | Two’s complement; counts zeroes then NOTs†|
|**[GMP][]**            |`mp_bitcnt_t(i)`                             | Bignum‡                   | Signed   | Special behavior‡                         |
|**[C++][]**            |`std::popcnt(i)`                             | 8/16/32/64-bit            | Unsigned | Forbidden by static typing                |
|**[Go][]**             |`bits.OnesCount(i)`, `bits.OnesCount8(i)`, … | 8/16/32/64-bit            | Unsigned | Forbidden by static typing                |
|**[Java][]**           |`Integer.bitCount(i)`, `Long.bitCount(i)`, … | 16/32-bit; bignum         | Signed   | Two’s complement (type dependent)         |
|**[Haskell][]**        |`popCount i`                                 | 8/16/≥29/32/64-bit; bignum| Signed   | Two’s complement (type dependent)         |
|**[Rust][]**           |`i.count_ones()`                             | 8/16/32/64/128-bit        | Signed   | Two’s complement (type dependent)         |
|**[WebAssembly][]**    |`i32.popcnt`, `i64.popcnt`                   | 32/64-bit                 | Signed   | Two’s complement (type dependent)         |
|**[MySQL][]**          |`BIT_COUNT(i)`                               | 64-bit                    | Signed   | Two’s complement (64-bit)                 |
|**[Swift][]**          |`i.nonzeroBitCount`§                         | 32/64-bit¶                | Signed   | Two’s complement¶                         |

<details>

<summary>Table footnotes</summary>

\* [Scheme (R7RS)][] here refers to SRFI 151, which is implemented in several
R7RS implementations, such as [in Chicken Scheme][].

† When R7RS’s `bit-count` or Common Lisp’s `logcount` receives a
negative integer, it returns its number of zeroes instead. For example, both
`(bit-count 255)` and `(bit-count -256)` are 8, and both `(logcount 256)` and
`(logcount -257)` are 1.

R6RS’s `bitwise-bit-count` additionally applies bitwise NOT (i.e., one’s
complement – i.e., two’s complement minus one) to the number of zeroes. For
example, `(bitwise-bit-count -256)` is -9, and `(bitwise-bit-count -257)` is -2.

‡ [GMP][]’s documentation about `mp_bitcnt_t` says, “If [the argument is
negative], the number of 1s is infinite, and the return value is the largest
possible `mp_bitcnt_t`.”

§ [Swift][]’s `nonzeroBitCount` property forms a trio with its
`leadingZeroBitCount` and `trailingZeroBitCount` properties.

¶ Whether Swift’s int type is either 32- or 64-bit depends on its compiler.

</details>

[C++]: https://en.cppreference.com/w/cpp/numeric/popcount
[Common Lisp]: http://www.lispworks.com/documentation/HyperSpec/Body/f_logcou.htm
[GMP]: https://gmplib.org/manual/Integer-Logic-and-Bit-Fiddling#index-mpz_005fpopcount
[Go]: https://pkg.go.dev/math/bits#OnesCount
[Haskell]: https://downloads.haskell.org/~ghc/9.2.3/docs/html/libraries/base-4.16.2.0/Data-Bits.html#v:popCount
[in Chicken Scheme]: https://wiki.call-cc.org/supported-standards
[Java]: https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/lang/Integer.html#bitCount(int)
[MySQL]: https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html#function_bit-count
[Python]: https://docs.python.org/3/library/stdtypes.html#int.bit_count
[Rust]: https://doc.rust-lang.org/std/?search=count_ones
[Scheme (R6RS)]: http://www.r6rs.org/final/html/r6rs-lib/r6rs-lib-Z-H-12.html
[Scheme (R7RS)]: https://srfi.schemers.org/srfi-151/srfi-151.html
[Swift]: https://developer.apple.com/documentation/swift/int/nonzerobitcount
[WebAssembly]: https://developer.mozilla.org/en-US/docs/webassembly/reference/numeric/population_count
[Wolfram]: https://reference.wolfram.com/language/ref/DigitCount.html

### Count trailing zeros

Counting the trailing 0-bits in an integer’s binary representation is similarly
valuable for common optimizations when working with bitmaps, similarly common
across CPU instructions and other programming languages, and similarly annoying
and inefficient to write with the existing ECMAScript language facilities (e.g.,
`const clz32 = x => { x = x|0 || 0; return x ? 32 - Math.clz32(x & -x) - 1 : 32; }`).

## API Possibilities

`Math.popcount` and `Math.ctz32` with uint32 coercion are obvious close cousins
of `Math.clz32`.

Beyond that, we can explore the variations for larger bit widths, most notably
64-bit integers (perhaps in the context of the [BigInt Math proposal][]).

[BigInt Math proposal]: https://github.com/tc39/proposal-bigint-math
