---
layout: default
title: Joshua 5.0 User Documentation
---

This page contains end-user oriented documentation for the 5.0 release of
[the Joshua decoder](http://joshua-decoder.org/).

## Download and Setup

1. Follow [this link](http://cs.jhu.edu/~post/files/joshua-v5.0.tgz) to download Joshua, or do it
from the command line:

       wget -q http://cs.jhu.edu/~post/files/joshua-v5.0.tgz

2. Next, unpack it, set environment variables, and compile everything:

       tar xzf joshua-v5.0.tgz
       cd joshua-v5.0

       # for bash
       export JAVA_HOME=/path/to/java
       export JOSHUA=$(pwd)
       echo "export JOSHUA=$JOSHUA" >> ~/.bashrc

       # for tcsh
       setenv JAVA_HOME /path/to/java
       setenv JOSHUA `pwd`
       echo "setenv JOSHUA $JOSHUA" >> ~/.profile
       
       ant all

(If you don't know what to set `$JAVA_HOME` to, try `/usr/java/default`)

3. If you have a Hadoop installation, make sure that the environment variable `$HADOOP` is set and
points to it.

4. If you want to use Cherry & Foster's
[batch MIRA tuner](http://aclweb.org/anthology-new/N/N12/N12-1047v2.pdf) (recommended), you need to
[install Moses](http://www.statmt.org/moses/?n=Development.GetStarted) and define the `$MOSES`
environment variable to point to the root of the Moses installation.

## Quick start

Our <a href="pipeline.html">pipeline script</a> is the quickest way to get started. For example, to
train and test a complete model translating from Bengali to English:

First, download the data:
   
    wget --no-check -O indian-languages.tgz https://github.com/joshua-decoder/indian-parallel-corpora/tarball/master
    tar xf indian-languages.tgz
    ln -s joshua-decoder-indian-parallel-corpora-b71d31a input

Then, train and test a model

    $JOSHUA/bin/pipeline.pl --source bn --target en \
        --no-prepare --aligner berkeley \
        --corpus input/bn-en/tok/training.bn-en \
        --tune input/bn-en/tok/dev.bn-en \
        --test input/bn-en/tok/devtest.bn-en

This will align the data with the Berkeley aligner, build a Hiero model, tune with MERT, decode the
test sets, and reports results that should correspond with what you find on <a
href="http://joshua-decoder.org/indian-parallel-corpora/">the Indian Parallel Corpora page</a>. For
more details, including information on the many options available with the pipeline script, please
see <a href="pipeline.html">its documentation page</a>.

## More information

For more detail on the decoder itself, including its command-line options, see
[the Joshua decoder page](decoder.html).  You can also learn more about other steps of
[the Joshua MT pipeline](pipeline.html), including [grammar extraction](thrax.html) with Thrax and
Joshua's [efficient grammar representation](packing.html).

If you have problems or issues, you might find some help [on our answers page](faq.html) or
[in the mailing list archives](https://groups.google.com/forum/?fromgroups#!forum/joshua_support).

A [bundled configuration](bundle.html), which is a minimal set of configuration, resource, and script files, can be created and easily transferred and shared.
