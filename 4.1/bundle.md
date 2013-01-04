---
layout: default
category: links
title: Bundling a configuration
---

A *bundled configuration* is a minimal set of configuration, resource, and script files. A script, `$JOSHUA/scripts/support/run-bundler.py` can be used to package up the run bundle. The resulting bundle can easily be transferred and shared.

**Example invocation:**

    ./run-bundler.py \
      --force \
      /path/to/rundir/runs/5/test/1/joshua.config \
      /path/to/rundir/runs/5 \
      bundled-configurations \
        "-top-n 1 \
        -output-format %S \
        -mark-oovs false \
        -server-port 5674 \
        -tm/pt "thrax pt 20 /path/to/rundir/runs/5/test/1/grammar.gz"

A new directory `./bundled-configurations` will be created, and all the bundled files will be copied or created in it.  To use the configuration with Joshua, run the executable file `./bundled-configurations/bundle-runner.sh`.

Note, the additional options between the pair of quotation marks are passed as arguments to the `$JOSHUA/scripts/copy-config.pl` script. That script has some special parameters, especially the `-tm/..` option.
