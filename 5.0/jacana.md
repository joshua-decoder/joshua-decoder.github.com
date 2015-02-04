---
layout: default
title: Alignment with Jacana
---

## Introduction

jacana-xy is a token-based word aligner for machine translation, adapted from the original
English-English word aligner jacana-align described in the following paper:

    A Lightweight and High Performance Monolingual Word Aligner. Xuchen Yao, Benjamin Van Durme,
    Chris Callison-Burch and Peter Clark. Proceedings of ACL 2013, short papers.

It currently supports only aligning from French to English with a very limited feature set, from the
one week hack at the [Eighth MT Marathon 2013](http://statmt.org/mtm13). Please feel free to check
out the code, read to the bottom of this page, and
[send the author an email](http://www.cs.jhu.edu/~xuchen/) if you want to add more language pairs to
it.

## Build

jacana-xy is written in a mixture of Java and Scala. If you build from ant, you have to set up the
environmental variables `JAVA_HOME` and `SCALA_HOME`. In my system, I have:

    export JAVA_HOME=/usr/lib/jvm/java-6-sun-1.6.0.26
    export SCALA_HOME=/home/xuchen/Downloads/scala-2.10.2

Then type:

    ant

build/lib/jacana-xy.jar will be built for you.

If you build from Eclipse, first install scala-ide, then import the whole jacana folder as a Scala project. Eclipse should find the .project file and set up the project automatically for you.

Demo
scripts-align/runDemoServer.sh shows up the web demo. Direct your browser to http://localhost:8080/ and you should be able to align some sentences.

Note: To make jacana-xy know where to look for resource files, pass the property JACANA_HOME with Java when you run it:

java -DJACANA_HOME=/path/to/jacana -cp jacana-xy.jar ......

Browser
You can also browse one or two alignment files (*.json) with firefox opening src/web/AlignmentBrowser.html:



Note 1: due to strict security setting for accessing local files, Chrome/IE won't work.

Note 2: the input *.json files have to be in the same folder with AlignmentBrowser.html.

Align
scripts-align/alignFile.sh aligns tab-separated sentence files and outputs the output to a .json file that's accepted by the browser:

java -DJACANA_HOME=../ -jar ../build/lib/jacana-xy.jar -src fr -tgt en -m fr-en.model -a s.txt -o s.json

scripts-align/alignFile.sh takes GIZA++-style input files (one file containing the source sentences, and the other file the target sentences) and outputs to one .align file with dashed alignment indices (e.g. "1-2 0-4"):

java -DJACANA_HOME=../ -jar ../build/lib/jacana-xy.jar -m fr-en.model -src fr -tgt en -a s1.txt -b s2.txt -o s.align

Training
java -DJACANA_HOME=../ -jar ../build/lib/jacana-xy.jar -r train.json -d dev.json -t test.json -m /tmp/align.model

The aligner then would train on train.json, and report F1 values on dev.json for every 10 iterations, when the stopping criterion has reached, it will test on test.json.

For every 10 iterations, a model file is saved to (in this example) /tmp/align.model.iter_XX.F1_XX.X. Normally what I do is to select the one with the best F1 on dev.json, then run a final test on test.json:

java -DJACANA_HOME=../ -jar ../build/lib/jacana-xy.jar -t test.json -m /tmp/align.model.iter_XX.F1_XX.X

In this case since the training data is missing, the aligner assumes it's a test job, then reads model file still from the -m option, and test on test.json.

All the json files are in a format like the following (also accepted by the browser for display):

[
    {
        "id": "0008",
        "name": "Hansards.french-english.0008",
        "possibleAlign": "0-0 0-1 0-2",
        "source": "bravo !",
        "sureAlign": "1-3",
        "target": "hear , hear !"
    },
    {
        "id": "0009",
        "name": "Hansards.french-english.0009",
        "possibleAlign": "1-1 6-5 7-5 6-6 7-6 13-10 13-11",
        "source": "monsieur le Orateur , ma question se adresse à le ministre chargé de les transports .",
        "sureAlign": "0-0 2-1 3-2 4-3 5-4 8-7 9-8 10-9 12-10 14-11 15-12",
        "target": "Mr. Speaker , my question is directed to the Minister of Transport ."
    }
]
Where possibleAlign is not used.

The stopping criterion is to run up to 300 iterations or when the objective difference between two iterations is less than 0.001, whichever happens first. Currently they are hard-coded. If you need to be flexible on this, send me an email!

Support More Languages
To add support to more languages, you need:

labelled word alignment (in the download there's already French-English under alignment-data/fr-en; I also have Chinese-English and Arabic-English; let me know if you have more). Usually 100 labelled sentence pairs would be enough
implement some feature functions for this language pair
To add more features, you need to implement the following interface:

edu.jhu.jacana.align.feature.AlignFeature

and override the following function:

addPhraseBasedFeature

For instance, a simple feature that checks whether the two words are translations in wiktionary for the French-English alignment task has the function implemented as:

def addPhraseBasedFeature(pair: AlignPair, ins:AlignFeatureVector, i:Int, srcSpan:Int, j:Int, tgtSpan:Int,
      currState:Int, featureAlphabet: Alphabet){
  if (j == -1) {
  } else {
    val srcTokens = pair.srcTokens.slice(i, i+srcSpan).mkString(" ")
    val tgtTokens = pair.tgtTokens.slice(j, j+tgtSpan).mkString(" ")
                
    if (WiktionaryMultilingual.exists(srcTokens, tgtTokens)) {
      ins.addFeature("InWiktionary", NONE_STATE, currState, 1.0, srcSpan, featureAlphabet) 
    }
        
  }       
}
This is a more general function that also deals with phrase alignment. But it is suggested to implement it just for token alignment as currently the phrase alignment part is very slow to train (60x slower than token alignment).

Some other language-independent and English-only features are implemented under the package edu.jhu.jacana.align.feature, for instance:

StringSimilarityAlignFeature: various string similarity measures

PositionalAlignFeature: features based on relative sentence positions

DistortionAlignFeature: Markovian (state transition) features

When you add features for more languages, just create a new package like the one for French-English:

edu.jhu.jacana.align.feature.fr_en

and start coding!

