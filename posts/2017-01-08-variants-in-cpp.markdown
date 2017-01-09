---
title: variants in C++14 without boost
tags: variants, C++, C++14
description: Getting variants in C++
---

# Variants in their native land

One of the best parts of functional programming are `Algebraic Data
Types`, also known as `Sum` types, also known as `variants`. They help
you make more type safe programs, self-documenting code.

Here are some examples in `OCaml`

```ocaml
type 'a option = Some of 'a | None
```

This is the famous option type of functional programming. A fun thing
about option is that its also a Monad, so `>>=` aka `bind` is easily
defined as:

```ocaml
let ( >>= ) x f = match x with Some h -> f h | _ -> None
```

Another example recently added to the OCaml standard library is the
result type.

```ocaml
type ('success, 'failure) result = Ok of 'success | Error of 'failure
```

The things prefixed with `'` are called type variables, they let you
write generic code from the getgo. We only need one for `option` but
we need two different ones for result so that the type of the `Ok`
variant need not be the same as the type of the `Error` variant.

Once you get an `result` value, you can pattern match on it: 

```ocaml
let process_result = function 
  | Ok r -> (* Do something with r *)
  | Error reason -> (* Do something with the error *)
```

# variants in C++

In `C++` variants are not first class citizens of the language, so
they need to be provided by a library. In modern `C++` our options are
`C++17`'s `std::variant`, `boost::variant` or someone's own rolled
version.

I don't want the `boost` dependency and I can't use `C++17` so I
looked for a library implementation. I found one
by [mapbox](https://github.com/mapbox/variant) and it fit my usecase
perfectly; its a header only template library and I like the API
provided.  Here's my implementation of `option` using the `mapbox`
variant library.

```c++
// Let's assume this file is named optional.hpp
#pragma once

#include <mapbox/variant.hpp>

namespace optional {
  struct None { };
  template<typename T>
  struct Some { T payload; };

  template<typename T>
  struct Optional : mapbox::util::variant<None, Some<T>> {
    using Base = mapbox::util::variant<None, Some<T>>;
    using Base::Base;
  };

  // These two are helper functions, like std::make_unique, etc.
  template<typename T>
  auto some(T x)  { return Optional<T>{Some<T>{x}}; }

  template<typename T>
  auto none(void) { return Optional<T>{None{}}; }

}
```

And here's an example usage: 

```c++
// Let's assume this file is named main.cpp
#include <string>
#include <iostream>

#include "optional.hpp"

// This lets just make std::string just by adding a suffix of 's'
// to things that otherwise look like char *
using namespace std::literals::string_literals;

// Pretend that this is something that could fail.
auto file_contents(void) {
  return optional::some("some file contents"s);
}

int main(void)
{
  file_contents()
    .match(
  	   [](optional::None) {
  	     std::cout << "Check if file existed\n";
  	   },
  	   [](auto some) {
  	     std::cout << "File contents: " << some.payload;
  	   });
}
```

and you can easily compile it with, assuming that you added
mapbox/variant.hpp to your compiler's include search path...

```shell
$ clang++ -std=c++14 main.cpp
```

Happy coding.
