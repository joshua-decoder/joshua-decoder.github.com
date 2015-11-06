---
layout: default6
title: Installation
---

### Download and install

To use Joshua as a standalone decoder (with [language packs](/language-packs/)), you only need to download and install the runtime version of the decoder. 
If you also wish to build translation models from your own data, you will want to install the full version.
See the instructions below.

1.  Set up some basic environment variables. 
    You need to define `$JAVA_HOME`

       export JAVA_HOME=/path/to/java

       # JAVA_HOME is not very standardized. Here are some places to look:
       # OS X:  export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home
       # Linux: export JAVA_HOME=/usr/java/default

1.  If you are installing the full version of Joshua, you also need to define `$HADOOP` to point to your Hadoop installation.
    (Joshua looks for the Hadoop executuble in `$HADOOP/bin/hadoop`)

       export HADOOP=/usr

    If you don't have a Hadoop installation, [Joshua's pipeline](pipeline.html) can install a standalone version for you.
    
1.  To install just the runtime version of Joshua, type

       wget -q http://cs.jhu.edu/~post/files/joshua-runtime-{{ site.data.joshua.release_version }}.tgz

    Then build everything

       tar xzf joshua-runtime-{{ site.data.joshua.release_version }}.tgz
       cd joshua-runtime-{{ site.data.joshua.release_version }}

       # Add this to your init files
       export JOSHUA=$(pwd)
       
       # build everything
       ant

1.  To instead install the full version, type

       wget -q http://cs.jhu.edu/~post/files/joshua-{{ site.data.joshua.release_version }}.tgz

       tar xzf joshua-{{ site.data.joshua.release_version }}.tgz
       cd joshua-{{ site.data.joshua.release_version }}

       # Add this to your init files
       export JOSHUA=$(pwd)
       
       # build everything
       ant

### Building new models

If you wish to build models for new language pairs from existing data (such as the [WMT data](http://statmt.org/wmt14/)), you need to install some additional dependencies.

1. For learning hierarchical models, Joshua includes a tool called [Thrax](thrax.html), which
is built on Hadoop. If you have a Hadoop installation, make sure that the environment variable
`$HADOOP` is set and points to it. If you don't, Joshua will roll one out for you in standalone
mode. Hadoop is only needed if you plan to build new models with Joshua.

1. You will need to install Moses if either of the following applies to you:

    - You wish to build [phrase-based models](phrase.html) (Joshua 6 includes a phrase-based
      decoder, but not the tools for building such a model)

    - You are building your own models (phrase- or syntax-based) and wish to use Cherry & Foster's
[batch MIRA tuner](http://aclweb.org/anthology-new/N/N12/N12-1047v2.pdf) instead of the included
MERT implementation, [Z-MERT](zmert.html). 

    Follow [the instructions for installing Moses
here](http://www.statmt.org/moses/?n=Development.GetStarted), and then define the `$MOSES`
environment variable to point to the root of the Moses installation.

## More information

For more detail on the decoder itself, including its command-line options, see
[the Joshua decoder page](decoder.html).  You can also learn more about other steps of
[the Joshua MT pipeline](pipeline.html), including [grammar extraction](thrax.html) with Thrax and
Joshua's [efficient grammar representation](packing.html).

If you have problems or issues, you might find some help [on our answers page](faq.html) or
[in the mailing list archives](https://groups.google.com/forum/?fromgroups#!forum/joshua_support).

A [bundled configuration](bundle.html), which is a minimal set of configuration, resource, and script files, can be created and easily transferred and shared.
