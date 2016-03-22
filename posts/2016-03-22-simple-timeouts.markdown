---
title: Simple timeout usage
tags: ocaml, unix, timeout
description: Example code timeout
---

I wanted to use a timeout in OCaml for some shell coding but I didn't
want to introduce a big dependency on `Lwt`. After some googling I
found
[this](https://www.reddit.com/r/ocaml/comments/3qapbv/question_about_writing_a_timeout_function_and_the/)

Here's my take on the timeout function, basically its the same but I
push everything into the timeout function itself.

```ocaml
let timeout ?(on_timeout = fun () -> ()) ~arg ~timeout ~default_value f =
  let module Wrapper = struct exception Timeout end in
  let sigalrm_handler = Sys.Signal_handle (fun _ -> raise Wrapper.Timeout) in
  let old_behavior = Sys.signal Sys.sigalrm sigalrm_handler in
  let reset_sigalrm () = Sys.set_signal Sys.sigalrm old_behavior in
  ignore (Unix.alarm timeout);
  try
    let res = f arg in
    reset_sigalrm ();
    res
  with exc ->
    reset_sigalrm ();
    if exc = Wrapper.Timeout
    then (on_timeout (); default_value)
    else raise exc
```

and you can can use it like so:

```ocaml
Sys.command
|> timeout 
   ~arg:"sleep 3" 
   ~timeout:2 
   ~default_value:(-1) 
   ~on_timeout:(fun () -> print_endline "func timed out")
```

You might notice that your wrapped function only gets one arg, so how
can we use this timeout wrapper on functions that take more than one
argument? By currying and using a dummy arg of unit. 

Example:

```ocaml
let () =
  let partialed first second third () = first + second + third in
  timeout ~arg:() ~timeout:4 ~default_value:(-1) (partialed 1 2 3)
  |> ignore
```

I've added this to my `podge` library found
[here](http://github.com/fxfactorial/podge), a collection of useful
utility functions.
