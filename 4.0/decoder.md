---
layout: default
category: links
title: The Joshua decoder
---
## Joshua command-line options and arguments

<table border="0">
  <tr>
    <th>
      option
    </th>
    <th>
      value
    </th>
    <th>
      description
    </th>
  </tr>

  <tr>
    <td>
      <code>--lm</code>
    </td>
    <td>
      String, e.g. <n /> <code>TYPE 5 false false 100 FILE</code>
    </td>
    <td markdown="1">
      Use once for each of one or language models.
    </td>
  </tr>

  <tr>
    <td>
      <code>--lm_file</code>
    </td>
    <td>
      String: path the the language model file
    </td>
    <td>
      ???
    </td>
  </tr>

  <tr>
    <td>
      <code>--parse</code>
    </td>
    <td>
      None
    </td>
    <td>
      whether to parse (if not then decode)
    </td>
  </tr>

  <tr>
    <td>
      <code>--tm_file</code>
    </td>
    <td>
      String
    </td>
    <td>
       path to the the translation model
    </td>
  </tr>

  <tr>
    <td>
      <code>--glue_file</code>
    </td>
    <td>
      String
    </td>
    <td>
      ???
    </td>
  </tr>

  <tr>
    <td>
      <code>--tm_format</code>
    </td>
    <td>
      String
    </td>
    <td>
      description
    </td>
  </tr>

  <tr>
    <td>
      <code>--glue_format</code>
    </td>
    <td>
      String
    </td>
    <td>
      description
    </td>
  </tr>

  <tr>
    <td>
      <code>--lm_type</code>
    </td>
    <td>
      value
    </td>
    <td>
      description
    </td>
  </tr>

</table>

### `--lm`

multiple language models. The value should be a string named <code>lm.gz</code>
(I seem to have had a failure when it was named something else and/or was an
absolute path. Specify multiple attributes of the language model(s) at once.
*test*

where the six fields correspond to the following values:
* LM type: one of "kenlm", "berkeleylm", "javalm" (not recommended), or "none"
* LM order: the N of the N-gram language model
* whether to use left equivalent state (currently not supported)
* whether to use right equivalent state (currently not supported
* the ceiling cost of any n-gram (currently ignored)
* LM file: the location of the language model file


### `--lm_file`
The differences between `lm` and `lm_file` are ... TBD.

### `--parse`
whether to parse (if not then decode)

### `--tm_file`
A String argument specifying the file name (Path?) of the translation model.

### `--glue_file`

### `--tm_format`

### `--glue_format`

### `--lm_type`
lm_type = String.valueOf(fds[1]);
if (!lm_type.equals("kenlm") && !lm_type.equals("berkeleylm")
    && !lm_type.equals("none") && !lm_type.equals("javalm")) {
  System.err.println("* FATAL: lm_type '" + lm_type + "' not supported");
  System.err
      .println("* supported types are 'kenlm' (default), 'berkeleylm', and 'javalm' (not recommended), and 'none'");
  System.exit(1);
}

`lm_ceiling_cost`
lm_ceiling_cost = Double.parseDouble(fds[1]);
`lm_ceiling_cost: %s`

`use_left_equivalent_state`
use_left_equivalent_state = Boolean.valueOf(fds[1]);
logger
`use_left_equivalent_state: %s`

`use_right_equivalent_state`
use_right_equivalent_state = Boolean.valueOf(fds[1]);
`use_right_equivalent_state: %s`
    use_right_equivalent_state));

`order`
lm_order = Integer.parseInt(fds[1]);
`g_lm_order: %s`

`use_sent_specific_lm`
use_sent_specific_lm = Boolean.valueOf(fds[1]);
`use_sent_specific_lm: %s`

`use_sent_specific_tm`
use_sent_specific_tm = Boolean.valueOf(fds[1]);
`use_sent_specific_tm: %s`

`span_limit`
span_limit = Integer.parseInt(fds[1]);
`span_limit: %s`

`phrase_owner`
phrase_owner = fds[1].trim();
`phrase_owner: %s`

`glue_owner`
glue_owner = fds[1].trim();
`glue_owner: %s`

`default_non_terminal`
default_non_terminal = "[" + fds[1].trim() + "]";
// default_non_terminal = fds[1].trim();
`default_non_terminal: %s`

`goalSymbol`
goal_symbol = "[" + fds[1].trim() + "]";
// goal_symbol = fds[1].trim();
`goalSymbol: `

`constrain_parse`
constrain_parse = Boolean.parseBoolean(fds[1]);

`oov_feature_index`
oov_feature_index = Integer.parseInt(fds[1]);

`true_oovs_only`
true_oovs_only = Boolean.parseBoolean(fds[1]);

`use_pos_labels`
use_pos_labels = Boolean.parseBoolean(fds[1]);

`fuzz1`
fuzz1 = Double.parseDouble(fds[1]);
`fuzz1: %s`

`fuzz2`
fuzz2 = Double.parseDouble(fds[1]);
`fuzz2: %s`

`max_n_items`
max_n_items = Integer.parseInt(fds[1]);
`max_n_items: %s`

`relative_threshold`
relative_threshold = Double.parseDouble(fds[1]);
`relative_threshold: %s`

`max_n_rules`
max_n_rules = Integer.parseInt(fds[1]);
`max_n_rules: %s`

`use_unique_nbest`
use_unique_nbest = Boolean.valueOf(fds[1]);
`use_unique_nbest: %s`

`add_combined_cost`
add_combined_cost = Boolean.valueOf(fds[1]);
`add_combined_cost: %s`

`use_tree_nbest`
use_tree_nbest = Boolean.valueOf(fds[1]);
`use_tree_nbest: %s`

`escape_trees`
escape_trees = Boolean.valueOf(fds[1]);
`escape_trees: %s`

`include_align_index`
include_align_index = Boolean.valueOf(fds[1]);
`include_align_index: %s`

`top_n`
topN = Integer.parseInt(fds[1]);
`topN: %s`

`parallel_files_prefix`
Random random = new Random();
int v = random.nextInt(10000000);// make it random
parallel_files_prefix = fds[1] + v;
`parallel_files_prefix: %s`

`num_parallel_decoders`
`threads`
num_parallel_decoders = Integer.parseInt(fds[1]);
if (num_parallel_decoders <= 0) {
  throw new IllegalArgumentException(
      "Must specify a positive number for num_parallel_decoders");
}
`num_parallel_decoders: %s`

`save_disk_hg`
save_disk_hg = Boolean.valueOf(fds[1]);
`save_disk_hg: %s`

`use_kbest_hg`
use_kbest_hg = Boolean.valueOf(fds[1]);
`use_kbest_hg: %s`

`forest_pruning`
forest_pruning = Boolean.valueOf(fds[1]);
`forest_pruning: %s`

`forest_pruning_threshold`
forest_pruning_threshold = Double.parseDouble(fds[1]);
`forest_pruning_threshold: %s`

`visualize_hypergraph`
visualize_hypergraph = Boolean.valueOf(fds[1]);
`visualize_hypergraph: %s`

`mark_oovs`
mark_oovs = Boolean.valueOf(fds[1]);
`mark_oovs: %s`

`pop-limit`
pop_limit = Integer.valueOf(fds[1]);
`pop-limit: %s`

`useCubePrune`
useCubePrune = Boolean.valueOf(fds[1]);
if (useCubePrune == false) logger.warning("useCubePrune=false");
logger.finest(String.format("useCubePrune: %s", useCubePrune));
else if (parameter.equals(normalize_key("useBeamAndThresholdPrune"))) {
useBeamAndThresholdPrune = Boolean.valueOf(fds[1]);
if (useBeamAndThresholdPrune == false)
  logger.warning("useBeamAndThresholdPrune=false");
logger.finest(String.format("useBeamAndThresholdPrune: %s", useBeamAndThresholdPrune));

else if (parameter.equals(normalize_key("oovFeatureCost"))) {
oov_feature_cost = Float.parseFloat(fds[1]);
logger.finest(String.format("oovFeatureCost: %s", oov_feature_cost));

else if (parameter.equals(normalize_key("useGoogleLinearCorpusGain"))) {
useGoogleLinearCorpusGain = new Boolean(fds[1].trim());
logger
    .finest(String.format("useGoogleLinearCorpusGain: %s", useGoogleLinearCorpusGain));

else if (parameter.equals(normalize_key("googleBLEUWeights"))) {
String[] googleWeights = fds[1].trim().split(";");
if (googleWeights.length != 5) {
  logger.severe("wrong line=" + line);
  System.exit(1);
}
linearCorpusGainThetas = new double[5];
for (int i = 0; i < 5; i++)
  linearCorpusGainThetas[i] = new Double(googleWeights[i]);

logger.finest(String.format("googleBLEUWeights: %s", linearCorpusGainThetas));

else if (parameter.equals(normalize_key("oracleFile"))) {
oracleFile = fds[1].trim();
logger.info(String.format("oracle file: %s", oracleFile));
if (!new File(oracleFile).exists()) {
  logger.warning("FATAL: can't find oracle file '" + oracleFile + "'");
  System.exit(1);
}

else if (parameter.equals("c") || parameter.equals("config")) {
// this was used to send in the config file, just ignore it
;

else {
logger.warning("FATAL: unknown configuration parameter '" + fds[0] + "'");
System.exit(1);

# Build a translation model

I recommend placing each week's model in a new directory, e.g., expts/scale12/model0.

Put the input files in a subdirectory "input".

Concatenate all the training files on each side.  The pipeline script normally does tokenization and normalization, but in this instance we have a custom tokenizer we need to apply to the source side, so we have to do it manually and then skip that step using "--first-step alignment".

* to tokenize the English data, do

      cat fisher.en | $JOSHUA/scripts/training/normalize-punctuation.pl en | $JOSHUA/scripts/training/penn-treebank-tokenizer.perl | $JOSHUA/scripts/lowercase.perl > fisher.norm.tok.lc.en

* to tokenize the Spanish data, you have to write an intermediate file since the Java tokenizer doesn't read from STDIN.

      export JOSHUA=/home/hltcoe/mpost/code/joshua
      cat fisher.es | $JOSHUA/scripts/training/normalize-punctuation.pl es \
      | $JOSHUA/scripts/support/twitter-tokenizer | $JOSHUA/scripts/lowercase.perl > fisher.norm.tok.lc.es
      export JOSHUA=/home/hltcoe/lorland/workspace/joshua

      paste fisher.norm.tok.lc.es fisher.norm.tok.lc.en | grep -Pv "^\t|\t$" \
      | ./splittabs.pl fisher.norm.tok.lc.noblanks.es fisher.norm.tok.lc.noblanks.en

  contents of `splittabls.pl` by Matt Post:

      #!/usr/bin/perl

      # splits on tab, printing respective chunks to the list of files given
      # as script arguments

      use FileHandle;

      my @fh;
      $| = 1;   # don't buffer output

      if (@ARGV < 0) {
        print "Usage: splittabs.pl < tabbed-file\n";
        exit;
      }

      my @fh = map { get_filehandle($_) } @ARGV;
      @ARGV = ();

      while (my $line = <>) {
        chomp($line);
        my (@fields) = split(/\t/,$line,scalar @fh);

        map { print {$fh[$_]} "$fields[$_]\n" } (0..$#fields);
      }

      sub get_filehandle {
          my $file = shift;

          if ($file eq "-") {
              return *STDOUT;
          } else {
              local *FH;
              open FH, ">$file" or die "can't open '$file' for writing";
              return *FH;
          }
      }

* Now you can run the pipeline.  You need the following args:


      set -u
      pair=es-en
      type=hiero
      #. ~/.bashrc
      #basedir=$(pwd)
      dir=grammar-$pair-$type
      [[ ! -d $dir ]] && mkdir -p $dir
      cd $dir                                                                                                                                                                                                                                                                                                                                                                           source=$(echo $pair | cut -d- -f 1)                                                                                                                                                      
      target=$(echo $pair | cut -d- -f 2)
      $JOSHUA/scripts/training/pipeline.pl \
        --source $source \
        --target $target \
        --corpus /home/hltcoe/lorland/expts/scale12/model1/input/fisher.norm.tok.lc.noblanks \
        --type $type \
        --no-prepare \
        --first-step align \
        --last-step thrax \
        --hadoop $HADOOP
