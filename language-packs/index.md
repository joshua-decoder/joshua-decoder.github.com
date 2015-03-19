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

- [Spanish--English phrase-based model](es-en-phrase/) [1.9 GB], built on Europarl and the Fisher
  and CALLHOME parallel dataset.

- [Arabic--English phrase-based model](ar-en-phrase/) [2.4 GB], built from
  [the LDC Arabic-Dialect/English parallel text](https://catalog.ldc.upenn.edu/LDC2012T09),
  [the ISI Arabic--English automatically extracted parallel text](https://catalog.ldc.upenn.edu/LDC2007T08), and
  [translations of the Arabic CALLHOME transcripts](https://catalog.ldc.upenn.edu/LDC97T19),
  and with an English Gigaword language model.

Have a request? Please email [Matt Post](http://cs.jhu.edu/~post/).
