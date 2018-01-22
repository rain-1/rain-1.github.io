# parsing libraries in racket scheme

This information was compiled in Jan 22 2018.

These are the libraries in the racket pkgs database that you can use to make your own parser. There are a few of them so it's difficult to choose. The aim of this article is to make it easier to choose.

At the moment *I recommend ragg*.

**NAME** [parser-tools-lib](http://docs.racket-lang.org/parser-tools/index.html?q=parser-tools)
**AUTHORS** Scott Owens
**LOC** 5885
**NOTE** this is part of racket
**SYNTAX** s-exp based yacc or CFG
**DEPS** none
**EXAMPLES** https://github.com/racket/parser-tools/tree/master/parser-tools-lib/parser-tools/examples

**NAME** [combinator-parser](http://docs.racket-lang.org/combinator-parser/index.html)
**AUTHORS** asumu, takikawa
**LOC** 1959
**NOTE** Deprecated "We recommend using either parser-tools/yacc or other parsing packages such as parsack or ragg."
**SYNTAX** combinators
**DEPS** parser-tools-lib
**EXAMPLES** https://github.com/takikawa/combinator-parser/blob/master/combinator-parser/examples/combinator-example.rkt

**NAME** [parsack](http://docs.racket-lang.org/parsack/index.html)
**AUTHORS** stchang
**LOC** 1984
**NOTE**
**SYNTAX** monadic notation
**DEPS** none
**EXAMPLES** https://github.com/stchang/parsack/tree/master/parsack/examples

**NAME** [megaparsack](http://docs.racket-lang.org/megaparsack/)
**AUTHORS** Alexis King
**LOC** 782
**NOTE** http://docs.racket-lang.org/megaparsack/differences-from-parsack.html
**SYNTAX** monadic notation
**DEPS** none
**EXAMPLES** https://github.com/lexi-lambda/megaparsack/blob/master/megaparsack-parser/megaparsack/parser/json.rkt

**NAME** [ragg](https://pkgs.racket-lang.org/package/ragg)
**AUTHORS** dyoo, clements, mb
**LOC** 4222
**NOTE** A design goal is to be easy for beginners to use
**SYNTAX** EBNF
**DEPS** parser-tools-lib
**EXAMPLES** https://github.com/jbclements/ragg/tree/master/ragg/examples

**NAME** [sparse](http://docs.racket-lang.org/sparse/index.html)
**AUTHORS** esco ricky
**LOC** 899
**NOTE** this is for recognizing s-expression languages
**SYNTAX** s-expressions
**DEPS** none
**EXAMPLES** http://docs.racket-lang.org/sparse/index.html#%28part._.Example%29

**NAME** [derp-3](https://pkgs.racket-lang.org/package/derp-3)
**AUTHORS** clements, michael adams
**LOC** 996
**NOTE** based on "Parsing with Derivatives". Seems to lack documentation.
**SYNTAX** production rules
**DEPS** none
**EXAMPLES** 

**NAME** [binary-class](https://pkgs.racket-lang.org/package/binary-class)
**AUTHORS** kalimehtar
**LOC** 701
**NOTE** for binary data
**SYNTAX** lisp struct definitions
**DEPS** none
**EXAMPLES** http://docs.racket-lang.org/binary-class/index.html#%28part._.Binary_class__.Base_system_%29
