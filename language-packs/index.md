---
layout: default6
title: Language packs
---

The simplest way to use Joshua is to use the provided "language packs", which are pre-built models
that enable translation in for particular language pairs. You can download and unpack each model and
then run the included script to translate new sentences.

It is important to note the assumptions underlying the translation engine:

- Joshua takes input on STDIN and outputs translations to STDOUT.

- Joshua expects its input to be one plain-text, UTF-8 encoded sentence per UNIX-delimited line. If
  you are translating documents, you must perform sentence segmentation yourself.
  
- Additionally, the input must be tokenized. To tokenize your data, you can use the script provided
  in each language pack.

## Available language packs

*Language packs are forthcoming*. 

Have a request? Please email [Matt Post](http://cs.jhu.edu/~post/).
