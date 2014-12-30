---
layout: default6
title: Welcome to Joshua
---

            <h4 class="blog-post-title">Welcome to Joshua!</h4>

            <p>This blog post shows a few different types of content that's supported and styled with Bootstrap. Basic typography, images, and code are all supported.</p>
            <hr>
            <p>Cum sociis natoque penatibus et magnis <a href="#">dis parturient montes</a>, nascetur ridiculus mus. Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Sed posuere consectetur est at lobortis. Cras mattis consectetur purus sit amet fermentum.</p>
            <blockquote>
              <p>Curabitur blandit tempus porttitor. <strong>Nullam quis risus eget urna mollis</strong> ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.</p>
            </blockquote>
            <p>Etiam porta <em>sem malesuada magna</em> mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.</p>
            <h2>Heading</h2>
            <p>Vivamus sagittis lacus vel augue laoreet rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.</p>
            <h3>Sub-heading</h3>
            <p>Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.</p>
            <pre><code>Example code block</code></pre>
            <p>Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.</p>
            <h3>Sub-heading</h3>
            <p>Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.</p>
            <ul>
              <li>Praesent commodo cursus magna, vel scelerisque nisl consectetur et.</li>
              <li>Donec id elit non mi porta gravida at eget metus.</li>
              <li>Nulla vitae elit libero, a pharetra augue.</li>
            </ul>
            <p>Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.</p>
            <ol>
              <li>Vestibulum id ligula porta felis euismod semper.</li>
              <li>Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.</li>
              <li>Maecenas sed diam eget risus varius blandit sit amet non magna.</li>
            </ol>
            <p>Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.</p>
          </div><!-- /.blog-post -->

          <!-- <nav> -->
          <!--   <ul class="pager"> -->
          <!--     <li><a href="#">Previous</a></li> -->
          <!--     <li><a href="#">Next</a></li> -->
          <!--   </ul> -->
          <!-- </nav> -->


This page contains end-user oriented documentation for the 6.0 release of
[the Joshua decoder](http://joshua-decoder.org/).

## Download and Setup

1. Download Joshua by clicking the big green button above, or from the command line:

       wget -q https://github.com/joshua-decoder/joshua-releases/joshua-6.0.tgz

1. Next, unpack it, set environment variables, and compile everything:

       tar xzf joshua-6.0.tgz
       cd joshua-6.0

       # for bash
       export JAVA_HOME=/path/to/java
       export JOSHUA=$(pwd)
       echo "export JOSHUA=$JOSHUA" >> ~/.bashrc

       # for tcsh
       setenv JAVA_HOME /path/to/java
       setenv JOSHUA `pwd`
       echo "setenv JOSHUA $JOSHUA" >> ~/.profile
       
       ant

   (If you don't know what to set `$JAVA_HOME` to, try `/usr/java/default`)

1. If you have a Hadoop installation, make sure that the environment variable `$HADOOP` is set and
points to it. If you don't, Joshua will roll one out for you in standalone mode. Hadoop is only
needed if you plan to build new models with Joshua.

1. In addition, you will need to install Moses if either of the following applies to you:

    - You wish to build phrase-based models (Joshua 6.0 includes a phrase-based decoder, but
      not the tools for building such a model)

    - You are building your own models (phrase- or syntax-based) and wish to use Cherry & Foster's
[batch MIRA tuner](http://aclweb.org/anthology-new/N/N12/N12-1047v2.pdf) instead of the included MERT. 

    Follow [the instructions for installing Moses
here](http://www.statmt.org/moses/?n=Development.GetStarted), and then define the `$MOSES`
environment variable to point to the root of the Moses installation.

## Quick start

Our <a href="pipeline.html">pipeline script</a> is the quickest way to get started. For example, to
train and test a complete model translating from Bengali to English:

First, download the Indian languages data:
   
    wget --no-check -O indian-languages.tgz https://github.com/joshua-decoder/indian-parallel-corpora/tarball/master
    tar xf indian-languages.tgz
    ln -s joshua-decoder-indian-parallel-corpora-b71d31a input

Then, train and test a model

    $JOSHUA/bin/pipeline.pl --source bn --target en \
        --no-prepare --aligner berkeley \
        --corpus input/bn-en/tok/training.bn-en \
        --tune input/bn-en/tok/dev.bn-en \
        --test input/bn-en/tok/devtest.bn-en

This will align the data with the Berkeley aligner, build a Hiero model, tune with MERT, decode the
test sets, and reports results that should correspond with what you find on <a
href="/indian-parallel-corpora/">the Indian Parallel Corpora page</a>. For
more details, including information on the many options available with the pipeline script, please
see <a href="pipeline.html">its documentation page</a>.

## More information

For more detail on the decoder itself, including its command-line options, see
[the Joshua decoder page](decoder.html).  You can also learn more about other steps of
[the Joshua MT pipeline](pipeline.html), including [grammar extraction](thrax.html) with Thrax and
Joshua's [efficient grammar representation](packing.html).

If you have problems or issues, you might find some help [on our answers page](faq.html) or
[in the mailing list archives](https://groups.google.com/forum/?fromgroups#!forum/joshua_support).

A [bundled configuration](bundle.html), which is a minimal set of configuration, resource, and script files, can be created and easily transferred and shared.
