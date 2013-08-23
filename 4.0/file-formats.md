---
layout: default4
category: advanced
title: Joshua file formats
---
This page describes the formats of Joshua configuration and support files.

## Translation models (grammars)

Joshua supports three grammar file formats.

1. Thrax / Hiero
1. SAMT [deprecated]
1. packed

The *Hiero* format is not restricted to Hiero grammars, but simply means *the format that David
Chiang developed for Hiero*.  It can support a much broader class of SCFGs containing an arbitrary
set of nonterminals.  Similarly, the *SAMT* format is not restricted to SAMT grammars but instead
simply denotes *the grammar format that Zollmann and Venugopal developed for their decoder*.  To
remove this source of confusion, "thrax" is the preferred format designation, and is in fact the
default.

The packed grammar format is the efficient grammar representation developed by
[Juri Ganitkevich](http://cs.jhu.edu/~juri) [is described in detail elsewhere](packing.html).

Grammar rules in the Thrax format follow this format:

    [LHS] ||| SOURCE-SIDE ||| TARGET-SIDE ||| FEATURES
    
Here are some two examples, one for a Hiero grammar, and the other for an SAMT grammar:

    [X] ||| el chico [X] ||| the boy [X] ||| -3.14 0 2 17
    [S] ||| el chico [VP] ||| the boy [VP] ||| -3.14 0 2 17
    
The feature values can have optional labels, e.g.:

    [X] ||| el chico [X] ||| the boy [X] ||| lexprob=-3.14 abstract=0 numwords=2 count=17
    
These feature names are made up.  For an actual list of feature names, please
[see the Thrax documentation](thrax.html).

The SAMT grammar format is deprecated and undocumented.

## Language Model

Joshua has three language model implementations: [KenLM](), [BerkeleyLM](), and an (unrecommended)
dummy Java implementation.  All language model implementations support the standard ARPA format
output by [SRILM]().  In addition, KenLM and BerkeleyLM support compiled formats that can be loaded
more quickly and efficiently.

### Compiling for KenLM

To compile an ARPA grammar for KenLM, use the (provided) `build-binary` command, located deep within
the Joshua source code:

    $JOSHUA/src/joshua/decoder/ff/lm/kenlm/build_binary lm.arpa lm.kenlm
    
This script takes the `lm.arpa` file and produces the compiled version in `lm.kenlm`.

### Compiling for BerkeleyLM

To compile a grammar for BerkeleyLM, type:

    java -cp $JOSHUA/lib/berkeleylm.jar -server -mxMEM edu.berkeley.nlp.lm.io.MakeLmBinaryFromArpa lm.arpa lm.berkeleylm

The `lm.berkeleylm` file can then be listed directly in the [Joshua configuration file](decoder.html).

## Joshua configuration

See [the decoder page](decoder.html).

## Pipeline configuration

See [the pipeline page](pipeline.html).

## Thrax configuration

See [the thrax page](thrax.html).
