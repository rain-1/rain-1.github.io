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
sh tarot-scripts/0-build-binaries.sh
sh tarot-scripts/1-bootstrap-tinyscheme.sh
sh tarot-scripts/2-selfhost.sh
sh tarot-scripts/3-check-differences.sh
sh tarot-scripts/run-tests.sh
```

I hope it all works smoothly, if there are any issues tell me.
