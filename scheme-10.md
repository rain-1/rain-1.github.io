# single_cream

This is a blog post about my new scheme interpreter, [single_cream](https://github.com/rain-1/single_cream).

I built it to support the [tarot](/scheme-9) scheme compiler. tarot is self hosting which creates a dependency loop between it and itself, I like to break this look by bootstrapping it off a much slower and simpler compiler written in a lower level language.

I was using tinyscheme for this but for reason, one day, tinyscheme gave GC errors instead of bootstrapping my compiler. I decided to implement my own scheme interpreter because I found tinyscheme's source code difficult and complex and I found that all the other small c based scheme interpreters were fundamentally lacking (for example, not implementing proper tail calls).

In total the program is a single file with 1600 lines of C code and 700 lines of C code that extend it from a primitive scheme interpreter to a more complete system with extra standard library functions and language constructs. I did my best to make the C code simple and easy to read.

## Implementing a AST based interpreter for scheme in C

single_cream is an AST based interpreter. This is the most conceptually simple kind of interpreter because it just parses in the input program and then executes that directly without processing it first. This also means it's also the slowest kind of interpreter.

How I designed this interpreter was to take a scheme based interpreter that looks like this:

(Let me take a code snippet from [matt might's blog here](http://matt.might.net/articles/implementing-a-programming-language/))

```
(define (eval exp env)
  (match exp
    [(? symbol?)          (env-lookup env exp)]
    [(? number?)          exp]
    [(? boolean?)         exp]
    [`(if ,ec ,et ,ef)    (if (eval ec env)
                              (eval et env)
                              (eval ef env))]
    [`(letrec ,binds ,eb) (eval-letrec binds eb env)]
    [`(let    ,binds ,eb) (eval-let binds eb env)]
    [`(lambda ,vs ,e)    `(closure ,exp ,env)]
    [`(set! ,v ,e)        (env-set! env v e)]
    [`(begin ,e1 ,e2)     (begin (eval e1 env)
                                 (eval e2 env))]
    [`(,f . ,args)        (apply-proc
                           (eval f env) 
                           (map (eval-with env) args))]))
```

and then I just manually translated that into C code, with a little extra care about tail-calls. So the C code version looks like this [sch3.c/eval_internal](https://github.com/rain-1/single_cream/blob/master/src/sch3.c#L991).

## Self interpreters and the gap between C and scheme

A self-interpreter is a funny thing. It turned out that self interpreters do not fully specify the semantics of a language. Lazy and strict lambda calculus would have the same self interpreter for example. Another tricky aspect of self interpreters is they let you take advantage of the host languages features to implement the object language.

For example writing a scheme self interpreter is really easy because you have symbols, s-expressions, garbage collection and tail call optimization already. But you have none of these in C - so I have to implement my own symbol table, s-expression objects and a garbage collector for them. I also have to ensure that all the tail calls in the scheme code are implemented using jumps in my C code.

Bridging this gap by implementing these key components is the essential part of bootstrapping a high level language from a lower level one. I feel that having a self interpreter and basing the lower level code provides a wonderful guide for both writing and understand the code.

I've written a FAQ explaining much more details about the [IMPLEMENTATION](https://github.com/rain-1/single_cream/blob/master/doc/IMPLEMENTATION.md) of single_cream.

## More

Read more here if you're interested in programming language development: https://rain-1.github.io/scheme
