---
layout: default
title: Grammar Packing
---

One day, we hope that Juri will write up nice instructions.  Until then:

From a February 22 email
------------------------
Hey,

Quick note: the packing code is working and pushed to my packed branch.

To use it:

1. Make sure the grammar is labeled.  A labeled grammar is one that has feature names attached to
each of the feature values in each row of the grammar file.  Here is a line from a labeled grammar:

        [X] ||| [X,1] অন্যান্য [X,2] ||| [X,1] other [X,2] ||| 0 0 1 0 0 1.02184

   and here is one from an labeled grammar (note that the labels are not very useful):

        [X] ||| [X,1] অন্যান্য [X,2] ||| [X,1] other [X,2] ||| f1=0 f2=0 f3=1 f4=0 f5=0 f6=1.02184

   If your grammar is not labeled, you can use the script `$JOSHUA/scripts/label_grammar.py`:
   
        zcat grammar.gz | $JOSHUA/scripts/label_grammar.py > grammar-labeled.gz

   As a side-effect of this step is to produce a file 'dense_map' in the current directory,
   containing the mapping between feature names and feature columns.  This file is needed in later
   steps.

1. The packer needs a sorted grammar.  It is sufficient to sort by the first word:

        zcat grammar-labeled.gz | sort -k3,3 | gzip > grammar-sorted.gz
      
   (The reason we need a sorted grammar is because the packer stores the grammar in a trie.  The
   pieces can't be more than 2 GB due to Java limitations, so we need to ensure that rules are
   grouped by the first arc in the trie to avoid redundancy across tries and to simplify the
   lookup).
    
1. In order to pack the grammar, we need two pieces of information: (1) a packer configuration file,
and (2) a dense map file.

   1. Write a packer config file.  This file specifies items such as the chunk size (for the packed
   pieces) and the quantization classes and types for each feature name.  Examples can be found at
   
       $JOSHUA/test/packed/packer.config
       $JOSHUA/test/bn-en/packed/packer.quantized
       $JOSHUA/test/bn-en/packed/packer.uncompressed
       
   The quantizer lines in the packer config file have the following format:
   
       quantizer TYPE FEATURES
       
   where `TYPE` is one of `boolean`, `float`, `byte`, or `8bit`, and `FEATURES` is a space-delimited
   list of feature names that have that quantization type.
   
   1. Write a dense_map file.  If you labeled an unlabeled grammar, this was produced for you as a
   side product of the `label_grammar.py` script you called in Step 1.  Otherwise, you need to
   create a file that lists the mapping between feature names and (0-indexed) columns in the
   grammar, one per line, in the following format:
   
      feature-index feature-name
    
1. To pack the grammar, type the following command:

      java -cp $JOSHUA/bin joshua.tools.GrammarPacker -c PACKER_CONFIG_FILE -p OUTPUT_DIR -g GRAMMAR_FILE

   This will read in your packer configuration file and your grammar, and produced a packed grammar
   in the output directory.

1. To use the packed grammar, just point to the packed directory in your Joshua configuration file.

      tm-file = packed-grammar/
      tm-format = packed
