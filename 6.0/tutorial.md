---
layout: default6
category: links
title: Pipeline tutorial
---

This document will walk you through using the pipeline in a variety of scenarios. Once you've gained a
sense for how the pipeline works, you can consult the [pipeline page](pipeline.html) for a number of
other options available in the pipeline.

## Download and Setup

Download and install Joshua as described on the [quick start page](index.html), installing it under
`~/code/`. Once you've done that, you should make sure you have the following environment variable set:

    export JOSHUA=$HOME/code/joshua-v{{ site.data.joshua.release_version }}
    export JAVA_HOME=/usr/java/default

If you have a Hadoop installation, make sure you've set `$HADOOP` to point to it. For example, if the `hadoop` command is in `/usr/bin`,
you should type

    export HADOOP=/usr

Joshua will find the binary and use it to submit to your hadoop cluster. If you don't have one, just
make sure that HADOOP is unset, and Joshua will roll one out for you and run it in
[standalone mode](https://hadoop.apache.org/docs/r1.2.1/single_node_setup.html). 

## A basic pipeline run

For today's experiments, we'll be building a Spanish--English system using data included in the
[Fisher and CALLHOME translation corpus](/data/fisher-callhome-corpus/). This
data was collected by translating transcribed speech from previous LDC releases.

Download the data and install it somewhere:

    cd ~/data
    wget --no-check -O fisher-spanish-corpus.zip https://github.com/joshua-decoder/fisher-callhome-corpus/archive/master.zip
    unzip fisher-spanish-corpus.zip

Then define the environment variable `$FISHER` to point to it:

    cd ~/data/fisher-spanish-corpus-master
    export FISHER=$(pwd)
    
### Preparing the data

Inside the tarball is the Fisher and CALLHOME Spanish--English data, which includes Kaldi-provided
ASR output and English translations on the Fisher and CALLHOME  dataset transcriptions. Because of
licensing restrictions, we cannot distribute the Spanish transcripts, but if you have an LDC site
license, a script is provided to build them. You can type:

    ./bin/build_fisher.sh /export/common/data/corpora/LDC/LDC2010T04

Where the first argument is the path to your LDC data release. This will create the files in `corpus/ldc`.

In `$FISHER/corpus`, there are a set of parallel directories for LDC transcripts (`ldc`), ASR output
(`asr`), oracle ASR output (`oracle`), and ASR lattice output (`plf`). The files look like this:

    $ ls corpus/ldc
    callhome_devtest.en  fisher_dev2.en.2  fisher_dev.en.2   fisher_test.en.2
    callhome_evltest.en  fisher_dev2.en.3  fisher_dev.en.3   fisher_test.en.3
    callhome_train.en    fisher_dev2.es    fisher_dev.es     fisher_test.es
    fisher_dev2.en.0     fisher_dev.en.0   fisher_test.en.0  fisher_train.en
    fisher_dev2.en.1     fisher_dev.en.1   fisher_test.en.1  fisher_train.es

If you don't have the LDC transcripts, you can use the data in `corpus/asr` instead. We will now use
this data to build our own Spanish--English model using Joshua's pipeline.
    
### Run the pipeline

Create an experiments directory for containing your first experiment. *Note: it's important that
this **not** be inside your `$JOSHUA` directory*.

    mkdir ~/expts/joshua
    cd ~/expts/joshua
    
We will now create the baseline run, using a particular directory structure for experiments that
will allow us to take advantage of scripts provided with Joshua for displaying the results of many
related experiments. Because this can take quite some time to run, we are going to add a crippling
restriction: Joshua will only use sentences in the training sets with ten or fewer words on either
side (Spanish or English):

    cd ~/expts/joshua
    $JOSHUA/bin/pipeline.pl           \
      --rundir 1                      \
      --readme "Baseline Hiero run"   \
      --source es                     \
      --target en                     \
      --type hiero                    \
      --corpus $FISHER/corpus/ldc/fisher_train \
      --tune $FISHER/corpus/ldc/fisher_dev \
      --test $FISHER/corpus/ldc/fisher_dev2 \
      --maxlen 10 \
      --lm-order 3
      
This will start the pipeline building a Spanish--English translation system constructed from the
training data and a dictionary, tuned against dev, and tested against devtest. It will use the
default values for most of the pipeline: [GIZA++](https://code.google.com/p/giza-pp/) for alignment,
KenLM's `lmplz` for building the language model, Z-MERT for tuning, KenLM with left-state
minimization for representing LM state in the decoder, and so on. We change the order of the n-gram
model to 3 (from its default of 5) because there is not enough data to build a 5-gram LM.

A few notes:

- This will likely take many hours to run, especially if you don't have a Hadoop cluster.

- If you are running on Mac OS X, KenLM's `lmplz` will not build due to the absence of static
  libraries. In that case, you should add the flag `--lm-gen srilm` (recommended, if SRILM is
  installed) or `--lm-gen berkeleylm`.

### Variations

Once that is finished, you will have a baseline model. From there, you might wish to try variations
of the baseline model. Here are some examples of what you could vary:

- Build an SAMT model (`--type samt`), GKHM model (`--type ghkm`), or phrasal ITG model (`--type phrasal`) 
   
- Use the Berkeley aligner instead of GIZA++ (`--aligner berkeley`)
   
- Build the language model with BerkeleyLM (`--lm-gen srilm`) instead of KenLM (the default)

- Change the order of the LM from the default of 5 (`--lm-order 4`)

- Tune with MIRA instead of MERT (`--tuner mira`). This requires that Moses is installed.
   
- Decode with a wider beam (`--joshua-args '-pop-limit 200'`) (the default is 100)

- Add the provided BN-EN dictionary to the training data (add another `--corpus` line, e.g., `--corpus $FISHER/bn-en/dict.bn-en`)

To do this, we will create new runs that partially reuse the results of previous runs. This is
possible by doing two things: (1) incrementing the run directory and providing an updated README
note; (2) telling the pipeline which of the many steps of the pipeline to begin at; and (3)
providing the needed dependencies.

# A second run

Let's begin by changing the tuner, to see what effect that has. To do so, we change the run
directory, tell the pipeline to start at the tuning step, and provide the needed dependencies:

    $JOSHUA/bin/pipeline.pl           \
      --rundir 2                      \
      --readme "Tuning with MIRA"     \
      --source bn                     \
      --target en                     \
      --corpus $FISHER/bn-en/tok/training.bn-en \
      --tune $FISHER/bn-en/tok/dev.bn-en        \
      --test $FISHER/bn-en/tok/devtest.bn-en    \
      --first-step tune \
      --tuner mira \
      --grammar 1/grammar.gz \
      --no-corpus-lm \
      --lmfile 1/lm.gz
      
 Here, we have essentially the same invocation, but we have told the pipeline to use a different
 MIRA, to start with tuning, and have provided it with the language model file and grammar it needs
 to execute the tuning step. 
 
 Note that we have also told it not to build a language model. This is necessary because the
 pipeline always builds an LM on the target side of the training data, if provided, but we are
 supplying the language model that was already built. We could equivalently have removed the
 `--corpus` line.
 
## Changing the model type

Let's compare the Hiero model we've already built to an SAMT model. We have to reextract the
grammar, but can reuse the alignments and the language model:

    $JOSHUA/bin/pipeline.pl           \
      --rundir 3                      \
      --readme "Baseline SAMT model"  \
      --source bn                     \
      --target en                     \
      --corpus $FISHER/bn-en/tok/training.bn-en \
      --tune $FISHER/bn-en/tok/dev.bn-en        \
      --test $FISHER/bn-en/tok/devtest.bn-en    \
      --alignment 1/alignments/training.align   \
      --first-step parse \
      --no-corpus-lm \
      --lmfile 1/lm.gz

See [the pipeline script page](pipeline.html#steps) for a list of all the steps.

## Analyzing the results

We now have three runs, in subdirectories 1, 2, and 3. We can display summary results from them
using the `$JOSHUA/scripts/training/summarize.pl` script.
