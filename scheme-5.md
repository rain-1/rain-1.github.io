# Programming Lessons from Tarot: 1 - qcode

To execute a scheme program with tarot, we compile it into "qcode" then load and execute it in the virtual machine. This document describes qcode. It's a lot like bytecode except instead being byte-based it's 64 bit (qword) based.

## Virtual machine

For the virtual machine to execute compiled scheme it needs to implement a runtime platform meaning: symbol table, datatypes, garbage collection, ... Every scheme object is represented in the VM as a tagged 64 bit word. The tag says whether it's a boolean, number, character, cons cell, vector or a file port. Once you take the tag off you either get the raw data for the number, char or you get a pointer into the heap.

I wanted to be able to easily refer to constant data in compiled code so instead of using a bytecode format, what about making it a sequence of 64 bit words? This makes the interpreter loop really simple: just fetch a qword, execute it and its execution might fetch more qwords as immediate. There is no "decoding" of complicated byte sequences. The previous scheme compiler I wrote used a horrible bytecode system that required lots of decoding, unescaping.. it was nasty. This new one is so much cleaner, I'm really happy with it.

From [qcode.h](https://github.com/rain-1/tarot-vm/blob/master/qcodes.h) you can see there is only 30 different instructions but we encode them using a whole 64 bit word, capable of expressing 18446744073709551616 different instructions. A lot of waste! I was worried that such waste would be a disaster for speed and make it very very slow - luckily it doesn't!

## bf test

Brainfuck interpreters are usually based on bytecode, so to test the idea about using 64 bit words I took (A) a standard brainfuck interpreter (B) A faster brainfuck interpreter that does a quick assembly pass during loading, to point open and close brackets to each other.

Then I modified them both to operate on 64 bit words instead of bytes, if we call these (A') and (B'). I ran the brainfuck mandelbrot render program in them. A' is a tiny bit slower than A. A is much much much slower than B'. B' is a tiny bit slower than B. `A' > A >>>>>>>>>>>>>>> B' > B`.

This gave a strong indication that using 64 bit words will be fine, only causing a tiny impact on speed compared against a much more compact encoding.

## qcode serialization and loading

The compiler doesn't actually emit a sequence of 64 bit words though. It emits a "tokens" which the virtual machine can read in and translate into 64 bit words before they get executed. When it seeds string data it allocates that and loads a single pointer to it into the code section! The loading stage also resolves global variables. Like brainfuck resolving []'s at load time, instead of every single time program flow reaches them this is an enormous speedup!

(Actually I forgot to do the global variable resolution at loading time, so originally the compiler was a lot slower. 1 min 30 to bootstrap. When this was found and fixed - by making that happen at load time - it cut the time down to one second! I cut this down to half a second later on by rewriting the parser.)

The serialized form is just text tokens separated by \0 the nul byte. Since strings can never contain a nul byte one magic aspect of using this format is that we do not have to escape or unescape strings! It's really simple to work with:

Here's an example of a scheme function:

```scheme
(define (not b) (if b #f #t))
```

and the qcode it gets compiled into:

```
stack-grow^@1^@allocate-closure^@0^@7^@clo-set-loc^@0
^@var-loc^@0^@set-glo^@not^@halt^@var-loc^@0^@branch
^@3^@datum-false^@jump^@1^@datum-true^@ret^@
```

(`^@` represents the nul byte which is a non printing character)

## Interpreter

Finally here is the [loader](https://github.com/rain-1/tarot-vm/blob/master/loader.c) and the VM interpreter loop: [interpreter.c](https://github.com/rain-1/tarot-vm/blob/master/interpreter.c) `0xC0FFEEEEEEEEEEEE` and `0xDEADBEEFDEADBEEF` are stack canaries.

I used a computed goto table instead of a switch/case because this paper says it's much better for the branch predictor: [The Structure and Performance of Efficient Interpreters](https://www.jilp.org/vol5/v5paper12.pdf).

## Conclusions

* I used a "scale model" (brainfuck) to validate an idea but how can we know when scale models are valid and when they aren't?
* Using 64 bit codewords is perfectly reasonable! My scheme compiler proves this by bootstrapping in half of a second. Other scheme compilers take minuites or even hours.
* The technique of ^@ separated tokens is wonderful. I learned it from the STOMP protocol and I think it can be fruitful in many other places. e.g. for UNIX command lines.
* The biggest speedups in VMs come from moving things from run time to loading time.

