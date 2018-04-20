# The Flatten and Assemble Passes

`parse` => `desugar` => `hoist` => `denest` => `tmp-alloc` => [flatten.scm](https://github.com/rain-1/tarot-compiler/blob/master/passes/flatte.scm) => [assembler.scm](https://github.com/rain-1/tarot-compiler/blob/master/passes/assembler.scm)

The flatten pass turns hoisted denested code into qcode/bytecode instruction language.

The input language is what comes after the hoist and denest pass: meaning that we are processing a scheme language that has no lambdas, explicit closure creation and setup, only simple non-nested procedure calls, LET and IF (which can nest).

The qcode language is a very simple bytecode type virtual machine.

In terms of data: it has a few registers, global variables, a stack and objects will be allocated in the heap. Everything that can be reached by those roots will be kept by the garbage collector.

For code: the VM simply loops forever executing the next word it sees. There are words to put constant data into registers, move data between registers, jumping, branching (conditional jump), pushing data onto the stack, allocating or manipulation local stack space, assembly style call and ret instructions. Have a look at interpreter.c for full details.

The main things flatten compiler pass does are
* break IF up into parts with labels and conditional branches between them
* break function applications up into a phase where we set up the stack frame
* implement proper tail calls
* lay down metadata about blocks of code for the stack trace printer to use in the case of a crash

To produce these sequences of instructions we use the 'seq' library a lot!

## Proper Tail Calls

How does this phase of the compiler implement proper tail calls?

The `flatten-exp` function is the main function that transforms the input language into the output qcode, it takes parameters for the piece of code that it to compile and also the output register that the code fragments execution result should end up in. It also takes an extra parameter called `tail?` which is #t only when we are in a tail position.

In a normal function application we simply build a new stack frame on top of the current one then use `call` to make the VM unpack the closure in the `clo` register and jump to the subroutines code.

In the special case of a tail call we just emit one extra instruction before the call, a `shiftback` which move the top stack frame down over the current one.


## Assembler

The next pass is the "assembler". this removes all the labels in the qcode, resolving label references to relative numeric values.

It works in two passes:
* pass 1: scan through the code noting the position of every label into a table, removing the label from the code.
* pass 2: scan through the label-free code, resolving every reference to a label using the table constructed in pass 1.


## Example

So we have a concrete example, let's look at this function to get to the end of a list (yes this is a silly function! but working through it demonstrates IF, a regular procedure call and a tail call):

```scheme
(define (end l)
  (if (null? l)
      l
      (end (cdr l))))
```

the flatten pass turns it into this

```
(information site0e0d31ff "end: end.scm")
(information site6ec68664 "end: end.scm")

(label def2e17eca7)          ;; this is just a stub to put the
(stack-grow 1)               ;; end function into the end global variable
(allocate-closure 0 closure3d2dd275)
(clo-set-loc 0)              ;; every definition gets a stub like this
(var-loc 0)
(set-glo end)
(halt)

(label closure3d2dd275)      ;; this is the actual code for END
(stack-grow 1)               ;; allocate space for 1 temporary variable
(stackframe site6ec68664)    ;; get ready to call a function
(var-loc 0)                  ;; lookup the l parameter
(push)                       ;; use it as the first argument in our call
(var-glo null?)              ;; look up the NULL? global variable
(call)                       ;; call the function
(label site6ec68664)         ;; once it returns, we end up here
                             ;; with the result in the acc register
(branch skip-then1af7f0ea)   ;; test the acc reguster (IF/THEN)
(var-loc 0)                  ;; lookup the l parameter
(jump skip-else4da32c7e)     ;; and just right to the end (RET)
(label skip-then1af7f0ea)    ;; (IF/ELSE)
(stackframe site0e0d31ff)    ;; get ready to call a function
(var-loc 0)                  ;; lookup l
(push)                       ;; use it as the first parameter
(var-glo cdr)
(call)                       ;; call (cdr l)
(label site0e0d31ff)
(set-loc 1)                  ;; move the accumulator reg into local stack
(var-loc 1)                  ;; move it back into the accumulator
(push)                       ;; (yes that was a pointless move, a better
(var-glo end)                ;;        optimized compiler wouldn't do it)
(shiftback 1)                ;; tail call
(call)
(label site2fd0ad81)
(label skip-else4da32c7e)
(ret)
```

and then after assembling it, we get

```
(information 39 "end: end.scm" information 22 "end: end.scm" stack-grow
1 allocate-closure 0 7 clo-set-loc 0 var-loc 0 set-glo end halt stack-grow
1 stackframe 6 var-loc 0 push var-glo null? call branch 4 var-loc 0 jump
18 stackframe 6 var-loc 0 push var-glo cdr call set-loc 1 var-loc 1 push
var-glo end shiftback 1 call ret)
```

which can simply be written out into a .q file!
