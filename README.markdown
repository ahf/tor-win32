# Cross Compiling Tor for Windows

This document aims to describe how to get a working toolchain for cross
compiling Tor for Windows on a Linux host. This was all tested in an x86-64
Debian Testing VM running on QubesOS. It is worth noting that cross compiling
Tor for Windows seems much slower than doing native compilation, but I have not
tried to understand why this is the case. Running Tor in Wine will also appear
more sluggish than when running Tor natively. Especially the bootstrap phase
seems slow (possibly due to some Wine bootup phase somewhere?).

And yes, all of this is slightly crazy. So much for avoiding a Windows
installation :-)

## Setting up the cross compilation environment

To begin with we must install the packages that are needed to setup the MinGW
32-bit Windows cross compiler environment.

The Debian package maintainers have been nice to us and made packages for the
zlib library where the library have already been cross compiled for 32-bit
Windows. Install the packages using:

    # apt install libz-mingw-w64 libz-mingw-w64-dev

### Building Tor

This Git repository contains a `Makefile` which includes rules for downloading
and building cross compiled versions of the dependencies of Tor that is needed
to run a minimal version of Tor. This includes `libevent` and `openssl`. We
currently do not enable Zstd and LZMA compression support in the cross compiled
version of Tor that are build using this method, but please submit a pull
request to this repository if you add support for additional feature
dependencies in the `Makefile`.

To fetch and build the dependencies of Tor and Tor itself run:

    $ git clone https://github.com/ahf/tor-win32.git
    $ cd tor-win32
    $ make

## Running Tor under Wine

Since we are running on an x86-64 VM in 64-bit mode we must ensure that Debian
can install 32-bit x86 packages as well. You can read more about the Multiarch
concept on the [Debian wiki](https://wiki.debian.org/Multiarch).

    # dpkg --add-architecture i386
    # apt update

Install the 32-bit version of Wine using:

    # apt install wine32

## Authors

- Alexander Færøy (<ahf@torproject.org>).
