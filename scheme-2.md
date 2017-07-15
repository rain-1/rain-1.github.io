# Scheme Programming Lessons from Tarot: 2 - accessors

## Accessors

All of the compiler passes classify and take apart various structures. Originally I wanted to do this with pattern matching, so I added a pattern matching macro to scheme. Later on I found a much simpler way to do it: generate test and accessor functions:

You describe the shape of your data objects like this:

```scheme
(go 'if '(if "test" "consequent" "antecedent"))
(go 'lambda '(lambda "vars" "body"))
(go 'begin '(begin . "statements"))
```

and then the `go` function produces scheme code like this:

```scheme
(define (if? exp)
  (and (pair? exp)
       (eq? 'if (car exp))
       (pair? (cdr exp))
       (pair? (cddr exp))
       (pair? (cdddr exp))
       (null? (cddddr exp))))
(define (if-get-test exp) (cadr exp))
(define (if-get-consequent exp) (caddr exp))
(define (if-get-antecedent exp) (cadddr exp))
(define (lambda? exp)
  (and (pair? exp)
       (eq? 'lambda (car exp))
       (pair? (cdr exp))
       (pair? (cddr exp))
       (null? (cdddr exp))))
(define (lambda-get-vars exp) (cadr exp))
(define (lambda-get-body exp) (caddr exp))
(define (begin? exp)
  (and (pair? exp) (eq? 'begin (car exp))))
(define (begin-get-statements exp) (cdr exp))
```

Now here's an example of it in use:

```scheme
(define (hoist exp scope stk filename)
  ...
  ...
	((if? exp)
	 `(if ,(hoist (if-get-test exp) scope stk filename)
	      ,(hoist (if-get-consequent exp) (clone-scope scope) stk filename)
	      ,(hoist (if-get-antecedent exp) (clone-scope scope) stk filename)))

	((lambda? exp)
	 (let ((vars (lambda-get-vars exp))
	       (body (lambda-get-body exp)))
            ...))
  
	((begin? exp)
	 `(begin . ,(mapply hoist (cdr exp) scope stk filename)))
   ...
   ...)
```

It's very easy to work with and there is much less code in total as well as less work for the compiler to do when processing this code (since it's just simple function calls in a cond rather than a whole pattern match system that needs expanded).

and [here](https://notabug.org/rain1/scheme-accessors/src/master/accessors.scm) is the script that implements the code-generator!
