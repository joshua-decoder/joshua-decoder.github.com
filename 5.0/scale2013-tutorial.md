---
layout: default
category: links
title: SCALE 2013 Joshua tutorial
---

If you're running Joshua on the HLTCOE file servers, you're in luck, because if anything the Joshua
setup is overfit to that environment. This page contains some notes tailored to a
[SCALE 2013](http://hltcoe.jhu.edu/research/scale-workshops/) tutorial that took place on June 5,
2013.

This page is designed to supplement the [pipeline walkthrough](pipeline.html), so I recommend you
keep both open in separate tabs.

## Download and Setup

You can copy Joshua 5.0rc2 from `~lorland/workspace/mt/joshua/release/joshua-v5.0rc2.tgz` instead of
downloading it directly. I recommend you install it under `~/code/`. Assuming you do so, you will
have

    export JOSHUA=$HOME/code/joshua-v5.0rc2

You should set the following environment variables. Add these to your ~/.bashrc:

    export HADOOP=/opt/apache/hadoop
    export HADOOP_CONF_DIR=/opt/apache/hadoop/conf/apache-mr/mapreduce
    export MOSES=/home/hltcoe/mpost/code/mosesdecoder
    export SRILM=/home/hltcoe/mpost/code/srilm
    
Then load them:

    source ~/.bashrc
    
## Installation

You don't need to install any external software since it is already installed. The environment
variable exports above will take care of this for you.

## A basic pipeline run

For today's experiments, we'll be translating with a Spanish-English translated dataset collected by
[Chris Callison-Burch](http://cs.jhu.edu/~ccb/), [Adam Lopez](http://cs.jhu.edu/~alopez/), and
[myself](http://cs.jhu.edu/~post/). This dataset contains translations of the LDC transcriptions of
the Fisher Spanish and CALLHOME Spanish datasets. These translations were collected using Amazon's
Mechanical Turk.

We plan to publicly release this dataset later this year, and in the meantime *we ask that you do
 not distribute this data or remove it from HLTCOE servers*.

The dataset is located at `/export/common/SCALE13/Text/fishcall`. Please set the following environment
variable for convenience:

    export FISHCALL=/export/common/SCALE13/Text/fishcall

If you're at CLSP, the data can be found instead at

    export FISHCALL=/home/mpost/data/fishcall
    
### Preparing the data

The data has already been split into training, development, held-out test, and test sets, both for
Fisher Spanish and CALLHOME. The prefixes are as follows:

    $FISHCALL/fisher_train
    $FISHCALL/fisher_dev
    $FISHCALL/fisher_dev2
    $FISHCALL/fisher_test
    
    $FISHCALL/callhome_train
    $FISHCALL/callhome_devtest
    $FISHCALL/callhome_evltest
    
### Run the pipeline

I'll assume here a run directory of `$HOME/expts/scale13/joshua-tutorial/runs/`. To run the complete
pipeline and output results for the Fisher held-out test set, type:

    cd $HOME/expts/scale13/joshua-tutorial/runs/
    $JOSHUA/bin/pipeline.pl           \
      --readme "Baseline run"         \
      --rundir 1                      \
      --corpus $FISHCALL/fisher_train \
      --tune $FISHCALL/fisher_dev     \
      --test $FISHCALL/fisher_dev2    \
      --source es                     \
      --target en
      
This will start the pipeline building a translation system trained on (Spanish transcript, English
translation) pairs, and evaluate on other Spanish transcripts. It will use the defaults for all
pieces of the pipeline: [GIZA++](https://code.google.com/p/giza-pp/) for alignment, BerkeleyLM for
building the language model, batch MIRA for tuning, KenLM for representing LM state in the decoder,
and so on.

### Variations

You can try different variations:

   - Build an SAMT model (`--type samt`), GKHM model (`--type ghkm`), or phrasal model (`--type phrasal`) 
   
   - Use the Berkeley aligner instead of GIZA++ (`--aligner berkeley`)
   
   - Build the language model with SRILM (recommended) instead of BerkeleyLM (`--lm-gen srilm`)

   - Tune with MERT instead of MIRA (`--tuner mert`)
   
   - Decode with a wider beam (`--joshua-args '-pop-limit 200'`) (the default is 100)

   - Add training data (add another `--corpus` line, e.g., `--corpus $FISHCALL/callhome_train`)
