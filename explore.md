Exploring Automation Patents
================
Katja Mann, Lukas Püttmann
June 14 2018

We provide some codes here, to explore the datasets we provide in our paper. You can cite this document as:

> Mann, Katja and Lukas Püttmann (2018). "Exploring Automation Patents". Online at: <https://github.com/lpuettmann/automation-patents>.

Load some packages:

``` r
library(tidyverse)
```

    ## ── Attaching packages ───────────────

    ## ✔ ggplot2 3.1.1     ✔ purrr   0.3.2
    ## ✔ tibble  2.1.1     ✔ dplyr   0.8.1
    ## ✔ tidyr   0.8.3     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.4.0

    ## ── Conflicts ────────────────────────
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

Get the two patent datasets and combine them:

``` r
patents <- read_csv("data/patents1.zip") %>%
  bind_rows(read_csv("data/patents2.zip"))
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   week = col_double(),
    ##   patent = col_double(),
    ##   automat = col_double(),
    ##   raw_automat = col_double(),
    ##   excl = col_double(),
    ##   post_yes = col_double(),
    ##   post_no = col_double(),
    ##   hjt1 = col_character(),
    ##   hjt2 = col_character(),
    ##   hjt2_num = col_double(),
    ##   uspc_primary = col_character(),
    ##   length_pattext = col_double(),
    ##   cts = col_double(),
    ##   cts_wt = col_double(),
    ##   assignee = col_character()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   week = col_double(),
    ##   patent = col_double(),
    ##   automat = col_double(),
    ##   raw_automat = col_double(),
    ##   excl = col_double(),
    ##   post_yes = col_double(),
    ##   post_no = col_double(),
    ##   hjt1 = col_character(),
    ##   hjt2 = col_character(),
    ##   hjt2_num = col_double(),
    ##   uspc_primary = col_character(),
    ##   length_pattext = col_double(),
    ##   cts = col_double(),
    ##   cts_wt = col_double(),
    ##   assignee = col_character()
    ## )

``` r
hjt <- patents %>% 
  group_by(year, hjt1, automat) %>% 
  summarise(patents = n()) %>% 
  mutate(classification = case_when(
    automat == 1 ~ "automation",
    automat == 0 ~ "rest"))

ggplot(hjt, aes(year, patents, fill = classification)) +
  geom_col(width = 0.5) +
  facet_wrap(~hjt1, scales = "free_y") +
  theme_minimal() +
  scale_y_continuous(labels=function(x) x / 1000) +
  labs(y = "patents (in 1000s)", x = NULL, subtitle = "1976-2014, annually", 
       fill = "Classification:  ", title = "Automation Patents by Technology Class") +
  theme(legend.position = "bottom") +
  scale_fill_manual(values = c("#d73027", "#4575b4"))
```

![](explore_files/figure-markdown_github/hjt-fig-1.png)

Industries
==========

Get the industry data:

``` r
ind <- read_csv("data/industry_data.zip") 
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   sic1 = col_double(),
    ##   sic_div = col_character(),
    ##   sic = col_double(),
    ##   nb = col_character(),
    ##   affil = col_character(),
    ##   weight = col_character(),
    ##   assignee = col_character(),
    ##   patents = col_double()
    ## )

Pick the versions of the data we're interested in: 1) where are patents used (not invented) and 2) patents not weighted by the number of their citations. The data comes split into bins for individual patent assignees. So to get a first look at the data, sum over all patent assignees:

``` r
df <- ind %>%
  filter(affil == "sector of use",
         weight == "none") %>%
  group_by(year, sic1, sic_div, sic, nb) %>% 
  summarise(patents = sum(patents)) 
```

Now, calculate the share of patents in industries that are automation patents:

``` r
df <- df %>% 
  spread(nb, patents) %>% 
  mutate(sh = automation / (automation + rest))
```

Plot the share of automation patents for all industries:

``` r
ggplot(df, aes(year, sh, group = sic)) +
  geom_line(alpha = 0.1) +
  labs(title = "Share of automation patents by industry",
       subtitle  = "1976-2014, annually",
       x = NULL,
       y = NULL) +
  theme_minimal() +
  theme(axis.line = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank()) +
  facet_wrap(~sic_div, ncol = 3)
```

    ## Warning: Removed 17 rows containing missing values (geom_path).

![](explore_files/figure-markdown_github/autom_sh-plot-1.png)

There are many more subindustries in *Manufacturing* and *Services* than in *Finance* or *Government*, that's why the plots are darker and denser.

The share of all patents that we classify as automation has risen across industries. But there's quite a bit of heterogeneity across industries.

The bump around 2002-2004 corresponds to a change in the files provided by [Google](https://www.google.com/googlebooks/uspto-patents-grants-text.html). This probably means that some category of patents was in the files but not before or after. If you know something about what changed around those dates, please let us know!

We were worried that this might be due to an error in our parsing of the patent files, but that does not seem to be the case.

To check whether this is a problem, we scraped data from the USPTO website and compared our aggregate counts per year and technological class with those in our dataset. You find the codes [here](https://github.com/lpuettmann/scrape-uspto) and the original website with the data [here](https://www.uspto.gov/web/offices/ac/ido/oeip/taf/cbcby.htm).

Load the scraped comparison dataset:

``` r
uspto <- read_csv("https://github.com/lpuettmann/scrape-uspto/raw/master/uspto_counts.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   class = col_character(),
    ##   class_title = col_character(),
    ##   counts = col_character(),
    ##   yr = col_double(),
    ##   patents = col_double(),
    ##   uspc = col_double(),
    ##   ctg = col_character(),
    ##   hjt_num = col_double(),
    ##   hjt = col_character()
    ## )

These counts run from 1995 to 2914.

``` r
cmp <- patents %>%
  filter(year >= 1995) %>% 
  group_by(year, hjt1, hjt2, hjt2_num) %>%
  summarise(my_pts = n()) %>%
  rename(yr = year) %>%
  full_join(uspto %>%
              rename(hjt2_num = hjt_num) %>% 
              group_by(yr, hjt2_num, counts) %>%
              summarise(uspc_pts = sum(patents)) %>%
              ungroup(),
            by = c("yr", "hjt2_num")) %>% 
  arrange(counts, yr, hjt2_num) %>% 
  drop_na(hjt2_num)
```

Calculate the ratio between our counts and the official ones and visualize:

``` r
annot_bump <- tribble(
  ~xmin, ~xmax,
  2002, 2004)

cmp %>% 
  mutate(rt = uspc_pts / my_pts,
         counts = ifelse(counts == "no_dupl", "no duplicates", counts)) %>% 
  ggplot() +
  geom_rect(aes(xmin = xmin, xmax = xmax, ymin = -Inf, ymax = Inf),
            data = annot_bump, fill = "#f4a582", alpha = 0.4,
            inherit.aes = FALSE) +
  geom_hline(yintercept = 0, size = 0.5, color = "grey80") +
  geom_hline(yintercept = 1, size = 0.3, color = "grey80") +
  geom_line(aes(yr, rt, group = counts, color = counts), alpha = 0.8) +
  geom_point(aes(yr, rt, color = counts), alpha = 0.8, size = 1.1,
             stroke = 0) +
  facet_wrap(~hjt2, scales = "free_y", ncol = 5) +
  theme_minimal() +
  theme(legend.position = "bottom") +
  labs(x = NULL, y = "ratio", title = "Patent numbers: USPTO vs. our dataset",
       subtitle = '1995-2014, annually. The potentially problematic "bump": 2002, 2003 and 2004.',
       caption = paste0("Source: Our dataset and the official USPTO numbers from:\n", 
                        "https://www.uspto.gov/web/offices/ac/ido/oeip/taf/cbcby.htm"),
       color = "Counting of patents\n in USPTO data:   ")
```

    ## Warning: Removed 72 rows containing missing values (geom_path).

    ## Warning: Removed 72 rows containing missing values (geom_point).

![](explore_files/figure-markdown_github/plot-comp-1.png)

``` r
ggsave("figures/cmp_uspto_counts.pdf", width = 8, height = 8)
```

    ## Warning: Removed 72 rows containing missing values (geom_path).

    ## Warning: Removed 72 rows containing missing values (geom_point).

You find a high resolution figure of this visualization [here](https://github.com/lpuettmann/automation-patents/tree/master/figures).

There's nothing special about 2002 to 2004 and the red line is very flat and wiggles around 1. From this, we conclude that this "bump" in the data is something that is already contained in the original data from the patent office. This might be due to some change in the eligibility criteria for which innovations can be protected with a patent or it could be do to the way that the USPTO publishes patents or structures their data.

In any way, we recommend to be cautious in interpreting changes around these years as reflecting underlying technological trends. In our own empirical analysis, we use five year moving sums of patents.

Aggregate statistics
====================

Let's also look at some aggregate statistics of patents over time.

``` r
aggr_patents <- ind %>% 
  filter(affil == "sector of use",
         weight == "none") %>% 
  group_by(year, nb) %>% 
  summarise(patents = sum(patents))
```

Plot the number of patents by year by category.

``` r
ggplot(aggr_patents, aes(year, patents, color = nb)) +
  geom_line() + 
  geom_point() +
  theme_minimal() +
  labs(title = "Total number of patents by category and year",
       subtitle = "1976-2014")
```

![](explore_files/figure-markdown_github/unnamed-chunk-2-1.png)

Calculate how the share of automation patents has changed over time:

``` r
aggr_stats <- aggr_patents %>% 
  spread(nb, patents) %>% 
  mutate(share_autom = automation / (automation + `chemical-pharma` + rest))

write_csv(aggr_stats, "out_data/aggr_stats.csv")
```

``` r
ggplot(aggr_stats, aes(year, share_autom)) +
  geom_line() + 
  geom_point() +
  theme_minimal() +
  labs(title = "Share of automation patents",
       subtitle = "1976-2014")
```

![](explore_files/figure-markdown_github/unnamed-chunk-4-1.png)
