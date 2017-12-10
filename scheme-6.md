# Scheme Programming Lessons from Tarot: 6 - efficient eval for macros

## Eval

I never use `eval` in my scheme programming but it is a crucial part of implementing macros. We have already seen [the macros](scheme-4) that build up the tarot scheme language but this blog explains how macros (and eval) are implemented in tarot.

Now tarot implements scheme by compiling it to qcode. qcode runs on top of the qcode virtual machine. This allows us to implement `eval` in a trivial way, just compile the code and pass it off to the virtual machine to execute.

Here's the snippet from `compiler.scm`:

```scheme
(define (eval exp)
  (let ((p (vm:open)))
     (compile exp #f p '(halt)))
     (vm:finish p))
```

In previous scheme implementations I didn't have the luxury of a VM to pass code off to for efficient execution, so instead I implemented `eval` by using a metacircular interpreter that would recursively walk the abstract syntax tree - this is very very slow in comparison!

## vm:open and vm:finish

These are functions that the virtual machine exposes to the underlying scheme. It does this by providing them as builtins. `vm/builtins.c`:

```c
	glo_define(intern("vm:open"), mk_numb(0), mk_bltn(bltn_vm_open));
	glo_define(intern("vm:finish"), mk_numb(1), mk_bltn(bltn_vm_finish));
```

The `vm:open` builtin creates a pipe, the `compile` command then compiles our code and writes the qcode into the pipe, finally `vm:finish` closes up the pipe and then loads and executes everything that was written into it:

```c
// VM

scm bltn_vm_open() {
	int fd[2];
	// make a pipe
	info_assert(!pipe(&fd[0]));

	// make a port out of it
	return mk_pipe(fdopen(fd[1], "w"), fdopen(fd[0], "r"));
}

scm bltn_vm_finish() {
	scm p = STACK(0);
	info_assert(scm_gettag(p) == ATOM_PRT);

	FILE *f1, *f2;

	scm *tmp = vm_code + vm_code_size;

	f1 = port_get_file(p);
	f2 = port_get_pipe_end(p);

	fclose(f1);
	load_code(f2);
	fclose(f2);

	// TODO remove it from the table
	// but dont re-close the fds

	vm_exec(tmp);

	return reg_acc;
}
```

and `vm_exec` is the main interpreter loop that [we already saw](scheme-5#interpreter) inside `interpreter.c.`

## Macros

That shows how the tarot compiler is able to implement an efficient `eval`. Now let's look at how macros are implemented.

The compiler processes the input source code by first parsing it then desugaring it then performing more serious compiler passes to crunch the code down into qcode. It is in the `desugar.scm` pass that macros are implemented.

The main purpose of the desugar pass is to insert the implicit `begin` in `lambda` and generally just make the scheme code fit a more uniform style. As it does this it expands all uses of macros.

We have an assoc list (in a mutable box) called `macro-definitions`, for an entry in this assoc table the key is the macro name and the value is the *evaluated* function implementing the expander.

```scheme
(define macro-definitions
  (box '()))

(define (load-macro mac)
  (if (head? 'defmacro mac '())
	(push-box! macro-definitions (cons (cadr mac) (eval (caddr mac))))
	(eval mac)))

(define (macro? exp)
  (and (pair? exp)
       (cond ((assoc (car exp) (unbox macro-definitions)) => cdr)
	     (else #f))))
```

Therefore to apply a macro when we see one, we simply call the expansion function and desugar the result:

```scheme
(define (desugar exp shadow)
  (cond ...

	((macro? exp)
	 => (lambda (expander)
	      (desugar (expander exp) shadow)))

        ...))
```

That's all there is to it!
