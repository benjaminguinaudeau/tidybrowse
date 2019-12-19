
<!-- README.md is generated from README.Rmd. Please edit that file -->

# tidybrowser

<!-- badges: start -->

<!-- badges: end -->

Tidybrowser is a meta package containing different packages easing web
scrapping and the deployment of docker containers from R.

## Installation

``` r
# install.packages("devtools")
devtools::install.packages("benjaminguinaudeau/tidybrowser")
```

## Packages

### dockeR

dockeR wraps up docker command line tools and allows to manage docker
containers from R. It can be use to deploy selenium servers, shiny-app
servers, rstudio-servers, etc…

### tidyselenium

This wraps up RSelenium function in a pipable way. It also offers
function to easily communicate with a selenium server running inside a
docker container.

### tidyweb

Tidyweb allows to represent xml-tree data in a tidy way. It works as
well with xml-nodes as with selenium elements.

### selinput

Selinput wraps up the python library pyautogui, which emulates mouse and
keyboard input. It allows to easily type, click and scroll inside a
docker container, with a running selenium server.

### bashR

bashR allows to run some simple bash functions from whithin R.
<<<<<<< HEAD

``` r
library(tidybrowser)
#> ── Attaching packages ──────────────────────────────────────────────────── tidybrowser 0.0.1 ──
#> ✓ RSelenium    1.7.5          ✓ selinput     0.0.0.9000
#> ✓ dockeR       0.1.0          ✓ bashR        0.0.0.9000
#> ✓ tidyselenium 0.0.0.9000     ✓ dplyr        0.8.3     
#> ✓ tidyweb      0.0.0.9000     ✓ usethis      1.5.1
#> ── Conflicts ─────────────────────────────────────────────────────── tidybrowser_conflicts() ──
#> x bashR::append() masks base::append()
#> x bashR::exec()   masks dockeR::exec()
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
```
=======
>>>>>>> 845aad7f7710f8a11e671c550efecbc2a952eda3
