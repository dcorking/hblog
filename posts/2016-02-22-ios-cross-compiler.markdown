---
title: Getting an iOS compiler on Linux, full Objective-C support
tags: ios, linux, cross compiler, nixos, nixpkgs, objective-c
description: iOS cross compiler
---

# The problem 

Lately I've been doing iPhone low level debugging, forensics, and
reverse engineering. For that I need an `iOS` compiler and that has
usually only been available on `OS X`. However my day time development
environment is Linux which clearly doesn't have an `objective-c`
runtime or libraries...how to solve this?

# The solution

I found this [project](https://github.com/tpoechtrager/osxcross) but
it was very hands on and difficult to reproduce on multiple
machines. How could I reliably build this code? My solution was
`NixOS`, specifically the `nixpkgs` package manager (I won't go into
the benefits of `Nix` in this post). Now you can also benefit from
this work by getting an `armv7` iOS cross compiler for up to `iOS 9.2`
by simply doing:

```shell
$ nix-env -i clang ios-cross-compile
```

This will crap out initially because I cannot redistribute iOS SDKs,
but it does give you instructions on how to manually add something to
the `nix-store`. After you follow the instructions run the command
again and then you'll have all these tools 

```
armv7-apple-darwin11-ar                 armv7-apple-darwin11-nm
armv7-apple-darwin11-as                 armv7-apple-darwin11-nmedit
armv7-apple-darwin11-bitcode_strip      armv7-apple-darwin11-ObjectDump
armv7-apple-darwin11-checksyms          armv7-apple-darwin11-otool
armv7-apple-darwin11-clang              armv7-apple-darwin11-pagestuff
armv7-apple-darwin11-clang++            armv7-apple-darwin11-ranlib
armv7-apple-darwin11-codesign_allocate  armv7-apple-darwin11-redo_prebinding
armv7-apple-darwin11-dyldinfo           armv7-apple-darwin11-seg_addr_table
armv7-apple-darwin11-indr               armv7-apple-darwin11-segedit
armv7-apple-darwin11-install_name_tool  armv7-apple-darwin11-seg_hack
armv7-apple-darwin11-ld                 armv7-apple-darwin11-size
armv7-apple-darwin11-libtool            armv7-apple-darwin11-strings
armv7-apple-darwin11-lipo               armv7-apple-darwin11-strip
armv7-apple-darwin11-machocheck         armv7-apple-darwin11-unwinddump
```

...installed and they will work for SDK `7.0-9.2`!

# Stay tuned

I'll have more blog posts lined up including: 

1) How to get correct code completion for objective-c on Linux in
emacs

2) Creating iPhone command line tools, tweaks in C++, Objective-C
using theos.
