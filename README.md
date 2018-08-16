# Automation Patents

This repositories makes available the data produced by our paper: 

>Mann, Katja and Lukas Püttmann (2017). "Benign Effects of Automation: New Evidence from Patent Texts". Unpublished manuscript.

Please cite us if you use our data.

Links:

- [Working paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2959584)
- [VoxEU column](https://voxeu.org/article/benign-effects-automation-new-evidence)
- [Blog post](http://lukaspuettmann.com//2017/09/22/automation-patents-paper/)

# Files

We keep the following datasets here:

| Name | Format | Level | Years | Rows | Approx. size (zipped) |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
| `patents1.zip`  | `.csv` | Patents | 1976-2001 | 2,409,118 | 44 MB |
| `patents2.zip`  | `.csv` | Patents | 2002-2014 | 2,646,518 | 43 MB |
| `industry_data.zip`  | `.csv` | Industries (SIC 4 digit) | 1976-2014 | 451,620 | 12 MB |
| `czone_dataset.zip`  | `.csv` | Commuting zones | 1976-2014 | 281,580 | 3.4 MB |


# What you don't find here

- Does not include the codes to do the initial parsing and classification of the patents.
- Does not provide the codes of how to go from one dataset to the other.
- This is not a replication kit for our paper. So you won't find data and codes here to reproduce all the figures and tables in our paper.

# Datasets

All datasets cover the period 1976 to 2014 and the regional coverage are the United States. 

## 1. Patent level dataset

Includes all US utility patents and contains the information for every patent if we classify it as automation or not. To construct some variables (`cts`, `cts_wt` and `assignee`)

The assignee and citation data is from the [Fung Institute](https://github.com/funginstitute/downloads) and stops in 2010.

- `year`: Grant year.
- `week`: Identifies the weekly files that the patent was published in.
- `patent`: Patent number (7 characters)
- `automat`: Classification as automation patent after excluding patents (see paper for details).
- `raw_automat`: Classification as automation patent before excluding patents (see paper for details).
- `excl`: Excluded patents. We exclude many chemical or pharmaceutical patents in our empirical analysis, see paper for details.
- `post_yes`: The posterior probability that a patent is an automation patent.
- `post_no`: The posterior probability that a patent is not an automation patent.
- `hjt1`: Hall-Jaffe-Trajtenberg top-level categories
- `hjt2`: Hall-Jaffe-Trajtenberg subcategories by name
- `hjt2_num`: Hall-Jaffe-Trajtenberg subcategories by number
- `uspc_primary`: Every patent is assigned one or several USPC (United States Patent Classification) numbers. This reports the first USPC number written in the patent documents. We use this number to assign Hall-Jaffe-Trajtenberg categories. This is not the classification we use to match patents to industries: We use the complete list of patents' IPC (International Patent Classification) numbers for this (not contained in this dataset). 
- `length_pattext`: Length of patent text as measured by the number of lines in [Google](https://www.google.com/googlebooks/uspto-patents-grants-text.html)'s text files. Number is missing for every last patent in the weekly files.
- `cts`: Number of citations using Fung Institute data.
- `cts_wt`: Number of weighted citations. See paper for explanation.
- `assignee`: The group of patent assignee ("US firm", "foreigners", "governments", "universities" or missing/"NA"). We use the Fung Institute data to identify US firms, foreigners and governments and our own coding to find universities and public research institutes. 


## 2. Industry level dataset

We distribute all patents probabilistically to industries where they are created ("sector of manufacture") and where they are likely to be used ("industry of use"). See paper and the links above for explanations. Industries are defined according to [SIC 1987](https://www.osha.gov/pls/imis/sic_manual.html).

- `year`: Year
- `sic1`: First digit of SIC number
- `sic_div`: Name of SIC division ("Agriculture", "Mining" and so on)
- `sic`: Four-digit SIC number (1987 SIC classification).
- `nb`: Our classification of patents as either "automation" or non-automation ("rest") according to the Naive Bayes algorithm.
- `affil`: Two options ("sector of manufacture" and "industry of use")
- `weight`: Uses either no weights ("none") or weighs patents by the number of their citations.
- `assignee`: Four options ("foreigners", "governments", "universities" and "other")
- `patents`: Number of patent (equivalents).

Be careful when you use this dataset, as some variables provide subsets to the dataset and some offer alternative datasets:
- `affil` and `weight` are options you can choose
- `nb` and `assignee` contain the values for subsets of patents. So if, for example, you want to know how many automation patents there are in some industry that are owned by any entity, then you need to sum across the `assignee` variable.

## 3. Commuting zone level dataset

Includes the number of patents that can be used in a US commuting zones.

- `cz`: Commuting zone
- `year`: Grant year
- `type`: Whether the patent measure has been constructed using levels of patents or logs as described in the paper.
- `assignee`: Group who is assigned the patent, as described above. `all` contains all groups.
- `weight`: Citation weights as described above.
- `autopats`: Automation patents
- `nonautopats`: All other (non-automation) patents.


# How to use

## Stata

Use the [maptile](https://michaelstepner.com/maptile/) program to create maps:

```stata
import delim data/czone_dataset.csv
keep if type == "level" & assignee == "all" & weight == "none"

maptile autopats if year==1976, geography(cz) conus nquantiles(4) /// 
	savegraph(figures\map_1976.png) replace legdecimals(0) resolution(0.5) /// 
	twopt(title(Automation patents: 1976))
```

<img src="https://github.com/lpuettmann/automation-patents/blob/master/figures/map_1976.png" width="500">

## R

See [here](/explore.md) ([codes](https://github.com/lpuettmann/automation-patents/blob/master/explore.Rmd)) for more examples, figures and usage recommendations.

# License

Our data and our codes are under the [MIT license](https://github.com/lpuettmann/automation-patents/blob/master/LICENSE.md). This means that you can use everything as you please for research or commercial purposes, as long as you refer back to us.

# Contributing

If you find irregularities or bugs, please open an issue here.

# References

Hall, B. H., A. B. Jaffe and M. Trajtenberg (2001). "The NBER patent citation data file: Lessons, insights and methodologial tools". NBER Working Paper No. 8498. ([website](http://www.nber.org/patents/)) ([pdf](http://www.nber.org/papers/w8498.pdf))

Lai, R., A. D’Amour, A. Yu, Y. Sun, D. M. Doolin and L. Fleming (2011). "Disambiguation and coauthorship networks of the u.s. patent inventor database (1975 -2010)". Fung Institute. ([doi](https://doi.org/10.1016/j.respol.2014.01.012))
