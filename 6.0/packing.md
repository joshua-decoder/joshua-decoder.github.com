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
it is packed, since a packed grammar is a directory).

Packing the grammar requires first sorting it by the rules source side,
which can take quite a bit of temporary space.

*CAVEAT*: You may run into problems packing very very large Hiero
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

### Using multiple packed grammars (Joshua 6.0.5)

Packed grammars serialize their vocabularies which prevented the use of multiple
packed grammars during decoding. With Joshua 6.0.5, it is possible to use multiple packed grammars during decoding if they have the same serialized vocabulary.
This is achieved by packing these grammars jointly using a revised packing CLI.

To pack multiple grammars:

    $JOSHUA/scripts/support/grammar-packer.pl grammar1.filtered.gz grammar2.filtered.gz [...] grammar1.packed grammar2.packed [...]

This will produce two packed grammars with the same vocabulary. To use them in the decoder, put this in your ```joshua.config```:

    tm = hiero -owner pt -maxspan 20 -path grammar1.packed
    tm = hiero -owner pt2 -maxspan 20 -path grammar2.packed

Note the different owners.
If you are trying to load multiple packed grammars that do not have the same
vocabulary, the decoder will throw a RuntimeException at loading time:

    Exception in thread "main" java.lang.RuntimeException: Trying to load multiple packed grammars with different vocabularies! Have you packed them jointly?
