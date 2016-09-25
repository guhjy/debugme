


# debugme

> Debug R Packages

[![Linux Build Status](https://travis-ci.org/gaborcsardi/debugme.svg?branch=master)](https://travis-ci.org/gaborcsardi/debugme)
[![Windows Build status](https://ci.appveyor.com/api/projects/status/github/gaborcsardi/debugme?svg=true)](https://ci.appveyor.com/project/gaborcsardi/debugme)
[![](http://www.r-pkg.org/badges/version/debugme)](http://www.r-pkg.org/pkg/debugme)
[![CRAN RStudio mirror downloads](http://cranlogs.r-pkg.org/badges/debugme)](http://www.r-pkg.org/pkg/debugme)


Specify debug messages as special string constants, and control debugging of
packages via environment variables.

## Installation and Usage

Install the package from GitHub:


```r
source("https://install-github.me/gaborcsardi/debugme")
```

To use `debugme` in your package, import it, and then add the following
`.onLoad` function to your package:
```r
.onLoad <- function(libname, pkgname) {
  debugme::debugme()
}
```

You can now add debug messages via character literals. No function calls
are necessary. For example:
```r
"!DEBUG Start up phantomjs"
private$start_phantomjs(phantom_debug_level)

"!DEBUG Start up shiny"
private$start_shiny(path)

"!DEBUG create new phantomjs session"
private$web <- session$new(port = private$phantom_port)

"!DEBUG navigate to Shiny app"
private$web$go(private$get_shiny_url())
```

The string literals are simply ignored when debugging is turned off. To
turn on debugging for a package, set the environment variable `DEBUGME` to
the package name you want to debug. E.g. from a `bash` shell:

```sh
export DEBUGME=mypackage
```

Or from within R:

```r
Sys.setenv(DEBUGME = "mypackage")
```

Separate multiple package names with commas:

```sh
export DEBUGME=mypackage,otherpackage
```

The debug messages will be prefixed by the package names, and assuming your
terminal supports color, will be colored differently for each package.

## Motivation

I have always wanted a debugging tool that
* is very simple to use,
* can be controlled via environment variables, without changing anything
  it the packages themselves,
* has zero impact on performance when debugging is off.

`debugme` is such a tool.

### Performance

Function calls are relatively cheap in R, but they still do have an impact.
If you never want to worry about the log messages making your code slower,
you will like `debugme`. `debugme` debug strings have practically no
performance penalty when debugging is off.

Here is a simple comparison between debugging with a function call, `f1()`,
debugging with debug strings, `f2()` and no debugging at all.


```r
debug <- function(msg) { cat(msg, file = "/dev/null", "\n") }
f1 <- function() {
  for (i in 1:100) {
    debug("foobar")
    Sys.sleep(.001)
  }
}
```


```r
f2 <- function() {
  for (i in 1:100) {
    "!DEBUG foobar"
    Sys.sleep(.001)
  }
}
```


```r
f3 <- function() {
  for (i in 1:100) {
    Sys.sleep(.001)
  }
}
```


```r
microbenchmark::microbenchmark(f1(), f2(), f3(), times = 10L)
```

```
#> Unit: milliseconds
#>  expr      min       lq     mean   median       uq      max neval cld
#>  f1() 153.4889 173.5416 172.8059 174.4914 176.3528 177.5417    10   b
#>  f2() 133.3437 134.5015 135.6928 135.6628 137.0485 138.1498    10  a 
#>  f3() 133.5342 134.4767 135.3831 134.8393 136.7823 137.1044    10  a
```

## License

MIT © [Gábor Csárdi](https://github.com/gaborcsardi)
