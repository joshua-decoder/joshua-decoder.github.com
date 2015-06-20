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

The script takes just two mandatory arguments in the following order:

1.  The path to the Joshua configuration file to base the bundle
    on. This file should contain the tuned weights from the tuning run, so
    you can use either the final tuned file from the tuning run
    (`tune/joshua.config.final`) or from the test run
    (`test/model/joshua.config`).
1.  The directory to place the language pack in. If this directory
    already exists, the script will die, unless you also pass `--force`.

In addition, there are a number of other arguments that may be important.

- `--root /path/to/root`. If file paths in the Joshua config file are
   not absolute, you need to provide relative root. If you specify a
   tuned pipeline file (such as `tune/joshua.config.final` above), the
   paths should all be absolute. If you instead provide a config file
   from a previous run bundle (e.g., `test/model/joshua.config`), the
   bundle directory above is the relative root.

- The config file options that are used in the pipeline are likely not
  the ones you want if you release a model. For example, the tuning
  configuration file contains options that tell Joshua to output 300
  translation candidates for each sentence (`-top-n 300`) and to
  include lots of detail about each translation (`-output-format '%i
  ||| %s ||| %f ||| %c'`).  Because of this, you will want to tell the
  run bundler to change many of the config file options to be more
  geared towards human-readable output. The default copy-config
  options are options are `-top-n 0 -output-format %S -mark-oovs
  false`, which accomplishes exactly this (human readability).
  
- A very important issue has to do with the translation model (the
  "TM", also sometimes called the grammar or phrase table). The
  translation model can be very large, so that it takes a long time to
  load and to [pack](packing.html). To reduce this time during model
  training, the translation model is filtered against the tuning and
  testing data in the pipeline, and these filtered models will be what
  is listed in the source config files. However, when exporting a
  model for use as a language pack, you need to export the full model
  instead of the filtered one so as to maximize your coverage on new
  test data. The `--tm` parameter is used to accomplish this; it takes
  an argument specifying the path to the full model. If you would
  additionally like the large model to be [packed](packing.html) (this
  is recommended; it reformats the TM so that it can be quickly loaded
  at run time), you can use `--pack-tm` instead. You can only pack one
  TM (but typically there is only TM anyway). Multiple `--tm`
  parameters can be passed; they will replace TMs found in the config
  file in the order they are found.

Here is an example invocation for packing a hierarchical model using
the final tuned Joshua config file:

    ./run-bundler.py \
      --force --verbose \
      /path/to/rundir/tune/joshua.config.final \
      language-pack-YYYY-MM-DD \
      --root /path/to/rundir \
      --pack-tm /path/to/rundir/grammar.gz \
      --copy-config-options \ 
        '-top-n 1 -output-format %S -mark-oovs false' \
      --server-port 5674

The copy config options tell the decoder to present just the
single-best (`-top-n 0`) translated output string that has been
heuristically capitalized (`-output-format %S`), to not append `_OOV`
to OOVs (`-mark-oovs false`), and to use the translation model
`/path/to/rundir/grammar.gz` as the main translation model, packing it
before placing it in the bundle. Note that these arguments to
`--copy-config` are the default, so you could leave this off entirely.
See [this page](decoder.html) for a longer list of decoder options.

This command is a slight variation used for phrase-based models, which
instead takes the test-set Joshua config (the result is the same):

    ./run-bundler.py \
      --force --verbose \
      /path/to/rundir/test/model/joshua.config \
      --root /path/to/rundir/test/model \
      language-pack-YYYY-MM-DD \
      --pack-tm /path/to/rundir/model/phrase-table.gz \
      --server-port 5674

In both cases, a new directory `language-pack-YYYY-MM-DD` will be
created along with a README and a number of support files.

