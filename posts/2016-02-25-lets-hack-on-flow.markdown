---
title: Let's hack on Facebook flow
tags: ocaml, facebook, flow, javascript
description: Hacking on Facebook flow
---

I'm writing this post for the JavaScript developer that wants to hack
on flow but doesn't know OCaml and is eager enough to learn some OCaml
in pursuit of their hacking goals.

# Getting started.

You'll need to have `opam` installed, it is OCaml's package
manager. On `OS X` you can get it with `brew install opam`, on
`debian` just use apt-get's version and if on `Ubuntu` use this
[ppa](https://launchpad.net/~avsm/+archive/ubuntu/ppa)

The very act of installing `opam` should have also installed a
compiler, which we'll need for building `flow`.

For completeness's sake, we'll do:
```shell
$ opam install ocamlfind ocamlbuild
```

This should provide us with all the tools we need to build flow.

# Building Flow and setting goals

Now let's build flow
```shell
$ git clone https://github.com/facebook/flow
$ cd flow
$ make
```

and a sanity check:

```shell
$ bin/flow examples/01_HelloWorld
The flow server's version didn't match the client's, so it exited.
Going to launch a new one.

Launching Flow server for /Users/Edgar/Repos/flow/examples/01_HelloWorld
Spawned flow server (child pid=37837)
Logs will go to /private/tmp/flow/zSUserszSEdgarzSReposzSflowzSexampleszS01_HelloWorld.log
examples/01_HelloWorld/hello.js:7
  7: foo("Hello, world!");
     ^^^^^^^^^^^^^^^^^^^^ function call
  4:   return x*10;
              ^ string. This type is incompatible with
  4:   return x*10;
              ^^^^ number


Found 1 error
```

this is expected of course. Let's say our goal is to add some extra
behavior around string `This type is incompatible`. First we try to
find it. 

```shell
$ cd src
$ grep -nIr "This type is incompatible" .
```

Of the many hits we get we see that only two end in `.ml` which is the
file extension for `OCaml` source code.

```
./typing/flow_js.ml:3586:      let msg = "This type is incompatible with" in
./typing/flow_js.ml:3772:    "This type is incompatible with"
```

Let's open the second one on line `3772`, we see this function

```ocaml
and err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else "This type is incompatible with"
```

it looks a little bit weird because it starts with an `and`. That's
okay, the `and` means that its being used in a function before its
defined, otherwise it would have been:

```ocaml
let err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else "This type is incompatible with"
```

The function is named `err_msg`, it takes `l` and `u` and produces a
`string`.

Let's add a prefix to the string, we'll change it to:
```ocaml
and err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else "Hello JS World-> This type is incompatible with"
```

for completeness sake I assume you're still in the `src` directory.

```shell
$ cd ..
$ make clean
$ make
$ bin/flow examples/01_HelloWorld 
The flow server's version didn't match the client's, so it exited.
Going to launch a new one.

Launching Flow server for /Users/Edgar/Repos/flow/examples/01_HelloWorld
Spawned flow server (child pid=40942)
Logs will go to /private/tmp/flow/zSUserszSEdgarzSReposzSflowzSexampleszS01_HelloWorld.log
examples/01_HelloWorld/hello.js:7
  7: foo("Hello, world!");
     ^^^^^^^^^^^^^^^^^^^^ function call
  4:   return x*10;
              ^ string. Hello JS World -> This type is incompatible with
  4:   return x*10;
              ^^^^ number


Found 1 error
```

Yay, we found the spot and changed it.

# Now let's iterate on our success

Great, we changed the string, but let's say we want to have some other
stuff happen, let's try some printing to the screen. We change
`err_msg` to this instead:

```ocaml
and err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else begin
    print_endline "Hey Coder Reading hyegar.com";
    "Hello JS World -> This type is incompatible with"
  end
```

## Interesting Sidenotes
The `begin`, `end` are just syntax. I could have also written it using
`( )` like so:

```ocaml
and err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else (
    print_endline "Hey Coder Reading hyegar.com";
    "Hello JS World -> This type is incompatible with")
```

More interesting is the usage of `;`. First think to yourself, what is
the return value of printing to the screen, nothing really. In OCaml
we represent side effect and dummy values with `()`, said outloud as
unit and written in type signatures as `unit`. When you see `unit` in
a signature then it usually means the function is doing some kind of
side effect like printing to screen or talking to a db.

So I could have also written it as:

```ocaml
and err_msg l u =
  if is_use u
  then spf "%s%s" (err_operation u) (err_value l)
  else (
    let () = print_endline "Hey Coder Reading hyegar.com" in
    "Hello JS World -> This type is incompatible with")
```

but the `;` is simplier and shorter.

Back to our coding:

...and now we do the same procedure as before including the `make
clean` step. Now we run the code

```shell
$ bin/flow examples/01_HelloWorld
The flow server's version didn't match the client's, so it exited.
Going to launch a new one.

Launching Flow server for /Users/Edgar/Repos/flow/examples/01_HelloWorld
Spawned flow server (child pid=43811)
Logs will go to /private/tmp/flow/zSUserszSEdgarzSReposzSflowzSexampleszS01_HelloWorld.log
examples/01_HelloWorld/hello.js:7
  7: foo("Hello, world!");
     ^^^^^^^^^^^^^^^^^^^^ function call
  4:   return x*10;
              ^ string. Hello JS World -> This type is incompatible with
  4:   return x*10;
              ^^^^ number


Found 1 error
```

But where is our printed string of `Hey Coder Reading hyegar.com`? 

# Put on your thinking hat

First clue is this: the error message is being created by the flow
server. Many servers use logs and will redirect stdout, stderr to
those logs, just makes sense to do that. Second clue is that
`print_endline` prints to `stdout` and the third and final clue is
that `flow` itself said at the beginning of invocation that:

```
Logs will go to /private/tmp/flow/zSUserszSEdgarzSReposzSflowzSexampleszS01_HelloWorld.log
```

and now we do a simple cat on this log file, yours will be named
something unique to your machine, and see our calls to `print_endline`
being made a surprisingly large number of times (Perhaps a question
for the `flow` developers themselves)

```shell
cat /private/tmp/flow/zSUserszSEdgarzSReposzSflowzSexampleszS01_HelloWorld.log             ⏎
[2016-02-25 00:58:54] Initializing Server (This might take some time)
[2016-02-25 00:58:54] executable=/Users/Edgar/Repos/flow/bin/flow
[2016-02-25 00:58:54] version=0.22.0
[2016-02-25 00:58:54] Parsing
[2016-02-25 00:58:54] Building package heap
[2016-02-25 00:58:54] Running local inference
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
Hey Coder Reading hyegar.com
[2016-02-25 00:58:55] Re-resolving directly dependent files
[2016-02-25 00:58:55] Calculating dependencies
[2016-02-25 00:58:55] Merging
[2016-02-25 00:58:55] Done
[2016-02-25 00:58:55] Server is READY
[2016-02-25 00:58:55] Took 0.431947 seconds to initialize.
[2016-02-25 00:58:55] Status: Error
```

And there you have it, now you start hacking on `OCaml` in flow. Be
sure to check out my numerous other blog posts about `OCaml` including
the jargon and build situation located
[here](http://hyegar.com/2015/10/20/so-youre-learning-ocaml/)
