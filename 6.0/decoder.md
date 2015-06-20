---
layout: default6
category: links
title: Decoder configuration parameters
---

Joshua configuration parameters affect the runtime behavior of the decoder itself.  This page
describes the complete list of these parameters and describes how to invoke the decoder manually.

To run the decoder, a convenience script is provided that loads the necessary Java libraries.
Assuming you have set the environment variable `$JOSHUA` to point to the root of your installation,
its syntax is:

    $JOSHUA/bin/decoder [-m memory-amount] [-c config-file other-joshua-options ...]

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
configuration file parameters is

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

      $JOSHUA/bin/decoder -mark-oovs true
      $JOSHUA/bin/decoder -mark-oovs

## Joshua configuration file

In addition to the decoder parameters described below, the configuration file contains the model
feature weights.  These weights are distinguished from runtime parameters in that they are delimited
by a space instead of an equals sign. They take the following
format, and by convention are placed at the end of the configuration file:

    lm_0 4.23
    tm_pt_0 -0.2
    OOVPenalty -100
   
Joshua can make use of thousands of features, which are described in further detail in the
[feature file](features.html).

## Joshua decoder parameters

This section contains a list of the Joshua run-time parameters.  An important note about the
parameters is that they are collapsed to canonical form, in which dashes (-) and underscores (-) are
removed and case is converted to lowercase.  For example, the following parameter forms are
equivalent (either in the configuration file or from the command line):

    {top-n, topN, top_n, TOP_N, t-o-p-N}
    {poplimit, pop-limit, pop-limit, popLimit,PoPlImIt}

This basically defines equivalence classes of parameters, and relieves you of the task of having to
remember the exact format of each parameter.

In what follows, we group the configuration parameters in the following groups:

- [General options](#general)
- [Pruning](#pruning)
- [Translation model options](#tm)
- [Language model options](#lm)
- [Output options](#output)
- [Alternate modes of operation](#modes)

<a id="general" />

### General decoder options

- `c`, `config` --- *NULL*

   Specifies the configuration file from which Joshua options are loaded.  This feature is unique in
   that it must be specified from the command line (obviously).

- `amortize` --- *true*

  When true, specifies that sorting of the rule lists at each trie node in the grammar should be
  delayed until the trie node is accessed. When false, all such nodes are sorted before decoding
  even begins. Setting to true results in slower per-sentence decoding, but allows the decoder to
  begin translating almost immediately (especially with large grammars).

- `server-port` --- *0*

  If set to a nonzero value, Joshua will start a multithreaded TCP/IP server on the specified
  port. Clients can connect to it directly through programming APIs or command-line tools like
  `telnet` or `nc`.
  
      $ $JOSHUA/bin/decoder -m 30g -c /path/to/config/file -server-port 8723
      ...
      $ cat input.txt | nc localhost 8723 > results.txt

- `maxlen` --- *200*

  Input sentences longer than this are truncated.

- `feature-function`

  Enables a particular feature function. See the [feature function page](features.html) for more information.

- `oracle-file` --- *NULL*

  The location of a set of oracle reference translations, parallel to the input.  When present,
  after producing the hypergraph by decoding the input sentence, the oracle is used to rescore the
  translation forest with a BLEU approximation in order to extract the oracle-translation from the
  forest.  This is useful for obtaining an (approximation to an) upper bound on your translation
  model under particular search settings.

- `default-nonterminal` --- *"X"*

   This is the nonterminal symbol assigned to out-of-vocabulary (OOV) items. Joshua assigns this
   label to every word of the input, in fact, so that even known words can be translated as OOVs, if
   the model prefers them. Usually, a very low weight on the `OOVPenalty` feature discourages their
   use unless necessary.

- `goal-symbol` --- *"GOAL"*

   This is the symbol whose presence in the chart over the whole input span denotes a successful
   parse (translation).  It should match the LHS nonterminal in your glue grammar.  Internally,
   Joshua represents nonterminals enclosed in square brackets (e.g., "[GOAL]"), which you can
   optionally supply in the configuration file.

- `true-oovs-only` --- *false*

  By default, Joshua creates an OOV entry for every word in the source sentence, regardless of
  whether it is found in the grammar.  This allows every word to be pushed through untranslated
  (although potentially incurring a high cost based on the `OOVPenalty` feature).  If this option is
  set, then only true OOVs are entered into the chart as OOVs. To determine "true" OOVs, Joshua
  examines the first level of the grammar trie for each word of the input (this isn't a perfect
  heuristic, since a word could be present only in deeper levels of the trie).

- `threads`, `num-parallel-decoders` --- *1*

  This determines how many simultaneous decoding threads to launch.  

  Outputs are assembled in order and Joshua has to hold on to the complete target hypergraph until
  it is ready to be processed for output, so too many simultaneous threads could result in lots of
  memory usage if a long sentence results in many sentences being queued up.  We have run Joshua
  with as many as 64 threads without any problems of this kind, but it's useful to keep in the back
  of your mind.
  
- `weights-file` --- NULL

  Weights are appended to the end of the Joshua configuration file, by convention. If you prefer to
  put them in a separate file, you can do so, and point to the file with this parameter.

### Pruning options <a id="pruning" />

- `pop-limit` --- *100*

  The number of cube-pruning hypotheses that are popped from the candidates list for each span of
  the input.  Higher values result in a larger portion of the search space being explored at the
  cost of an increased search time. For exhaustive search, set `pop-limit` to 0.

- `filter-grammar` --- false

  Set to true, this enables dynamic sentence-level filtering. For each sentence, each grammar is
  filtered at runtime down to rules that can be applied to the sentence under consideration. This
  takes some time (which we haven't thoroughly quantified), but can result in the removal of many
  rules that are only partially applicable to the sentence.

- `constrain-parse` --- *false*
- `use_pos_labels` --- *false*

  *These features are not documented.*

### Translation model options <a id="tm" />

Joshua supports any number of translation models. Conventionally, two are supplied: the main grammar
containing translation rules, and the glue grammar for patching things together. Internally, Joshua
doesn't distinguish between the roles of these grammars; they are treated differently only in that
they typically have different span limits (the maximum input width they can be applied to).

Grammars are instantiated with config file lines of the following form:

    tm = TYPE OWNER SPAN_LIMIT FILE

* `TYPE` is the grammar type, which must be set to "thrax". 
* `OWNER` is the grammar's owner, which defines the set of [feature weights](features.html) that
  apply to the weights found in each line of the grammar (using different owners allows each grammar
  to have different sets and numbers of weights, while sharing owners allows weights to be shared
  across grammars).
* `SPAN_LIMIT` is the maximum span of the input that rules from this grammar can be applied to. A
  span limit of 0 means "no limit", while a span limit of -1 means that rules from this grammar must
  be anchored to the left side of the sentence (index 0).
* `FILE` is the path to the file containing the grammar. If the file is a directory, it is assumed
  to be [packed](packed.html). Only one packed grammar can currently be used at a time.

For reference, the following two translation model lines are used by the [pipeline](pipeline.html):

    tm = thrax pt 20 /path/to/packed/grammar
    tm = thrax glue -1 /path/to/glue/grammar

### Language model options <a id="lm" />

Joshua supports any number of language models.  To add a language
model, add a line of the following format to the configuration file:

    lm = TYPE ORDER LEFT_STATE RIGHT_STATE CEILING_COST FILE

where the six fields correspond to the following values:

* `TYPE`: one of "kenlm", "berkeleylm", or "none"
* `ORDER`: the order of the language model
* `LEFT_STATE`: whether to use left-state minimization; currently only supported by KenLM
* `RIGHT_STATE`: whether to use right equivalent state (currently unsupported)
* `CEILING_COST`: the LM-specific ceiling cost of all n-grams (currently ignored)
* `FILE`: the path to the language model file.  All language model types support the standard ARPA
   format.  Additionally, if the LM type is "kenlm", this file can be compiled into KenLM's compiled
   format (using the program at `$JOSHUA/bin/build_binary`); if the the LM type is "berkeleylm", it
   can be compiled by following the directions in
   `$JOSHUA/src/joshua/decoder/ff/lm/berkeley_lm/README`. The [pipeline](pipeline.html) will
   automatically compile either type.

For each language model, you need to specify a feature weight in the following format:

    lm_0 WEIGHT
    lm_1 WEIGHT
    ...

where the indices correspond to the order of the language model declaration lines.

### Output options <a id="output" />

- `output-format` *New in 5.0*

  Joshua prints a lot of information to STDERR (making this more granular is on the TODO
  list). Output to STDOUT is reserved for decoder translations, and is controlled by the

   - `%i`: the sentence number (0-indexed)

   - `%e`: the source sentence

   - `%s`: the translated sentence

   - `%S`: the translated sentence, with some basic capitalization and denomralization. e.g.,

         $ echo "¿ who you lookin' at , mr. ?" | $JOSHUA/bin/decoder -output-format "%S" -mark-oovs false 2> /dev/null 
         ¿Who you lookin' at, Mr.? 

   - `%t`: the target-side tree projection, all printed on one line (PTB style)
   
   - `%d`: the synchronous derivation, with each rules printed indented on their own lines

   - `%f`: the list of feature values (as name=value pairs)

   - `%c`: the model cost

   - `%w`: the weight vector (unimplemented)

   - `%a`: the alignments between source and target words (currently broken for hierarchical mode)

  The default value is

      output-format = %i ||| %s ||| %f ||| %c
      
  i.e.,

      input ID ||| translation ||| model scores ||| score

- `top-n` --- *300*

  The number of translation hypotheses to output, sorted in decreasing order of model score

- `use-unique-nbest` --- *true*

  When constructing the n-best list for a sentence, skip hypotheses whose string has already been
  output.

- `escape-trees` --- *false*

- `include-align-index` --- *false*

  Output the source words indices that each target word aligns to.

- `mark-oovs` --- *false*

  if `true`, this causes the text "_OOV" to be appended to each untranslated word in the output.

- `visualize-hypergraph` --- *false*

  If set to true, a visualization of the hypergraph will be displayed, though you will have to
  explicitly include the relevant jar files.  See the example usage in
  `$JOSHUA/examples/tree_visualizer/`, which contains a demonstration of a source sentence,
  translation, and synchronous derivation.

- `dump-hypergraph` --- ""

  This feature directs that the hypergraph should be written to disk for each input sentence. If
  set, the value should contain the string "%d", which is replaced with the sentence number. For
  example,
  
      cat input.txt | $JOSHUA/bin/decoder -dump-hypergraph hgs/%d.txt

  Note that the output directory must exist.

  TODO: revive the
  [discussion on a common hypergraph format](http://aclweb.org/aclwiki/index.php?title=Hypergraph_Format)
  on the ACL Wiki and support that format.

### Lattice decoding

In addition to regular sentences, Joshua can decode weighted lattices encoded in
[the PLF format](http://www.statmt.org/moses/?n=Moses.WordLattices), except that path costs should
be listed as <b>log probabilities</b> instead of probabilities.  Lattice decoding was originally
added by Lane Schwartz and [Chris Dyer](http://www.cs.cmu.edu/~cdyer/).

Joshua will automatically detect whether the input sentence is a regular sentence (the usual case)
or a lattice.  If a lattice, a feature will be activated that accumulates the cost of different
paths through the lattice.  In this case, you need to ensure that a weight for this feature is
present in [your model file](decoder.html). The [pipeline](pipeline.html) will handle this
automatically, or if you are doing this manually, you can add the line

    SourcePath COST
    
to your Joshua configuration file.    

Lattices must be listed one per line.

### Alternate modes of operation <a id="modes" />

In addition to decoding input sentences in the standard way, Joshua supports both *constrained
decoding* and *synchronous parsing*. In both settings, both the source and target sides are provided
as input, and the decoder finds a derivation between them.

#### Constrained decoding

To enable constrained decoding, simply append the desired target string as part of the input, in
the following format:

    source sentence ||| target sentence

Joshua will translate the source sentence constrained to the target sentence. There are a few
caveats:

   * Left-state minimization cannot be enabled for the language model

   * A heuristic is used to constrain the derivation (the LM state must match against the
     input). This is not a perfect heuristic, and sometimes results in analyses that are not
     perfectly constrained to the input, but have extra words.

#### Synchronous parsing

Joshua supports synchronous parsing as a two-step sequence of monolingual parses, as described in
Dyer (NAACL 2010) ([PDF](http://www.aclweb.org/anthology/N10-1033‎.pdf)). To enable this:

   - Set the configuration parameter `parse = true`.

   - Remove all language models from the input file 

   - Provide input in the following format:

          source sentence ||| target sentence

You may also wish to display the synchronouse parse tree (`-output-format %t`) and the alignment
(`-show-align-index`).

