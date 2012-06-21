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
    
1. Write a packer config file.  Examples can be found in `example/packed-grammar` and
`test/bn-en/hiero`

1. Write a dense_map file and put it into the packed grammar directory. The file maps feature names
   to Joshua's dense phrasal feature indices. Format is 
   
    feature-index feature-name
    
   one per line. Again, examples are in the repository.

1. In your joshua.config, change `tm_file` to the packed directory name and the format to packed.

1. Enjoy faster loading and slower decoding. Woohoo?

The script `scripts/label_grammar.py` will take care of steps (1) and (4) for you. It'll just name
the features with their numeric column indices. You still need to make sure the dense_map file is in
the packed grammar directory.

I didn't run this on anything sizable yet, but it seems to work fine on two small examples, same
scores as standard grammars and all that.  A modified test on Bn-En Hiero that uses an unquantized
packed grammar passes (same BLEU score statistics on the output). I'll test more and put in some
more speed improvements soon.

From a May 18 email
-------------------
The packing process isn't exactly streamlined as of now, I apologize for that, so here are some
instructions (also, look at $JOSHUA/examples/packed-grammar for reference):

To pack a grammar you need a labeled, sorted grammar in Joshua/Hiero format. The sorting needs to be
by first source side word. That boils down to by source side, but only the first word matters. This
is so that we can chunk the grammar into less-than-2GB slices (needed because of Java-side
limitations) and guarantee that this slicing doesn't tear apart the source trie. The labeling refers
to the rule features, see hiero.tm.labeled.gz for an example. The $JOSHUA/scripts/label_grammar.py
will do the labeling for you (crudely, it'll just number the features assuming a dense setup).

You then need to specify a packer config. In it, you can request a slice size (in number of rules)
and specify the types/quantizations for the features by feature name. Check out packer.quantized and
packer.uncompressed for examples. You also need to create a file called `dense_map` which maps the
feature names to Joshua's internal dense feature representation (i.e., indices). After you have
packed a grammar, that file needs to be copied into the packed grammar directory for the decoder to
work with it. I'm sorry this is so hacky, we're working on cleaning that up a bit. To make things a
little bit easier, the `label_grammar.py` script I mentioned above also generates a (trivial)
`dense_map` file.

I just added a little usage info to `joshua.tools.GrammarPacker`, the packing tool. Run it with `-h`
to see it. Once you have a packed grammar (with the dense_map file added, you just substitute the
packed directory's path for the `tm-file` option in your `joshua.config` and set `tm-format` to
`packed`. Should work fine then.
