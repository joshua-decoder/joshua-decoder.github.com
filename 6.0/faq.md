---
layout: default6
category: help
title: Frequently Asked Questions
---

Solutions to common problems will be posted here as we become aware of
them.  If you need help with something, please check
[our support group](https://groups.google.com/forum/#!forum/joshua_support)
for a solution, or
[post a new question](https://groups.google.com/forum/#!newtopic/joshua_support).

### You get a message stating: "no ken in java.library.path"

This occurs when [KenLM](https://kheafield.com/code/kenlm/) failed to
build. This can occur for a number of reasons:
   
- [Boost](http://www.boost.org/) isn't installed. Boost is
  available through most package management tools, so try that
  first. You can also build it from source.

- Boost is installed, but not in your path. The easiest solution is
  to add the boost library directory to your `$LD_LIBRARY_PATH`
  environment variable. You can also edit the file
  `$JOSHUA/src/joshua/decoder/ff/lm/kenlm/Makefile` and define
  `BOOST_ROOT` to point to your boost location. Then rebuild KenLM
  with the command
  
      ant -f $JOSHUA/build.xml kenlm

- You have run into boost's weird naming of multi-threaded
  libraries. For some reason, boost libraries sometimes have a
  `-mt` extension applied when they are built with multi-threaded
  support. This will cause the linker to fail, since it is looking
  for, e.g., `-lboost_system` instead of `-lboost_system-mt`. Edit
  the same Makefile as above and uncomment the `BOOST_MT = -mt`
  line, then try to compile again with
  
      ant -f $JOSHUA.build.xml kenlm

You may find the following reference URLs to be useful.

    https://groups.google.com/forum/#!topic/joshua_support/SiGO41tkpsw
    http://stackoverflow.com/questions/12583080/c-library-in-using-boost-library

