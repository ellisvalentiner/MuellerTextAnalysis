
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Mueller Report Text Analysis

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
<!-- badges: end -->

Read in the report (downloaded as CSV from
[Factbase](https://%20f2.link/mr-sheet)). The Factbase version of the
report has been human-reviewed to fix errors in the OCR process.

``` r
report <- read_csv(
  file = "report.csv",
  col_names = c("page", "text", "definition", "NA"),
  col_types = cols(
    page = col_integer(),
    text = col_character(),
    definition = col_character(),
    `NA` = col_character()
  ),
  skip = 2
)
```

This is what the first few rows of the data looks like:

``` r
report %>%
  head() %>%
  knitr::kable()
```

| page | text                                                                                                                                                                                                      | definition | NA |
| ---: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :- |
|    1 | Report On The Investigation Into Russian Interference In The 2016 Presidential Election Volume I of II Special Counsel Robert S. Mueller, III Submitted Pursuant to 28 C.F.R. § 600.8(c) Washington, D.C. | NA         | NA |
|    1 | March 2019                                                                                                                                                                                                | NA         | NA |
|    2 | \[Blank, Not Redacted\]                                                                                                                                                                                   | NA         | NA |
|    3 | \[Table of Contents\]                                                                                                                                                                                     | NA         | NA |
|    4 | \[Table of Contents\]                                                                                                                                                                                     | NA         | NA |
|    5 | \[Table of Contents\]                                                                                                                                                                                     | NA         | NA |

This code chunk converts the report to a “tidy report” where each row is
a word.

``` r
tidy_report <- report %>%
  select(page, text) %>%
  unnest_tokens(output = word, input = text)

tidy_report %>%
  head() %>%
  knitr::kable()
```

| page | word          |
| ---: | :------------ |
|    1 | report        |
|    1 | on            |
|    1 | the           |
|    1 | investigation |
|    1 | into          |
|    1 | russian       |

The report has 197,140 words across 448 pages.

### Sentiment analysis

Sentiment analysis refers to estimating the sentiment – the general
positivity or negativity – of a text. Sometimes sentiment analysis is
used to characterize not just positivity/negativity but also other
emotions (such as anger, anticipation, disgust, fear, joy, sadness,
surprise, and trust).

Although the actual sentiment of text depends on the context of tokens,
it is common to simply use a lookup table that maps words to a
pre-identified sentiment. For instance the word “happy” would be given a
positive sentiment score (e.g. +3) and “unhappy” would be given a
negative sentiment score (e.g. -2). This method is easy to implement but
has obvious drawbacks – the sequence of tokens isn’t considered and some
words take on different meaning in different contexts.

This code chuck gets the sentiment of each word using the AFINN lexicon.

``` r
report_sentiment <- tidy_report %>%
  count(page, word) %>%
  left_join(
    y = get_sentiments("afinn"),
    by = "word"
  )
```

The report sentiment score is 0.14 (slightly positive) with a standard
deviation of 1.88 – so the text should probably be considered neutral.

This plot shows the “sentiment score” by each page. The page sentiment
score is calculated as the average (mean) of word scores scaled by their
number of occurences.

``` r
report_sentiment %>%
  group_by(page) %>%
  summarize(
    score = mean(n * score, na.rm = TRUE)
  ) %>%
  ungroup() %>%
  mutate(
    score = coalesce(score, 0)
  ) %>%
  ggplot(aes(x = page, y = score, fill = score)) +
  geom_bar(stat = "identity") +
  scale_x_continuous("Page") +
  scale_y_continuous("Sentiment Score") +
  scale_fill_viridis_c(option = "C") +
  guides(fill = FALSE) +
  theme_minimal()
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />
