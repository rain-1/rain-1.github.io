# The Hoist Pass

`parse` => `desugar` => [hoist.scm](https://notabug.org/rain1/tarrochi/src/master/compiler/passes/hoist.scm) => `denest` => `tmp-alloc` => `flatten` => `assemble`

Compiling is translating code from one language to another, in our case we are going from a high level language with first class/higher order functions/lambda to a low level bytecode language which doesn't have them. So we need to somehow remove all the lambdas by translating them into something lower lever. Doing this essentially "implements" lambda by expressing it in terms of something else. In this note I explain the hoist pass which does this and some other things.

The hoist pass does two things to the input code:
* closure conversion: hoists all lambdas up to the top level, by converting them to closures
* storage analysis: analyzing the storage type of each variable

I found that I was able to implement both these operations simultaneously in a very concise way. In fact I would say that the hoist pass is really the best part of the whole tarot compiler.

One of the really cool mind blowing bits from [SICP](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-14.html#%_thm_2.4) (alternatively lambda calculus/church encoding) was that you could implement `CONS` just using lambda. That makes a good example for demonstrating the hoist pass because it involves some nested procedures that close over free variables. So if the input program is this:

```scheme
(define (kons kar kdr) (lambda (sel) (sel kar kdr)))
(define (kar kons) (kons (lambda (x y) x)))
(define (kdr kons) (kons (lambda (x y) y)))
```

let's see what hoisting does to it.

## Closures

Procedures defined with lambda are converted into "closures". In tarot closures are implemented as a data structure with two parts: 1. A pointer to the code that implements the lambda procedure and 2. all of the closed over environment variables to be carried around with it. This is what allows us to pass procedures around as values.

## Storage Types

The storage types are:
* `tmp` - Temporary variables/`LET`-bound variables inside a procedure.
* `loc` - A function parameter.
* `env` - An environment variable, accessing the closured over variables.
* `glo` - A global variable, `DEFINE`d at the top level.

## Example

After the desugar pass function applications have been explicit:

```scheme
pass:desugar
(define "tests/scm/kons.scm" kons (lambda (kar kdr) (lambda (sel) (app sel kar kdr))))
(define "tests/scm/kons.scm" kar (lambda (kons) (app kons (lambda (x y) x))))
(define "tests/scm/kons.scm" kdr (lambda (kons) (app kons (lambda (x y) y))))
```

Now applying the hoist pass to the 3 functions defined above we get this soup of definitions and closures:

```scheme
pass:hoist
(def53b2564f "kons: tests/scm/kons.scm" kons 0 (closure 0 () closure2e17eca7))
(def4da32c7e "kar: tests/scm/kons.scm" kar 0 (closure 0 () closure1af7f0ea))
(def2fd0ad81 "kdr: tests/scm/kons.scm" kdr 0 (closure 0 () closure0e0d31ff))

(closure2e17eca7 "kons: tests/scm/kons.scm" #f 2 (closure 2 ((var loc 0) (var loc 1)) closure3d2dd275))
(closure3d2dd275 "kons: tests/scm/kons.scm" #f 1 (app (var loc 0) (var env 0) (var env 1)))

(closure1af7f0ea "kar: tests/scm/kons.scm" #f 1 (app (var loc 0) (closure 0 () closure75509d76)))
(closure75509d76 "kar: tests/scm/kons.scm" #f 2 (var loc 0))

(closure0e0d31ff "kdr: tests/scm/kons.scm" #f 1 (app (var loc 0) (closure 0 () closure6ec68664)))
(closure6ec68664 "kdr: tests/scm/kons.scm" #f 2 (var loc 1))
```

The format of the entries in this 'soup' that hoist outputs is:
```
(<label> <information> <definition-name> <parameters> <body>)
```

Each entry describes a first order procedure

* `label` is a unique identifier.
* `information` is used during execution so that after a crash stack traces can provide useful information.
* `definition-name` is the variable that this entry defines, or #f if it is just a lambda nested inside a definition or appearing elsewher.
* `paramaters` is the number of arguments this procedure expects when called.
* `body` is the actual code for the proc.

You can see how the two lambdas in the definition of `kons` have been turned into two top level closure definitions taking 2 and then 1 parameter. The first part just builds the inner closure, the second part performs the function application by picking the variables out based on whether they come as parameters or from the environment.

Hopefully this explains enough to make it clear what the goal of this pass is, and makes reading the source code for it easy: [hoist.scm](https://notabug.org/rain1/tarrochi/src/master/compiler/passes/hoist.scm).
