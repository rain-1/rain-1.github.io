# Bootstrapping relevant OS's

## a-Linux

* [http://asm.sourceforge.net/asmutils.html](http://asm.sourceforge.net/asmutils.html)

![a-Linux](a-linux.png)

a-Linux is a linux distribution implemented entirely in assembly, except for the linux kernel itself. It has a reasonably complete set of the standard userspace programs. This could be a good minimal platform to build things on, to ensure you are not making any implicit assumptions about C or libc. 42k lines of asm. GPL2.

## asmc

* [https://gitlab.com/giomasce/asmc](https://gitlab.com/giomasce/asmc)

![asmc](asmc.png)

Built up from assembly, a series of programming languages are used to create a bootable kernel that runs an interpreter or compiler in order to build proceeding stages. Very promising! 9 asm, 11k g, 3k c. GPL3.

## MenuetOS/KolibriOS

* [https://kolibrios.org/en/](https://kolibrios.org/en/)

![kolibrios](kolibrios.png)

## LFS

![lfs](lfs.png)

This is actually a book that explains how to build a detatched gcc compiler toolchain and then use it to build a complete linux distribution entirely from scratch. This process will be very important if one wishes to go from a non-linux OS that can run GCC to a linux OS. millions of lines of code. Primarily GPL.

## MikeOS

* [http://mikeos.sourceforge.net/](http://mikeos.sourceforge.net/)

![mikeos](mikeos.png)

A ground up OS written entirely in assembly. includes an addon 'c library' implemented in asm, allowing one to write in C! 16 lines of asm. BSD like license, don't use the term MikeOS for forks.

## Oberon OS

![oberon](oberon.png)

TODO.

## Sortix

* [https://sortix.org/](https://sortix.org/)

![sortix](sortix.png)

Sortix is a complete implementation of a POSIX OS from scratch in C (kernel in C++). GCC can be used on it. 169k lines of code. ISC License.

## ToaruOS

* [https://gitlab.com/toaruos/toaru-nih](https://gitlab.com/toaruos/toaru-nih)
* [https://toaruos.org/](https://toaruos.org/)
* [https://gitlab.com/toaruos/misaka](https://gitlab.com/toaruos/misaka)

![toaruos](toaruos.png)

ToaruOS is very complete with graphics, networking, package manager. It can run python. Implemented in C. 55k lines of code. University of Illinois/NCSA Open Source License.

## Xv6

* [https://pdos.csail.mit.edu/6.828/2017/xv6.html](https://pdos.csail.mit.edu/6.828/2017/xv6.html)
* [https://github.com/mit-pdos/xv6-public](https://github.com/mit-pdos/xv6-public)

![xv6](xv6.png)

This is a very basic UNIX OS implemented in C. It was developed for the purpose of teaching. The code is clear and it doesn't have rough edges. 8k lines of code. MIT License.

## Xinu

* [https://xinu.cs.purdue.edu/](https://xinu.cs.purdue.edu/)

![xinu](xinu.png)

> XINU stands for Xinu Is Not Unix -- although it shares concepts and even names with Unix, the internal design differs completely. Xinu is a small, elegant operating system that supports dynamic process creation, dynamic memory allocation, network communication, local and remote file systems, a shell, and device-independent I/O functions. The small size makes Xinu suitable for embedded environments.

22k lines of C. Unlicensed(?).
