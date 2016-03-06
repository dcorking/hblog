---
title: Date formating + hash digest example in OCaml
tags: ocaml, datetime, date, crypto, nocrypto
description: Datetime,Crypto tutorial in OCaml
---

This is another blog post in my quest to improve the state of OCaml
coding for the average programmer. Today's blog post is getting a date
string in OCaml, a seemingly trivial thing to do in many other
programming languages, and using it in a real world hashing example.

Low level way
================

You do could it like this:

```ocaml
(* Assume this is code.ml *)
let time_now () =
  Unix.(
    let localtime = localtime (time ()) in
    Printf.sprintf "[%02u:%02u:%02u]"
      localtime.tm_hour localtime.tm_min localtime.tm_sec)
	  
let () = time_now () |> print_endline
```

This will require you to link the `unix` package against your program,
for example:

```shell
$ ocamlfind ocamlopt -package unix -linkpkg code.ml -o Time_now
```

Higher level way + Crypto example
=========================================

At some point this low level interface might not be appropriate and
you'll want higher abstracts. `Calendar` is an package on opam that
provides such abstractions; install with:

```shell
$ opam install calendar
```

The API of `calendar`, which you'll find under the module
`CalendarLib`, is rather large. Here's some real world code that you
can reuse, built upon. The example also uses a cryptographic package
which you can get with 

```shell
$ opam install nocrypto
```

and the code:

```ocaml
(* Assume this is code.ml *)
module D = CalendarLib.Date

let nr_domains = 24

let seed_string ~date i =
  Printf.sprintf
    "%d-%d-%d:%d"
    (D.day_of_month date)
    (D.month date |> D.int_of_month)
    (D.year date)
    i

let domain_generate date =
  let rec helper count accum =
    if count = nr_domains then accum
    else seed_string ~date count :: helper (count + 1) accum
  in
  helper 0 []

let () =
  Nocrypto_entropy_unix.initialize ();
  CalendarLib.Date.today ()
  |> domain_generate
  |> List.iter begin fun date_stamp ->
    Nocrypto.Hash.digest `SHA256 (Cstruct.of_string date_stamp)
    |> Cstruct.to_string |> print_endline
  end
```

and compiling with:

```shell
$ ocamlfind ocamlopt -package calendar,nocrypto.unix -linkpkg code.ml -o Ex
```

Yay. Be sure to check my archives for many more quick and useful posts
on OCaml.
