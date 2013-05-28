---
layout: default
category: help
title: Releases
---

# [5.0](5.0/index.html)

2012 December ???

[download](http://cs.jhu.edu/~post/files/joshua-5.0.tgz)

[documentation](5.0/index.html)

## [5.0](5.0/index.html) Release Notes

# [4.0](4.0/index.html)

2012 July 2

[download](http://cs.jhu.edu/~post/files/joshua-4.0.tgz)

[documentation](4.0/index.html)

## [4.0](http://joshua-decoder.org/4.0/index.html) Release Notes

- Significantly improved and extended documentation.

  We have extensively documented many of Joshua's features, focusing especially on parameters affecting (1) the experimental pipeline and (2) the decoder itself.  We plan to continue updating this regularly and believe it will be very useful to end-users of Joshua.

- An official tarball download.

  We have separated the end-user version of Joshua (available through a tarball release) from the development version of Joshua (available through github).  This saves users from downloading many (hundreds of) megabytes of commit history that aren't relevant to people who mostly want to use Joshua as a black box.

- Grammar packing.

  Joshua contains a new grammar representation based on packed tries that increases end-to-end decoding speed and reduces the amount of memory needed store grammars.  At the moment, this is not automatically enabled in the pipeline, but usage instructions are available in the documentation.

- PRO.

  Joshua contains an implementation of Pairwise Ranking Optimization (Hopkins & May, EMNLP 2011).  This can be enabled from the pipeline by passing '--tuner pro'.

- Synchronous parsing.

  Joshua now supports synchronous parsing via two monolingual parses, as described in Dyer (NAACL 2010: http://www.aclweb.org/anthology/N/N10/N10-1033 ).

- Paraphrases.

  Thrax, Joshua's SCFG-based grammar extractor, now supports pivot-based extraction of syntactically-informed paraphrases.

These are in addition to many bugfixes and other small improvements.
