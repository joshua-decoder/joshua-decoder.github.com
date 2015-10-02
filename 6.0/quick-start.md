---
layout: default6
title: Quick Start
---

If you just want to use Joshua to translate data, the quickest way is
to download a [pre-built model](/language-packs/). 

If not language pack is available, or if you have your own parallel
data that you want to train the translation engine on, then you have
to build your own model. This takes a bit more knowledge and effort,
but is made easier with Joshua's [pipeline script](pipeline.html),
which runs all the steps of preparing data, aligning it, and
extracting and tuning component models. 

Detailed information about running the pipeline can be found in
[the pipeline documentation](/6.0/pipeline.html), but as a quick
start, you can build a simple Bengali--English model by following
these instructions.

*NOTE: We suggest you build models outside the `$JOSHUA` directory*.

First, download the dataset:
   
    mkdir -p ~/models/bn-en/
    cd ~/models/bn-en
    wget -q https://github.com/joshua-decoder/indian-parallel-corpora/archive/1.0.tar.gz
    tar xzf indian-parallel-corpora-1.0.tar.gz
    ln -s indian-parallel-corpora-1.0 input

Then, train and test a model

    $JOSHUA/bin/pipeline.pl --source bn --target en \
        --type hiero \
        --no-prepare --aligner berkeley \
        --corpus input/bn-en/tok/training.bn-en \
        --tune input/bn-en/tok/dev.bn-en \
        --test input/bn-en/tok/devtest.bn-en

This will align the data with the Berkeley aligner, build a Hiero
model, tune with MERT, decode the test sets, and reports results that
should correspond with what you find on
[the Indian Parallel Corpora page](/indian-parallel-corpora/). For
more details, including information on the many options available with
the pipeline script, please see [its documentation page](pipeline.html).

Finally, you can export the full model as a language pack:

    ./run-bundler.py \
      tune/joshua.config.final \
      language-pack-bn-en \
      --pack-tm grammar.gz
      
(or possibly `tune/1/joshua.config.final` if you're using an older version of
the pipeline).

This will create a [runnable model](bundle.html) in
`language-pack-bn-en`. See the `README` file in that directory for
information on how to run the decoder.
