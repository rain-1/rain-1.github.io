# Scheme Programming Lessons from Tarot: 4 - Syntactic Extension

One of the luxuries of scheme that also helps keep the compiler code short is macros. Other languages like [javascript](https://www.sweetjs.org/) can have macros too, it's just not as common. We only need to implement a few important primitives (things like numbers, strings, variables, function calls, lambda, letrec) and then we can extend and add to the language to make it easier to program in.

The hallmark of scheme is hygenic macros, they follow special rules to respect lexical scope. I just used a very basic unhygenic 'defmacro' style system instead, because it's much easier to implement - surprisingly there was absolutely no macro-expansion bugs from unwanted capture or anythnig like that. See [scope sets](http://www.cs.utah.edu/plt/scope-sets/) for the state of the art on hygene.

In the compiler we only implement the bare minimum and then we add the most important extra features in a file `mac/macros.scm`.

## quasiquote

Quasiquotation is an incredibly versatile tool for building up lists, it comes with an unquote (,) operator that lets you drop back into scheme. It's almost like a basic form of staging. For an example \`(1 ,(+ 1 1) 3) builds up the list (1 2 3). In terms of parsing \`x is short for (quasiquote x) and ,x is short for (unquote x).

Here's how we add this functionality to our scheme:

```scheme
(define (quasiquote^ t)
   (if (pair? t)
        (if (eq? (car t) 'unquote)
            (cadr t)
	    (cons 'cons 
                  (cons (quasiquote^ (car t))
                        (cons (quasiquote^ (cdr t))
			      '()))))
	(cons 'quote (cons t '()))))

(defmacro quasiquote
  (lambda (t)
    (quasiquote^ (cadr t))))
```

Real quasiquote should keep track of nested quotation "levels" but this more primitive version that doesn't that is fine for now.

Now that we have quasiquote, we can use it in defining other macros!

## when and unless

`when` and `unless` are useful short hands to use instead of a "one armed if".

```scheme
(defmacro when
  (lambda (exp)
    (let ((test (cadr exp))
          (body `(begin . ,(cddr exp))))
      `(if ,test
           ,body
           #f))))

(defmacro unless
  (lambda (exp)
    (let ((test (cadr exp))
          (body `(begin . ,(cddr exp))))
      `(if ,test
           #f
           ,body))))
```

## cond

`cond` is one of the most commonly used language constructs, it lets you define conditional expressions - like a sequence of if, else if, else if, else if, end. Since scheme considers #f as false and everything else as a true value, sometimes you condition on a function that returns a useful value. In those cases we can use `=>` to get the value. You can read a precise specification of [cond in r5rs](http://www.schemers.org/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html#%_idx_106).

For the implementation, we first define a few different shapes to classify parts of a COND expression:

```scheme
(go 'cond/0 '(cond))
(go 'cond/else '(cond (else . "else")))
(go 'cond/1 '(cond ("one") . "next"))
(go 'cond/=> '(cond ("test" => "thunk") . "next"))
(go 'cond/clause '(cond ("test" . "rest") . "next"))
```

```scheme
(defmacro cond
  (lambda (exp)
    (if (cond/0? exp)
        `(exit)
        (if (cond/else? exp)
            `(begin . ,(cond/else-get-else exp))
            (if (cond/1? exp)
                `(or ,(cond/1-get-one exp) ,(cond-get-next exp))
                (if (cond/=>? exp)
                    (let ((test (cond/clause-get-test exp))
                          (thunk (cond/=>-get-thunk exp))
                          (tmp (gensym "cond-tmp")))
                      `(let ((,tmp ,test))
                         (if ,tmp
                             (,thunk ,tmp)
                             ,(cond-get-next exp))))
                    (if (cond/clause? exp)
                        (let ((test (cond/clause-get-test exp))
                              (rest (cond/clause-get-rest exp)))
                                 `(if ,test
                                      (begin . ,rest)
                                      ,(cond-get-next exp)))
                        (exit))))))))
```

## mapply

Finally I'd like to introduce `mapply`, this is a new invention! and I've found it very useful in writing compiler passes. Compiler passes are generally functions that take in an AST and recurse on that variable, but they also carry along extra information in function parameters. For language constructs like function application or `begin` that have a whole list of subexpressions `mapply` is the perfect way to do the recursive call!

```scheme
(defmacro mapply
  (lambda (exp)
    ;;(mapply f xs arg ...)
    (let ((f (cadr exp))
          (xs (caddr exp))
          (args (cdddr exp))
          (x (gensym "x")))
      `(map (lambda (,x) (,f ,x . ,args)) ,xs))))
```
