---
layout: default6
category: advanced
title: Grammar Packing
---

Grammar packing refers to the process of taking a textual grammar
output by [Thrax](thrax.html) (or Moses, for phrase-based models) and
efficiently encoding it so that it can be loaded
[very quickly](https://aclweb.org/anthology/W/W12/W12-3134.pdf) ---
packing the grammar results in significantly faster load times for
very large grammars.  Packing is done automatically by the
[Joshua pipeline](pipeline.html), but you can also run the packer
manually.

The script can be found at
`$JOSHUA/scripts/support/grammar-packer.pl`. See that script for
example usage. You can then add it to a Joshua config file, simply
replacing a `tm` path to the compressed text-file format with a path
to the packed grammar directory (Joshua will automatically detect that
it is packed.

Packing the grammar requires first sorting it, which can take quite a
bit of temporary space.

*CAVEAT*: You may run into problems packing very large hiero
 grammars. Email the support list if you do.

### Examples

A Hiero grammar, using the compressed text file version:

    tm = hiero -owner pt -maxspan 20 -path grammar.filtered.gz
    
Pack it:

    $JOSHUA/scripts/support/grammar-packer.pl grammar.filtered.gz grammar.packed

Pack a really big grammar:

    $JOSHUA/scripts/support/grammar-packer.pl -m 30g grammar.filtered.gz grammar.packed

Be a little more verbose:

    $JOSHUA/scripts/support/grammar-packer.pl -m 30g grammar.filtered.gz grammar.packed

You have a different temp file location:

    $JOSHUA/scripts/support/grammar-packer.pl -T /local grammar.filtered.gz grammar.packed

Update the config file line:

    tm = hiero -owner pt -maxspan 20 -path grammar.packed
