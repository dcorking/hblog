Today was the first day of the Introduction to Functional Programming
in `OCaml` course, located [here](https://www.france-universite-numerique-mooc.fr/courses/parisdiderot/56002/session01/about). Apparently over 2,000 people signed up
and while doubtlessly many will drop out, there will still be 2,000
more programmers that are now aware of this amazing language called
`OCaml`.

# Crash course on the OCaml ecosystem.

These are some key notes that you should know.

1.  `opam` is the package manager for OCaml. It is very advanced and
    supports many features. The most basic of which is 
    
    ```shell
    $ opam install <some_package>
    ```
    
    For people on `OS X`, you can get it on `brew` and all the `Linux`
    distros should have `opam` for you to install. For Windows people,
    get a VM. EDIT: To be clear, OCaml can build **native** executables on
    Windows just fine, but opam doesn't work on this platform and for a
    beginner you'll waste a lot of time with environment issues or
    libraries that assume Unix.

2.  Once you have `opam` installed, you probably want to do:
    
    ```shell
    $ opam switch 4.02.3
    ```
    
    This will install the latest version of the compiler.

3.  `ocamlfind` is a program that predates `opam` and wraps the
    standard `OCaml` compilers: `ocamlc` and `ocamlopt`. The former is
    a byte code compiler and the latter creates native code.

4.  `ocamlbuild` is a tool that helps build `OCaml` programs, many
    people have strong opinions on it.

5.  `oasis` is a tool that helps abstract usage of 3, 4. I resisted it
    for a while and wrote Makefiles instead, don't do that, just use
    oasis. The oasis flow basically goes like this: (Be aware that
    oasis is really finicky and its error messages are useless)
    
    5.1) Create a directory.
    
    5.2) Go to the directory and create a file named **\_oasis** and
         directory named `src`
    
    5.3) Here is a template of the contents of the **\_oasis** file
    
    ```shell
    OASISFormat:  0.4
    OCamlVersion: >= 4.02.3
    Name:         opam_package_name
    Version:      0.1
    Maintainers:  New OCaml programmer
    Homepage:     http://my_coolsite.com
    Synopsis:     Some short description
    Authors:      Cool@me.com
    License:      BSD-3-clause
    Plugins:      META (0.4), DevFiles (0.4)
    AlphaFeatures: ocamlbuild_more_args
    
    Description:
      Some cool description
    
    # This is a comment and this below creates an binary program    
    Executable <some_program_name>
      Path: src
      BuildTools:ocamlbuild
      install: true
      MainIs: main.ml
      CompiledObject: native
      BuildDepends: package_one, package_two
    
    # Another comment, this builds a library called pg
    Library pg
      Path:         src
      # oasis will figure out the dependencies, 
      # Just list the modules you want public, 
      # Note that there's no .ml, just give the name
      Modules:      Pg
      CompiledObject: byte
      BuildDepends: some_package
    ```
    
    5.4) Generate the Makefile, setup.ml, configure and other build crap.
    
    ```shell
    $ oasis setup -setup-update dynamic
    ```
    
    5.5) Actually build your code, yes its just a call to make.
    
    ```shell
    $ make
    ```
    
    5.6) You can stop here, but you can go even further with
         `oasis2opam`. Install it with: `opam install oasis2opam`, then
         in your project's root directory, aka the directory with the
         \_oasis file, do: `oasis2opam --local`. This creates the `opam`
         directory and some meta data for the opam packaging
         system. Your local package can now be a first class citizen
         with opam just by doing this in the same project root
         directory: 
    
    ```shell
    $ opam pin add <your_package_name> . -y
    ```

6.  `merlin` is a OCaml program that is simply amazing it drives code
    completion for plugins available in `emacs` and `vim`. Once you
    have merlin installed and running, add a `.merlin` file to your
    project so that `merlin` knows what packages to code complete for,
    a sample `.merlin` file looks like this:
    
    ```shell
    B _build/src
    S src
    PKG cmdliner js_of_ocaml
    ```
    
    Notice how I put the `B _build/src` That sort of assumes you're
    using `_oasis` and you made the `src` directory I mentioned earlier.

7.  There are no full blown IDEs for OCaml, learn `emacs` or
    `vim`. EDIT: apparently `Sublime Text` has a merlin plugin, if
    you're already familiar with Sublime Text then just stick with it,
    merlin is really what matters here.

8.  `utop` is an enhanced repl, its better than the plain `ocaml`
       repl. Install it with `opam install utop`

# Library situation

`OCaml` does have a standard library but it sucks. It was only created
to serve the needs of the compiler programmers, ie its not like
`Python`'s standard library which has everything under the sun + the
moon. There are a few standard library replacements, one is called
`Core` and its provided by Jane Street. Its the library used in the
**Real World OCaml** book/website. Another standard library replacement
is called `Batteries`, this is more "community" supported. There is a
more recent contender called `Containers`. For a categorized list of
contemporary and well liked/must have libraries, checkout the
[awesome-ocaml](https://github.com/rizo/awesome-ocaml) repository.

# Speaking of Libraries...

This is "functional programming," so many of the real world libraries
you'll encounter will have Monadic interfaces, like `lwt` or Core's
`async`, both are asynchronous threading libraries, use Monads
and that wacky `>>=` function. But you really shouldn't fret about
what a Monad is or represents, just follow the type signature and
you'll be fine. For a more detailed treatment of Monads in OCaml and a
code example to talk to the `Stripe` API, see [this](http://hyegar.com/blog/2015/09/23/let's-just-use-monads/).

# Doing simple tasks (shameless plug)

I try using `OCaml` for literally everything and that includes going
to hackathons, to make this less painful I wrote a library called
`Podge` which helps with simple stuff. I don't claim its a standard
library replacement, just a library for getting stuff done. These two
code samples assume the file is named `code.ml` and can be run with
`utop code.ml`

First install with opam:

```shell
$ opam install podge
```

1.  Reading output of a process

```ocaml
#require "podge"
let () = 
  Podge.Unix.read_process_output "ls -halt" |> List.iter print_endline
```

The `|>` just means piping, its piping the output of
`read_process_output` into the input of the partially applied function
`iter`

1.  Reading a file

```ocaml
#require "podge"
let () = 
  Podge.Unix.read_lines "code.ml" |> List.iter print_endline
```

Similar to 1, this reads all lines of file and gives it to the input
of the partially applied function `iter`.

These are two simple code samples from `Podge`, check out the [repo](https://github.com/fxfactorial/podge)
for other useful modules like: (The README has code examples)

1.  `Web` for simple HTTP requests and getting data back as JSON,
2.  `Xml` for querying simple XML documents
3.  `ANSITerminal` for creating colored shell output
4.  `String` which is all due to [Rudi Grinberg](http://rgrinberg.com).

# What can you do with it?

Loads.

1.  Compilers!, lots of compilers/compiler tools are written in
    OCaml: Facebook uses OCaml for [pfff](https://github.com/facebook/pfff) and [flow](https://github.com/facebook/flow) and the first cut of
    Rust was written in OCaml.
2.  Financial world, [Jane Street](https://www.janestreet.com) uses OCaml for basically everything (AFAIK)
3.  Systems Programming: [ahrefs](https://ahrefs.com), my employer, uses OCaml for heavy
    systems programming.
4.  Kernels: Unikernels are hot right now, the most prominent one is
    the [Mirage-OS](https://mirage.io) project and its all OCaml.
5.  Shameless plug: I use OCaml as well for `js_of_ocaml`, in fact I'm
    using it to write an Electron app with a node backend (All code is
    OCaml compiled into JS, then run on node/Electron), see [here.](https://github.com/fxfactorial/ocaml-electron)
6.  Genomics/Bioinformatics: [Hammer Lab](https://github.com/hammerlab) in NYC uses OCaml for their
    genomics/sequencing work.

...And I'm sure there's more I haven't mentioned...

# Stick with it!

This style of coding might be new to you or maybe its your first
programming language, stick with it and continue. `OCaml` offers many
awesome features and has many strengths including a very professional
and pragmatic community.
