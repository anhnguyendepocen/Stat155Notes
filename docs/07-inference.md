---
title: "Math 155 - Statistical Inference"
author: "Prof. Heggeseth"
date: "September 6, 2018"
output: 
  html_document: default
---


```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```
## Loading required package: xml2
```

```
## Loading required package: lattice
```

```
## Loading required package: ggformula
```

```
## Loading required package: ggstance
```

```
## 
## Attaching package: 'ggstance'
```

```
## The following objects are masked from 'package:ggplot2':
## 
##     geom_errorbarh, GeomErrorbarh
```

```
## 
## New to ggformula?  Try the tutorials: 
## 	learnr::run_tutorial("introduction", package = "ggformula")
## 	learnr::run_tutorial("refining", package = "ggformula")
```

```
## Loading required package: Matrix
```

```
## Registered S3 method overwritten by 'mosaic':
##   method                           from   
##   fortify.SpatialPolygonsDataFrame ggplot2
```

```
## 
## The 'mosaic' package masks several functions from core packages in order to add 
## additional features.  The original behavior of these functions should not be affected by this.
## 
## Note: If you use the Matrix package, be sure to load it BEFORE loading mosaic.
## 
## Have you tried the ggformula package for your plots?
```

```
## 
## Attaching package: 'mosaic'
```

```
## The following object is masked from 'package:Matrix':
## 
##     mean
```

```
## The following object is masked from 'package:ggplot2':
## 
##     stat
```

```
## The following objects are masked from 'package:dplyr':
## 
##     count, do, tally
```

```
## The following objects are masked from 'package:infer':
## 
##     prop_test, t_test
```

```
## The following objects are masked from 'package:stats':
## 
##     binom.test, cor, cor.test, cov, fivenum, IQR, median, prop.test,
##     quantile, sd, t.test, var
```

```
## The following objects are masked from 'package:base':
## 
##     max, mean, min, prod, range, sample, sum
```

# Statistical Inference

Let's remember our goal of "turning data into information." Based on a sample data set, we want to be able to say something about the larger population of interest. This endeavor is called **statistical inference**. In statistical inference, we care about using sample data to make statements about "truths" in the larger population.

- To make causal inferences in the sample, we need to account for all possible confounding variables, or we need to randomize the "treatment" and assure there are no other possible reasons for an observed effect.
- To generalize to a larger population, we need the sample to be representative of the larger population. Ideally, that sample would be randomly drawn from the population. If we actually have a census in that we have data on country, state, or county-level, then we can consider the observed data as a "snapshot in time". There are random processes that govern how things behave over time, and we have just observed one period in time.

Let's do some statistical inference based on a simple random sample (SRS) of 100 flights leaving NYC in 2013. 

<div class="reflect">
<p>What is our population of interest? What population could we generalize to?</p>
</div>


```r
data(flights)
flights <- flights %>% 
    na.omit() %>%
    mutate(season = case_when(
      month %in% c(10:12, 1:3) ~ "winter",
      month %in% c(4:9) ~ "summer"
    )) %>% 
    mutate(day_hour = case_when(
      between(hour, 1, 12) ~ "morning",
      between(hour, 13, 24) ~ "afternoon"
    )) %>% 
    select(arr_delay, dep_delay, season, day_hour, origin, carrier)

set.seed(2018)
## This creates a dataset called flights_samp
## that contains the SRS of size 100
flights_samp <- flights %>%
    sample_n(size = 100)
```

We've already been thinking about random variation and how that plays a role in the conclusions we can draw. In this chapter, we will formalize two techniques that we use to do perform statistican inference: confidence intervals and hypothesis tests. 


## Confidence Intervals

A **confidence interval** (also known as an interval estimate) is an interval of plausible values of the unknown population parameter of interest based on randomly sampled data. However, the interval computed from a particular sample does not necessarily include the true value of the parameter. Since the observed data are random samples from the population, the confidence interval obtained from the data is also random.

The **confidence level** represents the proportion of possible random samples and thus confidence intervals that contain the true value of the unknown population parameter. Typically, the confidence level is represented by $(1-\alpha)$ such that if $\alpha = 0.05$, then the confidence level is 95% or 0.95. What is $\alpha$? We will define $\alpha$ when we get to hypothesis testing, but for now, we will describe $\alpha$ as an error probability. Because we want an error probability to be low, it makes sense that the confidence level is $(1-\alpha)$.


**Valid Interpretation:** Assuming the sampling distribution model is accurate, I am 95% confident that my confidence interval of (lower, upper) contains the true population parameter (*put in context*), which means that we'd expect 95% of samples to lead to intervals that contain the true population parameter value. We just don't know if our particular interval from our study contains that true population parameter value or not.

### Via Classical Theory

If we can use theoretical probability to approximate the sampling distribution, then we can create a confidence interval by taking taking our estimate and adding and subtracting a margin of error:

$$\text{Estimate }\pm \text{ Margin of Error}$$

The margin of error is typically constructed using z-scores from the sampling distribution (such as $z^* = 2$ that corresponds to a 95% confidence interval) and an estimate of the standard deviation of the estimate, called a **standard error**. 

Once we have an estimate of the standard deviation (through a formula or R output) and an approximate sampling distribution, we can create the interval estimate:

$$\text{Estimate }\pm z^* *SE(\text{Estimate})$$

The fact that confidence intervals can be created as above is rooted in the Central Limit Theorem (CLT). If you would like to see how the form above is derived, see the Math Box below.


<div class="mathbox">
<p>(Optional) Deriving confidence intervals from the CLT</p>
<p>The CLT originally expresses that</p>
<p><span class="math display">\[\frac{\text{sample mean} - \text{true mean}}{\text{true std. dev. of sample mean}} \sim \text{Normal}(0,1)\]</span></p>
<p>It turns out that the CLT also applies to regression coefficients:</p>
<p><span class="math display">\[\frac{\hat{\beta} - \beta}{\text{ESTIMATED std. dev. of }\hat{\beta}} \sim \text{Normal}(0,1)\]</span></p>
<p>From there we can write a probability statement using the 68-95-99.7 rule of the normal distribution and rearrange the expression using algebra:</p>
<p><span class="math display">\[P(-2\leq\frac{\hat{\beta} - \beta}{SE}\leq2) = 0.95\]</span></p>
<p><span class="math display">\[P(-2 SE\leq\hat{\beta} - \beta \leq2 SE ) = 0.95\]</span></p>
<p><span class="math display">\[P(-2 SE-\hat{\beta} \leq  -\beta \leq2 SE-\hat{\beta} ) = 0.95\]</span></p>
<p><span class="math display">\[P(2 SE+\hat{\beta} \geq \beta \geq -2 SE+\hat{\beta} ) = 0.95\]</span></p>
<p><span class="math display">\[P(\hat{\beta}-2 SE \leq \beta \leq\hat{\beta}+2 SE ) = 0.95\]</span></p>
<p>You’ve seen the Student T distribution introduced in the previous chapter. We used the Normal distribution in this derivation, but it turns out that the Student t distribution is more accurate for linear regression coefficients (especially if sample size is small). The normal distribution is appropriate for logistic regression coefficients.</p>
</div>

### Via Bootstrapping


In order to gauge the sampling variability, we can treat our sample as our "fake population" and generate repeated samples from this "population" using the technique of bootstrapping.

Once we have a distribution of sample statistics based on the generated data sets, we'll create a confidence interval by finding the $\alpha/2$th percentile and the $(1-\alpha/2)$th percentile for our lower and upper bounds. For example, for a 99% bootstrap confidence interval, $\alpha = 0.01$ and you would find the values that are the 0.5th and 99.5th percentiles.

## Confidence Interval Examples

### Proportion Outcome

Let's return to the flight data and estimate the proportion of afternoon flights based on a sample of 100 flights from NYC. 

First, the classical 95% confidence interval can be constructed using the theory of the Binomial Model (Do the 3 conditions hold? Is n large enough for it to look Normal?)


```r
flights_samp %>% 
  summarize(prop = count(day_hour)/100) %>%
  mutate(SE = sqrt(prop*(1-prop)/100)) %>%
  mutate(lb = prop - 2*SE, ub = prop + 2*SE)
```

```
## Warning: `data_frame()` is deprecated as of tibble 1.1.0.
## Please use `tibble()` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_warnings()` to see where this warning was generated.
```

```
## # A tibble: 1 x 4
##    prop     SE    lb    ub
##   <dbl>  <dbl> <dbl> <dbl>
## 1  0.53 0.0499 0.430 0.630
```

Or we could bootstrap and get our confidence interval that way. 


```r
alpha <- 0.05

boot_data <- mosaic::do(1000)*( 
    flights_samp %>% # Start with the SAMPLE (not the FULL POPULATION)
      sample_frac(replace = TRUE) %>% # Generate by resampling with replacement
      summarize(prop = count(day_hour)/100) # Calculate statistics
)

boot_data %>%
  summarize(lower = quantile(prop, alpha/2),
    upper = quantile(prop, 1-alpha/2))
```

```
##   lower upper
## 1  0.44  0.63
```

Our confidence interval gives a sense of the true proportion of flights departed NYC in the afternoon, keeping in mind that this sample could be one of the unlucky samples (the 5%) that have intervals that don't contain the true value.

### Mean and then Median

Perhaps you really care about the arrival delay time because you have somewhere important you need to be when you take flights out of NYC. Let's estimate the mean arrival delay based on a sample of 100 flights from NYC. 

First off, let's create a classical confidence interval. Since our sample size is relatively large, we can use the Normal model (instead of William Gosset's work). We use the sample standard deviation and plug into the SE formula. 


```r
flights_samp %>% 
  summarize(mean = mean(arr_delay), s = sd(arr_delay)) %>%
  mutate(SE = s/sqrt(100)) %>%
  mutate(lb = mean - 2*SE, ub = mean + 2*SE)
```

```
## # A tibble: 1 x 5
##    mean     s    SE     lb    ub
##   <dbl> <dbl> <dbl>  <dbl> <dbl>
## 1  6.36  33.9  3.39 -0.422  13.1
```



```r
alpha <- 0.05

boot_data <- mosaic::do(1000)*( 
    flights_samp %>% # Start with the SAMPLE (not the FULL POPULATION)
      sample_frac(replace = TRUE) %>% # Generate by resampling with replacement
      summarize(means = mean(arr_delay),medians = median(arr_delay)) # Calculate statistics
)

boot_data %>%
  summarize(lower = quantile(means, alpha/2),
    upper = quantile(means, 1-alpha/2))
```

```
## # A tibble: 1 x 2
##   lower upper
##   <dbl> <dbl>
## 1 0.230  13.4
```

Our confidence interval gives potential values for of the true mean arrival delay for flights that departed NYC, keeping in mind that this sample could be one of the unlucky samples (the 5%) that have intervals that don't contain the true value. Also remember that the mean is sensitive to outliers...Let's consider the median.

We get a slightly different story if we are interested in the middle number versus the average. But notice, we aren't using a classical CI here because the sampling distribution of a median is not necessarily Normal. 



```r
boot_data %>%
  summarize(lower = quantile(medians, alpha/2),
    upper = quantile(medians, 1-alpha/2))
```

```
## # A tibble: 1 x 2
##   lower upper
##   <dbl> <dbl>
## 1  -8.5   5.5
```

### Logistic Regression Model

Are the same relative numbers of morning flights in the winter and the summer? Let's fit a logistic regression model and see what our sample says.


```r
flights_samp$afternoon = flights_samp$day_hour == 'afternoon'

glm.afternoon <- glm(afternoon ~ season, data = flights_samp, family = 'binomial')
summary(glm.afternoon)
```

```
## 
## Call:
## glm(formula = afternoon ~ season, family = "binomial", data = flights_samp)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.332  -1.126   1.030   1.030   1.230  
## 
## Coefficients:
##              Estimate Std. Error z value Pr(>|z|)
## (Intercept)   -0.1226     0.2863  -0.428    0.668
## seasonwinter   0.4793     0.4036   1.188    0.235
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 138.27  on 99  degrees of freedom
## Residual deviance: 136.85  on 98  degrees of freedom
## AIC: 140.85
## 
## Number of Fisher Scoring iterations: 4
```

The output for the model gives standard errors for the slopes, so we can create the classical confidence intervals directly from the output, 


```r
confint(glm.afternoon)
```

```
## Waiting for profiling to be done...
```

```
##                   2.5 %    97.5 %
## (Intercept)  -0.6905940 0.4389059
## seasonwinter -0.3081988 1.2791414
```

Or with bootstrapping,


```r
boot_data <- mosaic::do(1000)*( 
    flights_samp %>% # Start with the SAMPLE (not the FULL POPULATION)
      sample_frac(replace = TRUE) %>% # Generate by resampling with replacement
      glm(afternoon ~ season, data = ., family = 'binomial') # Calculate statistics
)

boot_data %>%
  summarize(lower = quantile(seasonwinter, alpha/2),
    upper = quantile(seasonwinter, 1-alpha/2))
```

```
##        lower    upper
## 1 -0.3503645 1.367213
```

Knowing that the sample is random, the interval estimate for the logistic regression slope is given by the confidence interval. But for logistic regression, we exponentiate the slopes to get an more interpretable value, the odds ratio. Here, we are comparing the odds of having a flight in the afternoon between winter months (numerator) and summer months (denominator). Is 1 in the interval? If so, what does that tell you?


```r
exp(confint(glm.afternoon))
```

```
## Waiting for profiling to be done...
```

```
##                  2.5 %   97.5 %
## (Intercept)  0.5012782 1.551009
## seasonwinter 0.7347692 3.593553
```

```r
boot_data %>%
  summarize(lower = exp(quantile(seasonwinter, alpha/2)),
    upper = exp(quantile(seasonwinter, 1-alpha/2)))
```

```
##       lower    upper
## 1 0.7044313 3.924396
```



### Linear Regression Model Slope (Categorical Variable)

Everything says that there are longer delays in winter. Is that actually true? Let's fit a linear regression model to test it


```r
lm.delay <- lm(arr_delay ~ season, data = flights_samp)
summary(lm.delay)
```

```
## 
## Call:
## lm(formula = arr_delay ~ season, data = flights_samp)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -41.694 -21.694  -8.694  11.547 166.306 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)
## (Intercept)    6.6939     4.8687   1.375    0.172
## seasonwinter  -0.6547     6.8176  -0.096    0.924
## 
## Residual standard error: 34.08 on 98 degrees of freedom
## Multiple R-squared:  9.408e-05,	Adjusted R-squared:  -0.01011 
## F-statistic: 0.009221 on 1 and 98 DF,  p-value: 0.9237
```

The classical CI for the slope is given with by


```r
confint(lm.delay)
```

```
##                   2.5 %   97.5 %
## (Intercept)   -2.967923 16.35568
## seasonwinter -14.183889 12.87457
```

or with bootstrapping,


```r
boot_data <- mosaic::do(1000)*( 
    flights_samp %>% # Start with the SAMPLE (not the FULL POPULATION)
      sample_frac(replace = TRUE) %>% # Generate by resampling with replacement
      lm(arr_delay ~ season, data = .) # Calculate statistics
)

boot_data %>%
  summarize(lower = quantile(seasonwinter, alpha/2),
    upper = quantile(seasonwinter, 1-alpha/2))
```

```
##      lower  upper
## 1 -15.0511 11.794
```

The 95% confidence interval gives a sense of the difference of mean arrival delays of flights between winter and summer is given by the confidence interval. Is zero in the interval? If so, what does that tell you?


### Linear Regression Model Slope (Quantitative Var)

How well can the departure delay predict the arrival delay? What is the effect of departing 1 more minute later? Does that correspond to 1 minute later in arrival on average? Let's look at the estimated slope between departure and arrival delays for the sample of 100 flights from NYC.


```r
lm.delay2 <- lm(arr_delay ~ dep_delay, data = flights_samp)
summary(lm.delay2)
```

```
## 
## Call:
## lm(formula = arr_delay ~ dep_delay, data = flights_samp)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -27.396 -12.397  -2.904   9.588  60.529 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -2.58743    1.85522  -1.395    0.166    
## dep_delay    1.00420    0.06173  16.267   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 17.72 on 98 degrees of freedom
## Multiple R-squared:  0.7297,	Adjusted R-squared:  0.727 
## F-statistic: 264.6 on 1 and 98 DF,  p-value: < 2.2e-16
```

The classical CI for the slope is given with by


```r
confint(lm.delay2)
```

```
##                  2.5 %   97.5 %
## (Intercept) -6.2690537 1.094198
## dep_delay    0.8816967 1.126705
```

or with bootstrapping,


```r
boot_data <- mosaic::do(1000)*( 
    flights_samp %>% # Start with the SAMPLE (not the FULL POPULATION)
      sample_frac(replace = TRUE) %>% # Generate by resampling with replacement
      lm(arr_delay ~ dep_delay, data = .) # Calculate statistics
)

boot_data %>%
  summarize(lower = quantile(dep_delay, alpha/2),
    upper = quantile(dep_delay, 1-alpha/2))
```

```
##       lower    upper
## 1 0.9173677 1.181704
```


If the flight leaves an additional minute later, then we'd expect the arrival delay to be increased by a value in the interval above, on average. 

### Confidence Intervals for Prediction

Imagine we are on a plane, we left 15 minutes late, how late will arrive? Since we only have a sample of 100 flights, we are a bit unsure of our prediction. 

A classical CI can give us an interval estimate of what the prediction should be (if we had data on all flights).


```r
predict(lm.delay2, newdata = data.frame(dep_delay = 15), interval = 'confidence')
```

```
##        fit      lwr      upr
## 1 12.47558 8.881202 16.06996
```

This is taking into account how uncertain we are about our model prediction because our model is based on sample data rather than population data. 


### Prediction Intervals

We also know that every flight is different (different length, different weather conditions, etc), so the true arrival delay won't be exactly what we predict. 

So to get a better prediction for our arrival delay, we can account for the size of errors or residuals by creating a **prediction interval**. This interval will be much wider than the confidence interval because it takes into account how far the true values are from the prediction line. 


```r
predict(lm.delay2, newdata = data.frame(dep_delay = 15), interval = 'prediction')
```

```
##        fit       lwr      upr
## 1 12.47558 -22.86868 47.81985
```


### Probability Theory vs. Bootstrapping

In the modern age, computing power allows us to perform boostrapping easily to create confidence intervals. Before computing was as powerful as it is today, scientists needed mathematical theory to provide simple formulas for confidence intervals.

If certain assumptions hold, the mathematical theory proves to be just as accurate and less computationally-intensive than bootstrapping. Many scientists using statistics right now learned the theory because when they learned statistics, computers were not powerful enough to handle techniques such as bootstrapping.

Why do we teach both the mathematical theory and bootstrapping? You will encounter both types of techniques in your fields, and you'll need to have an understanding of what these techniques are to bridge the gap until statistical inference uses modern computational techniques more widely.

## Hypothesis Testing

Hypothesis testing is another tool that can be used for statistical inference. Let's warm up to the ideas of hypothesis testing by considering two broad types of scientific questions: (1) *Is there* a relationship? (2) *What* is the relationship?

Suppose that we are thinking about the relationship between housing prices and square footage. Accounting for sampling variation...

- ...**is there** a real relationship between price and living area?
- ...**what** is the real relationship between price and living area?

Whether by the Central Limit Theorem (mathematical theory) or bootstrapping, confidence intervals provide a *range of plausible values* for the true population parameter and allow us to answer both types of questions:

- **Is there** a real relationship between price and living area?
    - Is the null value in the interval?
- **What** is the relationship between price and living area?
    - Look at the estimate and the values in the interval


**Hypothesis testing** is a general framework for answering questions of the first type. It is a general framework for making decisions between two "theories".

- **Example 1**    
    Decide between: true support for a law = 50% vs. true support $\neq$ 50%

- **Example 2**    
    In the model $\text{Price} = \beta_0 + \beta_1\text{Living Area} + \text{Error}$, decide between $\beta_1 = 0$ and $\beta_1 \neq 0$.

In a hypothesis test, we use data to decide between two "hypotheses" labeled as follows:

1. **Null hypothesis** ($H_0$ = "H naught")    
Hypothesis that is assumed to be true by default.    
A status quo hypothesis: hypothesis of no effect/relationship/difference.

2. **Alternative hypothesis** ($H_A$ or $H_1$)    
A non-status quo hypothesis.    
Claim being made about the population.

### Test statistics

Let's consider the question: Is there a relationship between house price and living area? We can try to answer that with the linear regression model below:

$$\text{Price} = \beta_0 + \beta_1\text{Living Area} + \text{Error}$$

We would phrase our null and alternative hypotheses as follows:

$$H_0: \beta_1 = 0 \qquad \text{vs.} \qquad H_A: \beta_1 \neq 0$$

The null hypothesis $H_0$ describes the situation of "no relationship" because it hypothesizes that the true slope $\beta_1$ is 0. The alternative hypothesis posits a real relationship: the true population slope $\beta_1$ is not 0. That is, there is not no relationship. (*Double negatives!*)

To gather evidence, we collect data and fit a model. From the model, we can compute a **test statistic**, which tells us how far the observed slope is from the null hypothesis value of 0 (called the **null value**). The test statistic is a *discrepancy measure* where large values indicate higher discrepancy with the null hypothesis, $H_0$.

The test statistic below is a reasonable proposal for our modeling context:

$$\text{Test statistic} = \frac{\text{estimate} - \text{null value}}{\text{std. error of estimate}}$$

It looks like a z-score. It expresses: how far away is our estimate from the null value in units of standard error? With large values (in magnitude) of the test statistic, our data (and our estimate) is discrepant with what the null hypothesis proposes because our estimate is quite far away from the null value in standard error units.

### Logic of hypothesis testing

How large in magnitude must the test statistic be in order to make a decision between $H_0$ and $H_A$? We will use another metric called a **p-value**.

**Assuming $H_0$ is true**, we ask: What is the chance of observing a test statistic which is "as or even more extreme" than the one we just saw? This conditional probability is called a **p-value**, $P(|T| \geq t_{obs} | H_0\text{ is true})$.

If our test statistic is large, then our estimate is quite far away from the null value (in standard error units), and then the chance of observing someone this large or larger (assuming $H_0$ is true) would be very small. **A large test statistic leads to a small p-value.**

If our test statistic is small, then our estimate is quite close to the null value (in standard error units), and then the chance of observing someone this large or larger (assuming $H_0$ is true) would be very large. **A small test statistic leads to a large p-value.**

#### Making Decisions

If the p-value is "small", then we reject $H_0$ in favor of $H_A$. Why? A small p-value (by definition) says that if the null hypotheses were indeed true, we are unlikely to have seen such an extreme discrepancy measure (test statistic). We made an assumption that the null is true, and operating under that assumption, we observed something odd and unusual. This makes us reconsider our null hypothesis.

How small is small enough for a p-value? We will set a threshold $\alpha$. P-values less than this threshold will be "small enough". When we talk about error rates of the decisions associated with rejecting or not rejecting the null hypothesis, the meaning of $\alpha$ will become more clear.

### Summary of procedure

1. State hypotheses $H_0$ and $H_A$.
2. Select $\alpha$, a threshold for what is considered to be a small enough p-value.
3. Calculate a test statistic.
4. Calculate the corresponding p-value.
5. Make a decision:
    - If p-value $\leq\alpha$, reject $H_0$ in favor of $H_A$.
    - Otherwise, we fail to reject $H_0$ for lack of evidence.    
        (Jurors' decisions are "guilty" and "not guilty". Not "guilty" and "innocent".)

### Testing single model coefficients

A major emphasis of our course is regression models. It turns out that many scientific questions of interest can be framed using regression models.

In the `summary()` output, R performs the following hypothesis test by default (for any slope coefficient $\beta$):

$$H_0: \beta = 0 \qquad \text{vs} \qquad H_A: \beta \neq 0$$

$$t_{obs} = \text{Test statistic} = \frac{\text{estimate} - \text{null value}}{\text{std. error of estimate}} = \frac{\text{estimate} - 0}{\text{std. error of estimate}} $$

Note that test statistics are random variables! Why? Because they are based on our random sample of data. Thus, it will be helpful to understand the distributions of test statistics in terms of probability density functions.

<div class="reflect">
<p>If <span class="math inline">\(H_0\)</span> were true, where would the probability density function of the test statistic be centered?</p>
</div>

### Distributions of test statistics

What test statistics are we likely to get if $H_0$ is true? The probability density function of the test statistic "under $H_0$" (that is, if $H_0$ is true) is shown below. Note that it is centered at 0. This distribution shows that if indeed the null is true, there is variation in the test statistics we might obtain from random samples, but most test statistics are around zero.

It would be very unlikely for us to get a pretty large (extreme) test statistic if indeed $H_0$ were true. Why? The density drops rapidly at more extreme values.

<img src="07-inference_files/figure-html/unnamed-chunk-22-1.png" width="768" style="display: block; margin: auto;" />

### Graphical description of p-values

Suppose that our observed test statistic is 2.

What test statistics are "as or more extreme"?

- Absolute value of test statistic is at least 2: $|\text{Test statistic}| \geq 2$
- In other words: $\text{Test statistic} \geq 2$ or $\text{Test statistic} \leq -2$

The p-value corresponding to our test statistic is the area under the probability density function in those "as or more extreme" regions.

<img src="07-inference_files/figure-html/unnamed-chunk-23-1.png" width="960" style="display: block; margin: auto;" />

### Example: Linear Regression

Below we fit a linear regression model of house price on living area:


```r
homes <- read.delim("http://sites.williams.edu/rdeveaux/files/2014/09/Saratoga.txt")
mod_homes <- lm(Price ~ Living.Area, data = homes)
confint(mod_homes) ## 95% confidence interval by default
```

```
##                 2.5 %     97.5 %
## (Intercept) 3647.6958 23231.0922
## Living.Area  107.8616   118.3835
```

```r
summary(mod_homes)$coefficients
```

```
##               Estimate  Std. Error   t value      Pr(>|t|)
## (Intercept) 13439.3940 4992.352849  2.691996  7.171207e-03
## Living.Area   113.1225    2.682341 42.173065 9.486240e-268
```

The `t value` column is the test statistic, and the `Pr(>|t|)` column is the p-value. Note that the "t" comes from the Student t distribution.

- What are $H_0$ and $H_A$?    
    We write the model $\text{Price} = \beta_0 + \beta_1\text{Living Area} + \text{Error}$.    
    $H_0: \beta_1 = 0$ (There is no relationship between price and living area.)    
    $H_A: \beta_1 \neq 0$ (There is a relationship between price and living area.)    
- For a threshold $\alpha = 0.05$, what is the decision regarding $H_0$?    
    Note that when you see `e` in R output, this means "10 to the power". So `9.486240e-268` means $9.49 \times 10^{-268}$. This p-value is less than our threshold $\alpha = 0.05$, so we reject $H_0$ and say that we have significant evidence for a relationship between price and living area.
- Is this consistent with the confidence interval?    
    This result is consistent with the 95% confidence interval in that the interval does not contain 0.

### Example: Logistic Regression

Below we fit a logistic regression model of whether a movie made a profit (response) on whether it is a history film:


```r
movies <- read.csv("https://www.dropbox.com/s/73ad25v1epe0vpd/tmdb_movies.csv?dl=1")
mod_movies <- glm(profit==TRUE ~ History, data = movies, family = "binomial")
confint(mod_movies) ## 95% confidence interval by default
```

```
## Waiting for profiling to be done...
```

```
##                   2.5 %    97.5 %
## (Intercept)  0.09438169 0.2102419
## HistoryTRUE -0.26484980 0.3085225
```

```r
summary(mod_movies)$coefficients
```

```
##               Estimate Std. Error   z value     Pr(>|z|)
## (Intercept) 0.15226921 0.02955458 5.1521364 2.575356e-07
## HistoryTRUE 0.02074995 0.14604883 0.1420754 8.870204e-01
```

The `z value` column is the test statistic, and the `Pr(>|z|)` column is the p-value. Note that the "z" refers to z-score and the Normal distribution.

Try for yourselves!

- What are $H_0$ and $H_A$?
- For a threshold $\alpha = 0.05$, what is the decision regarding $H_0$?
- Is this consistent with the confidence interval?

### Errors 

Just as with model predictions, we may make errors when doing hypothesis tests. 

We may decide to reject $H_0$ when it is actually true. We may decide to not reject $H_0$ when it is actually false. 

We give these two types of errors names. 

**Type 1 Error** is when you reject $H_0$ when it is actually true. This is a false positive because you are concluding there is a real relationship when there is none. This would happen if one study published that coffee causes cancer in one group of people, but no one else could actually replicate that result since coffee doesn't actually cause cancer.  

**Type 2 Error** is when you don't reject $H_0$ when it is actually false. This is a false negative because you would conclude there is no real relationship when there is a real relationship. This happens when our sample size is not large enough to detect the real relationship due to the large amount of noise due to sampling variability.

We care about both of these types of errors. Sometimes we prioritize one over the other. Based on the framework presented, we control the chance of a Type 1 error through the confidence level/p-value threshold we used. In fact, the chance of a Type 1 Error is $\alpha$,

$$P(\text{ Type 1 Error }) = P(\text{ Reject }H_0 ~|~H_0\text{ is true} ) =  \alpha$$

Let $\alpha = 0.05$ for a moment. If the Null Hypothesis ($H_0$) is actually true, then about 5% of the time, we'd get unusual test statistics through random chance. With those samples, we would incorrectly conclude that there was a real relationship.

The chance of a Type 2 Error is often notated as $\beta$ (but this is not the same value as the slope),

$$P(\text{ Type 2 Error }) = P(\text{ Fail to Reject }H_0 ~|~H_0\text{ is false} ) =  \beta$$

We control the chance of a Type 2 error when choosing the sample size. With a larger sample size $n$, we will be able to more accurately detect real relationships. The **power** of a test, the ability to detect real relationships, is $P(\text{Reject }H_0 ~|~H_0\text{ is false}) = 1 - \beta$.  In order to calculate these two probabilities, we'd need to know the value (or at least a good idea) of the true effect. 


To recap, 

- If we lower $\alpha$, the threshold we use to determine the p-value is small enough to reject $H_0$, we can reduce the chance of a Type 1 Error. 
- Lowering $\alpha$ makes it harder to reject $H_0$, thus we might have a higher chance of a Type 2 Error. 
- Another way we can increase the power and thus decrease the chance of a Type 2 Error is to increase the sample size. 

## Statistical Significance v. Practical Significance

The common underlying question that we ask as Statisticians is "Is there a real relationship in the population?"

We can use confidence intervals or hypothesis testing to help us answer this question.

If we note that the no relationship value is NOT in the confidence interval or the p-value is less then $\alpha$, we can say that there is significant evidence to suggest that there is a real relationship. We can conclude there is a **statistically significant** relationship because the relationship we observed it is unlikely be due only to sampling variabliliy.

But as we discussed in class, there are two ways you can control the width of a confidence interval. If we increase the sample size $n$, the standard error decreases and thus decreasing the width of the interval. If we decrease our confidence level (increase $\alpha$), then we decrease the width of the interval. 

A relationship is **practically significant** if the estimated effect is large enough to impact real life decisions. For example, an Internet company may run a study on website design. Since data on observed clicks is fairly cheap to obtain, their sample size is 1 million people (!). With large data sets, we will conclude almost every relationship is statistically significant because the variability will be incredibly small. That doesn't mean we should always change the website design. How large of an impact did the size of the font make on user behavior? That depends on the business model. On the other hand, in-person human studies are expensive to run and sample sizes tended to be in the 100's. There may be a true relationship but we can't distinguish the "signal" from the "noise" due to the higher levels of sampling variability. While we may not always have statistical significance, the estimated effect is important to consider when designing the next study. 

Hypothesis tests are useful in determining statistical significance (Answering: "Is there a relationship?").

Confidence intervals are more useful in determining practical significance (Answering: "What is the relationship?")


## Model Selection

In practice, you need to decide which variables should be included in an model. This is referred to as **model selection** or **variable selection**. We have many tools that can help us make these decisions.

- Exploratory visualizations give us an indication for which variables have the strongest relationship with our response of interest (also if it is not linear)
- $R^2$ can tell us the percent of variation in our response that is explained by the model -- We want HIGH $R^2$
- The standard deviation of the residuals, $s_e$ (residual standard error), tells us the average magnitude of our residuals (prediction errors for our data set) -- We want LOW $s_e$

With the addition of statistical inference in our tool set, we now have many other ways to help guide our decision making process.


- Confidence intervals or tests for individual coefficients or slopes ($H_0: \beta_k = 0$, population slope for kth variable is 0 meaning no relationship)
    - See `summary(lm1)`
- Nested tests for a subset of coefficients or slopes ($H_0: \beta_k = 0$, population slope for kth variable is 0 meaning no relationship)
    - `anova(lm1, lm2)` for comparing two linear regression models (one larger model and one model with some variables removed)
    - `anova(glm1, glm2, test = 'LRT')` for comparing two logistic regression models (one larger model and one model with some variables removed)

In general, we want a simple model that works well. We want to follow Occam's Razor, a philosophy that suggests that it is a good principle to explain the phenomena by the simplest hypothesis possible. In our case, that mean the fewest variables in our models. 

So there are model selection criteria that we can use that penalizes you for having too many variables. Here are some below.


- Choose a model with a LOWER Akaike Information Criterion (AIC) and Bayesian Information Criterion (BIC) calculated as
  
 $$AIC = n\log(SSE) - n\log(n) + 2(k+1) = -2\log(L) + 2(k+1)$$
 $$BIC = n\log(SSE) -n\log(n) + (k+1)\log(n)= -2\log(L) + (k+1)\log(n)$$  
    - Calculated using `BIC(lm1)` and `AIC(lm1)`

- Choose a model with a higher adjusted $R^2$, calculated as,
$$R^2_{adj} = 1 - \frac{SSE/(n-k-1)}{SSTO/(n-1)}$$
where $k$ is the number of estimated coefficients in the model. 
    - Find the adjusted R squared in the output of `summary(lm1)`
    
    
If our goal is prediction, then we will want to choose a model that has the lowest prediction error. BUT, if we fit our model to our data and then calculate the prediction error from that SAME data, we aren't getting an accurate estimate of the prediction error because we are cheating. We aren't doing predictions for new data values. 
 
 
**Training and Testing**

In order to be able to predict for new data, we can randomly split our observed data into two groups, a **training** set and a **testing** set (also known as a validation or hold-out set). 

- Fit the model to the training set and do prediction on the observations in the testing set.
- The prediction Mean Squared Error, $\frac{1}{n}\sum(y_i - \hat{y}_i)^2$ can be calulated based on the predictions from the testing set. 
    
**Drawbacks of Validation Testing**

- 1) The MSE can be highly variable as it depends on how you randomly split the data. 

- 2) We aren't fully utilizing all of our data to fit the model; therefore, we will tend to overestimate the prediction error. 


**K-Fold Cross-Validation**

If we have a small data set and we want to fully use all of the data in our training, we can do **K-Fold Cross-Validation**. The steps are as follows:

- Randomly splitting the set of observations into $k$ groups, or folds, of about equal size.

- The first group is treated as the test set and the method or model is fit on the remaining $k-1$ groups. The MSE is calculated on the observations in the test set. 

- Repeat $k$ times; each group is treated as the test set once. 

- The k-fold CV estimate of MSE is an average of these values,
\[CV_{(k)} = \frac{1}{k}\sum^k_{i=1}MSE_i \]
where $MSE_i$ is the MSE based on the $i$th group as the test group. In practice, one performs k-fold CV with $k=5$ or $k=10$ as it reduces the computational time and it also is more accurate. 


```r
require(boot)
cv.err <- cv.glm(data,glm1, K = 5)
sqrt(cv.err$delta[1]) #out of sample average prediction error
```
