---
layout: default
category: advanced
title: Building Translation Models
---

# Build a translation model

I recommend placing each week's model in a new directory, e.g., expts/scale12/model0.

Put the input files in a subdirectory "input".

Concatenate all the training files on each side.  The pipeline script normally does tokenization and normalization, but in this instance we have a custom tokenizer we need to apply to the source side, so we have to do it manually and then skip that step using "--first-step alignment".

* to tokenize the English data, do

      cat fisher.en | $JOSHUA/scripts/training/normalize-punctuation.pl en | $JOSHUA/scripts/training/penn-treebank-tokenizer.perl | $JOSHUA/scripts/lowercase.perl > fisher.norm.tok.lc.en

* to tokenize the Spanish data, you have to write an intermediate file since the Java tokenizer doesn't read from STDIN.

      export JOSHUA=/home/hltcoe/mpost/code/joshua
      cat fisher.es | $JOSHUA/scripts/training/normalize-punctuation.pl es \
      | $JOSHUA/scripts/support/twitter-tokenizer | $JOSHUA/scripts/lowercase.perl > fisher.norm.tok.lc.es
      export JOSHUA=/home/hltcoe/lorland/workspace/joshua

      paste fisher.norm.tok.lc.es fisher.norm.tok.lc.en | grep -Pv "^\t|\t$" \
      | ./splittabs.pl fisher.norm.tok.lc.noblanks.es fisher.norm.tok.lc.noblanks.en

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

* Now you can run the pipeline.  You need the following args:


      set -u
      pair=es-en
      type=hiero
      #. ~/.bashrc
      #basedir=$(pwd)
      dir=grammar-$pair-$type
      [[ ! -d $dir ]] && mkdir -p $dir
      cd $dir                                                                                                                                                                                                                                                                                                                                                                           source=$(echo $pair | cut -d- -f 1)                                                                                                                                                      
      target=$(echo $pair | cut -d- -f 2)
      $JOSHUA/scripts/training/pipeline.pl \
        --source $source \
        --target $target \
        --corpus /home/hltcoe/lorland/expts/scale12/model1/input/fisher.norm.tok.lc.noblanks \
        --type $type \
        --no-prepare \
        --first-step align \
        --last-step thrax \
        --hadoop $HADOOP
