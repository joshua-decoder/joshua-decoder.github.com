---
layout: default
category: links
title: Pipeline tutorial
---

This document will walk you through using the pipeline in a variety of scenarios. Once you've gained a
sense for how the pipeline works, you can consult the [pipeline page](pipeline.html) for a number of
other options available in the pipeline.

## Download and Setup

Download and install Joshua as described on the [quick start page](index.html), installing it under
`~/code/`. Once you've done that, you should make sure you have the following environment variable set:

    export JOSHUA=$HOME/code/joshua-v5.0
    export JAVA_HOME=/usr/java/default

If you have a Hadoop installation, make sure you've set `$HADOOP` to point to it (if not, Joshua
will roll out a standalone cluster for you). If you'd like to use kbmira for tuning, you should also
install Moses, and define the environment variable `$MOSES` to point to the root of its installation.

## A basic pipeline run

For today's experiments, we'll be building a Bengali--English system using data included in the
[Indian Languages Parallel Corpora](/indian-parallel-corpora/). This data was collected by taking
the 100 most-popular Bengali Wikipedia pages and translating them into English using Amazon's
[Mechanical Turk](http://www.mturk.com/). As a warning, many of these pages contain material that is
not typically found in machine translation tutorials.

Download the data and install it somewhere:

    cd ~/data
    wget -q --no-check -O indian-parallel-corpora.zip https://github.com/joshua-decoder/indian-parallel-corpora/archive/master.zip
    unzip indian-parallel-corpora.zip

Then define the environment variable `$INDIAN` to point to it:

    cd ~/data/indian-parallel-corpora-master
    export INDIAN=$(pwd)
    
### Preparing the data

Inside this tarball is a directory for each language pair. Within each language directory is another
directory named `tok/`, which contains pre-tokenized and normalized versions of the data. This was
done because the normalization scripts provided with Joshua is written in scripting languages that
often have problems properly handling UTF-8 character sets. We will be using these tokenized
versions, and preventing the pipeline from retokenizing using the `--no-prepare-data` flag.

In `$INDIAN/bn-en/tok`, you should see the following files:

    $ ls $INDIAN/bn-en/tok
    dev.bn-en.bn     devtest.bn-en.bn     dict.bn-en.bn     test.bn-en.en.2
    dev.bn-en.en.0   devtest.bn-en.en.0   dict.bn-en.en     test.bn-en.en.3
    dev.bn-en.en.1   devtest.bn-en.en.1   test.bn-en.bn     training.bn-en.bn
    dev.bn-en.en.2   devtest.bn-en.en.2   test.bn-en.en.0   training.bn-en.en
    dev.bn-en.en.3   devtest.bn-en.en.3   test.bn-en.en.1

We will now use this data to test the complete pipeline with a single command.
    
### Run the pipeline

Create an experiments directory for containing your first experiment:

    mkdir ~/expts/joshua
    cd ~/expts/joshua
    
We will now create the baseline run, using a particular directory structure for experiments that
will allow us to take advantage of scripts provided with Joshua for displaying the results of many
related experiments.

    cd ~/expts/joshua
    $JOSHUA/bin/pipeline.pl           \
      --rundir 1                      \
      --readme "Baseline Hiero run"   \
      --source bn                     \
      --target en                     \
      --corpus $INDIAN/bn-en/tok/training.bn-en \
      --corpus $INDIAN/bn-en/tok/dict.bn-en     \
      --tune $INDIAN/bn-en/tok/dev.bn-en        \
      --test $INDIAN/bn-en/tok/devtest.bn-en    \
      --lm-order 3
      
This will start the pipeline building a Bengali--English translation system constructed from the
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

- Add the provided BN-EN dictionary to the training data (add another `--corpus` line, e.g., `--corpus $INDIAN/bn-en/dict.bn-en`)

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
      --corpus $INDIAN/bn-en/tok/training.bn-en \
      --tune $INDIAN/bn-en/tok/dev.bn-en        \
      --test $INDIAN/bn-en/tok/devtest.bn-en    \
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
      --corpus $INDIAN/bn-en/tok/training.bn-en \
      --tune $INDIAN/bn-en/tok/dev.bn-en        \
      --test $INDIAN/bn-en/tok/devtest.bn-en    \
      --alignment 1/alignments/training.align   \
      --first-step parse \
      --no-corpus-lm \
      --lmfile 1/lm.gz

See [the pipeline script page](pipeline.html#steps) for a list of all the steps.

## Analyzing the results

We now have three runs, in subdirectories 1, 2, and 3. We can display summary results from them
using the `$JOSHUA/scripts/training/summarize.pl` script.
