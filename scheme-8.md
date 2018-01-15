# Lessons from Tarot 8 - Information for stack traces and profiling

Throughout the whole compile pipeline we keep a small piece of information about the expressions we are compiling. Just the name of the file it was in and the function it came from. I have found that (at least) this is crucial for doing programming. You are going to encounter a lot of crashes and errors and it's essential to know where it came from.

So this information is used by tarot to create our stack traces.

## stack trace example

```scheme
(define (increment n) (+ n 1))
(define (main)
  (map increment '(4 65 9 foo 72)))
(main)
```

```
=======================
STACK TRACE
increment: a.scm
map: build/compiler/std/lists.scm
map: build/compiler/std/lists.scm
map: build/compiler/std/lists.scm
map: build/compiler/std/lists.scm
a.scm
***
=======================

Assertion failed: scm_gettag(p) == TAG_NUMB (builtins.c: bltn_add: 214)
Aborted
```

As you can see, the crash happened inside increment and map is not tail recursive!

## Profiling example

I have been slowly getting some of the larceny scheme benchmark programs to run in tarot. I was particularly surprised by the lattice count benchmark, it was 140 seconds in tarot compared to 0.3 seconds in chez. I used the profiler on it to find out that all the time was being spent in the comparator function and member. That pointed me to my very inefficient CASE implementation - when I fixed that it run in about 25 seconds. I hope it speed it more too.

Here's what the profiler output looks like:

```
(l2 3)
(l2-l3 6)
(l3 10)
(l3-l2 4)
(l4 120549)
125121
(took 125 seconds and 121 ms)
0 - allocate-closure?: build/compiler/gen/shapes.scm
0 - special:open: build/compiler/passes/parser.scm
1 - maps: ./tests/benchmarks/lattice.scm
3 - concat-map: ./tests/benchmarks/lattice.scm
786 - count-maps: ./tests/benchmarks/lattice.scm
34762 - sum: ./tests/benchmarks/lattice.scm
72469 - maps-rest: ./tests/benchmarks/lattice.scm
156371 - reverse!: ./tests/benchmarks/lattice.scm
198207 - zulu-select: ./tests/benchmarks/lattice.scm
380241 - select-map: ./tests/benchmarks/lattice.scm
449224 - map-and: ./tests/benchmarks/lattice.scm
717334 - maps-1: ./tests/benchmarks/lattice.scm
7432437 - run: ./tests/benchmarks/lattice.scm
12220816 - lexico: ./tests/benchmarks/lattice.scm
```

When you compile the virtual machine you can pass in a flag to create the version with profiling enabled that's a little slower but gives you all that information at the end of a run.

# Compiling

The file & function name information are placed into the compiled qcode output by this function from the 'flatten' pass:

```scheme
(define (flatten-toplevel top ender tail? stk)
  (let* ((nm (car top))
         (info (cadr top))
         (target (caddr top))
         (tmps (cadddr top))
         (exp (cadddr (cdr top)))
         (nm-end (gensym "info-end")))
    (stack-push! stk `(elt (information ,nm ,nm-end ,info))) ;; Emit an 'information' opcode that
    `(cat (elt (label ,nm))                                  ;; knows where the start
          ,(if (= 0 tmps)
               '(cat)
               `(elt (stack-grow ,tmps)))
          ,(flatten-exp exp 'reg:acc tail? info stk)
          (elt (label ,nm-end))                              ;; and end of the compiled function is.
          ,(if target
               `(elt (set-glo ,target))
               '(cat))
          ,ender)))
```

So qcode files start with a big chunk of information opcodes that describes each function that will be loaded.

```
information^@2194^@2273^@assoc: build/compiler/std/lists.scm^@information^@2139^@2189^@
length: build/compiler/std/lists.scm^@information^@2109^@2134^@length: build/compiler/s
td/lists.scm^@information^@2035^@2104^@map: build/compiler/std/lists.scm^@information^@
1900^@2030^@map/2: build/compiler/std/lists.scm^@information^@1839^@1895^@for-each: bui
ld/compiler/std/lists.scm^@information^@1771^@1834^@foldl: build/compiler/std/lists.scm
^@information^@1706^@1766^@append: build/compiler/std/lists.scm^@information^@1641^@170
1^@revappend: build/compiler/std/lists.scm^@information^@1626^@1636^@reverse: build/com
piler/std/lists.scm^@information^@1519^@1621^@filter: build/compiler/std/lists.scm^@inf
ormation^@1445^@1514^@concat-map: build/compiler/std/lists.scm^@
```

The opcodes do not do anything at execution time, they are only used by the loader:

```c
if(!strcmp("information",w)) {
    vm_add_codeword(CODE_INFORMATION);
    sym = load_number(fptr);
    sym2 = load_number(fptr);
    info = load_string(fptr);
    information_store(vm_code + vm_code_size + sym,
                      vm_code + vm_code_size + sym2,
                      info);
    continue;
}
```

Each information entry is a start address, end address and string describing the code block. The information manager inside data.c maintains these entries in a sorted linked list. `information_store` just merges a new entry into the list.

Then it has a `information_lookup` function that takes in any address and searches through the list to find out which code block contains it. When it finds one it returns the description.

## How it's used for stack traces

This is used for printing stack traces when a crash happens. We have a function inside vm/objects.h called `info_assert`. It checks a condition (Generally it used to check things like "Is this object a pair?" inside the car and cdr builtins) and when the condition fails it will call the stack trace function and exit.

To output a stack trace we walk down the stack printing out the information for each return address that it saved on there!

## How it's used for the profiler

To build our profiling information we have a hash table mapping information strings to a counter. Every 300 VM instruction ticks we check look up the function we are currently in using `information_lookup` and increment its counter. At the end we can just print out the counters in order to see where the majority of time is spent when executing a program. Thanks to hackerfoo from proglangdesign for telling me about this easy way to make a profiler!

# Conclusion

It's not enough to just implement an interpreter or compiler that correctly implements your programming language on valid inputs. Getting useful information on errors is essential for debugging and getting good profiling is crucial for both optimizing your own programs and adding compiler optimizations to the system.

Having the virtual machine's execution split into a loading and then execution step was perfect for loading in all the information before starting the program. Originally the format for information was listed for every return address that occurs in compiler output, but I changed it to ranges and while this was a little harder to work with it gives more precise information.
