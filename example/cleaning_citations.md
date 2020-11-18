Searching Citations on Google Scholar
================
Benjamin Guinaudeau
18. November 2020

In this example, I use `tidybrowse` to set up a docker container on
which a selenium server is running. Selenium emulate a web-browser (here
Chrome) and allows you to navigate websites as if you were a real user.
Thanks to selenium, you can overcome many limitations of `rvest` and
scrape websites which use javascript in the background.

In this example, I navigate google scholar and automatically extract
citations and the BibTex files.

## Background

Recently, I worked on an academic project with a colleague, who was not
fluent in latex or markdown. We relied on the usual common denominator:
MS Word. While editing the manuscript, we manually added references at
the end of the document using APA-Style citations. But, when we were
about to submit the paper to a journal, we realized that Chicago
citation-style was required.

Instead of manually editing each of the references, I wrote this small
script, which does two things: 1. Search on google scholar for any query
(DOI, Citation, Author, etc…) 2. Save the clean citations proposed by
scholar (APA, ISO 609 and MLA) as well as the underlying BibTex file.

## Packages

``` r
library(tidybrowse)
```

## A couple of helpers

``` r
# Navigate to google scholar page
go_to_citation <- function(chrome, citation){
  
  # Create query
  query <- glue::glue('https://scholar.google.com/scholar?hl=fr&as_sdt=0%2C5&q={stringr::str_replace_all(citation, "\\\\s+", "+")}&btnG=')
  
  # Navigate to url
  chrome %>% go(query)
  Sys.sleep(.5)
  
  # Happens once every session
  # you need to solve the captcha and it will not come back
  while(tidyselenium::check_element(chrome, '#gs_captcha_c')){
    message("Solve the captcha in the VNC window\nWaiting 20 seconds")
    Sys.sleep(20)
    # readline("Before continuing, you need to solve a captcha in the VNC window.
    #         Once you're done, press 'enter'")
  }
  
  # Find all citations button
  elems <- chrome %>%
    tidyselenium::elements('*[role="button"]') %>%
    tidyselenium::filter.webElement("title", "Citer") 
  
  # Click the button (or the first one if several results are provided)
  if(length(elems) > 1) elems <- elems[[1]]
  elems %>% tidyselenium::click() 
  
  return(chrome)
}


# Access the three proposed citations and the bibtex info
get_citation_info <- function(chrome){
  
  # Get proposed citations
  cites <- chrome %>%
    tidyselenium::elements(".gs_citr") %>%
    purrr::map_chr(get_text)
  
  # Find link to BibTex file
  link <- chrome %>%
    tidyselenium::elements("*[href]") %>%
    purrr::map_chr(tidyselenium::get_attribute, "href") %>%
    stringr::str_subset("scholar.bib")
  
  # Read BibText file
  bib <- try(paste(readLines(link), collapse = "\n"))
  
  if(inherits(bib, "try-error")){
    bib <- paste(readLines(link), collapse = "\n")
  }
  
  return(list(cites = cites, bib = bib))
  
}

# Parse the results and returns a tibble
parse_gs_res <- function(gs_res){
  gs_res$cites %>% 
    purrr::map2_dfc(c("apa", "iso_690", "mla"), ~tibble::tibble(.x) %>% purrr::set_names(.y)) %>%
    dplyr::mutate(bib = gs_res$bib)
}
```

## Init docker container

You need a running docker daemon in the background. For this, docker
must be installed and the docker daemon must be started.

``` r
# This function will create/start a container named "chrome" and 
# returns a Selenium Driver object associated with this container
chrome <- tidyselenium::chrome_init("chrome", view = F)
```

Scraping is really easier if you can observe what is happening under the
hood. Hence, the container also hosts a vnc server, that allows you to
view and interact in real time with the emulated browser.

``` r
# This opens vnc viewer and displays the selenium browser
dockeR::view_container("chrome")
# To know the port used by the vnc server run:
# dockeR::get_port("chrome", 5900)
```

## Read citations

For this example, I simly some references from the word document into
`citation.txt`. Each row corresponds to one full-text citation, but you
could paste any kind of keywords (DOI, Author, Keyword, etc…).

``` r
word_citation <- readLines("citation.txt") %>%
  dplyr::glimpse()
```

    ##  chr [1:4] "Baumgartner, Frank R, and Bryan D Jones. 2010.Agendas and Instability in American Politics. University of Chicago Press." ...

## Scraping

``` r
# Open a new window on selenium container
chrome %>%
  tidyselenium::new_window()

citation_dt <- word_citation %>% 
  purrr::map_dfr(~{
    cli::cli_alert_info(.x)
    
    out <- try({
      chrome %>%
        go_to_citation(.x) %>%
        get_citation_info 
    })
    
    if(inherits(out, "try-error")){
      
      cli::cli_alert_danger("Could not retrieve citation")
      return(tibble::tibble())
      
    } else {
      
      cat(out$bib)
      message("\n\n")
      return(parse_gs_res(out))
      
      Sys.sleep(2)
      
    }
  }) %>%
  dplyr::glimpse()
```

    ## ℹ Baumgartner, Frank R, and Bryan D Jones. 2010.Agendas and Instability in American Politics. University of Chicago Press.

    ## Solve the captcha in the VNC window
    ## Waiting 20 seconds
    ## Solve the captcha in the VNC window
    ## Waiting 20 seconds
    ## Solve the captcha in the VNC window
    ## Waiting 20 seconds

    ## @article{baumgartner2014punctuated,
    ##   title={Punctuated equilibrium theory: Explaining stability and change in public policymaking},
    ##   author={Baumgartner, Frank R and Jones, Bryan D and Mortensen, Peter B},
    ##   journal={Theories of the policy process},
    ##   volume={8},
    ##   pages={59--103},
    ##   year={2014},
    ##   publisher={Westview Press Boulder, CO}
    ## }

    ## 

    ## ℹ Bergman, Torbjörn, Svante Ersson, and Johan Hellström. 2015. “Government Formation and Breakdown in Western and Central Eastern Europe.”Comparative European Politics13 (3): 345–75.

    ## @article{bergman2015government,
    ##   title={Government formation and breakdown in Western and Central Eastern Europe},
    ##   author={Bergman, Torbj{\"o}rn and Ersson, Svante and Hellstr{\"o}m, Johan},
    ##   journal={Comparative European Politics},
    ##   volume={13},
    ##   number={3},
    ##   pages={345--375},
    ##   year={2015},
    ##   publisher={Springer}
    ## }

    ## 

    ## ℹ Beyer, Daniela. 2018. “The Neglected Effects of Europeanization in the Member States–Policy-Making inDirectly EU-Influenced and Sovereign Domains.”Journal of European Public Policy 25 (9): 1294–1316.

    ## @article{beyer2018neglected,
    ##   title={The neglected effects of Europeanization in the member states--policy-making in directly EU-influenced and sovereign domains},
    ##   author={Beyer, Daniela},
    ##   journal={Journal of European Public Policy},
    ##   volume={25},
    ##   number={9},
    ##   pages={1294--1316},
    ##   year={2018},
    ##   publisher={Taylor \& Francis}
    ## }

    ## 

    ## ℹ Boix, Carles. 2000. “Partisan Governments, the International Economy, and Macroeconomic Policies inAdvanced Nations, 1960-93.”World Politics, 38–73

    ## @article{boix2000partisan,
    ##   title={Partisan governments, the international economy, and macroeconomic policies in advanced nations, 1960-93},
    ##   author={Boix, Carles},
    ##   journal={World politics},
    ##   pages={38--73},
    ##   year={2000},
    ##   publisher={JSTOR}
    ## }

    ## 

    ## Rows: 4
    ## Columns: 4
    ## $ apa     <chr> "Baumgartner, F. R., Jones, B. D., & Mortensen, P. B. (2014).…
    ## $ iso_690 <chr> "BAUMGARTNER, Frank R., JONES, Bryan D., et MORTENSEN, Peter …
    ## $ mla     <chr> "Baumgartner, Frank R., Bryan D. Jones, and Peter B. Mortense…
    ## $ bib     <chr> "@article{baumgartner2014punctuated,\n  title={Punctuated equ…

``` r
# Close all opened windows
chrome %>%
  tidyselenium::close_all()
```

## Export as .bib

You can now write a bibtex file and use it in a latex document. For
markdown users, I’ve just discovered that you can cite all your
bibliography by adapting the yaml-header ([More
info](https://github.com/rstudio/rmarkdown/issues/1096)).

``` r
citation_dt$bib %>%
  paste(collapse = "\n\n") %>%
  readr::write_lines("mandate.bib")
```

## Read proposed citation

Google scholar proposed three citation format. You can also print them
and directly copy them back into the word document.

``` r
citation_dt$mla %>%
  sort %>%
  paste(collapse = "\n\n") %>%
  cat
```

    ## Baumgartner, Frank R., Bryan D. Jones, and Peter B. Mortensen. "Punctuated equilibrium theory: Explaining stability and change in public policymaking." Theories of the policy process 8 (2014): 59-103.
    ## 
    ## Bergman, Torbjörn, Svante Ersson, and Johan Hellström. "Government formation and breakdown in Western and Central Eastern Europe." Comparative European Politics 13.3 (2015): 345-375.
    ## 
    ## Beyer, Daniela. "The neglected effects of Europeanization in the member states–policy-making in directly EU-influenced and sovereign domains." Journal of European Public Policy 25.9 (2018): 1294-1316.
    ## 
    ## Boix, Carles. "Partisan governments, the international economy, and macroeconomic policies in advanced nations, 1960-93." World politics (2000): 38-73.
