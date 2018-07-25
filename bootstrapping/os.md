# Bootstrapping relevant OS's

## a-Linux

* http://asm.sourceforge.net/asmutils.html

![a-Linux](a-linux.png)

a-Linux is a linux distribution implemented entirely in assembly, except for the linux kernel itself. It has a reasonably complete set of the standard userspace programs. This could be a good minimal platform to build things on, to ensure you are not making any implicit assumptions about C or libc. 42k lines of asm. GPL2.

## sortix

* https://sortix.org/

![sortix](sortix.png)

Sortix is a complete implementation of a POSIX OS from scratch in C (kernel in C++). GCC can be used on it. 169k lines of code. ISC License.

## ToaruOS

* https://gitlab.com/toaruos/toaru-nih
* https://toaruos.org/
* https://gitlab.com/toaruos/misaka

![toaruos](toaruos.png)

ToaruOS is very complete with graphics, networking, package manager. It can run python. Implemented in C. 55k lines of code. University of Illinois/NCSA Open Source License.

## Xv6

* https://pdos.csail.mit.edu/6.828/2017/xv6.html
* https://github.com/mit-pdos/xv6-public

![xv6](xv6.png)

This is a very basic UNIX OS implemented in C. It was developed for the purpose of teaching. The code is clear and it doesn't have rough edges. 8k lines of code. MIT License.
