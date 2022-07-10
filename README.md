# Binary population count in JavaScript
ECMAScript Proposal. J. S. Choi, 2021. Stage 0.

## Rationale

Binary population count (aka “popcount” or “popcnt”) is a numeric operation that
counts the number of 1-bits in an integer’s binary representation. This is a
useful and versatile operation in numerics, scientific applications, binary parsing, and many other context—such that it is included as a
built-in instruction in many today CPUs; it’s also an instruction in
WebAssembly. It is also present in many programming languages’ standard
libraries.

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
propose exploring the addition of a popcount API to the JavaScript language.

If this proposal is approved for Stage 1, then we would explore various
directions for the API’s design. We would also assemble as many real-world use
cases as possible and shape our design to fulfill them.

## Description
We would probably add a static function to the Math constructor that would look
like one the following:

```js
Math.popCount(i)
Math.popcount(i) // Like in C++.
Math.popcnt(i) // Like in WebAssembly (WAT).
Math.bitCount(i) // Like in Python, Java, MySQL.
Math.nonzeroBitCount(i) // Like in Swift.
Math.onesCount(i) // Like in Go.
Math.countOnes(i) // Like in Rust.
Math.countZeroes(i) // Like in Rust.
```

We could restrict the function to safe integers; it is uncertain how it should
behave on non-safe integers, negative integers, or non-integer numbers.

It is also uncertain whether we should limit it to 32- and/or 64-bit integers
and, if not, whether we should split it up by size (e.g., `Math.popcnt(i)`
versus `Math.popcnt32(i)` and `Math.popcnt64(i)`). A related cross-cutting
concern is how its API would fit with the [BigInt Math proposal][].

[BigInt Math]: https://github.com/tc39/proposal-bigint-math

(Lastly, an alternative to adding a popcount function that acts on integers
would be to add a bit-array data type with a popcount method. This would be
considerably more complicated.)
