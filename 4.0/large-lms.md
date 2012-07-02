---
layout: default
title: Building large LMs with SRILM
category: advanced
---

The following is a tutorial for building a large language model from the
English Gigaword Fifth Edition corpus
[LDC2011T07](http://www.ldc.upenn.edu/Catalog/catalogEntry.jsp?catalogId=LDC2011T07)
using SRILM. English text is provided from seven different sources.

### Step 0: Clean up the corpus

The Gigaword corpus has to be stripped of all SGML tags and tokenized.
Instructions for performing those steps are not included in this
documentation. A description of this process can be found in a paper
called ["Annotated
Gigaword"](https://akbcwekex2012.files.wordpress.com/2012/05/28_paper.pdf).

The Joshua package ships with a script that converts all alphabetical
characters to their lowercase equivalent. The script is located at
`$JOSHUA/scripts/lowercase.perl`.

Make a directory structure as follows:

    gigaword/
    ├── corpus/
    │   ├── afp_eng/
    │   │   ├── afp_eng_199405.lc.gz
    │   │   ├── afp_eng_199406.lc.gz
    │   │   ├── ...
    │   │   └── counts/
    │   ├── apw_eng/
    │   │   ├── apw_eng_199411.lc.gz
    │   │   ├── apw_eng_199412.lc.gz
    │   │   ├── ...
    │   │   └── counts/
    │   ├── cna_eng/
    │   │   ├── ...
    │   │   └── counts/
    │   ├── ltw_eng/
    │   │   ├── ...
    │   │   └── counts/
    │   ├── nyt_eng/
    │   │   ├── ...
    │   │   └── counts/
    │   ├── wpb_eng/
    │   │   ├── ...
    │   │   └── counts/
    │   └── xin_eng/
    │       ├── ...
    │       └── counts/
    └── lm/
        ├── afp_eng/
        ├── apw_eng/
        ├── cna_eng/
        ├── ltw_eng/
        ├── nyt_eng/
        ├── wpb_eng/
        └── xin_eng/


The next step will be to build smaller LMs and then interpolate them into one
file.

### Step 1: Count ngrams

Run the following script once from each source directory under the `corpus/`
directory (edit it to specify the path to the `ngram-count` binary as well as
the number of processors):

    #!/bin/sh

    NGRAM_COUNT=$SRILM_SRC/bin/i686-m64/ngram-count
    args=""

    for source in *.gz; do
       args=$args"-sort -order 5 -text $source -write counts/$source-counts.gz "
    done

    echo $args | xargs --max-procs=4 -n 7 $NGRAM_COUNT

Then move each `counts/` directory to the corresponding directory under
`lm/`. Now that each ngram has been counted, we can make a language
model for each of the seven sources.

### Step 2: Make individual language models

SRILM includes a script, called `make-big-lm`, for building large language
models under resource-limited environments. The manual for this script can be
read online
[here](http://www-speech.sri.com/projects/srilm/manpages/training-scripts.1.html).
Since the Gigaword corpus is so large, it is convenient to use `make-big-lm`
even in environments with many parallel processors and a lot of memory.

Initiate the following script from each of the source directories under the
`lm/` directory (edit it to specify the path to the `make-big-lm` script as
well as the pruning threshold):

    #!/bin/bash
    set -x

    CMD=$SRILM_SRC/bin/make-big-lm
    PRUNE_THRESHOLD=1e-8

    $CMD \
      -name gigalm `for k in counts/*.gz; do echo " \
      -read $k "; done` \
      -lm lm.gz \
      -max-per-file 100000000 \
      -order 5 \
      -kndiscount \
      -interpolate \
      -unk \
      -prune $PRUNE_THRESHOLD

The language model attributes chosen are the following:

* N-grams up to order 5
* Kneser-Ney smoothing
* N-gram probability estimates at the specified order *n* are interpolated with
  lower-order estimates
* include the unknown-word token as a regular word
* pruning N-grams based on the specified threshold

Next, we will mix the models together into a single file.

### Step 3: Mix models together

Using development text, interpolation weights can determined that give highest
weight to the source language models that have the lowest perplexity on the
specified development set.

#### Step 3-1: Determine interpolation weights

Initiate the following script from the `lm/` directory (edit it to specify the
path to the `ngram` binary as well as the path to the development text file):

    #!/bin/bash
    set -x

    NGRAM=$SRILM_SRC/bin/i686-m64/ngram
    DEV_TEXT=~mpost/expts/wmt12/runs/es-en/data/tune/tune.tok.lc.es

    dirs=( afp_eng apw_eng cna_eng ltw_eng nyt_eng wpb_eng xin_eng )

    for d in ${dirs[@]} ; do
      $NGRAM -debug 2 -order 5 -unk -lm $d/lm.gz -ppl $DEV_TEXT > $d/lm.ppl ;
    done

    compute-best-mix */lm.ppl > best-mix.ppl

Take a look at the contents of `best-mix.ppl`. It will contain a sequence of
values in parenthesis. These are the interpolation weights of the source
language models in the order specified. Copy and paste the values within the
parenthesis into the script below.

#### Step 3-2: Combine the models

Initiate the following script from the `lm/` directory (edit it to specify the
path to the `ngram` binary as well as the interpolation weights):

    #!/bin/bash
    set -x

    NGRAM=$SRILM_SRC/bin/i686-m64/ngram
    DIRS=(   afp_eng    apw_eng     cna_eng  ltw_eng   nyt_eng  wpb_eng  xin_eng )
    LAMBDAS=(0.00631272 0.000647602 0.251555 0.0134726 0.348953 0.371566 0.00749238)

    $NGRAM -order 5 -unk \
      -lm      ${DIRS[0]}/lm.gz     -lambda  ${LAMBDAS[0]} \
      -mix-lm  ${DIRS[1]}/lm.gz \
      -mix-lm2 ${DIRS[2]}/lm.gz -mix-lambda2 ${LAMBDAS[2]} \
      -mix-lm3 ${DIRS[3]}/lm.gz -mix-lambda3 ${LAMBDAS[3]} \
      -mix-lm4 ${DIRS[4]}/lm.gz -mix-lambda4 ${LAMBDAS[4]} \
      -mix-lm5 ${DIRS[5]}/lm.gz -mix-lambda5 ${LAMBDAS[5]} \
      -mix-lm6 ${DIRS[6]}/lm.gz -mix-lambda6 ${LAMBDAS[6]} \
      -write-lm mixed_lm.gz

The resulting file, `mixed_lm.gz` is a language model based on all the text in
the Gigaword corpus and with some probabilities biased to the development text
specify in step 3-1. It is in the ARPA format. The optional next step converts
it into KenLM format.

#### Step 3-3: Convert to KenLM

The KenLM format has some speed advantages over the ARPA format. Issuing the
following command will write a new language model file `mixed_lm-kenlm.gz` that
is the `mixed_lm.gz` language model transformed into the KenLM format.

    $JOSHUA/src/joshua/decoder/ff/lm/kenlm/build_binary mixed_lm.gz mixed_lm.kenlm

