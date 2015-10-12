# Getting started

## Installation

Arpeggio is written in Python programming language and distributed with
setuptools support. If you have `pip` tool installed Arpeggio can be installed
form [PyPI](https://pypi.python.org/pypi/Arpeggio/) with the following command:

```bash
    $ pip install Arpeggio
```

You can also install Arpeggio from source. In this case you have two options.
Either download source distribution, unpack and install:

```bash
$ wget https://github.com/igordejanovic/Arpeggio/archive/v1.1.tar.gz
$ tar xzf v1.1.tar.gz
$ cd Arpeggio-1.1
$ python setup.py install
```

!!! note
    We strive to keep `master` branch always stable so it is fairly safe to use.
    To install current development branch of Arpeggio instead of the stable release
    use `https://github.com/igordejanovic/Arpeggio/archive/master.tar.gz` URL in the
    above download command.

Or clone source repository with git:

```bash
$ git clone git@github.com:igordejanovic/Arpeggio.git
$ cd Arpeggio
$ python setup.py install
```

!!! note
    By default, after cloning you code will be on the `master` branch. It is current
    development branch. If you want to install some stable release you
    must first switch to appropriate `tag` before issuing command `python
    setup.py install`.

        $ git checkout v1.1
        $ python setup.py install

To verify that you have installed Arpeggio correctly run the following command:

```bash
$ python -c 'import arpeggio'
```

If you get no error, Arpeggio is correctly installed.


## Quick start


**Write [a grammar](grammars.md)**. There are several ways to do that:

- [The canonical grammar format](grammars.md#grammars-written-in-python) uses
  Python statements and expressions.  Each rule is specified as Python function
  which should return a data structure that defines the rule. For example a
  grammar for simple calculator can be written as:

        from arpeggio import Optional, ZeroOrMore, OneOrMore, EOF
        from arpeggio import RegExMatch as _

        def number():     return _(r'\d*\.\d*|\d+')
        def factor():     return Optional(["+","-"]), [number, ("(", expression, ")")]
        def term():       return factor, ZeroOrMore(["*","/"], factor)
        def expression(): return term, ZeroOrMore(["+", "-"], term)
        def calc():       return OneOrMore(expression), EOF

    The python lists in the data structure represent ordered choices while the tuples represent sequences from the PEG.
    For terminal matches use plain strings or regular expressions.

- The same grammar could also be written using [traditional textual PEG
  syntax](grammars.md#grammars-written-in-peg-notations) like this:

        number <- r'\d*\.\d*|\d+';  // this is a comment
        factor <- ("+" / "-")? (number / "(" expression ")");
        term <- factor (( "*" / "/") factor)*;
        expression <- term (("+" / "-") term)*;
        calc <- expression+ EOF;

- Or similar syntax but a little bit more readable like this:

        number = r'\d*\.\d*|\d+'    # this is a comment
        factor = ("+" / "-")? (number / "(" expression ")")
        term = factor (( "*" / "/") factor)*
        expression = term (("+" / "-") term)*
        calc = expression+ EOF

    The second and third options are implemented using canonical first form.
    Feel free to implement your own grammar syntax if you don't like these
    (see modules `arpeggio.peg` and `arpeggio.cleanpeg`).

**Instantiate a parser**. Parser works as a grammar interpreter. There is no
code generation.

```python
from arpeggio import ParserPython
parser = ParserPython(calc)   # calc is the root rule of your grammar
                              # Use param debug=True for verbose debugging
                              # messages and grammar and parse tree visualization
                              # using graphviz and dot
```

**Parse your inputs**

```python
parse_tree = parser.parse("-(4-1)*5+(2+4.67)+5.89/(.2+7)")
```

If parsing is successful (e.g. no syntax error if found) you get a [parse
tree](parse_trees.md).

**Analyze parse tree** directly or write a [visitor class](semantics.md) to
transform it to a more usable form.

For [textual PEG syntaxes](grammars.md#grammars-written-in-peg-notations)
instead of `ParserPyton` instantiate `ParserPEG` from `arpeggio.peg` or
`arpeggio.cleanpeg` modules. See examples how it is done.

To [debug your grammar](debugging.md) set `debug` parameter to `True`. A verbose
debug messages will be printed and a dot files will be generated for parser
model (grammar) and parse tree visualization.

Here is an image rendered using graphviz of parser model for `calc` grammar.

<img src="../images/calc_parser_model.dot.png" style="display:block; width: 15cm; margin-left:auto; margin-right:auto;"/>


And here is an image rendered for parse tree for the above parsed `calc` expression.

<img src="../images/calc_parse_tree.dot.png"/>

