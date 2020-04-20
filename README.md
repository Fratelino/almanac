
<!-- README.md is generated from README.Rmd. Please edit that file -->

# almanac

<!-- badges: start -->

[![Codecov test
coverage](https://codecov.io/gh/DavisVaughan/almanac/branch/master/graph/badge.svg)](https://codecov.io/gh/DavisVaughan/almanac?branch=master)
[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![R build
status](https://github.com/DavisVaughan/almanac/workflows/R-CMD-check/badge.svg)](https://github.com/DavisVaughan/almanac/actions)
<!-- badges: end -->

``` r
library(almanac)
```

almanac provides tools for working with recurrence rules, the
fundamental building blocks used to identify calendar “events”, such as
weekends or holidays. Constructing recurrence rules looks a little like
this:

``` r
# Thanksgiving = "The fourth Thursday in November"
on_thanksgiving <- yearly() %>% 
  recur_on_ymonth("November") %>%
  recur_on_wday("Thursday", nth = 4)
```

After constructing a recurrence rule, it can be used to generate dates
that are in the “recurrence set”.

``` r
alma_search("2000-01-01", "2006-01-01", on_thanksgiving)
#> [1] "2000-11-23" "2001-11-22" "2002-11-28" "2003-11-27" "2004-11-25"
#> [6] "2005-11-24"
```

Or determine if a particular date is a part of the recurrence set.

``` r
alma_in(c("2000-01-01", "2000-11-23"), on_thanksgiving)
#> [1] FALSE  TRUE
```

It also allows you to shift an existing sequence of dates, “stepping
over” dates that are in the recurrence set.

``` r
wednesday_before_thanksgiving <- "2000-11-22"

# Step forward 2 non-event days, stepping over thanksgiving
alma_step(wednesday_before_thanksgiving, n = 2, on_thanksgiving)
#> [1] "2000-11-25"
```

By combining multiple recurrence rules together, you can create larger
*recurrence bundles* that form a collection of holidays or events, such
as weekends. almanac comes with a pre-built `calendar_usa_federal()`
recurrence bundle that includes recurrence rules for the US holidays.
Combining this with a recurrence rule of “on weekends” constructs a
business calendar, which we can use to shift dates by, for example, “2
business days”.

``` r
on_weekends <- weekly() %>% 
  recur_on_weekends()

on_us_holidays_and_weekends <- calendar_usa_federal() %>%
  add_rrule(on_weekends)

thursday_and_friday_before_labor_day <- c("2019-08-29", "2019-08-30")

# Steps over Saturday, Sunday, and Labor Day to the following Tuesday
# and Wednesday, aka "two business days" from now!
two_business_days_forward <- alma_step(
  thursday_and_friday_before_labor_day, 
  n = 2,
  rbundle = on_us_holidays_and_weekends
)

two_business_days_forward
#> [1] "2019-09-03" "2019-09-04"

lubridate::wday(two_business_days_forward, label = TRUE)
#> [1] Tue Wed
#> Levels: Sun < Mon < Tue < Wed < Thu < Fri < Sat
```

## Learning More

View the vignettes on [the
website](https://davisvaughan.github.io/almanac/index.html) to learn
more about how to use almanac.

  - `vignette("almanac")`

  - `vignette("adjust-and-shift")`

  - `vignette("quarterly")`

  - `vignette("icalendar")`

## Installation

You can NOT install the released version of almanac from
[CRAN](https://CRAN.R-project.org) with:

``` r
# NO! install.packages("almanac")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("DavisVaughan/almanac")
```

Mac (OS-X) and Windows users should not have any problems installing
almanac. Linux users need libv8 to install the dependency R package, V8.
See the [V8 installation
instructions](https://github.com/jeroen/V8#debian--ubuntu) for more
information. almanac uses ES5 JavaScript, so it does *not* require any
“modern” JavaScript features and should work with the “old” V8 engine
provided by Ubuntu versions before 19.04.

## Acknowledgements

almanac has developed as a composite of ideas from multiple different
libraries.

First off, it directly embeds the *amazing* JavaScript library
[rrule](https://github.com/jakubroztocil/rrule) for the core recurrence
set calculations. To do this, it uses the equally awesome R package,
[V8](https://github.com/jeroen/V8), from Jeroen Ooms.

The date shifting / adjusting functions are modeled after similar
functions in [QuantLib](https://github.com/lballabio/QuantLib).

The fast binary search based implementations of `alma_next()` and
`alma_step()` are inspired by Pandas and the implementation of Numpy’s
[busday\_offset()](https://docs.scipy.org/doc/numpy/reference/generated/numpy.busday_offset.html).

The author of [gs](https://github.com/jameslairdsmith/gs), James
Laird-Smith, has been a great collaborator as we have bounced ideas off
of each other. gs attempts to solve a similar problem, but with a
slightly different implementation.