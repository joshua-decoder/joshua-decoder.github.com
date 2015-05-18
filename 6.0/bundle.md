---
layout: default6
category: links
title: Building a language pack
---

*The information in this page applies to Joshua 6.0.3 and greater*.

Joshua distributes [language packs](/language-packs), which are models
that have been trained and tuned for particular language pairs. You
can easily create your own language pack after you have trained and
tuned a model using the provided
`$JOSHUA/scripts/support/run-bundler.py` script, which gathers files
from a pipeline training directory and bundles them together for easy
distribution and release.

The script takes a number of mandatory arguments in the following
order:

1.  The path to the Joshua configuration file to base the bundle
    on. This file should contain the tuned weights from the tuning run, so
    you can use either the final tuned file from the tuning run
    (`tune/1/joshua.config.final`) or from the test run
    (`test/1/joshua.config`).
1.  The base directory of the pipeline run directory. This is needed
    because all of the paths in the config file are relative to this
    directory.
1.  The directory to place the language pack in. If this directory
    already exists, the script will die, unless you also pass `--force`.

The config file options that are used in the pipeline are likely not
the ones you want if you release a model. For example, the tuning
configuration file contains options that tell Joshua to output 300
translation candidates for each sentence (`-top-n 300`) and to include lots of
detail about each translation (`-output-format '%i ||| %s ||| %f ||| %c'`). 
Because of this, you will want to tell the run bundler to change many
of the config file options to be more geared towards human-readable
output. 

Another important issue has to do with the translation model (the
"TM", also sometimes called the grammar). The translation model can be
very large, so that it takes a long time to load and to
[pack](packing.html). To reduce this time, the translation model is
filtered against the tuning and testing data in the pipeline, and
these filtered models will be what is listed in the source config
files. However, when exporting a model for use as a language pack, you
need to export the full model instead of the filtered one. The `--tm`
line is used to accomplish this; it takes two arguments: the first
identifies the TM's owner, and the second, the updated path.

Here is an example invocation:

    ./run-bundler.py \
      --force \
      /path/to/rundir/test/1/joshua.config \
      language-pack-YYYY-MM-DD \
      --root /path/to/rundir \
      --pack-tm /path/to/rundir/grammar.gz \
      --copy-config-options \ 
        '-top-n 1 -output-format %S -mark-oovs false' \
      --server-port 5674

The copy config options tell the decoder to present just the
single-best (`-top-n 0`) translated output string that has been
heuristically capitalized (`-output-format %S`), to not append `_OOV`
to OOVs (`-mark-oovs false`), and to use the grammar
`/path/to/rundir/grammar.gz` as the main translation grammar. See
[this page](decoder.html) for a longer list of decoder options.

The `--pack-tm` option tells the run bundler to
[pack the grammar](packing.html), which can take some time. Since it
is the first of any `--[pack-]tm` arguments, it applies to the first
TM encountered in the config file.

A new directory `language-pack-YYYY-MM-DD` will be created along with
a README and a number of support files.
