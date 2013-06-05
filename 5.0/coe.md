---
layout: default
category: links
title: Notes for running the Joshua Pipeline at HLTCOE
---

If you're running Joshua on the HLTCOE file servers, you're in luck, because if anything the Joshua
setup is overfit to that environment. This page contains some notes tailored to a
[SCALE 2013](http://hltcoe.jhu.edu/research/scale-workshops/) tutorial that took place on June 5,
2013.

This page is designed to supplement the [pipeline walkthrough](pipeline.html), so I recommend you
keep both open in separate tabs.

## Download and Setup

You can copy Joshua 5.0rc2 from `~lorland/workspace/mt/joshua/release/joshua-v5.0rc2.tgz` instead of
downloading it directly.

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

We plan to publicly release this dataset later this year, and in the meantim *we ask that you do not
 distribute this data or remove it from HLTCOE servers*.

The dataset is located at `/export/common/SCALE13/Text/fishcall`. Please set the following environment
variable for convenience:

    export FISHCALL=/export/common/SCALE13/Text/fishcall
    
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
    
