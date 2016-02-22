Programmers usually have a choice of language to use via what job they
decide to seek. Its probably the case that the majority of programmers
don't really care what language they use and really just care about
the paycheck and this is pretty much fine I think, at the end of the
day it really is just a job. That said there are indeed some
programmers that care about what language they use for the majority of
their adult waking life, I liken it to a carpenter caring about the
brand of hammers they're using.

# My choice

For me the best language to use is [OCaml](http://ocaml.org). Its a statically typed
functional programming language that has a powerful type system. In
addition to being a functional language, OCaml lets me write
imperative code with real mutation and this is often very useful, i.e
having a local reference variable really makes some code so much
cleaner and easier to deal with and that really ought to be our goal,
clean and easy to reason about code.

# What can you do with it?

I keep getting this question and its a little strange. What can I do
with this language? Well anything else you can do in any other
programming language. For me I'm working in the day-time as an OCamler
and I use it for my [personal projects](https://github.com/fxfactorial). I even write JavaScript in
OCaml via [js\_of\_ocaml](http://ocsigen.org/js_of_ocaml/).

Here's an example straight from my bindings to [libmaxminddb](https://github.com/maxmind/libmaxminddb).

```ocaml
(* File named loc_dump.ml *)
#require "maxminddb"

let () =
  let some_ip = "172.56.31.240" in
  let this_mmdb = Maxminddb.create "etc/GeoLite2-City.mmdb" in
  let loc = Maxminddb.location some_ip this_mmdb in
  let open Maxminddb in
  Printf.sprintf "%f %f %d %s" loc.latitude loc.longitude loc.metro_code loc.time_zone
  |> print_endline;
  let geo_borders = borders ~lang:French ~ip:some_ip this_mmdb in
  Printf.sprintf
    "%s %s %s %s %s"
    geo_borders.postal_code
    geo_borders.city_name
    geo_borders.country_name
    geo_borders.continent_name
    geo_borders.iso_code
  |> print_endline
```

And the corresponding result on the shell

```shell
$ utop loc_dump.ml
33.895900 -118.220100 803 America/Los_Angeles
90221 Compton États-Unis Amérique du Nord US
```

In any case if you're a programmer and reading this post then I
recommend that you try OCaml, especially if you have been burned by
Haskell's mental overhead but still want to do functional programming.
