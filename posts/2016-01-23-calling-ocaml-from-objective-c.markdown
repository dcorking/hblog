I love iOS and I love OCaml, so I've packaged up an OCaml compiler for
iOS via opam. Now all you need to build OCaml programs on the iPhone
is opam!

# First steps

You'll need the cross compiler. Follow the README instructions
mentioned [here](https://github.com/fxfactorial/opam-ios). This will provide you with the cross compiler based on
OCaml 4.02.0.

# Example Code

Now I'll show you an example of calling OCaml from Objective-C. Much
of this code will have simplifications as its an example for getting
you started. Also note that dealing at the C level and the OCaml
runtime can be tricky and is not for beginners. 

Here's the OCaml code first, assume it is named code.ml

```ocaml
1  let make_string () =
2    print_endline "Hello Word from OCaml";
3    "Hello World "
4  
5  let () =
6    Callback.register "make_string" make_string
```

We're essentially telling the runtime to make this function
available to grab as a handle, in this case it's a handle on a
closure.

And now the Objective-C code, assume it is named main.c

```objective-c
 1  #define CAML_NAME_SPACE
 2  
 3  #import <Foundation/Foundation.h>
 4  
 5  #include <caml/callback.h>
 6  #include <caml/mlvalues.h>
 7  
 8  int main (int argc, char **argv)
 9  {
10    caml_startup(argv);
11    caml_callback(*caml_named_value("make_string"), Val_unit);
12    NSLog(@"Now using objective-c code");
13    return 0;
14  }
```

Its important to call `caml_startup` before any OCaml callbacks. Then
we get a handle on the closure and call it with unit.

To compile this code you'll need the cross-compiler which was
installed by following the directions on my opam-ios repo, the command
you use is:

```shell
$ ocamloptrev -rev 8.3 -ccopt -ObjC -cclib '-framework Foundation' main.c code.ml -o F
```

The 8.3 is the iOS SDK you want to link against, again see the README
on the github repo for opam-ios for more details.

The output we get, `F` is an arm executable, when we run it on the
iPhone we will get an output of:

```shell
$ some_iphone :~/  ./F
Hello Word from OCaml
2016-01-23 22:44:04.889 F[1977:507] Now using objective-c code
```

Yay!
