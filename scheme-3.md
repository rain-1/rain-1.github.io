# Scheme Programming Lessons from Tarot: 3 - stacks and queues

You might be surprised that something simple like a compiler needs advanced data structures like stacks and queues, but they really are essential tools throughout the whole program.

Reminder that a stack is an object that you can push objects onto the front and pop objects off of the front. (first in, first out). A queue is more like a tube, you can push objects into it and get them out the other end (last in, first out).

Contrary to common knowledge, programming with mutation is a really useful and valid technique. The difference between a stack and simply consing an object onto a list in a referentially transparent way is that the stack is conceptually one object that procedures can have effects on and you get your results out at the end. No hassle about multiple return values or tying your code in knots monadically threading objects through your code.

## Usage

How are stacks and queues used in the tarot implementation?

### stacks

* The desugar pass maintains a stack of filenames include'd (our `(include "file.scm")` is a lot like C `#include "file.c"`), so that the same file is not included twice.
* Desugar also pushes metadata information about each toplevel expression (the definition name, file it was found in and an estimation of its arity) that gets compiled onto a stack for later passes to use.
* The hoist pass uses two stacks. One to keep track of "temporaries" (local variables) and one to push the code for each closure converted lambda to process more later.
* The flatten pass pushes metadata about each function it compiles while it flattens code out. This metadata is used by the runtime to print stack traces on a crash.
* The assembler processes a sequence of instruction. on the first pass it removes each label from the code pushing it and it's place into a stack, on the second pass it uses the label places to make relative jumps and addresses.

### queues

* Hoisting uses a queue to both compile a list of the variables captured by a closure, and since a queue keeps things in order it is also used to numberize them (turn them from a named variable into an array index)
* Hoisting also emits each toplevel expression it compiles into a queue, this means the next compiler pass gets the toplevel expressions in order.
* The pass to allocate temporary stack slots to each local variable works by just adding each new variable onto a queue, and at the end requesting that many stack slots for locals. (In future something like graph coloring/lifetime analysis or a linear scan allocator could be used to get more efficient compiled code)
* The tokenizer reads characters as input and emits tokens as output into a queue.A bit like stream processing. Similarly the parser reads its tokens off the queue producing parsed s-expressions.

## Implementation

Here's the code for a stack, it's implemented is a list in a box. We can mutate the contents of the box to change the stack.

```scheme
(define (empty-stack) (box '()))

(define (stack-push! s v) (set-box! s (cons v (unbox s))))

(define (stack-pop! s)
  (let ((stk (unbox s)))
    (if (null? stk)
        (error 'stack-pop/null 0 0)
        (begin
          (set-box! s (cdr stk))
          (car stk)))))
```

Here's the code for a queue, the implementation of a queue has a bit more finesse than the stack.

It's implemented as a vector containing a tag (queue) and 2 elements `top` and `bot`. `top` is simply a list that we can pop things off. `bot` is the very last cons cell of the list and we use `set-cdr!` on it to make it longer each time to push an element into the queue.

```scheme
(define (empty-queue) (list->vector (list 'queue '() #f)))

(define (queue:top q) (vector-ref q 1))
(define (queue:bot q) (vector-ref q 2))
(define (queue:top! q v) (vector-set! q 1 v))
(define (queue:bot! q v) (vector-set! q 2 v))

(define (queue-push! q v)
  (if (queue:bot q)
      (begin (set-cdr! (queue:bot q) (list v))
             (queue:bot! q (cdr (queue:bot q))))
      (begin (queue:top! q (list v))
             (queue:bot! q (queue:top q)))))

(define (queue-pop! q)
  (let ((top (queue:top q)))
    (if (null? top)
        (error 'queue-pop! 0 0)
        (begin
          (queue:top! q (cdr top))
          (when (null? (cdr top))
            (queue:bot! q #f))
          (car top)))))
```

Here is a little REPL session with the queue, to help understand how it is implemented:

```scheme
> (define q (empty-queue))
> (queue-push! q 1)
> q
#(queue (1) (1))
> (queue-push! q 2)
> q
#(queue (1 2) (2))
> (queue-push! q 3)
> q
#(queue (1 2 3) (3))
> (queue-pop! q)
1
> q
#(queue (2 3) (3))
> (queue-pop! q)
2
> q
#(queue (3) (3))
> (queue-pop! q)
3
> q
#(queue () #f)
```

I hope that all made sense and the value of stacks and queues became clear. assoc lists are also extremely useful throughout. You can get the stack and queue code [here](https://notabug.org/rain1/scheme-stackqueue) or implement your own.

