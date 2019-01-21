# Lessons from Tarot 7 - pointer tagging and data representation.

Scheme has all sorts of different types of objects (numbers, booleans, cons cells, vectors, file ports) and since it is an untyped/dynamically typed language you can freely mix them and pass them around however you like (including doing it wrong and making your program halt with an error!).

Therefore we need to create some kind of data representation for all these different sorts of objects in our runtime. The particular representation that tarot uses is based on a very clever trick that a friend taught me called pointer tagging!

Every scheme object is implemented in tarot as a 64 bit word - this word will be tagged to say what type of object it is and then the rest of the word will provide the data for that object. Some objects will be pointers into the heap (and the heap will contain the rest of a larger object that can't fit inside a single 64 bit word).

Now here's the math behind this: Since 64 bit is 8 bytes, and the CPU addresses memory at a 1 byte granularity. If all of our pointers are aligned nicely to 8 byte boundaries then their binary representation always end in 3 zeros (because 2^3 = 8). This means we have 3 bits at the very end of any aligned pointer that we can use for our tag!

(N.B. tarot also has "builtins" which are tagged function pointers. It was necessary to compile the builtins with `-falign-functions=8` so that their function addresses were all aligned.)

## Diagram

Here is a diagram showing the particular tagging scheme used. We have more than 8 types of objects, so we actually group all "atomic" objects under the 000 tag, differentiating them further using 3 more tag bits. Pointers need the entire 61 bits of data .

* `0` when the word contains no extra data.
* `#` for numerical data. or in the CHR case a single unicode character/codepoint.
* `*` for pointer data.

(`SYM` = symbol, `BLTN` = builtin function, `STRG` = string, `CLOS` = closure)

```
+----------+-------------------------+
| ATOM FLS | 0 0 ... 0 0 0 0 0 0 0 0 |
| ATOM TRU | 0 0 ... 0 0 1 0 0 0 0 0 |
| ATOM NUL | 0 0 ... 0 0 1 1 0 0 0 0 |
| ATOM SYM | # # ... # # 0 0 1 0 0 0 |
| ATOM CHR | # # ... # # 1 0 1 0 0 0 |
|     NUMB | # # ......... # # 1 0 0 |
|     BLTN | * * ......... * * 1 1 0 |
|     STRG | * * ......... * * 0 0 1 |
|     CONS | * * ......... * * 1 0 1 |
|     VECT | * * ......... * * 0 1 1 |
|     CLOS | * * ......... * * 1 1 1 |
+----------+-------------------------+
```

## The Code

The `objects.h` file in tarot's virtual machine runtime starts like this:

```c
typedef uint64_t scm;
typedef int64_t num;

#define SCM_PTR(p) ((scm)((void*)(p)))
#define PTR_SCM(s) ((void*)(s))

typedef enum {
	TAG_ATOM = 0b000,
	TAG_NUMB = 0b100,
	//UNUSED = 0b010
	TAG_BLTN = 0b110,
	TAG_STRG = 0b001,
	TAG_CONS = 0b101,
	TAG_VECT = 0b011,
	TAG_CLOS = 0b111,
} scm_tag;

typedef enum {
	ATOM_FLS = 0b000000,
	ATOM_TRU = 0b100000,
	ATOM_NUL = 0b010000,
	ATOM_SYM = 0b110000,
	ATOM_CHR = 0b001000,
	ATOM_PRT = 0b101000,
	// 011
	// 111
	//---
	//---
} atom_tag;
```

## Where do pointers point? + gc

Any scheme object that is based on a pointer will point into the heap, to a region of heap memory that has a header then "raw" data, then "scheme" data. Everything is done in multiples of 64 bit words.

The header is a single 64 bit word which has 3 fields: the color (white or black) which is used by the garbage collector. the "raw size" (how many 64 bit words of raw data this object has). and the "scm size" how many scheme objects the object contains.

For example:

* A cons cell points into a heap that has raw size 0 and scm size 2.
* A string with 'n' bytes is rounded up to a multiple of 64 bit words plus one nul byte, and has scm size 0 becaues it contains no child objects.
* A vector has raw size 0 and scm size 'n'. Where n is the vector length.
* A closure is a mixed object containing both: it has raw size 1 (and this one raw object is a pointer to the address or label of the compiled bytecode for this procedure) and scm size 'n' where n is the number of environment variables closed over by that procedure.

The garbage collector is able to process the heap objects based only on the header, it doesn't need to worry about what exact type the objects are - it just colors in the header, skips over the raw data and recursively walks each of the scheme objects:

```c
void mark_object(scm obj) {
  scm *p;
  scm hdr;

  scm raw_size, scm_size;

  int i;
  if (scm_isptr(obj)) {
    p = (scm*)(obj & ~0b111);
    hdr = *p;

    if(get_hdr_color(hdr) == HDR_BLACK)
	    return;
    
    *p = set_hdr_color(hdr, HDR_BLACK);
    
    raw_size = get_hdr_raw_size(hdr);
    scm_size = get_hdr_scm_size(hdr);
    
    for(i = 0; i < scm_size; i++) {
      mark_object(p[1 + raw_size + i]);
    }
  }
}
```

## Conclusion

Overall this system seems to work quite well for implementing the runtime of our scheme system.

The tagging system is very nice and compact, letting us use a single word everywhere to represent scheme objects. The uniform system of headers makes tracing the heap very easy without lots of cases/branches.

I feel that I would like a much more efficient garbage collection algorithm than mark and sweep. It doesn't seem to work with if you have a very large amount of data at play - for this I think I need a generational gc. I'm learning about that now and hope to implement it in future.
