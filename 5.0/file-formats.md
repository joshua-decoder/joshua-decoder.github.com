---
layout: default
category: advanced
title: Joshua file formats
---
This page describes the formats of Joshua configuration and support files.

## Translation models (grammars)

Joshua supports two grammar file formats: a text-based version (also used by Hiero, shared by
[cdec](), and supported by [hierarchical Moses]()), and an efficient
[packed representation](packing.html) developed by [Juri Ganitkevich](http://cs.jhu.edu/~juri).

Grammar rules follow this format.

    [LHS] ||| SOURCE-SIDE ||| TARGET-SIDE ||| FEATURES
    
The source and target sides contain a mixture of terminals and nonterminals. The nonterminals are
linked across sides by indices. There is no limit to the number of paired nonterminals in the rule
or on the nonterminal labels (Joshua supports decoding with SAMT and GHKM grammars).

    [X] ||| el chico [X,1] ||| the boy [X,1] ||| -3.14 0 2 17
    [S] ||| el chico [VP,1] ||| the boy [VP,1] ||| -3.14 0 2 17
    [VP] ||| [NP,1] [IN,2] [VB,3] ||| [VB,3] [IN,2] [NP,1] ||| 0.0019026637 0.81322956

    
The feature values can have optional labels, e.g.:

    [X] ||| el chico [X,1] ||| the boy [X,1] ||| lexprob=-3.14 abstract=0 numwords=2 count=17
    
One file common to decoding is the glue grammar, which for hiero grammar is defined as follows:

    [GOAL] ||| <s> ||| <s> ||| 0
    [GOAL] ||| [GOAL,1] [X,2] ||| [GOAL,1] [X,2] ||| -1
    [GOAL] ||| [GOAL,1] </s> ||| [GOAL,1] </s> ||| 0

Joshua's [pipeline](pipeline.html) supports extraction of Hiero and SAMT grammars via
[Thrax](thrax.html) or GHKM grammars using [Michel Galley](http://www-nlp.stanford.edu/~mgalley/)'s
GHKM extractor (included) or Moses' GHKM extractor (if Moses is installed).

## Language Model

Joshua has two language model implementations: [KenLM](http://kheafield.com/code/kenlm/) and
[BerkeleyLM](http://berkeleylm.googlecode.com).  All language model implementations support the
standard ARPA format output by [SRILM](http://www.speech.sri.com/projects/srilm/).  In addition,
KenLM and BerkeleyLM support compiled formats that can be loaded more quickly and efficiently. KenLM
is written in C++ and is supported via a JNI bridge, while BerkeleyLM is written in Java. KenLM is
the default because of its support for left-state minimization.

### Compiling for KenLM

To compile an ARPA grammar for KenLM, use the (provided) `build-binary` command, located deep within
the Joshua source code:

    $JOSHUA/bin/build_binary lm.arpa lm.kenlm
    
This script takes the `lm.arpa` file and produces the compiled version in `lm.kenlm`.

### Compiling for BerkeleyLM

To compile a grammar for BerkeleyLM, type:

    java -cp $JOSHUA/lib/berkeleylm.jar -server -mxMEM edu.berkeley.nlp.lm.io.MakeLmBinaryFromArpa lm.arpa lm.berkeleylm

The `lm.berkeleylm` file can then be listed directly in the [Joshua configuration file](decoder.html).

## Joshua configuration file

The [decoder page](decoder.html) documents decoder command-line and config file options.

## Thrax configuration

See [the thrax page](thrax.html) for more information about the Thrax configuration file.
