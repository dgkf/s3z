
# Easy, Peasy, `s3z`

A package to streamline S3-style authoring and dispatch

# Motivation

> S3 is already easy!

Syntactically, yes. Conceptually, yes. In practice, sorta. S3 (and S4)
dispatch still requires that a namespace is the central focal point to
aggregate methods and assumes that all methods are layered upon a
singular, central generic. The R world has devised different strategies
for working around this issue:

-   Creating “generics” packages, which are intended to be clear focal
    points for others to implement methods on top of. (examples:
    `generics`, `universals`)

-   Prefixing packages with abbreviations (examples: `vctrs::vec_`,
    `forcats::fct_*`, `stringi::stri_*`).

-   Embed type information in a function name to differentiate
    functions.

These are all considered best practices in one community or another,
largely because they circumvent conflicts among packages.

Perhaps there was a time in R’s history where one could keep tabs on all
the available packages and the generics that exist across its landscape.
As the number of available packages grows, this becomes more and more
challenging. Instead of keeping tabs on all the available generics, it
would be preferred if the generics systems didn’t require such
historical knowledge.

This work is an effort to explore methods to make it easy to develop
methods in isolation, without requiring accute awareness of what other
packages may be leveraging the same generic function names.

# Project Goals

Although I think it would be awesome if a system like this became mature
enough that it could be widely adopted, this isn’t intended to be that
project (yet!). For now, this is just a proof-of-concept, to explore
first whether this feature set is possible, and to explore what
trade-offs it would require.

# Functional Goals

Specifically, an ideal implementation would allow for a few key use
cases:

## 1. Isolated generics

*A package’s generic methods should be functional in isolation*

A package which provides a method for a generic (which may also be used
by other packages), should be able to use that method independently.

``` r
library(s3z.first.chr)

first(letters)
# [1] "a"
```

> This one is easy with S3, simply export the generic `first` and the
> method `first.character`.

## 2. Permits duplicate generics

*Packages may provide methods for the same generic*

``` r
library(s3z.first.chr)
library(s3z.first.list)

first(letters)
# [1] "a"

first(list(1, 2, 3))
# [[1]]
# [1] 1
```

> Already we’d run into hot water with S3. Since both packages aim to
> work in isolation, they both would have to export `first`.
> `s3z.first.list` would have to export its method both as a standalone
> S3 method, and as an S3 method to `s3z.first.chr`’s flavor of `first`.
> This would work, but we’ve already fragmented our ecosystem for any
> future packages.

## 3. Namespace method access

*Ensure packages using these generics can access methods*

Of course, internally, if a package is using generic methods it should
also be able to operate on new datatypes that implement those methods.
For this, the methods need to be registered as well in the affected
package namespaces.

``` r
library(s3z.first)
library(s3z.last.chr)

first.default
# function(x, ...)
# last(rev(x))
# <environment namespace:s3z.first>

first(letters)
# [1] "a"
```

## 4. Method masking

*Methods should conform to a consistent logic of method masking.*

There are a few key scenarios where masking of the generic itself or one
of its methods should produce predictable results:

-   New methods for identical signatures should mask existing methods
-   Unloading packages should restore any behaviors it had masked

## Bonus Goals

Although I think these four goals would make for a compelling
implementation, there are extension features that I would also like to
explore.

### 5. Existing non-generic methods automatically become `.default` methods

To extend the method masking behaviors, one could consider how
non-generic methods might compose with such generics. Perhaps

-   Masking a non-generc with a generic should relegate the existing
    non-generic method to a `.default` method if one isn’t provided
-   New non-generic methods should mask the `.default` method if one
    exists

``` r
library(stats)
library(s3z.dressup)

hat
# function (x, ...)
# UseMethod("hat")

hat.default
# function (x, ...)
# [...]
# <environment: namespace:stats>

hat(person("Bugs", "Bunny", comment = list(hat = "tophat")))
# [1] "tophat"
```

### 6. Extended type signature dispatch

S4 is great for its ability to dispatch on multiple arguments. If we’re
already reinventing wheels, this would be an awesome ability to bake in
(although certainly not a priority at the outset).
