# How do you import SRFI-1 (the list library) in scheme?

It depends...


## Scheme implementations checked

* chicken `(import srfi-1)`
* chibi `(import (srfi 1))`
* guile `(use-modules (srfi srfi-1))`
* sagittarius `(import (srfi :1 lists))`
* chez N/A
* larceny `(import (srfi 1 lists))`
* racket `(require srfi/1)`
* mit N/A
* bigloo `(module ex (library srfi1))`
* gauche `(use srfi-1)`
* gambit N/A
* ironscheme `(import (srfi :1))`
* tinyscheme `(load "srfi-1-reference.scm")`

# chicken
```
#;1> (import srfi-1)
; loading ./srfi-1.import.so ...
```

# chibi
```
> (import (srfi 1))
> 
```

# guile
```
scheme@(guile-user)> (use-modules (srfi srfi-1))
scheme@(guile-user)> 
```

# sagittarius
```
sash> debbie@debian:~/schemes/sagittarius-0.9.1/build/out$ LD_LIBRARY_PATH=./usr/local/lib/sagittarius/0.9.1/x86_64-pc-linux/ ./usr/local/bin/sagittarius -L ./usr/local/share/sagittarius/0.9.1/lib/ -L ./usr/locae/sagittarius/0.9.1/sitelib/
sash> (import (srfi :1 lists))
#<unspecified>
sash>
```

# chez scheme
does not have it
https://srfi.schemers.org/srfi-1/mail-archive/msg00018.html
https://github.com/arcfide/chez-srfi/tree/master/%253a1 has packaged the reference implementation

# larceny
```
(import (srfi 1 lists))
```

# racket
* http://docs.racket-lang.org/srfi/index.html
```
(require srfi/1)
```

# mit scheme
there doesn't seem to be any way to actually import or load a libary. only files. srfi-1 is already loaded by default when you start mit scheme.
```
$ ./bin/scheme 
MIT/GNU Scheme running under GNU/Linux
Type `^C' (control-C) followed by `H' to obtain information about interrupts.

Copyright (C) 2014 Massachusetts Institute of Technology
This is free software; see the source for copying conditions. There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Image saved on Saturday May 17, 2014 at 2:39:25 AM
  Release 9.2 || Microcode 15.3 || Runtime 15.7 || SF 4.41 || LIAR/x86-64 4.118 || Edwin 3.116

1 ]=> lset-intersection!

;Value 13: #[compiled-procedure 13 ("srfi-1" #x39) #x1a #x34e292]
```

# bigloo
The SRFI 1 is implemented as a Bigloo library. Hence, in order to use the functions it provides, a module must import it.
```
(module ex
   (library srfi1))

(print (find-tail even? '(3 1 37 -8 -5 0 0)))
 => '(-8 -5 0 0))
```

# gauche
```
$ gosh
gosh> (use srfi-1)
gosh> lset-intersection!
#<closure (lset-intersection! = lis1 . lists)>
```

# gambit
There is some kind of module system implementation called 'black hole' that brings srfi-1 with it, I couldn't figure out how to use it. The website is down.

# ironscheme
```
(import (srfi :1))
```

# tinyscheme
```
$ wget https://srfi.schemers.org/srfi-1/srfi-1-reference.scm
HTTP request sent, awaiting response... 200 OK
Length: 55173 (54K) [application/octet-stream]
Saving srfi-1-reference.

srfi-1-reference.scm                                 100%[=====================================================================================================================>]  53.88K  --.-KB/s    in 0.1s    

2018-04-25 23:23:14 (392 KB/srfi-1-reference. saved [55173/55173]

$ ./scheme 
TinyScheme 1.41
ts> (load "srfi-1-reference.scm")
Loading srfi-1-reference.scm
#<EOF>
ts> lset-intersection!
#<CLOSURE>
```
