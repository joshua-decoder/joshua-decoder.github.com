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

## Runtime decoder

The default release version of Joshua includes hundreds of megabytes of dependencies used
for building models from your own language pairs. If you only wish to run language packs
(effectively using Joshua as a black-box translation engine), you can install a "light" version
of Joshua that includes only dependencies needed to run the translation piece. See the notes
on [the installation page](../6.0/install.html).

## Available language packs

<table class="table table-condensed">
  <tr>
    <th>Language pair</th>
    <th>Type</th>
    <th>Size</th>
    <th>Details</th>
    <th>Link</th>
  </tr>
  <tr>
    <td>Spanish--English</td>
    <td>phrase</td>
    <td>1.9 GB</td>
    <td>Built on <a href="http://www.statmt.org/europarl/">Europarl</a> and the <a href="https://catalog.ldc.upenn.edu/LDC2014T23">Fisher and CALLHOME parallel dataset</a>.</td>
    <td><a href="es-en-phrase/">Download</a></td>
  </tr>
  <tr>
    <td>Arabic--English</td>
    <td>phrase</td>
    <td>2.1 GB</td>
    <td>Built from
  <a href="https://catalog.ldc.upenn.edu/LDC2012T09">the LDC Arabic-Dialect/English parallel text</a>,
  <a href="https://catalog.ldc.upenn.edu/LDC2007T08">the ISI Arabic--English automatically extracted parallel text</a>, 
  and <a href="https://catalog.ldc.upenn.edu/LDC97T19">translations of the Arabic CALLHOME transcripts</a>, and with an English Gigaword language model.
    </td>
    <td><a href="ar-en-phrase/">Download</a></td>
  </tr>
  <tr>
    <td>Chinese--English</td>
    <td>hiero</td>
    <td>2.4 GB</td>
    <td>Built from about 2 million sentences of parallel news. Contains the Joshua runtime,
      so there are no external dependencies (including Joshua).
    </td>
    <td><a href="zh-en-hiero/">Download</a></td>
  </tr>
</table>

Have a request? Please email [Matt Post](http://cs.jhu.edu/~post/).
