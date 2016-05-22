---
title: Using system threads with Lwt
tags: ocaml, lwt, threading
decription: Sample of Lwt_premptive
---

This blog post shows you how to use real system threads in OCaml by
using `Lwt`, `Lwt_preemptive`.

Common complaint about multicore
=====================================

A common complaint about OCaml is the lack of multicore, about the
single threadedness of the runtime. This is true but its not like
`OCaml` programmers don't have solutions! Here's an easy example that
you can instantly use in your coding.

Setup
=====

First we will need some way to verify that our system threads are
actually working, we'll use some math equation to purposefully cause
CPU load. Here's the `Sieve Of Eratosthenes` Algorithm that I copied
from [here](http://www.scriptol.com/programming/sieve.php#ocaml).

```ocaml
open List

type integer = Int of int
let number_two = Int(2)
let number_zero = Int(0)
let is_less_than_two (Int n) = n < 2
let incr (Int n) = Int(n + 1)
let decr (Int n) = Int(n - 1)
let is_number_zero (Int n) = n = 0

let iota n =
  let rec loop curr counter =
    if is_less_than_two counter then []
    else curr::(loop (incr curr) (decr counter))
  in
  loop number_two n

let sieve lst =
  let rec choose_pivot = function
    | [] -> []
    | car::cdr when is_number_zero car ->
      car::(choose_pivot cdr)
    | car::cdr ->
      car::(choose_pivot (do_sieve car (decr car) cdr))

  and do_sieve step current lst =
    match lst with
    | [] -> []
    | car::cdr ->
      if is_number_zero current
      then number_zero::(do_sieve step (decr step) cdr)
      else car::(do_sieve step (decr current) cdr)
  in
  choose_pivot lst

let is_prime n =
  match rev (sieve (iota n)) with
    x::_ -> not (is_number_zero x)
```

Now our `Lwt`, `Lwt_preemptive` code:

```ocaml
open Lwt.Infix

let do_example port =
  let address = Unix.(ADDR_INET (inet_addr_loopback, port)) in
  Lwt_io.establish_server address (fun (tcp_in, tcp_out) ->
      () |> Lwt_preemptive.detach (fun () ->
          while true do
            ignore (is_prime (Int port))
          done
        )
      |> Lwt.ignore_result
    )
  |> ignore |> Lwt.return

let () =
  let rec forever () = fst (Lwt.wait ()) >>= forever in
  Lwt_preemptive.init 5 10 ignore;
  ([2000; 2001; 2002; 2003; 2004]
   |> Lwt_list.iter_p do_example >>= forever)
  |> Lwt_main.run
```

The code that runs inside the callback to `Lwt_io.establish_server`
uses `Lwt_preemptive.detach`, this creates a new system thread
whenever there is something that connects on ports
`[2000; 2001; 2002; 2003; 2004]`. You don't have to call
`Lwt_preemptive.init` since detach will do it anyway, but I am doing
it to ensure that at least 5 threads are made with 10 being the max.

We compile it with:

```shell
$ ocamlfind ocamlopt -thread -package lwt.unix,lwt.preemptive test_case.ml -linkpkg -o TEST_CASE
```

And we test it by starting up `./TEST_CASE`, opening `htop` and
finding `TEST_CASE` (hit `t` in htop to see a tree based process view)
and running `socat STDIN TCP:localhost:<some_port>`, where
`<some_port>` is a number in our list of ports (remember
`[2000; 2001; 2002; 2003; 2004]`).

Thus we see in `htop` the CPU % utilization move for each of the
threads of `TEST_CASE`.

Success! Real system threads.
