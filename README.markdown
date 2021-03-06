# hblog

Source that uses [Hakyll](http://jaspervdj.be/hakyll/) to generate the
static pages for [hyegar.com](http://hyegar.com)

Since the binary is built dynamically, make sure that hakyll is installed or
updated with

    cabal install hakyll --enable-shared
	
and build with:

	ghc --make -threaded hblog.hs
	
watch with:
	./hblog watch --no-server 

Might need to pick dependency versions explicitly like so:

	cabal install hakyll --enable-shared --constraint=process==1.4.2.0

## Dependencies:

### Ruby

- sass
- compass

### setup

- _publish directory should be setup up as a clone of the upstream deployment
  repository with the remote name of `github`. This is not a submodule and the
  relevant entry has been made to the .gitignore file.

## License

This configuration and setup code for this blog is under the MIT License. See
the files LICENSE for the full text.
