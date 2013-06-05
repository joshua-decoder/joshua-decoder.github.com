---
layout: default
title: Joshua 5.0 User Documentation
---

This page contains end-user oriented documentation for the 5.0 release of
[the Joshua decoder](http://joshua-decoder.org/).

## Download and Setup

1. Follow [this link](http://cs.jhu.edu/~post/files/joshua-v5.0rc2.tgz) to download Joshua, or do it
from the command line:

       wget -q http://cs.jhu.edu/~post/files/joshua-v5.0rc2.tgz

2. Next, unpack it, set environment variables, and compile everything:

       tar xzf joshua-v5.0rc2.tgz
       cd joshua-v5.0rc2

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

If you just want to run the complete machine translation pipeline (beginning with data preparation,
through alignment, hierarchical model building, tuning, testing, and reporting), we recommend you
use our <a href="pipeline.html">pipeline script</a>.  

## More information

For more detail on the decoder itself, including its command-line options, see
[the Joshua decoder page](decoder.html).  You can also learn more about other steps of
[the Joshua MT pipeline](pipeline.html), including [grammar extraction](thrax.html) with Thrax and
Joshua's [efficient grammar representation](packing.html) (new with version 5.0).

If you have problems or issues, you might find some help [on our answers page](faq.html) or
[in the mailing list archives](https://groups.google.com/forum/?fromgroups#!forum/joshua_support).

A [bundled configuration](bundle.html), which is a minimal set of configuration, resource, and script files, can be created and easily transferred and shared.
