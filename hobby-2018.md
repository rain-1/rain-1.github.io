# Hobby programming in 2018


# Unixy

## s

The s shell. I made a very simple shell based on some 'least surprise' type design principles. This was a really good project to do but I think the main thing I learned from it is that we can't solve "the problem" by just improving the shell, we also need to change the way argv is done and the text formats that our userspace tools input and output.

## makes

This is the second version of a build tool that I made out of frustration with GNU make. I decided to stop using GNU make.

The first version was extremely simple and relatively powerful, it implemented only the 'build X out of Y,Z,W if X doesn't exist or is older than one of the sources' and you would use this in a shallow embedding inside a shell script. It was impossible to extend this technique to parallel builds.

The second version supports parallel builds. It's implemented in golang using the go routines, I create a high number of goroutines and have them executed in 8 threads. I think this is a very practical tool that can be used to replace make despite the fact that the implementation is much shorter (thousands of times). Unfortunately I don't think anybody will adopt it.

## linenoise-mob

This is a fork of the BSD licensed readline replacement, because pull requests were not really getting merged into the main repo. I applied over 100 patches to it. I hope this project survives and ideally gets merged back into the upstream source (although I'm not sure if that's possible anymore since it implements Unicode).

## lumbergh

I implemented a process supervisor. It's a unix tool that launches various processes and supports restarting them if they die and also takedown. It's a lot more difficult and involved than you may expect. I studied the book Advanced Programming in the Unix Environment. I never got it working 100% the way I wanted it.

## tests

This may be one of the most useful pieces of software I wrote. It's just a simple script to automatically run tests. Recommended.

* https://github.com/rain-1/tests

## bao

I find the vala GUI program 'baobab' very useful, I started on a very very stripped down CLI version in golang. It is not complete but it lists the sizes of things. A breadth first version of 'find' could be good too.

* https://github.com/rain-1/bao




# Scheme

## racket-peg

This is a PEG parser library. I did an analysis of the parser libraries [here](https://rain-1.github.io/racket-parsing-libraries) for racket scheme because I wasn't too happy with the 'standard' one since it had some hygiene issues. I implemented the peg system as a macro, and also provided the standard peg syntax as a racket lang using a self hosted parser. Later on I ported the library to guile scheme.

I implemented "nand-lang" to demo the power of this system. I think it really has a lot of value.

I want to give a big thanks to petrolifero who collaborated with me on this a lot. So much fun working with you!

## Continuations study group

I did a literature review on delimited continuations. https://github.com/rain-1/continuations-study-group/wiki/Reading-List

My conclusions from reading a lot about continuations is:
* You really need multiple-prompts if you want to implement composable language constructs using them.
* I've become less convinced that delimited continuations are a good tool to base the rest of the language on.

## scheme_interpreter

I implemented a basic scheme interpreter in C which supports GC and tail calls. I am very happy with the clarity and brevity of the implementation. It seems like a very promising project. I started this with the aim of bootstrapping my tarot compiler off of it, instead of tinyscheme. Did not get there yet, need to build up to that with a lot of tests and a new macro system.

I had a novel idea for the macro system which I'm quite keen on. Basically every object the interpreter reads from a file should be run through a procedure - which is initially the identity function - before execution. In scheme you can replace this procedure with an increasingly complex macro expander system. The benefit of this is that it allows you to do much more of the programming in scheme rather than C.

I intend to continue developing this.

* https://github.com/rain-1/scheme_interpreter

## tarot

This is my self hosting and bootstrappable scheme compiler that runs on top of a VM implemented in C. The source code is very short and it's very efficient. I'm proud of this because it's one of the projects I've worked the hardest on.

* https://rain-1.github.io/scheme
* https://rain-1.github.io/scheme-9


# Interesting projects

## ras52 Bootstrap

Incredible work.

* https://github.com/ras52/bootstrap

## Interim OS

Beautiful idea to combine lisp and plan9, but I did not manage to get it to run. Cursed bitrot!

## Eulex

A very cool forth based operating system. also includes a mini lisp interpreter.

## Okami

A nice forth implementation that I like.

* https://github.com/wolfgangj/okami

## commit-sudoku

This was a sudoku game we played/solved together by github commits. I used my own scheme compiler to run a sudoku solver I wrote for it.

## rust based userspace

linux and non-linux OSs built entirely with rust. There are a great deal of projects related to this.



# Blogging

The blog posts I've written:

* https://rain-1.github.io/racket-parsing-libraries
* https://rain-1.github.io/scheme-continuations
* https://rain-1.github.io/scheme-srfi-1 The struggling and sad state of scheme standardization.
* https://rain-1.github.io/shell about s.
* https://rain-1.github.io/shell-2 about the missed opportunity of using structured text instead of plaintext in UNIX.
* https://rain-1.github.io/escaping A study on escaping.
* https://gist.github.com/rain-1/a253e47b939fc0769524d8716541c96e canonical s-exps.
* https://gist.github.com/rain-1/e6293ec0113c193ecc23d5529461d322 tab separated values.
* https://gist.github.com/rain-1/a8d1bb04ad01ebe6a4ac68fa33f7961c a very valuable regular expression operation that most libraries don't seem to implement.
* https://rain-1.github.io/in-browser-localhostdiscovery POC demonstrating a surprising (to me) amount of information that arbitrary web pages can learn about your internal network.
* https://rain-1.github.io/bootstrapping/os - Made a list of minimal/hobby operating systems that people have made. WIP.

I spent a lot of time thinking about problems with UNIX/POSIX/linux. I think I made a lot of progress on understanding the directions we need to go to improve things but it's very difficult to get other people on board.


# Wikiing

* https://bootstrapping.miraheze.org/wiki/Main_Page I maintained this wiki a lot.
* https://softwarecrisis.miraheze.org/wiki/Special:AllPages I added a tiny amount of things to this one.
* http://rosettacode.org/wiki/Rosetta_Code I can't log in here, so I didn't really do much here.
