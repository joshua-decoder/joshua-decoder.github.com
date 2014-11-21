---
layout: default
category: help
title: Common questions, problems, and solutions
---

1.  How do I make Joshua produce better results?

    One way is to add a larger language model. Build on Gigaword, news
    crawl data, etc. `lmplz` makes it easy to build and efficient to
    represent (especially if you compress it with `build_binary). To
    include it in Joshua, there are two ways:

    - *Pipeline*. By default, Joshua's pipeline builds a language
       model on the target side of your parallel training data. But
       Joshua can decode with any number of additional language models
       as well. So you can build a language model separately,
       presumably on much more data (since you won't be constrained
       only to one side of parallel data, which is much more scarce
       than monolingual data). Once you've built extra language models
       and compiled them with KenLM's `build_binary` script, you can
       tell the pipeline to use them with any number of `--lmfile
       /path/to/lm/file` flags.

    - *Joshua* (directly).
      [This file](http://localhost:4000/6.0/file-formats.html)
      documents the Joshua configuration file format.

1.  I have already run the pipeline once. How do I run it again,
    skipping the early stages and just retuning the model?

    You would need to do this if, for example, you added a language
    model, or changed some other parameter (e.g., an improvement to the
    decoder). To do this, follow the following steps:
   
    - Re-run the pipeline giving it a new `--rundir N+1` (where `N` is the last
      run, and `N+1` is a new, non-existent directory). 
    - Give it all the other flags that you gave before, such as the
      tuning data, testing data, source and target flags, etc. You
      don't have to give it the training data.
    - Tell it to start at the tuning step with `--first-step TUNE`
    - Tell it where all of your language model files are with `--lmfile
      /path/to/lm` lines. You also have to tell it where the main
      language model is, which is usually `--lmfile N/lm.kenlm` (paths
      are relative to the directory above the run directory.
    - Tell it where the main grammar is, e.g., `--grammar
      N/grammar.gz`. If the tuning and test data hasn't changed, you
      can also point it to the filtered and packed versions to save a
      little time using `--tune-grammar N/data/tune/grammar.packed` and
      `--test-grammar N/data/test/grammar.packed`, where `N` here again
      is the previous run (or some other run; it can be anywhere).
    
    Here's an example. Let's say you ran a full pipeline as run 1, and
    now added a new language model and want to see how it affects the
    decoder. Your first run might have been invoked like this:
   
        $JOSHUA/scripts/training/pipeline.pl \
          --rundir 1 \
          --readme "Baseline French--English Europarl hiero system" \
          --corpus /path/to/europarl \
          --tune /path/to/europarl/tune \
          --test /path/to/europarl/test \
          --source fr \
          --target en \
          --threads 8 \
          --joshua-mem 30g \
          --tuner mira \
          --type hiero \
          --aligner berkeley

    Your new run will look like this:

        $JOSHUA/scripts/training/pipeline.pl \
          --rundir 2 \
          --readme "Adding in a huge language model" \
          --tune /path/to/europarl/tune \
          --test /path/to/europarl/test \
          --source fr \
          --target en \
          --threads 8 \
          --joshua-mem 30g \
          --tuner mira \
          --type hiero \
          --aligner berkeley \
          --first-step TUNE \
          --lmfile 1/lm.kenlm \
          --lmfile /path/to/huge/new/lm \
          --tune-grammar 1/data/tune/grammar.packed \
          --test-grammar 1/data/test/grammar.packed

    Notice the changes: we removed the `--corpus` (though it would have
    been fine to have left it, it would have just been skipped),
    specified the first step, changed the run directory and README
    comments, and pointed to the grammars and *both* language model files.

3.  How can I enable specific feature functions?

    Let's say you created a new feature function, `OracleFeature`, and
    you want to enable it. You can do this in two ways. Through the
    pipeline, simply pass it the argument `--joshua-args "list of
    joshua args"`. These will then be passed to the decoder when it is
    invoked. You can enable your feature functions, then using
    something like
    
        $JOSHUA/bin/pipeline.pl --joshua-args '-feature-function OracleFeature'   

    If you call the decoder directly, you can just put that line in
    the configuration file, e.g.,
    
        feature-function = OracleFeature
        
    or you can pass it directly to Joshua on the command line using
    the standard notation, e.g.,
    
        $JOSHUA/bin/joshua-decoder -feature-function OracleFeature
        
    These could be stacked, e.g.,
    
        $JOSHUA/bin/joshua-decoder -feature-function OracleFeature \
            -feature-function MagicFeature \
            -feature-function MTSolverFeature \
            ...

    
