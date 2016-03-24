# Starting an R Package

This is a hands-on tutorial on how to start an R package using RStudio.
The R package will be fully featured, that means:

- We can use it through `library()`
- We will provide Roxygen API documentation vignettes
- We will provide unit tests

Only uploading to CRAN is left open.

## Prerequisites

- Install stable version of RStudio (tested with 0.99.893)
- Install an up-to-date version of R (tested with 3.2.3)
- Install C/C++ compiler (e.g. GCC/Clang)
- Install Latex (required for building docs)

## Creating the Project

- File > Create Project
- New Directory > R Package
- Package name "leftpad"
- Change parent directory of the package
- Click "Create Project"
- Build the project via Build > Build and Reload (Ctrl+Shift+B)
- Type `hello()` at the command line prompt

That's it!

## Use testthat for testing

We will follow the mantra "what would Hadley Wickham do?" and use his `testthat` package for testing.
For this, we first install the `devtools` package that helps us modify our package.
Also, we install some dependencies used in testing.

```
> install.packages("devtools")
> install.packages("testthat")
> install.packages("roxygen2")
```

Then, we modify our package to use `testthat` by typing

```
> devtools::use_testthat()
```

This will automatically create the `tests/testthat` directory for tests, add `testthat` to the `Suggest` field in the `DESCRIPTION`, and create `tests/testthat.R` that runs the tests on `R CMD check`.

Clicking Build > Test Package (Ctrl+Shift+T) complains that no test files were found.
For this, we create the file `tests/testthat/test_hello.R` and fill it with the following content

```
library(leftpad)
context("Greeting")

test_that("hello() returns the correct value", {
  expect_equal(hello(), "Hello, world!");
})
```

Next, run the tests and the top right pane should read as follows.

```
==> devtools::test()

Loading leftpad
Loading required package: testthat
Testing leftpad
Greeting : [1] "Hello, world!"
.

DONE 
```

Yay, testing has been setup successfully.

## Implementing `leftpad()`

Apparently, left-padding strings (similar to `sprintf`) is important and [some software stacks break down](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm) if packages implementing this are removed.
Let's make sure that we implement our own `leftpad` just in case we need it in our R projects.

Rename `R/hello.R` to `R/leftpad.R` and fill it with the following

```
leftpad <- function(value, cols, fill = ' ') {
  result = value;
  while (nchar(result) < cols) {
    result = paste(fill, result, sep='', collapse='');
  }
  result;
}
```

Next, rename `tests/testthat/test_hello.R` to `tests/testthat/test_leftpad.R`.

```
library(leftpad)
context("left padding")

test_that("leftpad works with space", {
  expect_equal(leftpad('test', 10), "      test");
})

test_that("leftpad works with zeros", {
  expect_equal(leftpad('test', 10, 0), "000000test");
})

test_that("leftpad does not overpad", {
  expect_equal(leftpad('test', 2), "test");
})
```

After running the test, the "Build" window at the top right showould read as follows:

```
==> devtools::test()

Loading leftpad
Loading required package: testthat
Testing leftpad
left padding : ...

DONE 
```

## Adding Documentation

We're almost done.
Try Build > Check Package.
This triggers a more comprehensive test that includes building the documentation.
You will see it fail, with a message ending in something like this:

```
...
* DONE
Status: 1 ERROR, 3 WARNINGs, 1 NOTE

See
  ‘/home/manuel/Development/r_module_demo/leftpad.Rcheck/00check.log’
for details.

Error: Command failed (1)
Execution halted

Exited with status 1.
```

How can we resolve this?
Quite simply, above there is the error message `Error: could not find function "hello"`, so there appears to be some of the initially generated stuff around.
Remove the file `man/hello.Rd`.
Afterwards, the check will run through.
However, our package is missing a very important thing: the documentation.

For this, we use Roxygen, a tool similar to Javadoc or Doxygen, that uses specially formatted comments in the code for generating the R documentation files (`.Rd`).

Head back to the file `leftpad.R`, set the cursor into the `leftpad` function and click Code > Insert Roxygen Skelleton.
Update the code to look like this:

```
#' Left-padd a string to a given number of characters
#'
#' @param value The value to left-padd
#' @param cols  The length of the string that we aim for.
#' @param fill  Optionally, a character to use for filling.
#'
#' @return Value padded to cols characters
#' @export
#'
#' @examples
#' leftpad(3, 10)
#' # "         3"
#' leftpad('test', 2)
#' # "test"
#' leftpad(3, 10, 0)
#' # "0000000003"
leftpad <- function(value, cols, fill = ' ') {
  result = value;
  while (nchar(result) < cols) {
    result = paste(fill, result, sep='', collapse='');
  }
  result;
}
```

Then, activate building the documentation of the project with Roxygen, Tools > Project Options.
There, select "Build Tools" and check "Generate documentation with Roxygen".
For now, check all boxes in the window "Roxygen Options" that opens and click OK, then again OK.

Note that the `@export` tag is very important as it will make Roxygen update your project files such that this symbol is exported by the package and made available to the package users.

You can now also use `?leftpad` on the prompt to display the documentation we just updated above.
Now, open the `DESCRIPTION` file and make it look like this.

```
Package: leftpad
Type: Package
Title: Left-pad strings like it's 2016
Version: 0.1.0
Author: John Doe <john.doe@example.com>
Maintainer: John Doe <john.doe@example.com>
Description: Left-pad strings. Enough said.
License: Public Domain
LazyData: TRUE
Suggests:
    testthat
RoxygenNote: 5.0.1
```

## Generate Package Tarball

You can now generate a package tarball by using Build > Build Source Package.

And we're done.

## References and Further Reading

- https://support.rstudio.com/hc/en-us/articles/200486488-Developing-Packages-with-RStudio
- https://support.rstudio.com/hc/en-us/articles/200486508-Building-Testing-and-Distributing-Packages
- http://r-pkgs.had.co.nz/tests.html
- https://cran.r-project.org/web/packages/roxygen2/vignettes/roxygen2.html
- http://r-pkgs.had.co.nz/data.html
