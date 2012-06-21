---
layout: default
category: advanced
title: Lattice decoding
---

In addition to regular sentences, Joshua can decode weighted lattices encoded in [the PLF
format](http://www.statmt.org/moses/?n=Moses.WordLattices).  Lattice decoding was originally added
by Lane Schwartz and [Chris Dyer](http://www.cs.cmu.edu/~cdyer/).

Joshua will automatically detect whether the input sentence is a regular sentence
(the usual case) or a lattice.  If a lattice, a feature will be activated that accumulates the cost
of different paths through the lattice.  In this case, you need to ensure that a weight for this
feature is present in [your model file](decoder.html).
