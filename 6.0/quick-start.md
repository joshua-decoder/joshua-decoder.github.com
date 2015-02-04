---
layout: default6
title: Quick Start
---

The quickest way to use Joshua is to download a
[pre-built model](/6.0/models.html) and use them to start translating data.

Building your own models takes a bit more work, and requires you to
supply parallel data that the models can be trained from. Information
about how to do this can be found in [the pipeline documentation](/6.0/pipeline.html).

Our pipeline.html">pipeline script</a> is the quickest way to get started. For example, to
train and test a complete model translating from Bengali to English:

First, download the Indian languages data:
   
   curl -#L https://github.com/joshua-decoder/indian-parallel-corpora/tarball/master > indian-languages.tgz
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
href="/indian-parallel-corpora/">the Indian Parallel Corpora page</a>. For
more details, including information on the many options available with the pipeline script, please
see <a href="pipeline.html">its documentation page</a>.

## More information

For more detail on the decoder itself, including its command-line options, see
[the Joshua decoder page](decoder.html).  You can also learn more about other steps of
[the Joshua MT pipeline](pipeline.html), including [grammar extraction](thrax.html) with Thrax and
Joshua's [efficient grammar representation](packing.html).

If you have problems or issues, you might find some help [on our answers page](faq.html) or
[in the mailing list archives](https://groups.google.com/forum/?fromgroups#!forum/joshua_support).

A [bundled configuration](bundle.html), which is a minimal set of
configuration, resource, and script files, can be created and easily
transferred and shared. (This is what we use to distribute pre-built models).
