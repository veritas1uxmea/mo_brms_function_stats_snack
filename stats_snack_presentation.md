<style>
body {
    overflow: scroll;
}
</style>

<style>
head {
  font-size: 2px;
}
</style>

Ordinal variables,               what to do? 
========================================================
author: Won Lee
date: November 10th, 2020
autosize: false
font-family: 'Helvetica'

Modeling ordinal variables with brms package in R

Index
========================================================
- Ordinal variables
- why do we (have to) care about this?
- Ordinal predictor variables: Modelling monotonic effects
- Use and interpretation of `mo()` function
- Ordinal outcome variables
- Use and interpretation of `cumulative()` function


All materials are located at [my github,](https://github.com/veritas1uxmea/mo_brms_function_stats_snack.git)
and the slides can be found on [my website.](https://wonlee-neuroscientist.com/04_bioinfo.html)







What is ordinal factor 
========================================================

<img src="figures/datatype.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" style="display: block; margin: auto;" />

What is ordinal factor 
========================================================
<img src="figures/steps.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />


Sometimes the data you get is all you've got
========================================================
- Social rank, social status 

<small>Alpha, subdominant, Subordinate</small>

- Drug doses 

<small>Vehicle, 2ug, 20ug, 200ug</small>

- Likert scale items 

<small>Strongly Disagree, Disagree, Neutral, Agree, Strongly Agree</small>

- When they ask you your annual income

<small>Below 20K, 20-40K, 40-100K, greater than 100K</small>

- How many meals in a week do you eat cheese?

<small> 0-2 times, 3-5 times, 6-9 times, more than 10 times</small>


Why do we have to care about this ...?
========================================================
Whoever studied animal behavior, might have had the same question:
<img src="stats_snack_presentation-figure/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

========================================================
so someone replied...:
<img src="stats_snack_presentation-figure/unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />



We should not assume 
========================================================
the "equidistance" when it comes to ordinal variables

<small>Equidistance refers to intervals with values that are distributed in equal units. 

When we treat an ordinal predictor as numeric, the models will assume the 'equidistance' of the given variable

Know your data!! </small>


For example, 
========================================================
<img src="stats_snack_presentation-figure/unnamed-chunk-5-1.gif" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />


Ordinal predictor variables 
========================================================
Modelling monotonic effects
> In many practical settings, we do often expect the changes between adjacent categories to be monotonic, that is, consistently negative or positive across the full range of the ordinal variable 

> Burkner & Charpentier (2020)


========================================================
[Williamson et al., (2019)](https://www.nature.com/articles/s41598-019-43747-w)
<img src="figures/female_fig4.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" style="display: block; margin: auto;" />

========================================================
[Jimenez and Mesoudi (2020)](https://brill.com/view/journals/jocc/20/3-4/article-p238_4.xml)

<img src="figures/jimenez_et_all_2020_fig4.jpg" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" style="display: block; margin: auto;" />

========================================================
[Jimenez and Mesoudi (2020)](https://brill.com/view/journals/jocc/20/3-4/article-p238_4.xml)
<img src="figures/jimenez_et_all_2020_tab2.jpg" title="plot of chunk unnamed-chunk-8" alt="plot of chunk unnamed-chunk-8" style="display: block; margin: auto;" />



Let's try it out
========================================================
class: small-code
- Annual income: Likert scale itmes
- Life satisfaction: Scale from 0 to 100 

Full tutorial [here](https://cran.r-project.org/web/packages/brms/vignettes/brms_monotonic.html)

```r
income_options <- c("below_20", "20_to_40", "40_to_100", "greater_100")
income <- factor(sample(income_options, 100, TRUE), 
                  levels = income_options, ordered = TRUE)
mean_ls <- c(30, 60, 70, 75)
ls <- mean_ls[income] + rnorm(100, sd = 7)
dat <- data.frame(income, ls)
```

Plot it first!
========================================================
<img src="stats_snack_presentation-figure/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" style="display: block; margin: auto;" />

Monotonic model 
========================================================
class: small-code

```r
fit1 <- brm(ls ~ mo(income), 
            # mo() for monotonic effect
            data = dat,
            file = "fit1.RDS") 
# saves file so we don't have to run this model every time
```

Monotonic model 
========================================================
class: small-code
see if we have 'healthy' posterior chains

```r
plot(fit1) 
```

<img src="stats_snack_presentation-figure/unnamed-chunk-12-1.png" title="plot of chunk unnamed-chunk-12" alt="plot of chunk unnamed-chunk-12" style="display: block; margin: auto;" />


Monotonic model 
========================================================
class: small-code

```r
plot(conditional_effects(fit1)) 
```

<img src="stats_snack_presentation-figure/unnamed-chunk-13-1.png" title="plot of chunk unnamed-chunk-13" alt="plot of chunk unnamed-chunk-13" style="display: block; margin: auto;" />

Treat income to be continuous
========================================================
class: small-code

```r
dat$income_num <- as.numeric(dat$income)
fit2 <- brm(ls ~ income_num, 
            data = dat,
            file = "fit2.RDS")
```

Treat income to be continuous
========================================================
class: small-code

```r
plot(conditional_effects(fit2))
```

<img src="stats_snack_presentation-figure/unnamed-chunk-15-1.png" title="plot of chunk unnamed-chunk-15" alt="plot of chunk unnamed-chunk-15" style="display: block; margin: auto;" />

Treat income as an unordered factor
========================================================
class: small-code

```r
contrasts(dat$income) <- contr.treatment(4)
contrasts(dat$income)
```

```
            2 3 4
below_20    0 0 0
20_to_40    1 0 0
40_to_100   0 1 0
greater_100 0 0 1
```

```r
fit3 <- brm(ls ~ income, 
            data = dat,
            file = "fit3.RDS")
```


Treat income as an unordered factor
========================================================
class: small-code

```r
plot(conditional_effects(fit3))
```

<img src="stats_snack_presentation-figure/unnamed-chunk-17-1.png" title="plot of chunk unnamed-chunk-17" alt="plot of chunk unnamed-chunk-17" style="display: block; margin: auto;" />

Model comparison
========================================================
class: small-code
Leave-one-out cross validation (LOO)
- expected log posterior predictive density (ELPD) difference

```r
loo(fit1, fit2, fit3) -> loo_result
loo_result$diffs
```

```
     elpd_diff se_diff
fit1   0.0       0.0  
fit3  -0.1       0.3  
fit2 -23.3       5.8  
```

```r
# loo_compare(loo(fit1), loo(fit2), loo(fit3))
# will give you the same result as above
```

Okay, then how do I interpret the results?
========================================================
class: small-code

```r
summary(fit1)
```

```
 Family: gaussian 
  Links: mu = identity; sigma = identity 
Formula: ls ~ mo(income) 
   Data: dat (Number of observations: 100) 
Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
         total post-warmup samples = 4000

Population-Level Effects: 
          Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
Intercept    30.84      1.68    27.54    34.20 1.00     2572     2274
moincome     14.61      0.75    13.14    16.08 1.00     2526     2221

Simplex Parameters: 
             Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
moincome1[1]     0.65      0.04     0.57     0.74 1.00     2765     2453
moincome1[2]     0.23      0.05     0.13     0.34 1.00     3581     2597
moincome1[3]     0.11      0.05     0.02     0.21 1.00     2494     1284

Family Specific Parameters: 
      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sigma     7.91      0.57     6.90     9.09 1.00     3102     2524

Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
and Tail_ESS are effective sample size measures, and Rhat is the potential
scale reduction factor on split chains (at convergence, Rhat = 1).
```


Always be careful when building models
========================================================
It all depends on what questions you are asking. 

If you think it's possible you'll see a u-shaped effect, do not use monotonic effect model (or test both)

<small>[Kundakovic et al., 2013. PNAS](https://www.pnas.org/content/110/24/9956)</small>
<img src="figures/kundakovic.png" title="plot of chunk unnamed-chunk-20" alt="plot of chunk unnamed-chunk-20" style="display: block; margin: auto;" />


Ordinal outcome variables
========================================================
class: small-code
- Obesity: "normal", "overweight" and "obese"

Predict obesity level based on age, gender, and level of physical activity, etc.

- My example: does corticosterone hormone level prior to social hierarchy formation predict social rank? 
<img src="figures/mlr.png" title="plot of chunk unnamed-chunk-21" alt="plot of chunk unnamed-chunk-21" style="display: block; margin: auto;" />



Ordinal outcome variables
========================================================
[Burkner & Vuorre (2019)](https://journals.sagepub.com/doi/10.1177/2515245918823199)
<img src="figures/box1.png" title="plot of chunk unnamed-chunk-22" alt="plot of chunk unnamed-chunk-22" style="display: block; margin: auto;" />



Example from Burkner & Vuorre (2019)
========================================================
class: small-code

```r
stemcell <- readRDS('stemcell.RDS') # from https://osf.io/9ekxw/
head(stemcell)
```

```
# A tibble: 6 x 2
  belief         rating
  <fct>           <dbl>
1 fundamentalist      4
2 fundamentalist      4
3 fundamentalist      4
4 fundamentalist      4
5 fundamentalist      4
6 fundamentalist      4
```

```r
str(stemcell)
```

```
tibble [829 x 2] (S3: tbl_df/tbl/data.frame)
 $ belief: Factor w/ 3 levels "moderate","fundamentalist",..: 2 2 2 2 2 2 2 2 2 2 ...
 $ rating: num [1:829] 4 4 4 4 4 4 4 4 4 4 ...
```

Plot it first 
========================================================
class: small-code
<img src="stats_snack_presentation-figure/unnamed-chunk-24-1.png" title="plot of chunk unnamed-chunk-24" alt="plot of chunk unnamed-chunk-24" style="display: block; margin: auto;" />

We should use
========================================================
class: small-code

`family = cumulative("probit")` `<-` this one

`family = acat("probit")`

`family = sratio("cloglog")`

========================================================
class: small-code
We know it's a bad idea but let's do it to see how bad it is

```r
fit_sc_metric <- brm(
  formula = rating ~ 1 + belief,
  data = stemcell,
  file = "fit_sc_metric.RDS")
```

Use cumulative() family function
========================================================
class: small-code

```r
fit_sc_cumulative <- brm(
  formula = rating ~ 1 + belief,
  data = stemcell,
  family = cumulative("probit"),
  file = "fit_sc_cumulative.RDS")
```


========================================================
class: small-code

```r
plot(conditional_effects(fit_sc_cumulative, categorical = T))
```

<img src="stats_snack_presentation-figure/unnamed-chunk-27-1.png" title="plot of chunk unnamed-chunk-27" alt="plot of chunk unnamed-chunk-27" style="display: block; margin: auto;" />


Model comparison
========================================================
class: small-code


```r
loo(fit_sc_metric, fit_sc_cumulative) -> loo_result_sc
loo_result_sc$diffs
```

```
                  elpd_diff se_diff
fit_sc_cumulative   0.0       0.0  
fit_sc_metric     -86.5      10.1  
```


========================================================
class: small-code

```r
summary(fit_sc_cumulative)
```

```
 Family: cumulative 
  Links: mu = probit; disc = identity 
Formula: rating ~ 1 + belief 
   Data: stemcell (Number of observations: 829) 
Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
         total post-warmup samples = 4000

Population-Level Effects: 
                     Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS
Intercept[1]            -1.25      0.08    -1.40    -1.09 1.00     3140
Intercept[2]            -0.64      0.07    -0.78    -0.49 1.00     3613
Intercept[3]             0.57      0.07     0.43     0.71 1.00     3580
belieffundamentalist    -0.24      0.09    -0.42    -0.06 1.00     3505
beliefliberal            0.31      0.09     0.13     0.50 1.00     3403
                     Tail_ESS
Intercept[1]             2965
Intercept[2]             3190
Intercept[3]             3096
belieffundamentalist     3274
beliefliberal            2833

Family Specific Parameters: 
     Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
disc     1.00      0.00     1.00     1.00 1.00     4000     4000

Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
and Tail_ESS are effective sample size measures, and Rhat is the potential
scale reduction factor on split chains (at convergence, Rhat = 1).
```


Thank you! Any questions?
========================================================
class: small-code

More resources

[Monotonic effects vignette](https://cran.r-project.org/web/packages/brms/vignettes/brms_monotonic.html)

[Burkner & Vuorre (2019) repository](https://osf.io/9ekxw/)

[Paul Burkner's website](https://paul-buerkner.github.io/publications/)

[MEleath's Statistical Rethinking](https://xcelab.net/rm/statistical-rethinking/)

[Statistical Rethinking in tidyvese and brms!!!! ](https://bookdown.org/ajkurz/Statistical_Rethinking_recoded/)



```r
sessionInfo()
```

```
R version 4.0.3 (2020-10-10)
Platform: x86_64-w64-mingw32/x64 (64-bit)
Running under: Windows 10 x64 (build 14393)

Matrix products: default

locale:
[1] LC_COLLATE=English_United States.1252 
[2] LC_CTYPE=English_United States.1252   
[3] LC_MONETARY=English_United States.1252
[4] LC_NUMERIC=C                          
[5] LC_TIME=English_United States.1252    

attached base packages:
[1] grid      stats     graphics  grDevices utils     datasets  methods  
[8] base     

other attached packages:
 [1] tweetrmd_0.0.8  brms_2.14.0     Rcpp_1.0.5      gifski_0.8.6   
 [5] gganimate_1.0.7 forcats_0.5.0   stringr_1.4.0   dplyr_1.0.2    
 [9] purrr_0.3.4     readr_1.4.0     tidyr_1.1.2     tibble_3.0.4   
[13] ggplot2_3.3.2   tidyverse_1.3.0 knitr_1.30     

loaded via a namespace (and not attached):
  [1] TH.data_1.0-10       colorspace_1.4-1     ellipsis_0.3.1      
  [4] ggridges_0.5.2       rsconnect_0.8.16     estimability_1.3    
  [7] markdown_1.1         base64enc_0.1-3      fs_1.5.0            
 [10] rstudioapi_0.11      farver_2.0.3         rstan_2.21.2        
 [13] DT_0.16              fansi_0.4.1          mvtnorm_1.1-1       
 [16] lubridate_1.7.9      xml2_1.3.2           splines_4.0.3       
 [19] codetools_0.2-16     bridgesampling_1.0-0 shinythemes_1.1.2   
 [22] bayesplot_1.7.2      jsonlite_1.7.1       broom_0.7.2         
 [25] dbplyr_1.4.4         shiny_1.5.0          compiler_4.0.3      
 [28] httr_1.4.2           emmeans_1.5.1        backports_1.1.10    
 [31] assertthat_0.2.1     Matrix_1.2-18        fastmap_1.0.1       
 [34] cli_2.1.0            later_1.1.0.1        tweenr_1.0.1        
 [37] htmltools_0.5.0      prettyunits_1.1.1    tools_4.0.3         
 [40] igraph_1.2.6         coda_0.19-4          gtable_0.3.0        
 [43] glue_1.4.2           reshape2_1.4.4       V8_3.2.0            
 [46] cellranger_1.1.0     vctrs_0.3.4          nlme_3.1-149        
 [49] crosstalk_1.1.0.1    xfun_0.18            ps_1.4.0            
 [52] rvest_0.3.6          mime_0.9             miniUI_0.1.1.1      
 [55] lifecycle_0.2.0      gtools_3.8.2         MASS_7.3-53         
 [58] zoo_1.8-8            scales_1.1.1         colourpicker_1.1.0  
 [61] hms_0.5.3            promises_1.1.1       Brobdingnag_1.2-6   
 [64] sandwich_3.0-0       parallel_4.0.3       inline_0.3.16       
 [67] shinystan_2.5.0      curl_4.3             gridExtra_2.3       
 [70] StanHeaders_2.21.0-6 loo_2.3.1            stringi_1.5.3       
 [73] dygraphs_1.1.1.6     pkgbuild_1.1.0       rlang_0.4.8         
 [76] pkgconfig_2.0.3      matrixStats_0.57.0   evaluate_0.14       
 [79] lattice_0.20-41      rstantools_2.1.1     htmlwidgets_1.5.2   
 [82] tidyselect_1.1.0     processx_3.4.4       plyr_1.8.6          
 [85] magrittr_1.5         R6_2.5.0             generics_0.0.2      
 [88] multcomp_1.4-14      DBI_1.1.0            pillar_1.4.6        
 [91] haven_2.3.1          withr_2.3.0          xts_0.12.1          
 [94] survival_3.2-7       abind_1.4-5          modelr_0.1.8        
 [97] crayon_1.3.4         progress_1.2.2       readxl_1.3.1        
[100] blob_1.2.1           callr_3.5.1          threejs_0.3.3       
[103] reprex_0.3.0         digest_0.6.25        xtable_1.8-4        
[106] httpuv_1.5.4         RcppParallel_5.0.2   stats4_4.0.3        
[109] munsell_0.5.0        shinyjs_2.0.0       
```


Monotonic model 
========================================================
class: small-code

```r
summary(fit1)
```

```
 Family: gaussian 
  Links: mu = identity; sigma = identity 
Formula: ls ~ mo(income) 
   Data: dat (Number of observations: 100) 
Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
         total post-warmup samples = 4000

Population-Level Effects: 
          Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
Intercept    30.84      1.68    27.54    34.20 1.00     2572     2274
moincome     14.61      0.75    13.14    16.08 1.00     2526     2221

Simplex Parameters: 
             Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
moincome1[1]     0.65      0.04     0.57     0.74 1.00     2765     2453
moincome1[2]     0.23      0.05     0.13     0.34 1.00     3581     2597
moincome1[3]     0.11      0.05     0.02     0.21 1.00     2494     1284

Family Specific Parameters: 
      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sigma     7.91      0.57     6.90     9.09 1.00     3102     2524

Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
and Tail_ESS are effective sample size measures, and Rhat is the potential
scale reduction factor on split chains (at convergence, Rhat = 1).
```

Treat income to be continuous
========================================================
class: small-code

```r
summary(fit2)
```

```
 Family: gaussian 
  Links: mu = identity; sigma = identity 
Formula: ls ~ income_num 
   Data: dat (Number of observations: 100) 
Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
         total post-warmup samples = 4000

Population-Level Effects: 
           Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
Intercept     24.55      2.50    19.70    29.52 1.00     3926     2898
income_num    13.71      0.90    11.90    15.46 1.00     3933     3081

Family Specific Parameters: 
      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sigma    10.05      0.72     8.76    11.60 1.00     3548     2800

Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
and Tail_ESS are effective sample size measures, and Rhat is the potential
scale reduction factor on split chains (at convergence, Rhat = 1).
```


Treat income as an unordered factor
========================================================
class: small-code

```r
summary(fit3)
```

```
 Family: gaussian 
  Links: mu = identity; sigma = identity 
Formula: ls ~ income 
   Data: dat (Number of observations: 100) 
Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
         total post-warmup samples = 4000

Population-Level Effects: 
          Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
Intercept    30.43      1.66    27.19    33.68 1.00     2984     2599
income2      29.07      2.28    24.56    33.52 1.00     3162     3065
income3      39.29      2.38    34.68    44.02 1.00     3189     3127
income4      44.32      2.25    40.01    48.74 1.00     3096     3038

Family Specific Parameters: 
      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
sigma     7.89      0.58     6.86     9.11 1.00     4010     2779

Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
and Tail_ESS are effective sample size measures, and Rhat is the potential
scale reduction factor on split chains (at convergence, Rhat = 1).
```


My own data example 
========================================================
class: small-code

```r
pcr_df <- readRDS("mup_paper_pcr.RDS")
head(pcr_df)
```

```
       mup3      mup20        ar    id rank
1 1.1290013 0.43307281 0.9383179  53-1    9
2 0.9772141 2.05703828 1.0574974  53-2   10
3 1.4349767 0.55279928        NA  53-4    2
4 1.0944346 1.36284295        NA  53-6    4
5 0.5214349 0.08426399 0.9193408  53-8   11
6 0.6410699 1.04691670 2.1784629 53-11    3
```

```r
table(pcr_df$rank)
```

```

 1  2  3  4  9 10 11 12 
 5  6  6  6  5  6  6  6 
```

```r
pcr_df <- pcr_df %>% 
  mutate(social_status = ifelse(rank == 1, "Alpha",
                                ifelse(rank < 5, "Subdominant" , "Subordinate"))) %>% 
  mutate(social_status = factor(social_status, ordered = T))
```

========================================================
class: small-code

```r
pcr_df %>% 
  ggplot(aes(rank, mup20))+
  geom_point(size = 7, shape = 21) + 
  geom_smooth(method = "lm")+
  theme_minimal(base_size = 24)
```

<img src="stats_snack_presentation-figure/unnamed-chunk-35-1.png" title="plot of chunk unnamed-chunk-35" alt="plot of chunk unnamed-chunk-35" style="display: block; margin: auto;" />

========================================================
class: small-code
<img src="stats_snack_presentation-figure/unnamed-chunk-36-1.png" title="plot of chunk unnamed-chunk-36" alt="plot of chunk unnamed-chunk-36" style="display: block; margin: auto;" />

========================================================
class: small-code

```r
mup_fit1 <- brm(mup20 ~ mo(social_status),
                data = pcr_df,
                file = "mup_fit1.RDS")

mup_fit2 <- brm(mup20 ~ as.numeric(rank),
                data = pcr_df,
                file = "mup_fit2.RDS")

loo_compare(loo(mup_fit1), loo(mup_fit2))
```

```
         elpd_diff se_diff
mup_fit1  0.0       0.0   
mup_fit2 -1.4       1.5   
```

```r
# actually glad to see elpd difference is very minimal for this dataset...
```

========================================================

```r
plot(conditional_effects(mup_fit1))
```

<img src="stats_snack_presentation-figure/unnamed-chunk-38-1.png" title="plot of chunk unnamed-chunk-38" alt="plot of chunk unnamed-chunk-38" style="display: block; margin: auto;" />

