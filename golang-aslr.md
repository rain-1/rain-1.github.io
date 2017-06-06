# Golang ASLR

One of the reasons I like using golang is that I feel capable of writing a secure webapp with it because it provides langsec benefits like [context aware HTML templating](https://golang.org/pkg/html/template/) to stop XSS. If the web application logic itself is secure then the next place to worry about is the level below that: how the compiler and runtime implements the code you wrote. (Another would be any external helper binaries that the web apps calls, e.g. imagemagick tools. This is out of scope of this article).

One would like to say "Golang is a memory safe language therefore any code written in it is secure against things like heap and buffer overflows." but mistakes can happen in implementing something as complex as a programming language. For example [chicken scheme](http://www.cvedetails.com/product/26314/Call-cc-Chicken.html?vendor_id=12910) has had some interesting vulnerabilities like this. It is also common to link C libraries to golang. The google golang implementation has only had [one CVE relating to certificate checking](http://www.cvedetails.com/vendor/14185/Golang.html), but a race has been used for [heap corruption](https://blog.stalkr.net/2015/04/golang-data-races-to-break-memory-safety.html).

In a perfect world your compiled golang code could not be corrupted and exploited, but this is not a perfect world. One of the best protections is ASLR (address randomization), modern C programs on linux are all compiled with this which increases security significantly by protecting against code reuse attacks. With ASLR an attacker does not know where code is so cannot easily reuse it.

# A no-go: seat belts at restaraunts.

As Alex Reece pointed out, googles golang implementation not only does not support ASLR but explicity rejects it:

> Once you have an arbitrary write in go, it is really easy to get arbitrary code execution. We put a function pointer in our data segment (we wanted to put it in the heap, but that didn't work on 64bit Go - apparently the size of a struct is limited to 32 bits. Luckily, the data segment is in the lower 32 bits) and change it to point to our shell code using the arbitrary write. Since Go has no randomization at all, this is as simple as running the program twice.

From [Exploiting a Go Binary](http://codearcana.com/posts/2013/04/23/exploiting-a-go-binary.html).

There is a mailing list discussion about this [here](https://github.com/golang/go/issues/14327).

> (Russ Cox) Address space randomization is an OS-level workaround for a language-level problem, namely that simple C programs tend to be full of exploitable buffer overflows.  Go fixes this at the language level, with bounds-checked arrays and slices and no dangling pointers, which makes the OS-level workaround much less important.  In return, we receive the incredible debuggability of deterministic address space layout.  I would not give that up lightly. 

> (Thomas Bushnell, BSG) You're engaging in a common mistake: knowing that it is important to have seat belts on cars, because of the risks that cars are prone to, you're now arguing for seat belts in restaurants (in case a car should crash into the restaurant), seat belts in trains (even though the risk/reward ratio is entirely different), seat belts on bicycles, and so forth.

# gccgo to the rescue!

Ian Lance Taylor, author of the GOLD linker also wrote the GCC implementation of golang. **It supports ASLR**: [Go Execution Modes - Ian Lance Taylor](https://docs.google.com/document/d/1nr-TQHw_er6GOQRsF6T43GGhFDelrAP0NqSS_00RgZQ/). So for security relevant programs like web applications I would recommend using it with the `-fPIC -pie' flags.

I used the following test program to verify the impact of these flags:

```
package main

import "fmt"
import "reflect"

func FuncPtrAddr(i interface {}) {
        fmt.Printf("%p\n",reflect.ValueOf(i).Pointer())
}

func main() {
        i := int(42)
        fmt.Printf("%p\n", &i)
        FuncPtrAddr(main)
        FuncPtrAddr(FuncPtrAddr)
        FuncPtrAddr(fmt.Printf)
}
```

* Running this with googles golang prints the same values every single time. Completely static binary.
* Running this with default gcc only randomizes the Printf function pointer.
* Running this compiled with the `-fPIC -pie' results in all addresses except the stack being randomized.

Overall golang is a very secure language, the google implementation is solid: [Looking for security trouble spots in Go code](http://0xdabbad00.com/2015/04/12/looking_for_security_trouble_spots_in_go_code/), but ASLR is a very important mitigation to protect against code reuse attacks and we can benefit from it by using the gcc golang implementation.

Thank you and happy hacking!
