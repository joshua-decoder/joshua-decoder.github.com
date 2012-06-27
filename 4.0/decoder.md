---
layout: default
category: links
title: Decoder configuration parameters
---
Joshua configuration parameters affect the runtime behavior of the decoder itself.  This page
describes the complete list of these parameters and describes how to invoke the decoder manually.

To run the decoder, a convenience script is provided that loads the necessary Java libraries.
Assuming you have set the environment variable `$JOSHUA` to point to the root of your installation,
its syntax is:

    $JOSHUA/joshua-decoder [-m memory-amount] [-c config-file other-joshua-options ...]
    
The `-m` argument, if present, must come first, and the memory specification is in Java format
(e.g., 400m, 4g, 50g).  Most notably, the suffixes "m" and "g" are used for "megabytes" and
"gigabytes", and there cannot be a space between the number and the unit.  The value of this
argument is passed to Java itself in the invocation of the decoder, and the remaining options are
passed to Joshua.  The `-c` parameter has special import because it specifies the location of the
configuration file.

The Joshua decoder works by reading from STDIN and printing translations to STDOUT as they are
received, according to a number of [output options](#output).  If no run-time parameters are
specified (e.g., no translation model), sentences are simply pushed through untranslated.  Blank
lines are similarly pushed through as blank lines, so as to maintain parallelism with the input.

Parameters can be provided to Joshua via a configuration file and from the command
line.  Command-line arguments override values found in the configuration file.  The format for
configuration file paramters is

    parameter = value
    
Command-line options are specified in the following format

    -parameter value    

Values are one of four types (which we list here mostly to call attention to the boolean format):

- STRING, an arbitrary string (no spaces)
- FLOAT, a floating-point value
- INT, an integer
- BOOLEAN, a boolean value.  For booleans, `true` evaluates to true, and all other values evaluate
  to false.  For command-line options, the value may be omitted, in which case it evaluates to
  true.  For example, the following are equivalent:
  
      $JOSHUA/joshua-decoder -show-align-index true
      $JOSHUA/joshua-decoder -show-align-index

## Joshua configuration file

Before describing the list of Joshua parameters, we present a note about the configuration file.
In addition to the decoder parameters described below, the configuration file contains the feature
weight values for the model.  The weight values are distinguished from runtime parameters in two
ways: (1) they cannot be overridden on the command line, and (2) they do not have an equals sign
(=).  Parameters are described in further detail in the [feature file](features.html).  They take
the following format, and by convention are placed at the end of the configuration file:

    lm 0 4.23
    phrasement pt 0 -0.2
    oovpenalty -100

## Joshua decoder parameters

This section contains a list of the Joshua run-time parameters.  An important note about the
parameters is that they are collapsed to canonical form, in which dashes (-) and underscores (-) are
removed and case is converted to lowercase.  For example, the following parameter forms are
equivalent (either in the configuration file or from the command line):

    {top-n, topN, top_n, TOP_N, t-o-p-N}
    {poplimit, pop-limit, pop-limit, popLimit}

This basically defines equivalence classes of parameters, and relieves you of the task of having to
remember the exact format of each parameter.

In what follows, we group the configuration parameters in the following groups:

- [Alternate modes of operation](#modes)
- [General options](#general)
- [Pruning](#pruning)
- [Translation model options](#tm)
- [Language model options](#lm)
- [Output options](#output)

<a name="modes" />
### Alternate modes of operation

In addition to decoding (which is the default mode), Joshua can also produce synchronous parses of a
(source,target) pair of sentences.  This mode disables the language model (since no generation is
required) but still requires a translation model.  To enable it, you must do two things:

1. Set the configuration parameters `parse = true`.
2. Provide input in the following format:
   
       source sentence ||| target sentence
       
You may also wish to display the synchronouse parse tree (`-use-tree-nbest`) and the alignment
(`-show-align-index`).

The synchronous parsing implementation is that of [Dyer (2010)]().  

If parsing is enabled, the following features become relevant:

- `forest-pruning` --- *false*

  If true, the synchronous forest will be pruned.

- `forest-pruning-threshold` --- *10*

  The threshold used for pruning.

<a name="general" />
### General decoder options

- `c`, `config` --- *NULL*

   Specifies the configuration file from which Joshua options are loaded.  This feature is unique in
   that it must be specified from the command line.

- `oracle-file`


- `default-nonterminal` --- *"X"*

   This is the nonterminal symbol assigned to out-of-vocabulary (OOV) items.  

- `goal-symbol` --- *"GOAL"*

   This is the symbol whose presence in the chart over the whole input span denotes a successful
   parse (translation).  It should match the LHS nonterminal in your glue grammar.  Internally,
   Joshua represents nonterminals enclosed in square brackets (e.g., "[GOAL]"), which you can
   optionally supply in the configuration file.

- `true-oovs-only` --- *false*

  By default, Joshua creates an OOV entry for every word in the source sentence, regardless of
  whether it is found in the grammar.  This allows every word to be pushed through untranslated
  (although potentially incurring a high cost based on the `oovPenalty` feature).  If this option is
  set, then only true OOVs are entered into the chart as OOVs.

- `use-sent-specific-tm` --- *false*

  If set to true, Joshua will look for sentence-specific filtered grammars.  The location is
  determined by taking the supplied translation model (`tm-file`) and looking for a `filtered/`
  subdirectory for a file with the same name but with the (0-indexed) sentence number appended to
  it.  For example, if 
  
      tm-file = /path/to/grammar.gz
  
  then the sentence-filtered grammars should be found at
  
      /path/to/filtered/grammar.0.gz
      /path/to/filtered/grammar.1.gz
      /path/to/filtered/grammar.2.gz      
      ...
      
- `threads`, `num-parallel-decoders` --- *1*

  This determines how many simultaneous decoding threads to launch.  
  
  Outputs are assembled in order and Joshua has to hold on to the complete target hypergraph until
  it is ready to be processed for output, so too many simultaneous threads could result in lots of
  memory usage if a long sentence results in many sentences being queued up.  We have run Joshua
  with as many as 48 threads without any problems of this kind, but it's useful to keep in the back
  of your mind.

- `oov-feature-cost` --- *100*

  Each OOV word incurs this cost, which is multiplied against the `oovPenalty` feature (which is
  tuned but can be held fixed).

- `use-google-linear-corpus-gain`
- `google-bleu-weights`


<a name="pruning" />
### Pruning options
  
There are three different approaches to pruning in Joshua.  

1. No pruning.  Exhaustive decoding is triggered by setting `pop-limit = 0` and
`use-beam-and-threshold-prune = false`.

1. The old approach.  This approach uses a handful of pruning parameters whose specific roles are
hard to understand and whose interaction is even more difficult to quantify.  It is triggered by
setting `pop-limit = 0` and `use-beam-and-threshold-prune = true`.

1. Pop-limit pruning (the new approach).  The pop limit determines the number of hypotheses that are
  popped from the candidates list for each of the O(n^2) spans of the input.  A nice feature of this
  approach is that it provides a single value to control the size of the search space that is
  explored (and therefore runtime).

Pop-limit pruning is enabled by default, and it is the recommended approach.

- `pop-limit` --- *100*

  The number of hypotheses to examine for each span of the input.

- `use-beam-and-threshold-pruning` --- *false*

  Enables the use of beam-and-threshold pruning, and makes the following five features relevant.
  
  - `fuzz1` --- *0.1*
  - `fuzz2` --- *0.2*
  - `max-n-items` --- *30*
  - `relative-threshold` --- *10.0*
  - `max-n-rules` --- *50*

- `constrain-parse` --- *false*
- `use_pos_labels` --- *false*


<a name="tm" />
### Translation model options

At the moment, Joshua supports only two translation models, which are designated as the (main)
translation model and the glue grammar.  Internally, these grammars are distinguished only in that
the `span-limit` parameter applies only to the glue grammar.  In the near future we plan to
generalize the grammar specification to permit an unlimited number of translation models.

The main translation grammar is specified with the following set of parameters:

- `tm_file STRING` --- *NULL*, `glue_file STRING` --- *NULL*

  This points to the file location of the translation grammar for text-based formats or to the
  directory for the [packed representation](packed.html).
  
- `tm_format STRING` --- *thrax*, `glue_format STRING` --- *thrax*

  The format the file is in.  The permissible formats are `hiero` or `thrax` (which are equivalent),
  `packed` (for [packed grammars](packed.html)), or `samt` (for grammars encoded in the format
  defined by [Venugopal]().  This parameter will be done away with in the near future since it is
  easily inferrable.  See [the formats page](file-formats.html) for more information about file
  formats.

- `phrase_owner STRING` --- *pt*, `glue-owner STRING` --- *pt*

  The ownership concept is used to distinguish the set of feature weights that apply to each
  grammar.  See the [page on features](features.html) for more information.  By default, these
  parameters have the same value, meaning the grammars share a set of features.

- `span-limit` --- *10*

  This controls the maximum span of the input that grammar rules loaded from `tm-file` are allowed
  to apply.  The span limit is ignored for glue grammars.

<a name="lm" />
### Language model options

Joshua supports the incorporation of an arbitrary number of language models.  To add a language
model, add a line of the following format to the configuration file:

    lm = lm-type lm-order 0 0 lm-ceiling-cost lm-file

where the six fields correspond to the following values:

* *TYPE*: one of "kenlm", "berkeleylm", "javalm" (not recommended), or "none"
* *ORDER*: the N of the N-gram language model
* *0*: whether to use left equivalent state (currently not supported)
* *0*: whether to use right equivalent state (currently not supported)
* *0*: the LM-specific ceiling cost of any n-gram (currently ignored; `lm-ceiling-cost` applies to
   all language models)
* *FILE*: the path to the language model file.  All types support the standard ARPA format.
   Additionally, if the LM type is "kenlm", this file can be compiled into KenLM's compiled format
   (using the program at `$JOSHUA/src/joshua/decoder/ff/lm/kenlm/build_binary`), and if the LM type
   is "berkeleylm", it can be compiled by following the directions in
   `$JOSHUA/src/joshua/decoder/ff/lm/berkeley_lm/README`.

For each language model, you need to specify a feature weight in the following format:

    lm 0 WEIGHT
    lm 1 WEIGHT
    ...
    
where the indices correspond to the language model declaration lines in order.

For backwards compatibility, Joshua also supports a separate means of specifying the language model,
by separately specifying each of `lm-file` (NULL), `lm-type` (kenlm), `order` (5), and
`lm-ceiling-cost` (100).


<a name="output" />
### Output options

- top-n
- use-unique-nbest
- add-combined-cost
- use-tree-nbest
- escape-trees
- include-align-index
- save-disk-hg
- use-kbest-hg
- visualize-hypergraph
- mark-oovs


