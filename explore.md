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

    ## ── Attaching packages ─────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 2.2.1.9000     ✔ purrr   0.2.5     
    ## ✔ tibble  1.4.2          ✔ dplyr   0.7.5     
    ## ✔ tidyr   0.8.1          ✔ stringr 1.3.1     
    ## ✔ readr   1.1.1          ✔ forcats 0.3.0

    ## ── Conflicts ────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

Get the two patent datasets and combine them:

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
    ##   year = col_integer(),
    ##   sic1 = col_integer(),
    ##   sic_div = col_character(),
    ##   sic = col_integer(),
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
    ##   yr = col_integer(),
    ##   patents = col_integer(),
    ##   uspc = col_integer(),
    ##   ctg = col_character(),
    ##   hjt_num = col_integer(),
    ##   hjt = col_character()
    ## )

    ## Warning in rbind(names(probs), probs_f): number of columns of result is not
    ## a multiple of vector length (arg 1)

    ## Warning: 936 parsing failures.
    ## row # A tibble: 5 x 5 col     row col   expected               actual file                           expected   <int> <chr> <chr>                  <chr>  <chr>                          actual 1  4681 yr    no trailing characters e3     'https://github.com/lpuettman… file 2  4682 yr    no trailing characters e3     'https://github.com/lpuettman… row 3  4683 yr    no trailing characters e3     'https://github.com/lpuettman… col 4  4684 yr    no trailing characters e3     'https://github.com/lpuettman… expected 5  4685 yr    no trailing characters e3     'https://github.com/lpuettman…
    ## ... ................. ... .......................................................................... ........ .......................................................................... ...... .......................................................................... .... .......................................................................... ... .......................................................................... ... .......................................................................... ........ ..........................................................................
    ## See problems(...) for more details.

These counts run from 1995 to .

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

    ## Warning: Removed 180 rows containing missing values (geom_path).

    ## Warning: Removed 180 rows containing missing values (geom_point).

![](explore_files/figure-markdown_github/plot-comp-1.png)

``` r
ggsave("figures/cmp_uspto_counts.pdf", width = 8, height = 8)
```

    ## Warning: Removed 180 rows containing missing values (geom_path).

    ## Warning: Removed 180 rows containing missing values (geom_point).

You find a high resolution figure of this visualization [here](https://github.com/lpuettmann/automation-patents/tree/master/figures).

There's nothing special about 2002 to 2004 and the red line is very flat and wiggles around 1. From this, we conclude that this "bump" in the data is something that is already contained in the original data from the patent office. This might be due to some change in the eligibility criteria for which innovations can be protected with a patent or it could be do to the way that the USPTO publishes patents or structures their data.

In any way, we recommend to be cautious in interpreting changes around these years as reflecting underlying technological trends. In our own empirical analysis, we use five year moving sums of patents.
