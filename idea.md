##### Rename in scope

Also rename for glue(" {to_rename}")

#### 
Fehler in glue("{SteveAI_dir}/scraper_rvest.RData") : 
  konnte Funktion "glue" nicht finden
  
 Auto check for installed packages and offer to load package to namespace and re-try line of code.

#### optimize create function from code

related:
https://github.com/rstudio/rstudio/issues/5160

As a potential improvement I would add function calls from the pipe operator:

  Input:

  `mtcars %>% cat`

**Received output:**

  ```
asd <- function(mtcars, cat) {
  mtcars %>% cat
}
```

**Expected output:**

  ```
asd <- function(mtcars) {
  mtcars %>% cat
}
```

**Attempt to solve / Preperation of PR:**

  If i look in the function definition of the pipe operator `?magrittr::`%>%`` (here from the magrittr package), the usage of the pipe operator is defined as `lhs %>% rhs`, where the argument `rhs` is defined as

> A function call using the magrittr semantics.

So, by definition, everything following the pipe operator is a function and not a value.

code <- "a <- 1
mtcars %>% mean
b <- 2"


**Arguments against this implementation:**

  It could be the case that the argument `rhs` is a custom function that should be passed as an argument to the newly defined function.  From my point of view that would be rather the exception.

display_mtcars <- function(mtcars, cat) {
  mtcars %>% cat()
}



asd <- function(mtcars, mean) {
  mtcars %>% mean()
}



code <- "a <- 1
  mtcars %>% mean %>% mean(1)
mtcars %>%
var %>%
var(1)
b <- 2"

library(magrittr)
Exclude_Pipe_Matches <- function(code) {
  lines <- code %>%
    strsplit(split = "\n") %>%
    unlist()

  pipe_Match_Idx <- lines %>%
    grepl(pattern = "%>%") %>%
    which()

  rhs_matches <- lines[pipe_Match_Idx] %>%
    lapply(FUN = function(line) {
      strsplit(x = line, split = "%>%") %>%
        unlist() %>%
        .[-1] %>%
        trimws()
    })


  for (nr in seq(rhs_matches)) {
    code_On_Next_Line <- rhs_matches[[nr]] %>%
      nchar() %>%
      sum() %>%
      magrittr::not()

    if (code_On_Next_Line) {
      rhs_matches[[nr]] <- lines[pipe_Match_Idx[nr] + 1] %>%
        strsplit(split = "%>%") %>%
        unlist() %>%
        .[1] %>%
        trimws()
    }
  }

  falsePositives <- rhs_matches %>% unlist()

  falsePositives
}

# sample code - not supposed to make sense
code <- "a <- 1
  mtcars %>% mean %>% mean(1)
mtcars %>%
var %>%
var(1)
b <- 2"

code %>% Exclude_Pipe_Matches()
