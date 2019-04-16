# Bootstappable Tarot Release

I am releasing a fully bootstrappable release of the Tarot scheme compiler today!

* It is a scheme compiler written in 1907 lines of scheme. 
* It (when compiled) runs on top of a virtual machine implemented in 2033 lines of C.
* It is able to compile itself.
* It can be run from plain sources on the tinyscheme interpreter.

You can bootstrap it by first compiling the compiler by running it on the tinyscheme interpreter. This takes about 10 mins on a fast CPU. After that you can self host it in about 20 seconds.

You can read more about the internal works of this system [here](scheme). If there are any aspects of the compiler that are unclear, you are very welcome to ask me about it!

(Note: It doesn't implement any scheme specification. It's just the parts of R5RS that I tend to use. No first class continuations. It doesn't have hygienic macros.)

To give it a go you can get all the sources:

```
git clone https://github.com/rain-1/tinyscheme.git
git clone https://github.com/rain-1/tarot-vm.git
git clone https://github.com/rain-1/tarot-compiler.git
git clone https://github.com/rain-1/tarot-scripts.git
git clone https://github.com/rain-1/tarot-tests.git
```

then run the scripts:

```
bash tarot-scripts/0-build-binaries.sh
bash tarot-scripts/1-bootstrap-tinyscheme.sh
bash tarot-scripts/2-selfhost.sh
bash tarot-scripts/3-check-differences.sh
bash tarot-scripts/run-tests.sh
```

You need [tests](https://github.com/rain-1/tests) to run the tests!

I hope it all works smoothly, if there are any issues tell me.


**UPDATE** Jan 2019.

tinyscheme suddenly stopped working. Some kind of GC error.

So I implemented a new tiny scheme interpreter <https://github.com/rain-1/single_cream> and bootstrapped tarot off that!

**UPDATE** April 2019.

Further work making the single_cream based bootstrap smoother.

## Infosheet

```
tarot is a scheme compiler
it is written in scheme (1917 lines)
it runs on top of a virtual machine
the virtual machine is written in C (2036 lines)
it can compile itself
it takes 14 seconds to self host

single_cream is a scheme interpreter
it is written in C (1705 lines)
it has supporting code and a standard library in scheme (691 lines)
it can run tarot
it takes 1 min 40 seconds to bootstrap tarot

$ wc -l tarot/src/*c
 1705 src/sch3.c
$ wc -l tarot/src/*scm
  221 src/init.scm
  358 src/preprocessor.scm
  112 src/std.scm
  691 total
$ linecount tarot-compiler/ '*.scm'
1917
$ linecount tarot-vm '*.c'
2036
```
