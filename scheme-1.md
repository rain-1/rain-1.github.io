# Scheme Programming Lessons from Tarot: 1 - sequences

## The append problem

```scheme
(define (append x y)
  (if (null? x)
      y
      (cons (car x) (append (cdr x) y))))
```

When you calculate `(append '(a b c) '(1 2 3 4))` the result is `(a b c 1 2 3 4)`. This requires copying the first 3 elements which attach onto the second list. This can become a problem if we use `append` to build up a large list, consider:

```scheme
(define example-a (append '(a) (append '(b) (append '(c) (append '(d) (append '(e) '()))))))
(define example-b (append (append (append (append (append '() '(a)) '(b)) '(c)) '(d)) '(e)))
```

`example-a` is okay, it does 5 copies *O(n)*.

`example-b` is very inefficient, it does 10 copies *O(n^2)*.

In the tarot scheme compiler we need to build up all sorts of long lists in various directions. So we cant allow this sort of quadratic inefficiency to creep in. If we are creating a list of 10 elements we should only do 10 copies at most. 

So how can we achieve this efficiency?

## Sequences

So we created a list-creation DSL. A `seq` data type that represents the list we want to construct, along with a function used at the end to 'bake' it into a real list. This data type has two constructors:

```
<seq> ::= (elt <element>)
        | (cat <seq> ...)
```

we can use it like this:

```scheme
(define example-a (seq->list `(cat (elt a) (cat (elt b) (cat (elt c) (cat (elt d) (cat (elt e) (cat))))))))
(define example-b (seq->list `(cat (cat (cat (cat (cat (cat) (elt a)) (elt b)) (elt c)) (elt d)) (elt e))))
(define example-c (seq->list `(cat (elt a) (elt b) (elt c) (elt d) (elt e)))
(define example-d (seq->list `(cat (cat (elt a) (elt b)) (elt c) (cat (elt d) (elt e)))))
```

The sequence library makes use of scheme's quasiquote operator and this gives us huge flexibility for compiling lists.

## Implementation

To efficiently implement this, we are using *difference lists*. That's a key technique in functional programming where instead of working directly with lists (that end with nil `'()`) you instead work with lists whose tail is left undecided and could be any other list.

```scheme
(define (seq->dlist seq tail)
  (cond ((elt? seq) (cons (elt-get-elt seq) tail))
	((cat? seq) (fold seq->dlist tail (cat-get-seqs seq)))
	(else (error 'seq->dlist "?" seq))))
(define (seq->list seq) (seq->dlist seq '()))
```

Essentially what this recursion does is walk through a seq data structure threading each element together into one final list. Allocating the fewest number of cons cells that it needs to!

## Example

Here's just a snippet of what its use might look like in a program, this is a section of the 'flatten' compiler pass processing an 'if' construct:

```scheme
(define (flatten-exp exp into tail? info stk)
  ...
  ...
  ...
	((if? exp)
	 (let ((test (if-get-test exp))
	       (consequent (if-get-consequent exp))
	       (antecedent (if-get-antecedent exp)))
	   (let ((lbl-skip-then (gensym "skip-then"))
		 (lbl-skip-else (gensym "skip-else")))
	     `(cat ,(flatten-exp test 'reg:acc #f info stk)
		   (elt (branch ,lbl-skip-then))
		   ,(flatten-exp consequent into tail? info stk)
		   (elt (jump ,lbl-skip-else))
		   (elt (label ,lbl-skip-then))
		   ,(flatten-exp antecedent into tail? info stk)
		   (elt (label ,lbl-skip-else))))))
  ...
  ...
  ...)
  ```
  
If you'd like to use seq in your own programs [here's my version](https://notabug.org/rain1/scheme-seq). You're also welcome to reimplement it in your favorite language!

