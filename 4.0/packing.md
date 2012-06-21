---
layout: default
title: Grammar Packing
---

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
packer.uncompressed for examples. You also need to create a file called dense_map which maps the
feature names to Joshua's internal dense feature representation (i.e. indices). After you have
packed a grammar, that file needs to be copied into the packed grammar directory for the decoder to
work with it. I'm sorry this is so hacky, we're working on cleaning that up a bit. To make things a
little bit easier, the label_grammar.py script I mentioned above also generates a (trivial)
dense_map file.

I just added a little usage info to joshua.tools.GrammarPacker, the packing tool. Run it with "-h"
to see it. Once you have a packed grammar (with the dense_map file added, you just substitute the
packed directory's path for the tm-file option in your joshua.config and set tm-format to
packed. Should work fine then.

Let me know how it does for you, I think you'd be the first person to run packing experiments,
excluding myself. I'm actively working on the packed grammar code, so I'm really interested in what
you find (ideas, complaints, stuff). Right now the grammar is packed mostly aiming towards loading
speed and flexibility, not so much size reduction. I'm still looking to do a bunch of experiments
with varying quantizations and have a few things in mind for better performance/grammar size, so
stay tuned (or let me know if you'd be interested in cooperating/competing/fighting to the
death). Are you working on something similar to this?
