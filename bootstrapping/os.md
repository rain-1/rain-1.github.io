# Bootstrapping relevant OS's

## a-Linux

* http://asm.sourceforge.net/asmutils.html

![a-Linux](a-linux.png)

a-Linux is a linux distribution implemented entirely in assembly, except for the linux kernel itself. It has a reasonably complete set of the standard userspace programs. This could be a good minimal platform to build things on, to ensure you are not making any implicit assumptions about C or libc. 42k lines of asm. GPL2.

## sortix

* https://sortix.org/

![sortix](sortix.png)

Sortix is a complete implementation of a POSIX OS from scratch in C (kernel in C++). GCC can be used on it. 169k lines of code. ISC License.
