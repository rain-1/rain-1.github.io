# Scheme Programming Lessons from Tarot

This page is for documenting things learned from implementing tarot, a basic self hosted scheme compiler that I did.

## Table of contents

* [sequences](scheme-1) - constructing complicated lists efficiently.
* [accessors](scheme-2) - breaking apart data structures in a simple way.
* [stacks and queues](scheme-3) - hammer and screwdriver, for programming.
* [syntactic extension](scheme-4) - making things easy with macros!
* [qcode](scheme-5) - some low level details about the compiler target and virtual machine.
* [eval and macros](scheme-6) - how we implement an efficient eval, and how is it used to implement macros.
* [pointer tagging and data representation](scheme-7) - how is data represented and processed in the runtime.
* [information for stack traces and profiling](scheme-8) - how we print useful stack traces, and profiling info.

* [bootstrappable release](scheme-9) - tarot is released! and it is bootstrappable from sources.

## Compiler Passes

* desugar
* [hoist](tarot-hoist)
* [denest](tarot-denest)
* tmp-alloc
* [flatten & assemble](tarot-flatten)

## Blogging

* [scheme-continuations](scheme-continuations) my opinion about first class continuations.
* [scheme-parse](scheme-parse) some work on parsing s-expressions.
* [scheme-10](scheme-10) single_cream interpreter
* [closure-conversion.rkt](https://gist.github.com/rain-1/36c4851b7c29cf8e42f23ba6eec37be6) closure conversion distilled.
* [serialize.rkt](https://gist.github.com/rain-1/dc738599829ed40b55f170b574628bd0) extending the above to support serializable closures


