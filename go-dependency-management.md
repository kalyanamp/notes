# Go dependency management: Godep and Glide

Golang's dependency management methods are, at large, not as evolved as in
languages like Python and Ruby. The main reason for that is that Golang is
a compiled language which makes dependency management important for development
but redundant for running software written in Go.

The model around non-standard Golang dependencies is composed with the use of
the `GOPATH` environment variable. `GOPATH` is a directory with the
following structure:

```
$GOPATH/
  |__ src/
  |     The source code organised in packages and .go files.
  |__ pkg/
  |     Compiled libraries. The pkg directory contains subdirectories for
  |     specific architectures.
  |__ bin/
        Contains the binary artifacts that are produced by the Go compiler.
```

Before Go 1.5, tools like [Godeps](https://github.com/tools/godep) came out.
They are based into the idea of rewriting the `GOPATH` or installing vendored
versions of Go libraries in the system `GOPATH`.

Since Go 1.5, the go commands (`run`, `build`, `test` and `install`) look into
the `vendor/` directory of a package for dependencies. This feature was
initially exposed as an experiment. In Go 1.6 it became a built-in part of the
Go tool chain.

In these notes, we are going to explore the usage of Godeps and Glide, the
most popular, currently, tools for dependency management in Go.

## Using Godeps

## Using Glide
