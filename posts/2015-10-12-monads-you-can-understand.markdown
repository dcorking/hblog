Let's learn about monads as programmers, that is let's use them for
building something and in the process we'll build up a intuition;
practice first, then you can go back to theory later.

This post is split in two parts:

Part 1 is just to get the ideas out the door

Part 2 (Currently not written) is for an OCaml audience,
just to improve the quality of the OCaml code itself

As Bryan O'Sullivan said (From Real World Haskell) "...we need
something that's non-trivial but still chewey." So we'll build a real
working example in OCaml that will be the starting point of a library
to the [Stripe API](https://stripe.com/docs/api) and it will use monads.

# Prereqs

We need to make sure we have all the same starting point for this
code, I assume you have [opam](https://opam.ocaml.org) installed. Now do:

```shell
$ opam install lwt cohttp yojson uri
```

These are the libaries we will use: `lwt` is for threading,
`cohttp.lwt` is for our HTTP requests, `uri` for handling URIs and
`yojson`, an easy to use and **de facto** json library.

# Stripe

The Stripe API is a straight forward REST API. You'll need to have a
developer account from which you'll get your Test Key, hence forth
known as `key`. We'll focus on one task, we'll start the process of
charging a credit card. Point your browser to this [link](https://stripe.com/docs/api#create_charge) which shows
how to create the "Charge" object. Be sure to be looking at that page
while logged in because we'll need the value for the `source` parameter,
we'll call it the **source token** From there we see we need to do a
`POST` to <https://api.stripe.com/v1/charges> with several required
arguments:

1.  The amount to charge
2.  The ISO code for currency
3.  Either the Customer or Source

All of Stripe's API returns JSON, so we can already sketch out our
function's signature and helper data structure, something like:

```ocaml
type t = { authed : Cohttp.Headers.t; 
           end_point : string; }

val make_handle : unit -> t

val create_charge : int -> 
                    string -> 
                    string ->
                    t ->  
                    Yojson.Basic.json Lwt.t
```

Mostly straight forward, although that `Lwt.t` should look at
little strange. The quick answer that that `create_charge` will give
back an `Lwt.t` monad which in this case will contain a `json` object.

Let's start filling out the `make_handle` function, 

```ocaml
let make_handle () = 
  let auth_me k = 
    let starter = Cohttp.Header.init () in 
    Cohttp.Auth.credential_of_string ("Bearer " ^ k)
    |> Cohttp.Header.add_authorization starter
  in
  {authed = auth_me "YOUR STRIPE KEY"; 
   (** Hardcoding to the charge create path *)
   end_point = "https://api.stripe.com/v1/charges"}
```

This creates our record needed for making a post. Stripe requires a
key in each request so we'll make this record to encapsulate that,
otherwise we have to write boilerplate code, never fun.

Now the first cut of out `create_charge` function:

(**META NOTE** This is horrible indentation, I'm only doing it because
of the sizing done in the exporting of this document and I want it to
be clear what's happening and the coloring is wrong because ~'~ can be
used in a value's name but its messing up the coding exporter)

```ocaml
 1  let create_charge amount currency source handle = 
 2    let open Cohttp_lwt_unix in
 3    let this_uri = Uri.of_string handle.end_point in
 4    [("amount", string_of_int amount);
 5     ("currency", currency);]
 6     ("source", source)]
 7    |> Uri.add_query_params' this_uri
 8    |> Client.post ~headers:handle.authed >>= 
 9       fun (resp, body) -> 
10       (* This can also be done more succinctly
11          with:
12          Cohttp_lwt_body.to_string body >|=
13          Yojson.Basic.from_string  *)
14       Cohttp_lwt_body.to_string body >>= fun j -> 
15       Yojson.Basic.from_string j |> return
```

Okay, so what the hell is going on. Line 2 open the `Cohttp_lwt_unix`
module locally to this function so that we can type `Client.post`
instead of `Cohttp_lwt_unix.Client.post`. Line 3 creates a `uri`
object, lines 4-6 are the query parameters we want to add to our `uri`
object. The `|>` symbol is a function, you can call it reverse apply,
you can define it as

```ocaml
let ( |> ) f x = x f
```

but you don't need to, it comes with OCaml. It just says take the left
side as the input to the right side, aka a Unix Pipe. Now let's see
the signature of `Client.post`, its:

```ocaml
val post ?ctx:Cohttp_lwt_unix.Client.ctx ->
         ?body:Cohttp_lwt_body.t ->
         ?chunked:bool ->
         ?headers:Cohttp.Header.t ->
         Uri.t -> 
         (Cohttp.Response.t * Cohttp_lwt_body.t) Lwt.t
```

Looks big but we don't care about most of it, in fact we could just
care about the `Uri.t` parameter since the rest of the parameters, the
ones with `?`, have default values but we can override them as we do
on line 8's `Client.post ~headers:handle.authed`. Now notice the final
value of a call to `Client.post`, its:

```ocaml
(Cohttp.Response.t * Cohttp_lwt_body.t) Lwt.t
```

This says that `post` will give back an `Lwt.t` monad which contains a
tuple of a response object and the body, again completely reasonable.
The line of line 8 features the famous `>>=` operator, aka `bind` it's
signature is:

```ocaml
val ( >>= ) : 'a Lwt.t -> ('a -> 'b Lwt.t) -> 'b Lwt.t
```

And this says that `>>=` takes something wrapped in the `Lwt.t` monad
on the left side and passes the unwrapped value to a function on the
right side which has to return something wrapped in the `Lwt.t` monad
where the two somethings can be different or the same. So in our code
that right side is this anonymous function, this lambda:

```ocaml
fun (resp, body) ->
```

Now we have a handle on the http response and the body, we won't do
any error checking so let's just look at the body with line 10's usage
of `Cohttp_lwt_body.to_string` whose signature is:

```ocaml
val to_string : Cohttp_lwt_body.t -> string Lwt.t
```

Translation: Takes a body and gives back a string wrapped in a Lwt.t
monad. Remember our goal is to get the body as a json object, so we
could just pipe it to `Yojson.Basic.from_string` but still recall that
our `create_charge` function had final value of `Yojson.Basic.json
Lwt.t`, not a plain `Yojson.Basic.json` so we pipe it to the other
famous monad related function, `return`. `return` really should have
been called inject because it takes a plain value and "injects" it
into a monad, let's see its value here:

```ocaml
val return : 'a -> 'a Lwt.t
```

We are using it to turn our plain `Yojson.Basic.json` into a
`Yojson.Basic.json Lwt.t` Now let's use this code to actually do
something.

```ocaml
let program = 
  let this_handle = make_handle () in
  let st = "SOURCE_TOKEN_MENTIONED_EARLIER" in
  create_charge 500 "usd" this_handle >>= fun j -> 
  Yojson.Basic.pretty_to_string j |> Lwt_io.printl 

let () = 
  Lwt_main.run program
```

At this point this should be understandable, our create\_charge is
returning a monad so we pass its output to `>>=`, which passes a json
object to a lambda, we turn the json value to a pretty printed string
and pipe it out to `Lwt_io.printl`, a `printf` for the `Lwt` library.

# Build and Run

Now let's build our program and run it, assuming all the code is in
`code.ml` we invoke the OCaml toolchain as so:

```ocaml
$ ocamlfind ocamlopt -linkpkg code.ml -packages lwt.unix,cohttp.lwt,yojson,uri -o T
```

assuming everything well, you should have an executable T, which will
print something like:

```ocaml
$ ./T
{
  "object": "charge",
  "created": 1443072037,
  "livemode": false,
  "paid": true,
  "status": "succeeded",
  "amount": 500,
  "currency": "usd",
  "refunded": false,
  "source": {
    "object": "card",
    "brand": "Visa",
    "funding": "credit",
    "exp_month": 8,
    "exp_year": 2016,
    "country": "US",
    "name": null,
    "address_line1": null,
    "address_line2": null,
    "address_city": null,
    "address_state": null,
    "address_zip": null,
    "address_country": null,
    "cvc_check": null,
    "address_line1_check": null,
    "address_zip_check": null,
    "tokenization_method": null,
    "dynamic_last4": null,
    "metadata": {},
    "customer": null
  },
  "captured": true,
  "failure_message": null,
  "failure_code": null,
  "amount_refunded": 0,
  "customer": null,
  "invoice": null,
  "dispute": null,
  "metadata": {},
  "statement_descriptor": null,
  "fraud_details": {},
  "receipt_email": null,
  "receipt_number": null,
  "shipping": null,
  "destination": null,
  "application_fee": null,
  "refunds": {
    "object": "list",
    "total_count": 0,
    "has_more": false,
    "url": "/v1/charges/ch_16oS9pJDURztdKY9Z7QS8c8D/refunds",
    "data": []
  }
}
```

Yay success.

# Moral of the Story

So in terms of actually day to day coding, you don't actually need to
know what a monad "is", you just need to know how to use it and
honestly that's completely fine, in fact just following the type
signatures can take you pretty far in a new, unfamiliar library.
