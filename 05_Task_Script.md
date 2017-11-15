Task 05: Prediction Model
================
Mark Blackmore
2017-11-14

setup

``` r
go <- Sys.time()
suppressPackageStartupMessages({
  library(tidyverse)
  library(stringr)
})

# Training Data
bi_words <- readRDS("./clean_repos/bi_words_fast.rds")
tri_words  <- readRDS("./clean_repos/tri_words_fast.rds")
quad_words <- readRDS("./clean_repos/quad_words_fast.rds")

# Ngram Matching Functions
bigram <- function(input_words){
                    num <- length(input_words)
                    filter(bi_words, 
                          word1==input_words[num]) %>% 
                    top_n(1, n) %>%
                    filter(row_number() == 1L) %>%
                    select(num_range("word", 2)) %>%
                    as.character() -> out
                    ifelse(out =="character(0)", "?", return(out))
}

trigram <- function(input_words){
                    num <- length(input_words)
                    filter(tri_words, 
                            word1==input_words[num-1], 
                            word2==input_words[num])  %>% 
                    top_n(1, n) %>%
                    filter(row_number() == 1L) %>%
                    select(num_range("word", 3)) %>%
                    as.character() -> out
                    ifelse(out=="character(0)", bigram(input_words), return(out))
}

quadgram <- function(input_words){
                    num <- length(input_words)
                    filter(quad_words, 
                            word1==input_words[num-2], 
                            word2==input_words[num-1], 
                            word3==input_words[num])  %>% 
                    top_n(1, n) %>%
                    filter(row_number() == 1L) %>%
                    select(num_range("word", 4)) %>%
                    as.character() -> out
                    ifelse(out=="character(0)", trigram(input_words), return(out))
}
```

Data Cleaning and Input Function; Calls the matching functions

``` r
ngrams <- function(input){
  # Create a dataframe
  input <- data_frame(text = input)
  # Clean the Inpput
  replace_reg <- "[^[:alpha:][:space:]]*"
  input <- input %>%
    mutate(text = str_replace_all(text, replace_reg, ""))
  # Find word count, separate words, lower case
  input_count <- str_count(input, boundary("word"))
  input_words <- unlist(str_split(input, boundary("word")))
  input_words <- tolower(input_words)
  # Call the matching functions
  y <- ifelse(input_count == 1, bigram(input_words), 
              ifelse (input_count == 2, trigram(input_words), quadgram(input_words)))
  # Output
  paste(input, y, sep = " ")
}
```

User Input and Program Ouput

``` r
input <- "In case of a"
ngrams(input)
```

    ## [1] "In case of a fire"

Time it

``` r
stop <- Sys.time()
(how_long <- stop - go)
```

    ## Time difference of 13.83419 secs