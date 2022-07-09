# Popcount for 1s and 0s in JavaScript
ECMAScript Proposal. J. S. Choi, 2021. Stage 0.

## Rationale

Bit population count (aka “popcount” or “popcnt”) is a numeric operation that
counts the number of 1-bits in an integer’s binary representation. This is a
useful and versatile operation in many contexts, such that it is included as a
built-in instruction in many today CPUs; it’s also an instruction in
WebAssembly. It is also present in many programming languages’ standard
libraries.

Some known use cases are detailed in an [article by Vaibhav Sagar][]. These
include:

* [Succinct data structures][] such as [rank–select bitmaps][], [Roaring
  Bitmaps][].
* [Hash-array-mapped tries (HAMTs)][HAMTs].
* [Error-correcting codes for strings using their Hamming distance][Hamming].
* [Molecular fingerprinting][].
* [Chess programming][] [bitboards][], e.g., [calculating mobility][].
* Graph analysis when represented by bitmaps, e.g., calculating adjacency matrices.

[article by Vaibhav Sagar]: https://vaibhavsagar.com/blog/2019/09/08/popcount/
[succinct data structures]: https://en.wikipedia.org/wiki/Succinct_data_structure
[rank–select bitmaps]: http://www.cs.cmu.edu/~./dga/papers/zhou-sea2013.pdf
[Roaring bitmaps]: https://roaringbitmap.org
[HAMTs]: https://vaibhavsagar.com/blog/2018/07/29/hamts-from-scratch/
[Hamming]: https://en.wikipedia.org/wiki/Hamming_distance#Error_detection_and_error_correction
[molecular fingerprinting]: http://www.dalkescientific.com/writings/diary/archive/2008/06/26/fingerprint_background.html
[chess programming]: https://www.chessprogramming.org/Population_Count
[bitboards]: https://www.chessprogramming.org/Bitboards
[calculating mobility]: https://www.chessprogramming.org/Mobility#Mobility_with_Bitboards

Popcount is so pervasive in programs that both [GCC][] and [Clang][] will try
to detect popcount implementations and replace them with the built-in CPU
instruction. (See [LLVM’s implementation][].)

[GCC]: https://godbolt.org/z/JUzmD8
[Clang]: https://godbolt.org/z/AVqMGl
[LLVM’s implementation]: https://github.com/llvm-mirror/llvm/blob/f36485f7ac2a8d72ad0e0f2134c17fd365272285/lib/Transforms/Scalar/LoopIdiomRecognize.cpp#L960

Popcount is annoying and inefficient to write in JavaScript. We therefore
propose exploring the addition of a memoization API to the JavaScript language.

If this proposal is approved for Stage 1, then we would explore various
directions for the API’s design. We would also assemble as many real-world use
cases as possible and shape our design to fulfill them.

## Description
We would probably add a static function to the Math constructor that would look
like one the following:

```js
Math.popCount(i)
Math.popcount(i)
Math.popcnt(i)
Math.countOnes(i) // Like Rust’s count_ones. We would also include countZeroes.
```

We could restrict it to safe integers; it is uncertain how it should behave on
non-safe integers or non-integer numbers. It is also uncertain whether we
should split it up by size (e.g., `Math.popcnt(i)` versus `Math.popcnt32(i)`
and `Math.popcnt64(i)`).
