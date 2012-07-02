---
layout: default
category: links
title: The Joshua Pipeline
---

This page describes the Joshua pipeline script, which manages the complexity of training and
evaluating machine translation systems.  The pipeline eases the pain of two related tasks in
statistical machine translation (SMT) research:

1. Training SMT systems involves a complicated process of interacting steps that are time-consuming
and prone to failure.

1. Developing and testing new techniques requires varying parameters at different points in the
pipeline.  Earlier results (which are often expensive) need not be recomputed.

To facilitate these tasks, the pipeline script:
- Runs the complete SMT pipeline, from corpus normalization and tokenization, through model
  building, tuning, test-set decoding, and evaluation.

- Caches the results of intermediate steps (using robust SHA-1 checksums on dependencies), so the
  pipeline can be debugged or shared across similar runs with (almost) no time spent recomputing
  expensive steps.
 
- Allows you to jump into and out of the pipeline at a set of predefined places (e.g., the alignment
  stage), so long as you provide the missing dependencies.

The Joshua pipeline script is designed in the spirit of Moses' `train-model.pl`, and shares many of
its features.  It is not as extensive, however, as Moses'
[Experiment Management System](http://www.statmt.org/moses/?n=FactoredTraining.EMS).

## Installation

The pipeline has no *required* external dependencies.  However, it has support for a number of
external packages, some of which are included with Joshua.

-  [GIZA++](http://code.google.com/p/giza-pp/)

   GIZA++ is the default aligner.  It is included with Joshua, and should compile successfully when
   you typed `ant all` from the Joshua root directory.  It is not required because you can use the
   (included) Berkeley aligner (`--aligner berkeley`).

-  [SRILM](http://www.speech.sri.com/projects/srilm/)

   By default, the pipeline uses a Java program from the
   [Berkeley LM](http://code.google.com/p/berkeleylm/) package that constructs an
   Kneser-Ney-smoothed language model in ARPA format from the target side of your training data.  If
   you wish to use SRILM instead, you need to do the following:
   
   1. Install SRILM and set the `$SRILM` environment variable to point to its installed location.
   1. Add the `--lm-gen srilm` flag to your pipeline invocation.
   
   More information on this is available in the [LM building section of the pipeline](#lm).  SRILM
   is not used for representing language models during decoding (and in fact is not supported,
   having been supplanted by [KenLM](http://kheafield.com/code/kenlm/) and BerkeleyLM).

-  [Hadoop](http://hadoop.apache.org/)

   The pipeline uses the [Thrax grammar extractor](thrax.html), which is built on Hadoop.  If you
   have a Hadoop installation, simply ensure that the `$HADOOP` environment variable is defined, and
   the pipeline will use it automatically at the grammar extraction step.  If you are going to
   attempt to extract very large grammars, it is best to have a good-sized Hadoop installation.
   
   (If you do not have a Hadoop installation, you might consider setting one up.  Hadoop can be
   installed in a
   ["pseudo-distributed"](http://hadoop.apache.org/common/docs/r0.20.2/quickstart.html#PseudoDistributed)
   mode that allows it to use just a few machines or a number of processors on a single machine.
   The main issue is to ensure that there are a lot of independent physical disks, since in our
   experience Hadoop starts to exhibit lots of hard-to-trace problems if there is too much demand on
   the disks.)
   
   If you don't have a Hadoop installation, there are still no worries.  The pipeline will unroll a
   standalone installation and use it to extract your grammar.  This behavior will be triggered if
   `$HADOOP` is undefined.

Make sure that the environment variable `$JOSHUA` is defined, and you should be all set.

## A basic pipeline run

The pipeline takes a set of inputs (training, tuning, and test data), and creates a set of
intermediate files in the *run directory*.  By default, the run directory is the current directory,
but it can be changed with the `--rundir` parameter.

For this quick start, we will be working with the example that can be found in
`$JOSHUA/examples/pipeline`.  This example contains 1,000 sentences of Urdu-English data (the full
dataset is available as part of the
[Indian languages parallel corpora](http://joshua-decoder.org/indian-parallel-corpora/) with
100-sentence tuning and test sets with four references each.

Running the pipeline requires two main steps: data preparation and invocation.

1. Prepare your data.  The pipeline script needs to be told where to find the raw training, tuning,
   and test data.  A good convention is to place these files in an input/ subdirectory of your run's
   working directory (NOTE: do not use `data/`, since a directory of that name is created and used
   by the pipeline itself).  The expected format (for each of training, tuning, and test) is a pair
   of files that share a common path prefix and are distinguished by their extension:

       input/
             train.SOURCE
             train.TARGET
             tune.SOURCE
             tune.TARGET
             test.SOURCE
             test.TARGET

   These files should be parallel at the sentence level (with one sentence per line), should be in
   UTF-8, and should be untokenized (tokenization occurs in the pipeline).  SOURCE and TARGET denote
   variables that should be replaced with the actual target and source language abbreviations (e.g.,
   "ur" and "en").
   
1. Run the pipeline.  The following is the minimal invocation to run the complete pipeline:

       $JOSHUA/scripts/training/pipeline.pl  \
         --corpus input/train                \
         --tune input/tune                   \
         --test input/devtest                \
         --source SOURCE                     \
         --target TARGET

   The `--corpus`, `--tune`, and `--test` flags define file prefixes that are concatened with the
   language extensions given by `--target` and `--source` (with a "." in betwee).  Note the
   correspondences with the files defined in the first step above.  The prefixes can be either
   absolute or relative pathnames.  This particular invocation assumes that a subdirectory `input/`
   exists in the current directory, that you are translating from a language identified "ur"
   extension to a language identified by the "en" extension, that the training data can be found at
   `input/train.en` and `input/train.ur`, and so on.
   
   Assuming no problems arise, this command will run the complete pipeline in about 20 minutes,
   producing BLEU scores at the end.  As it runs, you will see output that looks like the following:
   
       [train-copy-en] rebuilding...
         dep=/Users/post/code/joshua/test/pipeline/input/train.en 
         dep=data/train/train.en.gz [NOT FOUND]
         cmd=cat /Users/post/code/joshua/test/pipeline/input/train.en | gzip -9n > data/train/train.en.gz
         took 0 seconds (0s)
       [train-copy-ur] rebuilding...
         dep=/Users/post/code/joshua/test/pipeline/input/train.ur 
         dep=data/train/train.ur.gz [NOT FOUND]
         cmd=cat /Users/post/code/joshua/test/pipeline/input/train.ur | gzip -9n > data/train/train.ur.gz
         took 0 seconds (0s)
       ...
   
   And in the current directory, you will see the following files (among other intermediate files
   generated by the individual sub-steps).
   
       data/
           train/
               corpus.ur
               corpus.en
               thrax-input-file
           tune/
               tune.tok.lc.ur
               tune.tok.lc.en
               grammar.filtered.gz
               grammar.glue
           test/
               test.tok.lc.ur
               test.tok.lc.en
               grammar.filtered.gz
               grammar.glue
       alignments/
           0/
               [berkeley aligner output files]
           training.align
       thrax-hiero.conf
       thrax.log
       grammar.gz
       lm.gz
       tune/
           1/
               decoder_command
               joshua.config
               params.txt
               joshua.log
               mert.log
               joshua.config.ZMERT.final
           final-bleu

   These files will be described in more detail in subsequent sections of this tutorial.

   Another useful flag is the `--rundir DIR` flag, which chdir()s to the specified directory before
   running the pipeline.  By default the rundir is the current directory.  Changing it can be useful
   for organizing related pipeline runs.  Relative paths specified to other flags (e.g., to
   `--corpus` or `--lmfile`) are relative to the directory the pipeline was called *from*, not the
   rundir itself (unless they happen to be the same, of course).

   The complete pipeline comprises many tens of small steps, which can be grouped together into a
   set of traditional pipeline tasks:
   
       1. [Data preparation](#prep)
       1. [Alignment](#alignment)
       1. [Grammar extraction](#tm)
       1. [Language model building](#lm)
       1. [Tuning](#tuning)
       1. [Testing](#testing)

   These steps are discussed below, after a few intervening sections about high-level details of the
   pipeline.  

## Restarting failed runs

If the pipeline dies, you can restart it with the same command you used the first time.  If you
rerun the pipeline with the exact same invocation as the previous run (or an overlapping
configuration -- one that causes the same set of behaviors), you will see slightly different
output compared to what we saw above:

    [train-copy-en] cached, skipping...
    [train-copy-ur] cached, skipping...
    ...

This indicates that the caching module has discovered that the step was already computed and thus
did not need to be rerun.  This feature is quite useful for restarting pipeline runs that have
crashed due to bugs, memory limitations, hardware failures, and the myriad other problems that
plague MT researchers across the world.

Often, a command will die because it was parameterized incorrectly.  For example, perhaps the
decoder ran out of memory.  This allows you to adjust the parameter (e.g., `--joshua-mem`) and rerun
the script.  Of course, if you change one of the parameters a step depends on, it will trigger a
rerun, which in turn might trigger further downstream reruns.
   
## Skipping steps, quitting early

You will also find it useful to start the pipeline somewhere other than data preparation (for
example, if you have already-processed data and an alignment, and want to begin with building a
grammar) or to end it prematurely (if, say, you don't have a test set and just want to tune a
model).  This can be accomplished with the `--first-step` and `--last-step` flags, which take as
argument a case-insensitive version of the following steps:

- *FIRST*: Data preparation.  Everything begins with data preparation.  This is the default first
   step, so there is no need to be explicit about it.

- *ALIGN*: Alignment.  You might want to start here if you want to skip data preprocessing.

- *PARSE*: Parsing.  This is only relevant for building SAMT grammars (`--type samt`), in which case
   the target side (`--target`) of the training data (`--corpus`) is parsed before building a
   grammar.
   
- *THRAX*: Grammar extraction [with Thrax](thrax.html).  If you jump to this step, you'll need to
   provide an aligned corpus (`--alignment`) along with your parallel data.  
   
- *TUNE*: Tuning.  The exact tuning method is determined with `--tuner {mert,pro}`.  With this
   option, you need to specify a grammar (`--grammar`) or separate tune (`--tune-grammar`) and test
   (`--test-grammar`) grammars.  A full grammar (`--grammar`) will be filtered against the relevant
   tuning or test set unless you specify `--no-filter-tm`.  If you want a language model built from
   the target side of your training data, you'll also need to pass in the training corpus
   (`--corpus`).  You can also specify an arbitrary number of additional grammars with one or more
   `--lmfile` flags.

- *TEST*: Testing.  If you have a tuned model file, you can test new corpora by passing in a test
   corpus with references (`--test`).  You'll need to provide a run name (`--name`) to store the
   results of this run, which will be placed under `test/NAME`.  You'll also need to provide a
   Joshua configuration file (`--joshua-config`), one or more language models (`--lmfile`), and a
   grammar (`--grammar`); this will be filtered to the test data unless you specify
   `--no-filter-tm`) or unless you directly provide a filtered test grammar (`--test-grammar`).

- *LAST*: The last step.  This is the default target of `--last-step`.

We now discuss these steps in more detail.

## 1. Data preparation

Data prepare involves doing the following to each of the training data (`--corpus`), tuning data
(`--tune`), and testing data (`--test`).  Each of these values is an absolute or relative path
prefix.  To each of these prefixes, a "." is appended, followed by each of SOURCE (`--source`) and
TARGET (`--target`), which are file extensions identifying the languages.  The SOURCE and TARGET
files must have the same number of lines.  

For tuning and test data, multiple references are handled automatically.  A single reference will
have the format TUNE.TARGET, while multiple references will have the format TUNE.TARGET.NUM, where
NUM starts at 0 and increments for as many references as there are.

The following processing steps are applied to each file.

1.  **Copying** the files into `RUNDIR/data/TYPE`, where TYPE is one of "train", "tune", or "test".
    Multiple `--corpora` files are concatenated in the order they are specified.  Multiple `--tune`
    and `--test` flags are not currently allowed.
    
1.  **Normalizing** punctuation and text (e.g., removing extra spaces, converting special
    quotations).  There are a few language-specific options that depend on the file extension
    matching the [two-letter ISO 639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)
    designation.

1.  **Tokenizing** the data (e.g., separating out punctuation, converting brackets).  Again, there
    are language-specific tokenizations for a few languages (English, German, and Greek).

1.  (Training only) **Removing** all parallel sentences with more than `--maxlen` tokens on either
    side.  By default, MAXLEN is 50.  To turn this off, specify `--maxlen 0`.

1.  **Lowercasing**.

This creates a series of intermediate files which are saved for posterity but compressed.  For
example, you might see

    data/
        train/
            train.en.gz
            train.tok.en.gz
            train.tok.50.en.gz
            train.tok.50.lc.en
            corpus.en -> train.tok.50.lc.en

The file "corpus.LANG" is a symbolic link to the last file in the chain.  

## 2. ALIGNMENT

Alignments are between the parallel corpora at `RUNDIR/data/train/corpus.{SOURCE,TARGET}`.  To
prevent the alignment tables from getting too big, the parallel corpora are grouped into files of no
more than ALIGNER\_CHUNK\_SIZE blocks (controlled with a parameter below).  The last block is folded
into the penultimate block if it is too small.

-   `aligner ALIGNER` {giza (default), berkeley}

    Which aligner to use.  The default is [GIZA++](http://code.google.com/p/giza-pp/), but
    [the Berkeley aligner](http://code.google.com/p/berkeleyaligner/) can be used instead.  When
    using the Berkeley aligner, you'll want to pay attention to how much memory you allocate to it
    with `--aligner-mem` (the default is 10g).

-   `aligner-chunk-size SIZE` (1,000,000)

    The number of sentence pairs to compute alignments over.

## 3. PARSING


## TUNING

Here are
a number of arguments that define what is done:

-  `--lm` {kenlm (default), berkeleylm}

   This determines the language model code that will be used when decoding.  These implementations
   are described in their respective papers (PDFs:
   [KenLM](http://kheafield.com/professional/avenue/kenlm.pdf),
   [BerkeleyLM](http://nlp.cs.berkeley.edu/pubs/Pauls-Klein_2011_LM_paper.pdf)).
   
- `--lmfile FILE`

  Specifies a pre-built language model to use when decoding.  This language model can be in ARPA
  format, or in KenLM format when using KenLM or BerkeleyLM format when using that format.

- `--lm-gen` {berkeleylm (default), srilm}, `--buildlm-mem MEM`, `--witten-bell`

  At the tuning step, an LM is built from the target side of the training data (unless
  `--no-corpus-lm` is specified).  This controls which code is used to build it.  The default is a
  [BerkeleyLM java class](http://code.google.com/p/berkeleylm/source/browse/trunk/src/edu/berkeley/nlp/lm/io/MakeKneserNeyArpaFromText.java)
  that computes a Kneser-Ney LM with a constant discounting and no count thresholding.  The flag
  `--buildlm-mem` can be used to control how much memory is allocated to the Java process.  The
  default is "2g", but you will want to increase it for larger language models.
  
  If SRILM is used, it is called with the following arguments:
  
        $SRILM/bin/i686-m64/ngram-count -interpolate SMOOTHING -order 5 -text TRAINING-DATA -unk -lm lm.gz
        
  Where SMOOTHING is `-kndiscount`, or `-wbdiscount` if `--witten-bell` is passed to the pipeline.

## Corpus preparation

- `--corpus CORPUS1`
- `[--corpus CORPUS2]`
- `[...]`

  Specifies a file prefix for a training corpus.  If this flag is found multiples times, the corpora
  are all concatenated together for alignment and grammar extraction.  The use of multiple
  `--corpora` flags is only supported when running the script from the beginning; if you are
  skipping steps, only a single corpus can be provided (so you must do the concatenation yourself,
  if you so need).

- `--tune PREFIX`
- `--test PREFIX`
  
  Specify file prefixes for tuning and test data.  Unlike `--corpus`, only one instance each of
  `--tune` or `--test` is allowed (if you provide multiple ones, only the last value is used).

- `--type {hiero,samt}`

  Whether to learn a Hiero or SAMT grammar.  The default is Hiero.




- `--alignment FILE`

  Provide an alignment for the training data.  The format is the standard format where 0-indexed
  many-many alignment pairs for a sentence are provided on a line, source language first, e.g.,

      0-0 0-1 1-2 1-7 ...

  This value is required if you start at the grammar extraction step.

-  `--no-mbr`

   Skip Minimum Bayes-risk decoding of the n-best list of the output on the test data.  MBR is
   usually worth about 0.3 - 0.5 BLEU points, but takes a while.

-  `--lmfile FILE`

   Use the specified file as the language model for decoding (for both tuning and decoding of the
   test set).  If not provided, a Java program provided with the Berkeley LM is used to build a
   Kneser-Ney interpolated 5-gram language model from the target side of the training data (unless
   you specify `--lm-gen srilm`, in which case SRILM is used).
   
   You can specify as many `--lmfile` options as you wish.  A language model is built from the
   target side of the training data unless `--no-corpus-lm` is given.

-  `--filter-lm [default]`
-  `--no-filter-lm`

   Use Kenneth Heafield's "filter" program to filter the language model to the training data.  This
   is only available if a training corpus was provided.

- `--maxlen LEN [default=50]`

  Remove parallel sentences from the training data that are longer than this length (on either
  side).  This only applies to the training data, and is computed after tokenization.

- `--grammar FILE`

  Use the specified grammar instead of learning one with Thrax.

- `--joshua-config TEMPLATE`

  Use the specified file as the Joshua config file instead of the default template.  This file is a
  template into which are substituted run-specific information; see the TEMPLATE section below for
  more information.

- `--decoder-command TEMPLATE`

  Use the specified file as the decoding command.  This file is a template into which are
  substituted run-specific information; see the TEMPLATE section below for more information.

- `--thrax-conf FILE`

  Use the provided Thrax configuration file instead of the (grammar-specific) default.

- `--no-subsample [default]`
- `--subsample`

  Subsampling is a means of throwing away a portion of the training data by only retaining sentence
  pairs that appear relevant to the tuning and development data.  This makes the pipeline run
  significantly faster (particularly alignment), and often comes at only a small cost in accuracy on
  the development set.  Subsampling works by comparing the training data to the tuning and test sets
  passed in with --tune and --test.

- `--joshua-mem MEM [3100m]`

  Provide the maximum heap size available to instances of the Joshua decoder.  Uses Java notation
  (the value here is passed to Java's -Xmx flag).

- `--hadoop-mem MEM [4g]`

   Provide the maximum heap size for the hadoop instances used to learn the grammar in Thrax.

- `--jobs N`

  The number of parallel Joshua decoders to start (via qsub, by default) when decoding a input file,
  during both tuning and decoding.

- `--qsub-args "ARGS"`

  Provide the specified qsub arguments to the Joshua decoder command.  Only used if the Joshua
  decoder_command file contains the appropriate template variable (see the TEMPLATES section below).

## TEMPLATES

A few pieces of the pipeline are subject to enough variation across runs and across computing
environment that it is easier to provide template files than to use command-line arguments.  The
Joshua pipeline allows the Joshua configuration file and the Joshua decoder command to be
templatized.  Here are the options available to those files:

- `<JOSHUA>` is the root of the Joshua installation ($JOSHUA env. var.)
- `<INPUT>` is the file being decoded
- `<OUTPUT>` is the output file that the decoder command creates
- `<SOURCE>` is the source language (--source)
- `<TARGET>` is the target language (--target)
- `<LMFILE>` is the language model
- `<MEM>` is the amount of memory available to the Joshua decoder instances
- `<OOV>` is the OOV tag used in the grammar ("OOV" for SAMT, "X" for Hiero)
- `<NUMJOBS>` is the degree of parallelization for the Joshua decoder command
- `<QSUB_ARGS>` is the qsub arguments (--qsub-args)
- `<REF>` is the reference file
- `<CONFIG>` is the location of the joshua configuration file
- `<LOG>` is where the Joshua decoder should put its log file

The default configuration file is the following, located in
`$JOSHUA/scripts/training/templates/mert/decoder_command.sequential`.  It can be overridden with the
`--decoder_command` flag.

    cat <INPUT> | <JOSHUA>/joshua-decoder -m <MEM> -threads <NUMTHREADS> -c <CONFIG> > <OUTPUT> 2> <LOG>

If `--jobs` is set to something greater 1, the following template (located at
`$JOSHUA/scripts/training/templates/mert/decoder_command.qsub`) is selected instead:

    cat <INPUT> | awk 'BEGIN { num = 0 } {print "<seg id=\"" num "\">" $0 "</seg>"; num++}' \
      | <JOSHUA>/scripts/training/parallelize/parallelize.pl -j <NUMJOBS> -m --qsub-args '<QSUB_ARGS>' -- <JOSHUA>/joshua-decoder -m <MEM> -threads <NUMTHREADS> -c <CONFIG> > <OUTPUT> 2> <LOG>

## COMMON USE CASES AND PITFALLS 

- Memory usage is a major consideration in decoding with Joshua and hierarchical grammars.  In
  particular, SAMT grammars often require a large amount of memory.  Many steps have been taken to
  reduce memory usage, including beam settings and test-set- and sentence-level filtering of
  grammars.  However, memory usage can still be in the tens of gigabytes.

  To accommodate this kind of variation, the pipeline script allows you to specify both (a) the
  amount of memory used by the Joshua decoder instance and (b) the amount of memory required of
  nodes obtained by the qsub command.  These are accomplished with the --joshua-mem MEM and
  --qsub-args ARGS commands.  For example,

      pipeline.pl --joshua-mem 32g --qsub-args "-l pvmem=32g -q himem.q" ...

## FEEDBACK 

Please email joshua_technical@googlegroups.com with problems or suggestions.

