---
title: Intermediate js_of_ocaml
tags: ocaml, js_of_ocaml, javascript, ppx, depth_first_search
description: intermediate of_ocaml
---

This post is mainly aimed at intermediate level users of
`js_of_ocaml`, the `OCaml` to `JavaScript` compiler and at improving
the sad state of documentation for `js_of_ocaml`. (Plus I'm
interviewing for jobs, hire me!, and wanted to use both OCaml,
JavaScript while preparing for interviews)

OCaml typing of JavaScript (Uses PPX)
==========================================

Say we want to express some algorithms in OCaml but using JavaScript
as the runtime execution language/environment. Let's start with our
main data structure, a tree node.

```javascript
function TreeNode(value) {
  this.value = value;
  this.left = this.right = null;
}
```

Our OCaml version will be:

```ocaml
class type ['data] tree_node = object
  method value : 'data Js.prop
  method left : 'data tree_node Js.t Js.prop
  method right : 'data tree_node Js.t Js.prop
end
```

Note that there is a small thing about typing the nullability of the
left, right fields, I will return to this at the end of the blog post
and it was done for convenience.

So explanations:

0. We are creating a `class type`, this just describes the object, how
   we describe it is key.
1. The `'data` is a type variable and lets us use any kind of type for
   the value in this node.
2. OCaml objects only expose methods to the outside world, so
   properties need `Js.prop`, this lets us read and write to this
   field. We can control it to be read only by instead using
   `Js.readonly_prop` or even write only with
   `Js.writeonly_prop`.
3. Methods `left`, `right` are also properties. Read the signature
   from right to left, aka `left` is a JavaScript read or write
   property which has a JavaScript object typed as `tree_node` and
   parameterized with the `'data` type variable. Aka tree_nodes of
   `string` or `int` or whatever the type of `'data` is.
   
Now we need to provide a way to make this object.

```ocaml
let __hidden__ =
  Js.Unsafe.pure_js_expr "function TreeNode(value) {\
                          this.value = value; \
                          this.left = this.right = null;}"
```

This is the JavaScript we'll be using, essentially. Do note that the
field names match up, this is important. Now we provide a
constructor function: 

```ocaml
let node : ('data -> 'data tree_node Js.t) Js.constr = __hidden__
```

This `node` constructor says that its a special JavaScript constructor
that will invoke `__hidden__` with `new` and such that it expects one
argument. An example is: 

```ocaml
let root = new%js node "Hello"
```

We can also make specialized constructors,

```ocaml
let node_from_int : (int -> int tree_node Js.t) Js.constr = __hidden__
```

and a simple usage: 

```ocaml
let () =
  let root = new%js node "Hello" in
  root##.left := new%js node "Left side";
  root##.left##.left := new%js node "Grand Kid";

  (* Note that this just prints the raw object representation of the
  field value *)
  Firebug.console##log root##.left##.left##.value;

  (* This shows the value as expected *)
  print_endline root##.left##.left##.value
```

You can compile and run it on node with: (Assuming file name is
`trees_in_js.ml`)

```shell
$ ocamlfind ocamlc -package js_of_ocaml.ppx -linkpkg trees_in_js.ml
$ js_of_ocaml a.out -o T.js
$ node T.js
```
And you should get something like the following printed out:

```shell
h { t: 0, c: 'Grand Kid', l: 9 }
Grand Kid
```

Depth First Search, complete example
=========================================

Now here's an example of a preorder depth first search.

```ocaml
class type ['data] tree_node = object
  method value : 'data Js.prop
  method left : 'data tree_node Js.t Js.prop
  method right : 'data tree_node Js.t Js.prop
end

let node : ('data -> 'data tree_node Js.t) Js.constr = __hidden__

let depth_first_search starting_node =
  let stack = Stack.create () in
  Stack.push starting_node stack;
  while not (Stack.is_empty stack) do
    let iter_node = Stack.pop stack in

    Printf.sprintf "%s " iter_node##.value
    |> print_string;

    if Js.Opt.return iter_node##.right |> Js.Opt.test
    then Stack.push iter_node##.right stack;

    if Js.Opt.return iter_node##.left |> Js.Opt.test
    then Stack.push iter_node##.left stack

  done

let () =
  let root = new%js node "F" in
  root##.left := new%js node "B";
  root##.right := new%js node "G";
  root##.right##.right := new%js node "I";
  root##.left##.left := new%js node "A";
  root##.left##.right := new%js node "D";
  root##.left##.right##.left := new%js node "C";
  root##.left##.right##.right := new%js node "E";
  root##.right##.right##.left := new%js node "H";

  depth_first_search root |> print_newline
```

and compile it just like given earlier in the blog post. Shameless
plug, you can also turn it into an executable with a feature I added
to the `js_of_ocaml` compiler, 

```ocaml
$ ocamlfind ocamlc -package js_of_ocaml.ppx -linkpkg trees_in_js.ml
$ js_of_ocaml --custom-header='#!/usr/bin/env node' a.out -o T.js
$ chmod +x T.js
$ /T.js
```

Yay, we used the resources of two programming languages standard
libaries in one program!

Now we can return to why the typing of the `class type` matters. The
fully correct typing of `tree_node` is:

```ocaml
class type ['data] tree_node = object
  method value : 'data Js.prop
  method left : 'data tree_node Js.t Js.Opt.t Js.prop
  method right : 'data tree_node Js.t Js.Opt.t Js.prop
end
```

Notice the addition of `Js.Opt.t`. Since the `left`, `right` are
nullable, we should capture that in the OCaml API, this however does
force us to use `Js.Opt` and so we'd have to do things like:

```ocaml
root##.left := new%js node "Left side" |> Js.Opt.return;
```

When trying to set the field, etc. But since we didn't expose that
nullability of the field in the type signature, the
`depth_first_search` code needs to check if the field is indeed
`null`, which would have been otherwise forced by the type system had
we used `Js.Opt`, aka:

```ocaml
if Js.Opt.return iter_node##.right |> Js.Opt.test
then Stack.push iter_node##.right stack;
```

These are tradeoffs that you can make in your own usage. 

I hope this makes it easier for you to use OCaml, JavaScript together.
