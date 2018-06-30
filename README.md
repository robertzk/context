# An R package for importing dependencies

Importing and managing dependencies in an R script can be a pain. We do not refer merely
to packages, but also to keeping mental track of the current implicit [search path](http://stat.ethz.ch/R-manual/R-devel/library/base/html/search.html)
and determining the origin of unannotated local and global variables.

The context package provides a simple harness for constructing R files in larger
projects that may require more careful management of dependencies as in
the definition above. The only package whose names are included by 
default is "base", whilst everything else must be specified explicitly.

This mirrors dependency inclusion in other general purpose languages such as
C++, Golang, Python, Java, etc.

## Example

```r
# custom_plot.R
# Translated from https://www.rdocumentation.org/packages/fields/versions/9.6/topics/ribbon.plot
from("cran:fields^9.6") %import% c("ribbon.plot" = "ribbon")
from("graphics") %import% c("plot", "contour")
from("graphicDevices") %import% "trans3d"
import("graphics") %as% "g"
from("github.com/robertzk/s3mpi") %import% c("s3store" = "%s3>%") # or %import% "s3store" %as% "%s3>%"

plot(c(-1.5,1.5), c(-1.5,1.5), type = "n")
temp <- list(x = seq( -1, 1,, 40), y = seq( -1, 1,, 40))
temp$z <- outer(temp$x, temp$y, "+")
contour(temp, add = TRUE)

t <- seq(0, 0.5,, 50)
y <- sin(2 * pi * t)
x <- cos(pi * t)
z <- x + y

ribbon(x, y, z, lwd = 10)

pm <- g::persp(temp, phi = 15, shade = 0.8, col = "grey") 
uv <- trans3d(x, y, z, pm)
rd <- ribbon(uv$x, uv$y, z ** 2, lwd = 5)
rd %s3>% paste0("ribbon_data", format(Sys.time(), "%Y%M%d"))
```

We can execute the file from the global console using

```r
context::include_file("custom_plot.R")
context:::source("custom_plot.R") # Equivalent
```

## Internals

Under the hood, context uses [lockbox](https://github.com/robertzk/lockbox) for ensuring
that package dependencies are handled in an isolated way from the global `.libPaths()`.

Aliased names surrounded with percentage symbols, such as `c("s3store" = "%s3>%")` above,
are converted to an infix operator if the function is arity 2 or above..

In the example above, an imports "layer" is constructed from the `from` and `import`
statements. This is an environment whose parent environment is the base environment.
