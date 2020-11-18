
<!-- README.md is generated from README.Rmd. Please edit that file -->

# tidybrowse <img src="man/figures/tidybrowse_logo.jpeg" width="400px" align="right" />

[![](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![](https://img.shields.io/github/languages/code-size/benjaminguinaudeau/tidybrowse.svg)](https://github.com/benjaminguinaudeau/tidybrowse)
[![](https://img.shields.io/github/last-commit/benjaminguinaudeau/tidybrowse.svg)](https://github.com/benjaminguinaudeau/tidybrowse/commits/master)

Tidybrowse is a meta package containing different packages easing web
scrapping and the deployment of docker containers from R.

## Examples

A list of use-cases

-   [Searching Citations on Google
    Scholar](example/cleaning_citations.md): Use Selenium to scrape
    citations from Google Scholar

## Installation

``` r
# install.packages("devtools")
devtools::install_github("benjaminguinaudeau/tidybrowse")
```

## Packages

### [dockeR](https://github.com/benjaminguinaudeau/dockeR)

dockeR wraps up docker command line tools and allows to manage docker
containers from R. It can be use to deploy selenium servers, shiny-app
servers, rstudio-servers, etc…

### [tidyselenium](https://github.com/benjaminguinaudeau/tidyselenium)

This wraps up RSelenium function in a pipable way. It also offers
function to easily communicate with a selenium server running inside a
docker container.

### [tidyweb](https://github.com/benjaminguinaudeau/tidyweb)

Tidyweb allows to represent xml-tree data in a tidy way. It works as
well with xml-nodes as with selenium elements.

### [selinput](https://github.com/benjaminguinaudeau/selinput)

Selinput wraps up the python library pyautogui, which emulates mouse and
keyboard input. It allows to easily type, click and scroll inside a
docker container, with a running selenium server.

``` r
library(tidybrowse)
#> ── Attaching packages ────────────────────────────────────── tidybrowse 0.0.1 ──
#> ✓ RSelenium    1.7.7          ✓ selinput     0.0.0.9000
#> ✓ dockeR       0.1.0          ✓ bashR        0.0.0.9000
#> ✓ tidyselenium 0.0.0.9000     ✓ hideR        0.0.0.9000
#> ✓ tidyweb      0.0.0.9000     ✓ rvest        0.3.6
#> ── Conflicts ───────────────────────────────────────── tidybrowse_conflicts() ──
#> x bashR::exec()    masks dockeR::exec()
#> x dockeR::system() masks base::system()
```

## Thanks

A huge thank you to [Favstats](https://github.com/favstats) for
designing each of the hex-stickers.
