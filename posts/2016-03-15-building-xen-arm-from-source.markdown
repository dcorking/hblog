---
title: Building xen-arm-builder from source
tags: mirageos, sdcard, xen, ocaml
description: Tutorial/troubleshooting
---

I'm in Morocco for the first
[mirageos](http://marrakech2016.mirage.io/) hackathon. One of the
things I've done here is build
[xen-arm-builder](https://github.com/mirage/xen-arm-builder) from
source and it was a big challenge. Here's my general notes and
pointers to help you get over this hump. I assume you know what
mirageos/xen are.

Much MUCH thanks to [Thomas Leonard](http://roscidus.com/blog/) and
[Mindy Preston](http://somerandomidiot.com/) for lending me a
cubieboard2 and helping me with all the debugging, compiling issues;
wouldn't have been able to do it without them.

Machine Setup
================

I did this on Ubuntu 15 and was deploying to a
[cubieboard2](http://cubieboard.org/2013/06/19/cubieboard2-is-here/).

Example Workflow
====================
Assuming that you are connecting the cubieboard2 to your laptop over a
serial connection then you can get a shell on the machine with 

```shell
$ screen -h 10000 /dev/ttyUSB0 115200
```

Then say you are using
[mirage-skeleton](https://github.com/mirage/mirage-skeleton), here's
an example flow.

```shell
$ git clone https://github.com/mirage/mirage-skeleton
$ cd mirage-skeleton
$ make configure MODE=xen
$ cd console
$ make
$ sudo xl create console.xe
```

Yay Unikernel on Cubieboard2!

Troublesome issues
======================

I had many, many issues in building the code from source, here are
some troubleshooting steps that might help you as well.

0. Be sure to have fast internet because building from source
   including making a clone of the Linux source, ouch.
1. `OPAMVERBOSE=1 opam <anything>` is incredibly useful.
2. Ubuntu 15 will install a `gcc-5.0` version of the cross-compiler,
   be sure to do `make build CC=arm-linux-gnueabihf-gcc-4.8` instead
   of a plain `make build`.
3. Be sure to **not** use the `4.02.0` compiler, the cubieboard2 is
   super slow and that OCaml compiler had a performance bug, use the
   `4.02.3` compiler instead.
4. Check `top` to see some kind of xp binary, it eats the CPU and that
   really slows down compilation, best to kill them.
5. Do make sure that the time on the machine is correct, otherwise
   you'll have weird hanging by opam which will seem like its building
   but actually its just stuck on a timing issue, aka be sure to do:
   `sudo ntpdate uk.pool.ntp.org` before doing anything with
   mirage/opam/aptitude.
6. If you get odd errors like:

```shell
Parsing config from console.xl
xc: error: panic: xc_dom_core.c:185: failed to open file: No such file or directory: Internal error
libxl: error: libxl_dom.c:377:libxl__build_pv: xc_dom_kernel_file failed: No such file or directory
libxl: error: libxl_create.c:1022:domcreate_rebuild_done: cannot
(re-)build domain: -3
```

   Then check that the libraries are correctly set in `/usr/lib`,
   check with 

```shell
$ sudo strace -e open xl create console.xl
```
and either `cp` or `ln` them correctly.

7. I recommend just installing all the `depopts` of all the mirage
   projects, there are some bugs in mirage opam files that don't
   install other needed packages.
