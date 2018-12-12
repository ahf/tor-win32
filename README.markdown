# Cross Compiling Tor for Windows [![Build Status](https://travis-ci.org/ahf/tor-win32.svg?branch=master)](https://travis-ci.org/ahf/tor-win32)

<img width="33%" align="right" src="https://raw.githubusercontent.com/ahf/tor-win32/master/brain.jpg" />

This document aims to describe how to get a working toolchain for cross
compiling Tor for Windows on a Linux host. All of this was tested on an x86-64
Debian Testing VM running on QubesOS. It is worth noting that cross compiling
Tor for Windows seems much slower than doing native compilation, but I have not
tried to understand why this is the case. Running Tor in Wine will also appear
more sluggish than when running Tor natively. Especially the bootstrap phase
seems slow (possibly due to some Wine bootup phase somewhere?).

And yes, all of this is slightly crazy. So much for avoiding a Windows
installation :-)

## Continuous Integration with Travis CI

This repository contains a `.travis.yml` file which will try to
cross compile Tor for both 32-bit and 64-bit Windows using MinGW
on Ubuntu Xenial.

Additionally the Travis CI configuration for this repository contains a cron
job, which is executed once every 24 hour even when there are no new commits on
the repository itself. This allows us to use this repository as a very cheap
daily indicator on whether Tor is currently able to be cross compiled using the
method described below.

You can see the most recent build status of this repository at
https://travis-ci.org/ahf/tor-win32

## Setting up the cross compilation environment

To begin with we must install the packages that are needed to setup the MinGW
32-bit Windows cross compiler environment:

    $ sudo apt install gcc-mingw-w64-i686

If you are more interested in building Tor for 64-bit Windows you should
install the MinGW64 x86-64 cross compiler environment instead using:

    $ sudo apt install gcc-mingw-w64-x86-64

To build Tor directly from Git we need to install some of the autotools
packages. Install these using:

    $ sudo apt install autoconf automake libtool

The Debian package maintainers have been nice to us and made packages for the
zlib library where the library have already been cross compiled for both 32-bit
and 64-bit Windows. Install the packages using:

    $ sudo apt install libz-mingw-w64-dev

### Building Tor

This Git repository contains a `Makefile` which contains rules for downloading
and building cross compiled versions of the dependencies of Tor. This includes
`libevent` and `openssl`. We currently do not enable Zstd and LZMA compression
support in the cross compiled version of Tor that is build using this method,
but please submit a pull request to this repository if you add support for
additional feature dependencies in the `Makefile`.

To fetch and cross compile the dependencies of Tor and Tor itself for 32-bit
Windows run:

    $ git clone https://github.com/ahf/tor-win32.git
    $ cd tor-win32
    $ make

If you are more interested in building Tor 64-bit binaries you should instead
specify the MinGW version and HOST variable explicitly using:

    $ git clone https://github.com/ahf/tor-win32.git
    $ cd tor-win32
    $ make HOST="x86_64-w64-mingw32" MINGW="mingw64"

If nothing failed during compilation, you should now have a `tor.exe` in the
`src/tor/src/app/`directory. Verify that it is actually a 32-bit or 64-bit
Windows binary using `file`:

    $ file src/tor/src/app/tor.exe
    src/tor/src/app/tor.exe: PE32 executable (console) Intel 80386, for MS Windows

We have now succesfully build a cross compiled version of Tor for Windows.

## Running Tor under Wine

This section assumes that you have cross compiled Tor as a 32-bit binary
already. You can probably skip a lot of steps below if you are working with a
64-bit binary and just install the "native" 64-bit Wine from the Debian package
archive and use that directly instead of adding the 32-bit wine32 package.

Since we are running on an x86-64 VM in 64-bit mode we must ensure that Debian
can install 32-bit x86 packages for Wine to be able to run 32-bit Windows
applications. You can read more about the Multiarch concept on the [Debian
wiki](https://wiki.debian.org/Multiarch).

    $ sudo dpkg --add-architecture i386
    $ sudo apt update

Install the 32-bit version of Wine using:

    $ sudo apt install wine32

We need to copy some DLL files to the location of our `tor.exe` file.  There is
probably an environment variable we can set to make Wine search for DLL files
elsewhere, but for now we do the following:

Copy over `zlib1.dll`, which was installed together with the `libz-mingw-w64`
packages we installed above, to the location of `tor.exe`:

    $ cp /usr/i686-w64-mingw32/lib/zlib1.dll src/tor/src/app/

Copy over the `libssp-0.dll`, from the MinGW installation, to the location of
`tor.exe`:

    $ cp /usr/lib/gcc/i686-w64-mingw32/7.2-posix/libssp-0.dll src/tor/src/app/

You should now be able to start `tor.exe` with Wine using:

    $ wine src/tor/src/app/tor.exe
    Feb 18 20:57:38.997 [notice] Tor 0.3.4.0-alpha-dev (git-e0427b6bf6bd9ea4) running on Windows 7 with Libevent 2.1.8-stable, OpenSSL 1.0.2n, Zlib 1.2.11, Liblzma N/A, and Libzstd N/A.
    Feb 18 20:57:38.998 [notice] Tor can't help you if you use it wrong! Learn how to be safe at https://www.torproject.org/download/download#warning
    Feb 18 20:57:38.998 [notice] This version is not a stable Tor release. Expect more bugs than usual.
    Feb 18 20:57:39.006 [notice] Configuration file "C:\users\john\Application Data\tor\torrc" not present, using reasonable defaults.
    Feb 18 20:57:39.027 [warn] Path for GeoIPFile (<default>) is relative and will resolve to Z:\home\john\tor-win32\<default>. Is this what you wanted?
    Feb 18 20:57:39.028 [warn] Path for GeoIPv6File (<default>) is relative and will resolve to Z:\home\john\tor-win32\<default>. Is this what you wanted?
    Feb 18 20:57:39.032 [notice] Scheduler type KISTLite has been enabled.
    Feb 18 20:57:39.033 [notice] Opening Socks listener on 127.0.0.1:9050
    Feb 18 20:57:40.000 [notice] Bootstrapped 0%: Starting
    Feb 18 20:57:40.000 [notice] Starting with guard context "default"
    Feb 18 20:57:40.000 [notice] Bootstrapped 5%: Connecting to directory server
    Feb 18 20:57:40.000 [notice] Bootstrapped 10%: Finishing handshake with directory server
    Feb 18 20:57:40.000 [notice] Bootstrapped 15%: Establishing an encrypted directory connection
    Feb 18 20:57:40.000 [notice] Bootstrapped 20%: Asking for networkstatus consensus
    Feb 18 20:57:40.000 [notice] Bootstrapped 25%: Loading networkstatus consensus
    Feb 18 20:57:42.000 [notice] I learned some more directory information, but not enough to build a circuit: We have no usable consensus.
    Feb 18 20:57:42.000 [notice] Bootstrapped 40%: Loading authority key certs
    Feb 18 20:57:43.000 [notice] Bootstrapped 45%: Asking for relay descriptors
    Feb 18 20:57:43.000 [notice] I learned some more directory information, but not enough to build a circuit: We need more microdescriptors: we have 0/6080, and can only build 0% of likely paths. (We have 0% of guards bw, 0% of midpoint bw, and 0% of exit bw = 0% of path bw.)
    Feb 18 20:57:44.000 [notice] I learned some more directory information, but not enough to build a circuit: We need more microdescriptors: we have 0/6080, and can only build 0% of likely paths. (We have 0% of guards bw, 0% of midpoint bw, and 0% of exit bw = 0% of path bw.)
    Feb 18 20:57:44.000 [notice] Bootstrapped 50%: Loading relay descriptors
    Feb 18 20:57:46.000 [notice] Bootstrapped 56%: Loading relay descriptors
    Feb 18 20:57:46.000 [notice] Bootstrapped 61%: Loading relay descriptors
    Feb 18 20:57:46.000 [notice] Bootstrapped 66%: Loading relay descriptors
    Feb 18 20:57:46.000 [notice] Bootstrapped 71%: Loading relay descriptors
    Feb 18 20:57:47.000 [notice] Bootstrapped 80%: Connecting to the Tor network
    Feb 18 20:57:47.000 [notice] Bootstrapped 85%: Finishing handshake with first hop
    Feb 18 20:57:47.000 [notice] Bootstrapped 90%: Establishing a Tor circuit
    Feb 18 20:57:48.000 [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
    Feb 18 20:57:48.000 [notice] Bootstrapped 100%: Done

### Running Tests under Wine

To run Tor's different test suites under Wine you have to copy `libssp-0.dll`
and `zlib1.dll` into the `src/tor/src/test` directory. Just like we did above
with `src/tor/src/app/`. We should now be able to run the individual test suites
using:

    $ wine src/tor/src/test/test.exe
    onion_handshake: OK
    bad_onion_handshake: OK
    ...

Look at the different `*.exe` files in `src/tor/src/test/` to get an overview
of the different test suites to run :-)

## Authors

- Alexander Færøy (<ahf@torproject.org>).
