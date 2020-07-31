

# Logistic Regression Models

If you want to predict a binary categorical variable (only 2 possible outcomes), the standard linear regression models don't apply. If you let the two possible outcomes be 0 and 1, you'll never get a straight line relationship with any $x$ variable. 

Throughout this section, we will refer to one outcome as "success" (denoted 1) and "failure" (denoted 0). Depending on the context of the data, the success could be a negative thing such as "heart attack" or "20 year mortality" or it could be a positive thing such as "passing a course".

We will denote $p$ to be the chance of success and $1-p$ be the chance of failure. We want to build a model to explain why the chance of one outcome (success) may be higher for one group of people in comparison to another. 

## Logistic and Logit

The **logistic function** is an S shaped curve (sigmoid curve). For our purposes, the function will take the form

$$f(x) = \frac{1}{1 + e^{b_0 +b_1x}}$$

<img src="04-logistic-regression_files/figure-html/unnamed-chunk-1-1.png" width="672" />

For any real value $x$, this function, $f(x)$, will be a value between 0 and 1. This is perfect for us since probabilities or chances should also be between 0 and 1. 

In fact, we'll let the chance of the other outcome (failure), $1-p$, be modeled by this S function.

$$1-p = \frac{1}{1 + e^{b_0 +b_1x}}$$
<div class="mathbox">
<p>With a bit of algebra and rearranging terms, we can write this equation in terms of <span class="math inline">\(p\)</span>, the chance of success.</p>
<p><span class="math display">\[1-p = \frac{1}{1 + e^{b_0 +b_1x}}\]</span> <span class="math display">\[p = 1-\frac{1}{1 + e^{b_0 +b_1x}}\]</span></p>
<p><span class="math display">\[p = 
\frac{1 + e^{b_0 +b_1x}}{1 + e^{b_0 +b_1x}}-\frac{1}{1 + e^{b_0 +b_1x}}\]</span></p>
<p><span class="math display">\[p = \frac{e^{b_0 +b_1x}}{1 + e^{b_0 +b_1x}}\]</span></p>
</div>

Let's define one more term. The **odds** of an success outcome is the ratio of the chance of success to the chance of failure, odds $=p/(1-p)$.

<div class="mathbox">
<p>With a bit more algebra and rearranging terms, we can write the above model as a linear regression model.</p>
<p><span class="math display">\[p/(1-p) = \frac{e^{b_0 +b_1x}}{1 + e^{b_0 +b_1x}}/\frac{1}{1 + e^{b_0 +b_1x}}\]</span> <span class="math display">\[p/(1-p) = e^{b_0 +b_1x}\]</span> <span class="math display">\[\log(p/(1-p)) = b_0 +b_1x\]</span></p>
</div>

This is a **simple logistic regression model**. On the left hand side, we have the natural log of the odds, called the **logit** function. Think of this as a transformed version of our outcome. On the right hand side, we have a familiar linear equation. 

$$\log\left(\frac{p}{1-p}\right) = b_0 +b_1x$$

Like a linear regression model, we can extend this model to a **multiple logistic regression model** by adding additional $x$ variables,

$$\log\left(\frac{p}{1-p}\right) = b_0 +b_1x_1+b_2x_2+b_3x_3+\cdots +b_kx_k$$

## Fitting the Model

Based on observed data that includes an indicator variable for the responses (1 for success, 0 for failure) and predictor variables, we need to find the slope coefficients, $b_0$,...,$b_k$ that best fits the data. The way we do this is through a technique called **maximum likelihood estimation**. We will not discuss the details in this class; we'll save this for an upper level stats class such as Mathematical Statistics.

In R, we do this with the **g**eneral **l**inear **m**odel function, `glm()`.

For a data set, let's go back in history to January 28, 1986. On this day, the U.S. space shuttle called Challenger took off and tragically exploded about minute after the launch. After the fact, scientists ruled that the disaster was due to an o-ring seal failure. Let's look at experimental data on the o-rings prior to the fateful day.


```r
require(vcd)
data(SpaceShuttle)

SpaceShuttle %>%
  filter(!is.na(Fail)) %>%
  ggplot(aes(x = factor(Fail), y = Temperature) )+ 
  geom_boxplot() +
  theme_minimal()
```

<img src="04-logistic-regression_files/figure-html/unnamed-chunk-4-1.png" width="672" />

```r
SpaceShuttle %>%
  filter(!is.na(Fail)) %>%
  group_by(Temperature) %>%
  summarise(PercentFail = mean(Fail == 'yes')) %>%
  ggplot(aes(x = Temperature, y = PercentFail)) + 
  geom_point() +
  theme_minimal()
```

<img src="04-logistic-regression_files/figure-html/unnamed-chunk-4-2.png" width="672" />

<div class="reflect">
<p>What are the plots above telling us about the relationship between chance of o-ring failure and temperature?</p>
</div>


Let's fit a simple logistic regression model to predict the chance of o-ring failure (which is our "success" here -- we know it sounds morbid) based on the temperature using the experimental data. 


```r
model.glm <- glm(Fail ~ Temperature, data = SpaceShuttle, family = binomial)
summary(model.glm)
```

```
## 
## Call:
## glm(formula = Fail ~ Temperature, family = binomial, data = SpaceShuttle)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -1.0611  -0.7613  -0.3783   0.4524   2.2175  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)  
## (Intercept)  15.0429     7.3786   2.039   0.0415 *
## Temperature  -0.2322     0.1082  -2.145   0.0320 *
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 28.267  on 22  degrees of freedom
## Residual deviance: 20.315  on 21  degrees of freedom
##   (1 observation deleted due to missingness)
## AIC: 24.315
## 
## Number of Fisher Scoring iterations: 5
```

Based on a logistic regression model, the predicted log odds of an o-ring failure is given by

$$\log\left(\frac{\hat{p}}{1-\hat{p}}\right) = 15.0429 -0.2322\cdot Temperature$$


## Coefficient Interpretation

Let's take a look at the estimates from the model. What do they mean?


```r
summary(model.glm)
```

```
## 
## Call:
## glm(formula = Fail ~ Temperature, family = binomial, data = SpaceShuttle)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -1.0611  -0.7613  -0.3783   0.4524   2.2175  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)  
## (Intercept)  15.0429     7.3786   2.039   0.0415 *
## Temperature  -0.2322     0.1082  -2.145   0.0320 *
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 28.267  on 22  degrees of freedom
## Residual deviance: 20.315  on 21  degrees of freedom
##   (1 observation deleted due to missingness)
## AIC: 24.315
## 
## Number of Fisher Scoring iterations: 5
```

The slope coefficient (-0.232) tells you how much the predicted log odds increase with an increase of 1 degree increase in temperature. But what does a 1 unit increase in log odds mean? We need to do a bit of algebra. 

<div class="mathbox">
<p>Our estimated model is</p>
<p><span class="math display">\[\log\left(\frac{\hat{p}}{1-\hat{p}}\right) = b_0 +b_1x\]</span> where <span class="math inline">\(b_0 = 15.04\)</span> and <span class="math inline">\(b_1 = -0.232\)</span>.</p>
<p>If we imagine increasing <span class="math inline">\(x\)</span> by 1, then we get a different set of predicted chances of success, <span class="math inline">\(\hat{p}^*\)</span>,</p>
<p><span class="math display">\[\log\left(\frac{\hat{p}^*}{1-\hat{p}^*}\right) = b_0 +b_1(x+1)\]</span></p>
<p>Let’s find the difference between these two equations,</p>
<p><span class="math display">\[\log\left(\frac{\hat{p}^*}{1-\hat{p}^*}\right) - \log\left(\frac{\hat{p}}{1-\hat{p}}\right) = (b_0 +b_1(x+1)) - (b_0 +b_1x)\]</span> and simplify the right hand side (don’t you love it when things cancel!),</p>
<p><span class="math display">\[\log\left(\frac{\hat{p}^*}{1-\hat{p}^*}\right) - \log\left(\frac{\hat{p}}{1-\hat{p}}\right) = b_1\]</span> and then simplify the left hand side (using our rules of logarithms),</p>
<p><span class="math display">\[\log\left( \frac{\hat{p}^*/(1-\hat{p}^*)}{\hat{p}/(1-\hat{p})}\right) = b_1\]</span></p>
<p>Let’s exponentiate both sides,</p>
<p><span class="math display">\[\left( \frac{\hat{p}^*/(1-\hat{p}^*)}{\hat{p}/(1-\hat{p})}\right) = e^{b_1}\]</span></p>
</div>

We find that $e^{b_1} = e^{-0.232} = 0.793$ is the **odds ratio** based on increasing $x$ by 1 unit (it is a ratio of odds). The odds ratio is the ratio of the odds of success between two groups of units (those with temperature = T and those with temperature = T + 1 such as 28 v 29 degrees)

If a ratio is greater than 1, that means that the denominator is less than the numerator or equivalently the numerator is greater than denominator (odds of success are greater with T+1 as compared to T --> positive relationship between $x$ variable and odds of success). If a ratio is less than 1, that means that the denominator is greater than the numerator or equivalently the numerator is less than denominator (odds of success are less with T+1 as compared to T --> negative relationship between $x$ variable and odds of success). If the ratio is equal to one, the numerator equals the denominator (odds of success are equal for the two groups --> no relationship).


In this case, we have an odds ratio < 1 which means that the estimated odds of o-ring failure is lower for increased temperatures (in particular by increasing by 1 degree). This makes sense since we saw that the chance of o-ring failure decrease with warmer temperatures. 

## Prediction

On January 28, 1986, the temperature was 26 F degrees. Let's predict the chance of "success," which is a failure of o-rings in our data context, at that temperature.  


```r
predict(model.glm, newdata = data.frame(Temperature = 26), type = 'response') #type = 'response' gives chance, rather than odds
```

```
##         1 
## 0.9998774
```

They didn't have any experimental data testing o-rings at this low of temperatures, but even based on the data collected, we predict the chance of failure to be nearly 1 (near certainty). 

### Hard Predictions/Classifications

These predicted chances of "success" are useful to give us a sense of uncertainty in our prediction. If the chance of o-ring failure were 0.8, we would be fairly certain that a failure were likely but not absolutely. If the chance of o-ring failure were 0.01, we would be very certain that a failure were not likely to occur. 


But if we had to decide whether or not we should let the shuttle launch go, how high should the predicted chance be to delay the launch (even if it would cost a lot of money to delay)?

It depends. 

If we used a threshold of 0.8, then we'd say that for any experiment with a predicted chance of o-ring failure 0.8 or greater, we'll predict that there will be o-ring failure. As with any predictions, we may make an error. With this threshold, what is our **accuracy** (# of correctly predicted/# of data points)? 

In the table below, we see that there were three data points in which we correctly predicted o-ring failure (using a threshold of 0.8). There were 4 data points in which we erroneously predicted that it wouldn't fail when it actually did and correctly predicted no failure for 16 data points. So in total, our accuracy is (16+3)/(16+4+3) =  0.82 or 82%. 

The only errors we made were **false negatives**; we didn't predict o-ring failure but the o-rings did actual happen in the experiment. In this data context, false negatives have real consequences on human lives because a shuttle would launch and potentially explode because we had predicted there would be no o-ring failure. 

The **false negative rate** is the number of false negatives divided by the false negatives + true positives (denominator should be total number of experiments with actual o-ring failures). With a threshold of 0.80, our false negative rate is 4/(3+4) = 0.57 = 57%. We failed to predict 57% of the o-ring failures. This is fairly high when there are lives on the line.

A **false positive** (predicting failure when it doesn't happen) would delay launch but have minimal impact on human lives. 

The **false positive rate** is the number of false positives divided by the false positives + true negatives (denominator should be total number of experiments with no o-ring failures). With a threshold of 0.80, our false positive rate is 0/16 = 0. We always accurately predicted the o-rings would not fail.


```r
augment(model.glm, type.predict ='response') %>%
  mutate(predictOFail = .fitted >= 0.8) %>%
  count(Fail, predictOFail)
```

```
## Warning: `count_()` is deprecated as of dplyr 0.7.0.
## Please use `count()` instead.
## See vignette('programming') for more help
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_warnings()` to see where this warning was generated.
```

```
## # A tibble: 3 x 3
##   Fail  predictOFail     n
##   <fct> <lgl>        <int>
## 1 no    FALSE           16
## 2 yes   FALSE            4
## 3 yes   TRUE             3
```

What if we used a lower threshold to reduce the number of false negatives (those with very real human consequences)? Let's lower it to 0.25 so that we predict o-ring failure more easily. Let's find our accuracy: (10+4)/(10+3+6+4) = 0.61. Worse than before, but let's check false negative rate: 3/(3+4) = 0.43. That's lower. But now we have a non-zero false positive rate: 6/(6+10) = 0.375. So of the experiments with no o-ring failure, we predicted wrong 37.5% of the time. 


```r
augment(model.glm, type.predict ='response') %>%
  mutate(predictOFail = .fitted >= 0.25) %>%
  count(Fail, predictOFail)
```

```
## # A tibble: 4 x 3
##   Fail  predictOFail     n
##   <fct> <lgl>        <int>
## 1 no    FALSE           10
## 2 no    TRUE             6
## 3 yes   FALSE            3
## 4 yes   TRUE             4
```

Where the threshold goes depends on the real consequences of making those two types of errors. 

## Logistic Model Evaluation

In deciding whether a logistic regression model is a good and useful model, we need to consider the accuracy, the false positive rate, and the false negative rate. Depending on the context, we may focus on maximizing the overall accuracy or we may focus on minimizing the false negative rate or minimizing the false positive rate. 

There are other measures of model fit for a logistic regression such as AIC and BIC (lower is better), but those are beyond the scope of this course. You'll learn about them in future statistics courses. 

## Alternative Classification Models

Logistic regression is a very useful model to predict a binary outcome. However, it has its limitations. We are assuming a linear relationship between explanatory variables and the log odds of success. This is hard to check because we don't have a variable for odds that we could quickly plot. 

Other methods out there are more flexible but also more complex. Here is a list of some of the most popular classification methods. 

- Classification Trees can predict a binary outcome and choose the variables that are most important by recursively partitioning the data into groups that are more similar in terms of the outcome as well as in chosen predictor variables.

- Random Forests are an ensemble of classification tress that together are more stable than any one classification tree.

- Boosted Trees are classification trees that are sequentially created to target the errors from the last tree.

- Neural Networks are a type of classification algorithm that creates new features based on the original data that are the best predictors of the outcome. 

Take Statistical Machine Learning to learn more about these methods. But keep in mind that sometimes for a task, complex is not necessary: see https://www.huffingtonpost.com/2014/02/10/klemens-torggler-evolution-door_n_4762261.html.
