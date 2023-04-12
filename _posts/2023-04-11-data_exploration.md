---
layout: single
title:  "Test from R"
categories: jekyll update
classes: wide
toc: true
toc_icon: "cog"
---

# Libraries

``` r
library(tidyverse)
library(bnlearn)
library(lattice)
```

# Data

``` r
data <- read_csv("data/sachs1.csv")
```

    ## Rows: 853 Columns: 11
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (11): praf, pmek, plcg, PIP2, PIP3, p44/42, pakts473, PKA, PKC, P38, pjnk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
head(data)
```

    ## # A tibble: 6 × 11
    ##    praf  pmek  plcg  PIP2  PIP3 `p44/42` pakts473   PKA   PKC   P38  pjnk
    ##   <dbl> <dbl> <dbl> <dbl> <dbl>    <dbl>    <dbl> <dbl> <dbl> <dbl> <dbl>
    ## 1  26.4 13.2   8.82 18.3  58.8      6.61     17     414 17     44.9  40  
    ## 2  35.9 16.5  12.3  16.8   8.13    18.6      32.5   352  3.37  16.5  61.5
    ## 3  59.4 44.1  14.6  10.2  13       14.9      32.5   403 11.4   31.9  19.5
    ## 4  73   82.8  23.1  13.5   1.29     5.83     11.8   528 13.7   28.6  23.1
    ## 5  33.7 19.8   5.19  9.73 24.8     21.1      46.1   305  4.66  25.7  81.3
    ## 6  18.8  3.75 17.6  22.1  10.9     11.9      25.7   610 13.7   49.1  57.8

``` r
glimpse(data)
```

    ## Rows: 853
    ## Columns: 11
    ## $ praf     <dbl> 26.4, 35.9, 59.4, 73.0, 33.7, 18.8, 44.9, 47.4, 104.0, 21.1, …
    ## $ pmek     <dbl> 13.20, 16.50, 44.10, 82.80, 19.80, 3.75, 36.50, 15.00, 61.50,…
    ## $ plcg     <dbl> 8.82, 12.30, 14.60, 23.10, 5.19, 17.60, 10.40, 14.60, 10.60, …
    ## $ PIP2     <dbl> 18.30, 16.80, 10.20, 13.50, 9.73, 22.10, 132.00, 30.50, 21.10…
    ## $ PIP3     <dbl> 58.80, 8.13, 13.00, 1.29, 24.80, 10.90, 16.30, 17.50, 41.80, …
    ## $ `p44/42` <dbl> 6.61, 18.60, 14.90, 5.83, 21.10, 11.90, 8.66, 20.20, 11.50, 1…
    ## $ pakts473 <dbl> 17.00, 32.50, 32.50, 11.80, 46.10, 25.70, 17.90, 45.30, 23.50…
    ## $ PKA      <dbl> 414, 352, 403, 528, 305, 610, 835, 466, 445, 213, 449, 389, 5…
    ## $ PKC      <dbl> 17.00, 3.37, 11.40, 13.70, 4.66, 13.70, 15.00, 6.44, 29.20, 1…
    ## $ P38      <dbl> 44.9, 16.5, 31.9, 28.6, 25.7, 49.1, 35.9, 24.4, 61.0, 26.7, 4…
    ## $ pjnk     <dbl> 40.00, 61.50, 19.50, 23.10, 81.30, 57.80, 18.10, 20.00, 25.30…

``` r
sum(is.na(data))
```

    ## [1] 0

# Visualizations

The data is continuous, float, has no missing values. Let’s do some
visualizations.

``` r
df_long <- data %>% gather(key = "molecules", value = "concentration")

ggplot(df_long, aes(molecules, concentration)) +
  geom_boxplot()
```

![](data_exploration_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->
There are some extreme conecntration values for certain molcules. Let’s
filter for concentration less than 1000.

``` r
df_test <- df_long %>% 
  filter(concentration < 1000)

ggplot(df_test, aes(molecules, concentration)) +
  geom_boxplot()
```

![](data_exploration_files/figure-gfm/unnamed-chunk-4-1.png)<!-- --> PKA
seems to be the dominant molecule. Should check what these mean

``` r
summary(data)
```

    ##       praf             pmek             plcg             PIP2       
    ##  Min.   :  1.61   Min.   :  1.00   Min.   :  1.00   Min.   :  1.11  
    ##  1st Qu.: 35.50   1st Qu.: 17.90   1st Qu.: 11.50   1st Qu.: 21.10  
    ##  Median : 52.30   Median : 24.40   Median : 17.00   Median : 48.30  
    ##  Mean   : 59.31   Mean   : 30.03   Mean   : 19.49   Mean   : 81.55  
    ##  3rd Qu.: 74.30   3rd Qu.: 33.70   3rd Qu.: 23.70   3rd Qu.:106.00  
    ##  Max.   :552.00   Max.   :389.00   Max.   :167.00   Max.   :843.00  
    ##       PIP3            p44/42           pakts473            PKA         
    ##  Min.   :  1.00   Min.   :   1.00   Min.   :   1.70   Min.   :   1.95  
    ##  1st Qu.: 13.90   1st Qu.:   8.43   1st Qu.:  19.80   1st Qu.: 325.00  
    ##  Median : 23.90   Median :  14.90   Median :  29.20   Median : 437.00  
    ##  Mean   : 30.46   Mean   :  22.18   Mean   :  41.98   Mean   : 567.02  
    ##  3rd Qu.: 38.50   3rd Qu.:  22.10   3rd Qu.:  39.60   3rd Qu.: 649.00  
    ##  Max.   :764.00   Max.   :2571.00   Max.   :3555.00   Max.   :4491.00  
    ##       PKC              P38              pjnk       
    ##  Min.   :  1.00   Min.   :  1.53   Min.   :  1.00  
    ##  1st Qu.:  5.94   1st Qu.: 21.70   1st Qu.: 12.20  
    ##  Median : 13.60   Median : 30.20   Median : 20.00  
    ##  Mean   : 15.02   Mean   : 34.02   Mean   : 38.46  
    ##  3rd Qu.: 20.70   3rd Qu.: 40.70   3rd Qu.: 57.30  
    ##  Max.   :106.00   Max.   :170.00   Max.   :343.00

## Concentration distributions

``` r
histogram(~ PIP2 + PIP3 + pmek + P38, data = data, type = "density",
  breaks = NULL, xlab = "expression levels", ylab = "density",
  scales = list(x = list(relation = "free"),
                y = list(relation = "free", at = NULL)),
  panel = function(x, ...) {

    panel.abline(h = 0)
    panel.histogram(x, ...)
    panel.mathdensity(dmath = dnorm, col = "black",
      args = list(mean = mean(x), sd = sd(x)))

  }
)
```

![](data_exploration_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
ggplot(data, aes(pmek)) +
  geom_histogram(aes(y = ..density..), bins = 50, color = "white") +
  geom_density()
```

    ## Warning: The dot-dot notation (`..density..`) was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `after_stat(density)` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

![](data_exploration_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
# create a histogram for each column and arrange them in a grid
ggplot(df_long, aes(x = concentration)) +
  geom_histogram(aes(y = ..density..), binwidth = 0.2, color = "lightblue") +
  geom_density(alpha = .2, fill = "#FF6666") +
  labs(x = "Value", y = "Density", title = "Histograms of 11 Variables") +
  facet_wrap(~molecules, nrow = 4, ncol = 3, scales = "free")
```

![](data_exploration_files/figure-gfm/unnamed-chunk-8-1.png)<!-- --> The
distributions of the molecule concentrations are heavily skewed. Most of
them are small and cluster around zero. The variables are not symmetric
and clearly violate the distributional assumptions underlying GBNs. \##
Dependence relationships

``` r
ggplot(data, aes(x = PKC, y = PKA)) +
  geom_point() +
  geom_smooth(method = lm)
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](data_exploration_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
ggplot(data, aes(x = PKC, y = PIP3)) +
  geom_point() +
  geom_smooth(method = lm)
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](data_exploration_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->
Looks like the dependence relationships in the data are not always
linear. Most conditional independence tests and network scores are
designed to capture linear relationships have very low poer in detecting
nonlinear ones. Structure learning algorithms using such statistics are
unable to correctly learn the arcs in the DAG.
