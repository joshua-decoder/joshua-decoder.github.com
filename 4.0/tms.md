---
layout: default
category: advanced
title: Building Translation Models
---

# Build a translation model

Extracting a grammar from a large amount of data is a multi-step process. The first requirement is parallel data. The Europarl, Call Home, and Fisher corpora all contain parallel translations of Spanish and English sentences.

We will copy (or symlink) the parallel source text files in a subdirectory called `input/`.

Then, we concatenate all the training files on each side. The pipeline script normally does tokenization and normalization, but in this instance we have a custom tokenizer we need to apply to the source side, so we have to do it manually and then skip that step using the `pipeline.pl` option `--first-step alignment`.

* to tokenize the English data, do

    cat callhome.en europarl.en fisher.en > all.en | $JOSHUA/scripts/training/normalize-punctuation.pl en | $JOSHUA/scripts/training/penn-treebank-tokenizer.perl | $JOSHUA/scripts/lowercase.perl > all.norm.tok.lc.en

The same can be done for the Spanish side of the input data:

    cat callhome.es europarl.es fisher.es > all.es | $JOSHUA/scripts/training/normalize-punctuation.pl es | $JOSHUA/scripts/training/penn-treebank-tokenizer.perl | $JOSHUA/scripts/lowercase.perl > all.norm.tok.lc.es

By the way, an alternative tokenizer is a Twitter tokenizer found in the [Jerboa](http://github.com/vandurme/jerboa) project.

The final step in the training data preparation is to remove all examples in which either of the language sides is a blank line.

    paste all.norm.tok.lc.es all.norm.tok.lc.en | grep -Pv "^\t|\t$" \
      | ./splittabs.pl all.norm.tok.lc.noblanks.es all.norm.tok.lc.noblanks.en

contents of `splittabls.pl` by Matt Post:

    #!/usr/bin/perl

    # splits on tab, printing respective chunks to the list of files given
    # as script arguments

    use FileHandle;

    my @fh;
    $| = 1;   # don't buffer output

    if (@ARGV < 0) {
      print "Usage: splittabs.pl < tabbed-file\n";
      exit;
    }

    my @fh = map { get_filehandle($_) } @ARGV;
    @ARGV = ();

    while (my $line = <>) {
      chomp($line);
      my (@fields) = split(/\t/,$line,scalar @fh);

      map { print {$fh[$_]} "$fields[$_]\n" } (0..$#fields);
    }

    sub get_filehandle {
        my $file = shift;

        if ($file eq "-") {
            return *STDOUT;
        } else {
            local *FH;
            open FH, ">$file" or die "can't open '$file' for writing";
            return *FH;
        }
    }

Now we can run the pipeline to extract the grammar. Run the following script:

    #!/bin/bash

    # this creates a grammar

    # NEED:
    # pair
    # type

    set -u

    pair=es-en
    type=hiero

    #. ~/.bashrc

    #basedir=$(pwd)

    dir=grammar-$pair-$type

    [[ ! -d $dir ]] && mkdir -p $dir
    cd $dir

    source=$(echo $pair | cut -d- -f 1)
    target=$(echo $pair | cut -d- -f 2)

    $JOSHUA/scripts/training/pipeline.pl \
      --source $source \
      --target $target \
      --corpus /home/hltcoe/lorland/expts/scale12/model1/input/all.norm.tok.lc.noblanks \
      --type $type \
      --joshua-mem 100g \
      --no-prepare \
      --first-step align \
      --last-step thrax \
      --hadoop $HADOOP \
      --threads 8 \
