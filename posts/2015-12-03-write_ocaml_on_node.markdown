---
title: Write OCaml, run on node
tags: OCaml, nodejs, javascript, js_of_ocaml
description: write OCaml, compile to JavaScript
---

I've been exposed to Node, its an amazing ecosystem with great cross
platform support and a great standard library; my only issue is
**JavaScript**. So I've been writing several libraries, bindings, that
compile OCaml to JavaScript which I then run on `node`.

# Sample Code

Here are some working examples, many are directly from the README at
[ocaml-nodejs](https://github.com/fxfactorial/ocaml-nodejs/blob/master/README.md).

**Multicast DNS over UDP sockets, only for the local network, like a
 no config p2p chat application.**

```ocaml
 1  open Nodejs
 2  
 3  module U = Yojson.Basic.Util
 4  
 5  let (multicast_addr, bind_addr, port) = "224.1.1.1", "0.0.0.0", 6811
 6  
 7  let () =
 8    Random.self_init ();
 9    let p = new process in
10    let user_name = ref (Printf.sprintf "User:%d" (Random.int 10000)) in
11    let listener = Udp_datagram.(create_socket ~reuse_address:true Udp4) in
12    let sender = Udp_datagram.(create_socket ~reuse_address:true Udp4) in
13  
14    listener#bind ~port ~address:multicast_addr ~f:begin fun () ->
15      listener#add_membership multicast_addr;
16      listener#set_broadcast true;
17      listener#set_multicast_loopback true
18    end ();
19  
20  
21    listener#on_message begin fun b resp ->
22  
23      let handle = b#to_string () |> json_of_string in
24      if (handle <!> "id" |> Js.to_string) <> !user_name
25      then print_string (handle <!> "message" |> Js.to_string)
26  
27    end;
28  
29    p#stdin#on_data begin function
30      | String _ -> ()
31      | Buffer b ->
32        let msg = b#to_string () in
33        (* This needs to be redone with Re_pcre *)
34        if String.length msg > 10 then begin
35          let modify = String.sub msg 0 9 in
36          if modify = "set name:"
37          then begin
38            let as_string = Js.string (String.trim msg) in
39            let chopped =
40              as_string##split (Js.string ":") |> to_string_list |> Array.of_list
41            in
42            user_name := chopped.(1)
43          end
44        end;
45  
46        let msg = Printf.sprintf "%s>>>%s" !user_name (b#to_string ()) in
47        let total_message = (object%js
48          val id = !user_name |> to_js_str
49          val message = msg |> to_js_str
50          end) |> stringify
51        in
52        sender#send
53          ~offset:0
54          ~length:(String.length total_message)
55          ~port
56          ~dest_address:multicast_addr
57          (String total_message)
58      end
```

**Create a site and render directly from jade templates**

```ocaml
open Nodejs

let () =
  let exp = new Express.express in
  let app = new Express.app ~existing:None in

  app#set_app_value (`View_engine "jade");
  app#use (exp#static ".");
  app#get ~path:"/" (fun _ res -> res#render "index.jade");

  app#listen ~port:8080
```

**Create a raw server from the Net module**

```ocaml
let () =
  let server = Net.create_server ~conn_listener:begin fun sock ->
      sock#on_end (fun () -> print_endline "client disconnected");
      sock#write "Hello\r\n";
      sock >|> sock |> ignore
    end ()
  in
  server#listen ~port:8124 begin fun () ->
    let info = server#address in
    print_endline info.Net.address;
    print_endline (info.Net.ip_family |> string_of_ip);
    print_endline (info.Net.port |> string_of_int);
    print_endline "started server"
  end
```

**Create a file stream, gzip it, write it**

```ocaml
1  open Nodejs
2  
3  let _ =
4    Fs.create_read_stream "code.ml" >|>
5    Zlib.create_gzip () >|>
6    Fs.create_write_stream "NEWCODE_TEST.ml"
```

**Typed Decoding of Buffers**

```ocaml
open Nodejs

let () =
  let string_decoder = new String_decoder.decoder Utf_8 in
  let cent = new Buffer.buffer (`Array [|0xE2; 0x82; 0xAC|]) in
  (string_decoder#write cent) |> print_endline
```

This one is a bit more low level as it its a general idea of how these
bindings are implemented.

```ocaml
 1  (* Assume this file is called c.ml *)
 2  open Nodejs
 3  
 4  class child_process = object
 5  
 6    val raw_js = require_module "child_process"
 7  
 8    (* Clearly not finished *)
 9    method spawn_sync cmd args : (string * string) list =
10      let handle =
11        [|i (Js.string cmd);
12          i (List.map Js.string args |> Array.of_list |> Js.array)|]
13        |> m raw_js "spawnSync"
14      in
15      (handle <!> "envPairs")
16      |> Js.to_array |> Array.map begin fun (s : Js.js_string Js.t) ->
17        let chop = s##split (Js.string "=") |> to_string_list |> Array.of_list in
18        (chop.(0), chop.(1))
19      end
20      |> Array.to_list
21  
22  end
23  
24  let () =
25    let ls_proc = (new child_process)#spawn_sync "ls" [] in
26    ls_proc |> List.iter begin fun (key, value) ->
27      Printf.sprintf "Key was: %s and value: %s" key value
28      |> print_endline
29    end
```

(Line one comes from my `nodejs` package, install it with `opam install
nodejs`). 

This example is a subset of my bindings to the builtin node module,
[child\_process](https://nodejs.org/api/child_process.html). Here we spawn a separate process and create an OCaml
`alist` out of the environment variables of the spawned process. A
point of interest is the poverty of OCaml StdLib's `String` module, so
much so that I get more functionality out of JavaScript's string
methods! (There's no split in the StdLib). 

To actually run this code you'll need to do:

```shell
$ ocamlfind ocamlc c.ml -linkpkg -package nodejs -o T.out
$ js_of_ocaml T.out
$ node T.js
```

# Projects

This approach surprisingly works and I've written similar bindings to
[socket.io](https://github.com/fxfactorial/ocaml-npm-socket-io) for which I have a working chat server:

```ocaml
open Nodejs

let () =
  let io = Socket_io.require () in
  let server =
    Http.create_server begin fun incoming response ->

      Fs.read_file ~path:"./client.html" begin fun err data ->
        response#write_head ~status_code:200 [("Content-type", "text/html")];
        response#end_ ~data:(Http.String data) ()

      end
    end
  in
  let app = server#listen ~port:8080 begin fun () ->
      Printf.sprintf
        "Started Server and Running node: %s" (new process#version)
      |> print_endline
    end
  in

  let io = io#listen app in
  io#sockets#on_connection begin fun socket ->

    socket#on "message_to_server" begin fun data ->

      io#sockets#emit
        ~event_name:"message_to_client"
        !!(object%js val message = data <!> "message" end)

    end
  end
```

Notice the features of OCaml that don't exist in JavaScript at all,
like named parameters.

Another project in this vein are my bindings to Github's `Electron`
project, here's a project I did with a friend using Basecamp's
recently released `Trix` editor.

![img](./electron_working.gif)

# Like what you see?

Writing out these bindings is a bit of work, Node's API is pretty big
in addition to third party code like socket.io, express, and
Electron. Much of these bindings are quite formulaic, although some
ideas don't easily match up between the OCaml and JavaScript boundary,
like `varargs` so that requires some more thought at times. 

To any reader interested in OCaml open-source or for whatever reason,
send me a PR, its mostly an issue of translating [the Node API](https://nodejs.org/api/index.html) into the
equivalent bindings:

[nodejs repo](https://github.com/fxfactorial/ocaml-nodejs)

[electron repo](https://github.com/fxfactorial/ocaml-electron)
