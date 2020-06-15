---
title: "Implementing Katz's BackOff Model"
---
Having decided to formalize my foray into Data Science, I dived into the
Data Science Specialization hosted by Dr. Roger Peng and company of
Johns Hopkins University on Coursera (A pretty good introduction into
the basics, I’ll perhaps review the Specialization as a whole one of
these days). Anyways, the Capstone Project of the specialization has us
dealing with text corpuses from blogs, Twitter and the news and we’re
asked to build a predictive model for text input.

While reading about different n-gram and backoff models, I came across
Kneser-Ney Smoothing and Katz’s Back-off Model and thought that it would
be a decent place to start to try and implement these. In this post,
I’ll be covering Katz’s Backoff Model.

What is Katz’s Back-off Model?
------------------------------

Katz’s Backoff Model is a generative model used in language modeling to
estimate the conditional probability of a word, given its history given
the previous few words. However, probability estimates can change
suddenly on adding more data when the back-off algorithm selects a
different order of n-gram model on which to base the estimate.

Before getting into it, let’s talk a little bit about the concept of
discounting as it’s pretty important for the Katz Back-off Model.
Essentially, in discounting, we take away some of the probability mass
from observed n-grams and instead give them to unobserved n-grams so
that we can account for the n-grams that we haven’t seen yet.

Anyways, let’s get started!

Implementation
--------------

Let’s use the ngram “I was just” as a tester for the implementation.
\#\#\# Organizing Data We require tables of n-grams and their
frequencies. I’ve already loaded in the three text corpuses, stripped
all punctuation except those in common contractions and emojis, and
removed profanity using the lexicon package.  
Now, let’s write a function to give us the n-gram frequency tables for
the corpuses.

``` r
library(dplyr)
library(tokenizers)
library(reshape)

ngram <- "I was just"
ngram_freqs <- function(m){
  ngrams <- melt(tokenize_ngrams(data$sample, n = m))
  ngrams<-ngrams[ngrams$value %in% na.omit(ngrams$value),]
  ngrams<- ngrams %>% count(ngrams$value, sort = TRUE, )
  colnames(ngrams) <- c("ngram", "frequency")
  ngrams$ngram <- as.character(ngrams$ngram)
  ngrams
}
o_gram <- ngram_freqs(1)
bigram <- ngram_freqs(2)
trigram <- ngram_freqs(3)
tetragram <- ngram_freqs(4)
head(bigram)
```

    ##     ngram frequency
    ## 1  of the      2011
    ## 2  in the      1878
    ## 3  to the       974
    ## 4  on the       852
    ## 5 for the       834
    ## 6   to be       753

### Calculating Probabilities of Words Completing the n-gram

Let’s write a function that takes an input n-gram string and after
checking it’s length makes a dataframe consisting of words that would
complete a tetra/tri/bi-gram with the given string. We’ll keep an
absolute discounting of 0.5 overall (Look into cross validation to get
the best possible values for discounts).

``` r
gamma <- 0.5
kbo <- function(ngram, input,s){
    ngram <- tolower(ngram)
    subset <- data.frame(ngram = as.character(), frequency = as.numeric())
    if(s==2)  regex <-sprintf("%s%s", ngram, " ") 
    if(s==3) regex <-sprintf("%s%s%s%s", 
                             unlist(strsplit(ngram, " "))[1]," ",
                             unlist(strsplit(ngram, " "))[2], " ") 
    if(s==4) regex <-sprintf("%s%s%s%s%s%s", 
                             unlist(strsplit(ngram, " "))[1], " ",
                             unlist(strsplit(ngram, " "))[2], " ",
                             unlist(strsplit(ngram, " "))[3], " ") 
    contained_ngrams <- grep(regex, input$ngram)
    if(length(contained_ngrams)>0) subset <- input[contained_ngrams,]
    subset
}
length_pass <- function(ngram){
  if(length(unlist(strsplit(ngram, " ")))==1) output <- kbo(ngram, bigram,2)
  if(length(unlist(strsplit(ngram, " ")))==2) output <- kbo(ngram, trigram,3)
  if(length(unlist(strsplit(ngram, " ")))==3) output <- kbo(ngram, tetragram,4)
  output$probability <- (output$frequency-gamma)/sum(output$frequency)
  output
}
head(length_pass(ngram))
```

    ##                           ngram frequency probability
    ## 2604           i was just about         2      0.1875
    ## 2605          i was just trying         2      0.1875
    ## 162200        i was just called         1      0.0625
    ## 162201 i was just flabbergasted         1      0.0625
    ## 162202      i was just speaking         1      0.0625
    ## 162203       i was just talking         1      0.0625

### Finding Unobserved ngrams

Now, we find words that would make up tetra/tri/bi-grams but haven’t
been observed.

``` r
library(stringr)
unobserved <- function(ngram){
  s <- length_pass(ngram)
  k<-strsplit(s$ngram, " ")
  ex <- o_gram$ngram
  for (i in 1:length(k)) {
    ex <- ex[!(ex %in% unlist(k[i])[length(unlist(k[i]))])]
  }
  ex
}

head(unobserved(ngram), 10)
```

    ##  [1] "the"  "to"   "and"  "a"    "of"   "i"    "in"   "that" "is"   "for"

### Discounted Probability Mass

Now, we should go ahead and find the amount of discounted probability
mass taken from the n-gram.

``` r
alpha <- function(ngram, n){
  temp <- unlist(strsplit(ngram, " "))
  f <- c()
  for (i in 2:n) {
    f<- paste(temp[length(temp)-i+1], f)    
  }
  e <- paste(trimws(f), temp[length(temp)])
  t<- length_pass(trimws(f))
  
  a <- 1 - t[t$ngram==tolower(trimws(e)),]$frequency*(1-gamma)/sum(t$frequency)

  a
}
alpha(ngram, 3)
```

    ## [1] 0.9891008

### Calculating Backed Off Probabilities

Having found the amount of probability mass to be discounted for a
level, we will find the backed off probabilities for observed and
unobserved n-grams. Combinging the two dataframes, we’ll get our
predictions!

``` r
qbo_observed <- function(ngram, n){
  temps <- unlist(strsplit(ngram, " "))
  f <- c()
  for (i in 1:(n-1)) {
    f <- paste(temps[length(temps)-i], f)    
  }
  temp <- strsplit(length_pass(trimws(f))$ngram, " ")
  y <- length_pass(trimws(f))
  for (i in 1:length(temp)) {
    y$ngram[i] <- unlist(temp[i])[length(unlist(temp[i]))]
  }
  y <- subset(y, select = -c(frequency))
  y
}

qbo_unobserved <- function(ngram, n){

  temps <- unlist(strsplit(ngram, " "))
  f <- c()
  for (i in 1:(n-1)) {
    f<- paste(temps[length(temps)-i], f)    
  }
  temp <- o_gram[o_gram$ngram %in% unobserved(f),]

  temp$probability <- alpha(ngram, n)*temp$frequency/sum(temp$frequency)
  temp<- subset(temp, select = -c(frequency))
  temp
}
net_table <- function(ngram, n){
comb <- rbind(qbo_unobserved(ngram, n), qbo_observed(ngram, n))
output <- comb[order(comb$probability, decreasing = TRUE),]
output
}
head(net_table(ngram, 2))
```

    ##        ngram probability
    ## 37100      a  0.09089323
    ## 101100   the  0.04972087
    ## 1        the  0.04824658
    ## 2         to  0.02632334
    ## 3        and  0.02499493
    ## 4          a  0.02360031

### Final Predictions

Most Katz Backoff Implementations go for trigrams as the highest order
considered but I wanted to implement it for the tetragram level as well.
So let’s join the two tables and group them by ngram and summing up
their respective probabilities.

``` r
final_table <- function(ngram){
 output <- rbind(net_table(ngram, 2),net_table(ngram, 3))
 output <- output %>% 
   group_by(`ngram`) %>% 
   summarise_at(vars(probability),funs(sum(.,na.rm=TRUE)))  %>% 
   arrange(desc(probability))
 output
}
head(final_table(ngram))
```

    ## Warning: `funs()` is deprecated as of dplyr 0.8.0.
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_warnings()` to see where this warning was generated.

    ## # A tibble: 6 x 2
    ##   ngram probability
    ##   <chr>       <dbl>
    ## 1 a          0.139 
    ## 2 the        0.106 
    ## 3 in         0.0706
    ## 4 and        0.0595
    ## 5 going      0.0521
    ## 6 i          0.0429

Et voilà! We have our predictions for an ngram (“I was just”) using the
Katz Backoff Model using tetragram and trigram tables with backing off
to the trigram and bigram levels respectively. Further scope for
improvement is with respect to the speed and perhaps applying some sort
of smoothing technique like Good-Turing Estimation.
