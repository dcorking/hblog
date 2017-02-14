---
title: let binding in OCaml class definition 
tags: ocaml, objects, mistakes
description: avoid this trap
---

I like the `object` layer in OCaml but here's one quirk of the
language that sometimes I forget about and it can bite you...like I
just got bit in my
`OCaml`
[bindings](https://github.com/fxfactorial/ocaml-java-scriptengine) to
`Java`'s `ScriptEngine`. (Let's you evaluate `JavaScript` in `OCaml`
using the JVM)

OCaml lets you define an object like so: 

```ocaml
class thing = object
  method speak = print_endline "Hello"
end

let () = (new thing)#speak
```

Note that `method`s don't need arguments, they will always go off when
you call them.

and you can also have fields

```ocaml
class thing = object
  val coder = "coder"
  val mutable name = "Edgar"

  method speak = print_endline (name ^ coder)
  method set_name s = name <- s
end

let () = 
  let p = new thing in
  p#speak;
  p#set_name "Gohar";
  p#speak
```

Notice that we can also make fields `mutable` and they are private by
default. 

Now here's one situation you might encounter: 

```ocaml
class compute = object
  val first_field = Other_module.init ()
  val second_field = Some_module.use_it first_field
end
```

This won't work though because you can't use one field in another
field.

One solution might be: 

```ocaml

class compute = 

  let first_field = Other_module.init () in
  let second_field = Some_module.use_it first_field in
  object
    val first = first_field
    val second = second_field
  end

```

Now question, are `first_field` and `second_field` created each time
a new instance of `compute` is made? 

...

The answer is no and this might be counter intuitive to some, at least
it was to me and I sometimes forget this.

Verify it with: 

```ocaml
class thing =
  let foo = 1 + 2 in
  let () = print_endline "I was called" in
  object

  end


let () =
  let a = new thing in
  let b = new thing in
  ()
```

And see how many times `I was called` is printed to the screen; hence
be aware of this when you use objects in OCaml.
