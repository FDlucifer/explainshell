#[explainshell.com](http://www.explainshell.com) - match command-line arguments to their help text

explainshell is a tool (with a web interface) capable of parsing man pages, extracting options and
explain a given command-line by matching each argument to the relevant help text in the man page.

## How?

explainshell is built from the following components:

1. man page reader which converts a given man page from raw format to html (manpage.py)
2. classifier which goes through every paragraph in the man page and classifies
   it as contains options or not (algo/classifier.py)
3. an options extractor that scans classified paragraphs and looks for options (options.py)
4. a storage backend that saves processed man pages to mongodb (store.py)

When querying explainshell, it:

1. tokenzies the query (options.py)
2. does a look up in the store to see if we know how to explain the given program
3. goes through the rest of the tokens, trying to match each one to the list of extracted options
4. returns a list of matches that are rendered with Flask

## [TODO](https://raw.github.com/idank/explainshell/master/TODO) file

## Missing man pages

Right now explainshell.com contains the entire [archive of Ubuntu](http://manpages.ubuntu.com/). It's not
possible to directly add a missing man page to the live site (it might be in the future). Instead, submit a link [here](https://github.com/idank/explainshell/issues/1)
and I'll add it.

## Running explainshell locally

To setup a working environment that lets you run the web interface locally, you'll need to:

    $ pip install -r requirements.txt

    # load classifier data, needs a mongodb
    $ mongorestore dump/explainshell && mongorestore -d explainshell_tests dump/explainshell
    $ make tests
    .....................................................
    ----------------------------------------------------------------------
    Ran 53 tests in 2.847s

    OK

### Processing a man page

Use the manager to parse and save a gzipped man page in raw format:

    $ python explainshell/manager.py --log info echo
    INFO:explainshell.store:creating store, db = 'explainshell_tests', host = 'mongodb://localhost'
    INFO:explainshell.algo.classifier:train on 994 instances
    INFO:explainshell.manager:handling manpage echo (from /tmp/es/manpages/1/echo.1.gz)
    INFO:explainshell.store:looking up manpage in mapping with src 'echo'
    INFO:explainshell.manpage:executing '/tmp/es/tools/w3mman2html.cgi local=%2Ftmp%2Fes%2Fmanpages%2F1%2Fecho.1.gz'
    INFO:explainshell.algo.classifier:classified <paragraph 3, DESCRIPTION: '-n     do not output the trailing newlin'> (0.991381) as an option paragraph
    INFO:explainshell.algo.classifier:classified <paragraph 4, DESCRIPTION: '-e     enable interpretation of backslash escape'> (0.996904) as an option paragraph
    INFO:explainshell.algo.classifier:classified <paragraph 5, DESCRIPTION: '-E     disable interpretation of backslash escapes (default'> (0.998640) as an option paragraph
    INFO:explainshell.algo.classifier:classified <paragraph 6, DESCRIPTION: '--help display this help and exi'> (0.999215) as an option paragraph
    INFO:explainshell.algo.classifier:classified <paragraph 7, DESCRIPTION: '--version'> (0.999993) as an option paragraph
    INFO:explainshell.store:inserting mapping (alias) echo -> echo (52207a1fa9b52e42fb59df36) with score 10
    successfully added echo

### Start up a local web server:

    $ make serve
    python runserver.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader