# MMPHF-Experiments, Revisited

This repository is a fork of the [original MMPHF-Experiments
repository](https://github.com/ByteHamster/MMPHF-Experiments) for the paper
“[Learned Monotone Minimal Perfect
Hashing](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ESA.2023.46)“.
It contains updated code, including Rust implementation of the LCP-based
functions and a fix to the Java experiments. If you're looking for advice on the
choice of a MMPHF, the results in the paper are somewhat misleading for two main
reasons:

- An “apples-and-orangres“ problem: the experiments tested a wired implementation
  for integer or sequence of bytes of LeMonHash against Java code designed to turn
  any object in a bit vector _via_ a runtime-specified transformation strategy, and
  then process it. This level of genericity has a high cost not shared by the C++
  implementations.

- C++ vs. Java for this type of data structures implies at least a 2x slowdown.

- In an honest mistake, the authors used a UTF-16 transformation strategy that
  doubled the length of all ASCII strings passed to the Java data structures,
  squaring the size of the underlying universe. For a 10-bytes key passed to
  C++ structures, Java would get a 20-byte key. This impacted both the size and
  the speed of the Java implementations.

Here we try to give a more balanced set of results:

- The LCP-based MMPHFs are now available in the Rust
  [`sux`](https://crates.io/crates/sux) crate, providing, at least for those
  types of MMPHF, a way out of the “apples-and-oranges“ problem.

- We modified the test so that the Java code would use the cheapest available
  transformation strategy, which simply maps the input to byte arrays. This is
  the same setup used in the paper “[Theory and Practice of Monotone Minimal Perfect
  Hashing](https://doi.org/10.1145/1963190.2025378)“ that introduced them.

- From the results of the paper, a scaling problem was already rather evident at
  larger key sizes. We added a test on 1B URLs showing clear nonconstant behavior:
  on 100M URLs LeMonHash is 4 times slower than an LCP-based MMPHF, but at 1B is
  11 times slower.

The picture one gets from the new experiments is that LeMonHash is probably the
best contender in the “high-compression, slow queries“ corner of the design
space. It certainly is for integer keys. If speed is essential, however,
LCP-based solution use more space but are an order of magnitude faster, and it
is likely the situation is only gonna improve on larger datasets. Given the
indexing nature of these structure (they usually map to some other ancillary
data), the space saving of increased compression is often not worth the
slowdown, but this must be checked on a case-by-case basis.

A lesson learned from this experience is that the experiments in “[Theory and
Practice of Monotone Minimal Perfect
Hashing](https://doi.org/10.1145/1963190.2025378)“ flattened down the
differences between different structures much more than we thought at that time.
The complete genericity was necessary to develop all the structures in a reasonable
amount of time, but the resulting view of the design space hid some relevant
difference in speed.

![eu-2015.urls](./png/eu-2015.urls.png)
![trec-text.terms](./png/trec-text.terms.png)
![dna-31-mer.txt](./png/dna-31-mer.txt.png)
![uk-2007-05.urls](./png/uk-2007-05.urls.png)
![5GRAM_1](./png/5GRAM_1.png)
![fb_200M_uint64](./png/fb_200M_uint64.png)
![osm_cellids_800M_uint64](./png/osm_cellids_800M_uint64.png)
![uniform_uint64](./png/uniform_uint64.png)
![normal_uint64](./png/normal_uint64.png)
![exponential_uint64](./png/exponential_uint64.png)
