---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Dplyr functions with string"
subtitle: ""
summary: "A trick to use string as arguments in dplyr functions"
authors: [Yihui Fan]
tags: [R, dplyr]
categories: [Data Science]
date: 2019-11-01T14:30:17Z
lastmod: 2019-11-01T14:30:17Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Let’s say we have a simple data frame as below and we want to select the
female rows only.

``` r
df <- data.frame(id = c(1, 2, 3, 4, 5), 
                 gender = c("male", "female", "male", "female", "female"))

df
```

    ##   id gender
    ## 1  1   male
    ## 2  2 female
    ## 3  3   male
    ## 4  4 female
    ## 5  5 female

``` r
library(dplyr)

df %>%
    filter(., gender == "female")
```

    ##   id gender
    ## 1  2 female
    ## 2  4 female
    ## 3  5 female

The `filter()` function in `dplyr` (and other similar functions from the
package) use something called non-standard evaluation (NSE). In NSE,
names are treated as string literals. So just using ‘gender’ (without quotes) in the
function above works fine. This is in contrast with functions using
standard evaluation (SE). For example, the following code of indexing column
in a data frame will give an error.

``` r
df[, gender] # this will give error
```

    ## Error in `[.data.frame`(df, , gender): object 'gender' not found

This is because data frame indexing with `[]` uses SE, in which names
are treated as references to values. To make this work we will need to
pass a string.

``` r
df[, "gender"] # this works
```

    ## [1] male   female male   female female
    ## Levels: female male

This behaviour of `dplyr` is usually quite helpful as it makes the
expression succinct. However, occasionally we want to pass a string to
the function, for example, if we use the `filter()` function in a loop
or part of a more complicated function. Because of NSE, the codes below
will not work. Note that it doesn’t throw an error but give 0 rows. This
is because the function takes ‘var’ literally and couldn’t find it in
the data frame!

``` r
var <- "gender" # this is a string
val <- "female" # this is a string

df %>%
    filter(., var == val)
```

    ## [1] id     gender
    ## <0 rows> (or 0-length row.names)

To make this work, we will need a trick like below. The `sym()` function
convert a string to symbol, contrarily to `as.name()`. Then use the
`!!` to say that you want to unquote an input so that it’s evaluated, not
quoted.

``` r
var <- "gender" # this is a string
val <- "female" # this is a string

df %>%
    filter(., !!sym(var) == val)
```

    ##   id gender
    ## 1  2 female
    ## 2  4 female
    ## 3  5 female

Alternatively, you can use the `get()` function, which returns the value
of a named object.

``` r
var <- "gender" # this is a string
val <- "female" # this is a string

df %>%
    filter(., get(var) == val)
```

    ##   id gender
    ## 1  2 female
    ## 2  4 female
    ## 3  5 female

The first method addresses the NSE of the `filter()` function while the
second method tricks it to get the job done. Both work just fine.

If we don't want to pass a string but a name instead, the `tidyverse` has recently introduced a `{{}}` (#curly-curly') operator for tidy evaluation.

```{r}
library(tidyverse)

squirrels <- read_csv(str_c(
  "https://raw.githubusercontent.com/",
  "rfordatascience/tidytuesday/master/",
  "data/2019/2019-10-29/nyc_squirrels.csv"))

count_groups <- function(df, groupvar){
  df %>%
    group_by({{ groupvar }}) %>%
    count()
}

count_groups(squirrels, climbing)
```

See now we can pass the variable name `climbing` to the `group_by` function using the `{{}}` operator. In the past, we have to use the more cumbersome `!! enquo` (quote-unquote) trick to achieve somthing similar. 

```{r}
count_groups_old <- function(df, groupvar){
  df %>%
    group_by(!! enquo(groupvar)) %>%
    count()
}

count_groups_old(squirrels, climbing)
```

Happy hacking!