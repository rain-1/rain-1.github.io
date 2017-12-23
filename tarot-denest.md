# The Denest Pass

`parse` => `desugar` => [hoist.scm](https://notabug.org/rain1/tarrochi/src/master/compiler/passes/hoist.scm) => [denest.scm](https://notabug.org/rain1/tarrochi/src/master/compiler/passes/denest.scm) => `tmp-alloc` => `flatten` => `assemble`

The next step of compiliation is denesting. Scheme allows expressions like function application and even `IF` to be nested freely, meaning you can write `(+ 1 (if ...))` but in lower level languages like C if is a statement not an expression. That's why this compilation pass pulls every nested expression apart into a linear sequence of binding statements.

## Example 1

A good example of a deeply nested expression is

```scheme
(define l '(1 2 3 4 5))
```

which desugars to

```scheme
(define "tests/scm/12.scm" l
  (app cons (datum 1)
            (app cons (datum 2)
	              (app cons (datum 3)
		                (app cons (datum 4)
				          (app cons (datum 5) (datum ())))))))
```

and then (hoisting doesn't change it much) the result of denest is:

```scheme
((def3d2dd275 "l: tests/scm/12.scm" l 0
   (let ((tmp4da32c7e (app (var glo cons) (datum 5) (datum ())))
	 (tmp6ec68664 (app (var glo cons) (datum 4) (var tmp tmp4da32c7e)))
	 (tmp0e0d31ff (app (var glo cons) (datum 3) (var tmp tmp6ec68664)))
	 (tmp2fd0ad81 (app (var glo cons) (datum 2) (var tmp tmp0e0d31ff))))
     (app (var glo cons) (datum 1) (var tmp tmp2fd0ad81))))
```

As you can see the innermost function application is executed first. (Note: Internal to tarot we use `LET` to refer to `LET*`) and each temporary expression is given a name.

## BNF

So the denest pass turns scheme expressions into a big long `LET` expression where function applications are always "simple". This is similar to ANF/A-normal form from compilers which you can read more about here: [matt.might.net/articles/a-normalization](https://matt.might.net/articles/a-normalization/)

A rough BNF of the output format of denest is as follows:

```
<let> ::= (let ((<var>|#f <expr>) ...) <expr>)

<expr> ::= <basic>
         | (app <simple> ...)
         | (if <expr> <let> <let>)
	 | (allocate-closure <size> <label>)
	 | (set-closure! <var> <index> <simple>)

thu closure stuff is used closure AND for letrec

<simple> ::= <variable>
   	   | <atomic-data>
```

## LET-objects

To implement this we have created a LET-object type. LET-objects can be joined together and merged and manipulated in various ways. A LET-object has a sequence of bindings called its "table" and it has its "body" too.

To denest a function application we denest all its part then join all those resulting LETs into one big sequence. similarly, to denest a let we denest each binding of the let table and join the resulting LETs into one long one.


## Example 2

I also wanted to go over and example where `if` is used in an expression context:

```scheme
(print
 (fold (lambda (x tot)
 	(+ tot (if (or^ (zero? (remainder x 3)) (zero? (remainder x 5))) x 0)))
       0
       (iota 1000)))
```

after hoist we have the following form:

``scheme
(closure0e0d31ff "tests/scm/rosetta-sum35-b.scm" #f 2
		 (app (var glo +) (var loc 1)
		      (if (app (var glo boolean-or) (app (var glo zero?) (app (var glo remainder) (var loc 0) (datum 3)))
			       (app (var glo zero?) (app (var glo remainder) (var loc 0) (datum 5))))
			  (var loc 0)
			  (datum 0))))
```

and denesting does this to it:

```scheme
(closure0e0d31ff "tests/scm/rosetta-sum35-b.scm" #f 2
		 (let ((tmp47caa567 (app (var glo remainder) (var loc 0) (datum 3)))
		       (tmp48249dbf (app (var glo zero?) (var tmp tmp47caa567)))
		       (tmp1f0e5d0d (app (var glo remainder) (var loc 0) (datum 5)))
		       (tmp026baae9 (app (var glo zero?) (var tmp tmp1f0e5d0d)))
		       (tmp2c02fe8c (if (app (var glo boolean-or) (var tmp tmp48249dbf) (var tmp tmp026baae9))
					(let () (var loc 0))
					(let () (datum 0)))))
		   (app (var glo +) (var loc 1) (var tmp tmp2c02fe8c))))
```

You can see that the `(+ 1 ..)` part is at the very end because it's the last thing that happens. 
You can also note that some calculations are done before the `IF` condition, in order to make the `IF` condition a simple application. Finally the two branches of a conditional will contain their own `LET` expressions.

## Summary

In summary the denest pass is about making every function application "simple" by transforming nested code into linear sequenced statements. This is a very general sort of compiler pass that may be useful for any 'expression' based language to target a 'statement' based langauge like C, bytecode or assembly.

