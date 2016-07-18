---
title: Imperatively traverse binary trees in OCaml, print zigzag
tags: ocaml, trees, traversal, in_order, post_order, zigzag
description: tree traversals
---

OCaml tree examples tend to be defined with algebraic data types and
tend to be functional examples. Here are two imperative tree
traversals, a pre-order and in-order. I'm still trying to work out a
nice post-order imperative solution in OCaml so if you have one then
please tweet it at me: @edgararout

EDIT: I've added a zigzag function procedure as well. This question
came up recently at an interview and I messed it up. The point is to
print the tree in a zigzag pattern.

```ocaml
type 'a node = {mutable data: 'a;
                mutable left : 'a node option;
                mutable right: 'a node option; }

let new_node data = {data; left = None; right = None;}

let insert tree new_data =
  let module Wrapper = struct exception Stop_loop end in
  let iter = ref tree in
  try
    while true do
      if new_data < !iter.data
      then match !iter.left with
        | None ->
          !iter.left <- Some (new_node new_data);
          raise Wrapper.Stop_loop
        | Some left_tree -> iter := left_tree
      else if new_data > !iter.data
      then match !iter.right with
        | None ->
          !iter.right <- Some (new_node new_data);
          raise Wrapper.Stop_loop
        | Some right_tree -> iter := right_tree
    done
  with Wrapper.Stop_loop -> ()

let pre_order_traversal tree =
  let s = Stack.create () in
  Stack.push tree s;
  while not (Stack.is_empty s) do
    let iter_node = Stack.pop s in

    Printf.sprintf "%s " iter_node.data
    |> print_string;

    (match iter_node.right with
       None -> ()
     | Some right -> Stack.push right s);

    (match iter_node.left with
       None -> ()
     | Some left -> Stack.push left s)
  done

let in_order_traversal tree =
  let module W = struct exception Stop_loop end in
  let visited_stack = Stack.create () in
  let iter_node = ref (Some tree) in
  try while true do
      (* Inner loop, we keep trying to go left *)
      (try while true do
           match !iter_node with
           | None -> raise W.Stop_loop
           | Some left ->
             Stack.push left visited_stack;
             iter_node := left.left
         done;
       with W.Stop_loop -> ());

      (* If we have no more to process in the stack, then we're
         done *)
      if Stack.length visited_stack = 0
      then raise W.Stop_loop
      else
        (* Here we're forced to start moving rightward *)
        let temp = Stack.pop visited_stack in
        Printf.sprintf "%s " temp.data |> print_string;
        iter_node := temp.right
    done
  with W.Stop_loop -> ()

let print_spiral root =
  let (current, next) = Stack.(ref (create ()), ref (create ())) in
  let left_to_right = ref true in

  let swap a b = let (a_, b_) = !a, !b in a := b_; b := a_ in

  Stack.push root !current;

  while not (Stack.is_empty !current) do
    let r = Stack.top !current in
    Stack.pop !current |> ignore;
    Printf.sprintf "%s " r.data |> print_string;
    if !left_to_right then
      begin
        (match r.left with None -> () | Some l -> Stack.push l !next);
        (match r.right with None -> () | Some r -> Stack.push r !next)
      end
    else begin
      (match r.right with None -> () | Some r -> Stack.push r !next);
      (match r.left with None -> () | Some l -> Stack.push l !next)
    end;

    if Stack.length !current = 0
    then (left_to_right := not !left_to_right; swap current next)

  done

let () =
  let root = new_node "F" in

  ["B";"G";"A";"D";"I";"C";"E";"H"] |> List.iter (insert root);

  pre_order_traversal root;
  print_newline ();
  in_order_traversal root
  print_newline ();
  print_spiral root
```

