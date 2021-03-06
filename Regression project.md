---
title: "Peer Review Assigment"
output: pdf_document
---
========================================================

## Automatic or Manual: Transmission type impact on car fuel consumption

### Executive Summary

The goal of this analysis is to explore the effect of transmission type, automatic and manual, on a car fuel consumption. 
Using a dataset from the 1974 Motor Trend US magazine, this analysis will provide answers to following questions: 

- Is an automatic or manual transmission achieving better miles per gallon (MPG)? 
- How different is the MPG between automatic and manual transmissions?

Using hypothesis testing and simple linear regression, we were able to determine that there is a signficant difference between the mean MPG for automatic and manual transmission cars, with the latter achieving 7.24 higher MPGs on average. 

In order to adjust for other confounding variables such as the weight and horsepower of the car, we ran a multivariate regression to get a better estimate the impact of transmission type on MPG. Other variables such as weight of the vehicle, horsepower and number of cilinders turned out to be more significant than transmission type in affecting MPG values.
After validating the model using ANOVA, the results from the multivariate regression reveal that, on average, manual transmission cars get 2.08 miles per gallon more than automatic transmission cars.


### Data Processing

Loading the data set and converting some of the variables names to make them more descriptive:

```r
data(mtcars)
```

Converting the predictor variable to factor to make the analysis easier.


```r
mtcars$am <- as.factor(mtcars$am)
levels(mtcars$am) <- c("automatic", "manual")
```

### Exploratory Data Analysis

We will now check the distribution of the dependent variable `mpg` to make sure the assumptions for running a linear regression are met.

#### Testing for normality
The test result depends on p-value. When p < 0.05, then population is likely not normaly distributed.


```r
shapiro.test(mtcars$mpg)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  mtcars$mpg
## W = 0.9476, p-value = 0.1229
```
We also conducted a Histogram with Normal Curve and the Kernel Density Plot tests (Appendix Figure 1) which show that the distribution of `mpg` is approximately normal and there are no apparent outliers skewing the distribution.


#### Comparing the means

We now look if there is a difference, in fuel consumption for the two types of transmision. A p < 0.05 indicates that means are likely to be different.


```r
t.test(mtcars$mpg ~ mtcars$am)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  mtcars$mpg by mtcars$am
## t = -3.7671, df = 18.332, p-value = 0.001374
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -11.280194  -3.209684
## sample estimates:
## mean in group automatic    mean in group manual 
##                17.14737                24.39231
```

The low p-value (less than 0.05) indicates that there is a difference between these two groups. Figure 2 in the Appendix shows a comparison of the means.

### Model Selection

#### Correlation analysis
In order to select the best model, we need to find the variables that have the biggest impact on fuel consumption. We create a correlation matrix for the mtcars dataset and look at the `mpg` row.


```r
data(mtcars)
sort(cor(mtcars)[1,])
```

```
##         wt        cyl       disp         hp       carb       qsec 
## -0.8676594 -0.8521620 -0.8475514 -0.7761684 -0.5509251  0.4186840 
##       gear         am         vs       drat        mpg 
##  0.4802848  0.5998324  0.6640389  0.6811719  1.0000000
```
In addition to `am`, which has to be included in our regression model, we observe that `wt, cyl, disp`, and `hp` are highly correlated with our dependent variable mpg. Thus, they may be good candidates to include in our model. However, if we look at the correlation matrix we also see that `cyl` and `disp` are highly correlated with each other. Since predictors should not exhibit collinearity, we would remove `cyl` and `disp` from our model.

#### Regression analysis

We begin our model testing fitting a simple linear regression for `mpg` on `am`.


```r
fit <- lm(mpg ~ am, data = mtcars)
summary(fit)
```

```
## 
## Call:
## lm(formula = mpg ~ am, data = mtcars)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -9.3923 -3.0923 -0.2974  3.2439  9.5077 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   17.147      1.125  15.247 1.13e-15 ***
## am             7.245      1.764   4.106 0.000285 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.902 on 30 degrees of freedom
## Multiple R-squared:  0.3598,	Adjusted R-squared:  0.3385 
## F-statistic: 16.86 on 1 and 30 DF,  p-value: 0.000285
```

This analysis doesn't provide additional information for our hypothesis test. Interpreting the coefficient and intercepts, we say that, on average, automatic cars have 17.147 MPG and manual transmission cars have 7.24 MPGs more. In addition, we see that the R^2 value is 0.3598. This means that our model only explains 35.98% of the variance.  


Next, we fit a multivariate linear regression for mpg on `am, wt`, and `hp`. We run an ANOVA to compare the two models and see if they are significantly different.


```r
bestfit <- lm(mpg ~ am + wt + hp, data = mtcars)
anova(fit, bestfit)
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ am
## Model 2: mpg ~ am + wt + hp
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     30 720.90                                  
## 2     28 180.29  2    540.61 41.979 3.745e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

With a p-value of 3.745e-09, we reject the null hypothesis and claim that our multivariate model is significantly different from our simple model.


Finally, we check the residuals for any signs of non-normality and examine the residuals vs. fitted values plot (Appendix Figure 3) to check for signs of heteroskedasticity. Figure 3 in the Appendix shows that the residuals are normally distributed and homoskedastic.

### Estimates from final model


```r
summary(bestfit)
```

```
## 
## Call:
## lm(formula = mpg ~ am + wt + hp, data = mtcars)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.4221 -1.7924 -0.3788  1.2249  5.5317 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 34.002875   2.642659  12.867 2.82e-13 ***
## am           2.083710   1.376420   1.514 0.141268    
## wt          -2.878575   0.904971  -3.181 0.003574 ** 
## hp          -0.037479   0.009605  -3.902 0.000546 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.538 on 28 degrees of freedom
## Multiple R-squared:  0.8399,	Adjusted R-squared:  0.8227 
## F-statistic: 48.96 on 3 and 28 DF,  p-value: 2.908e-11
```
This model explains over 83.99% of the variance. Furthermore, we observe that `wt` and `hp` did confound the relationship between `am` and `mpg`. Reading the coefficient for `am`, we can see that, on average, manual transmission cars achieve 2.08 MPGs more than automatic transmission cars.


### Appendix

#### Figure 1. Histogram with Normal Curve and Kernel Density Plot

```r
par(mfrow = c(1, 2))
x <- mtcars$mpg
h<-hist(x, breaks = 10, col = "green", xlab = "MPG",
   main="Histogram of Miles per Gallon")
xfit <- seq(min(x), max(x), length = 40)
yfit <- dnorm(xfit, mean = mean(x), sd = sd(x))
yfit <- yfit * diff(h$mids[1:2])*length(x)
lines(xfit, yfit, col="blue", lwd = 2)

d <- density(mtcars$mpg)
plot(d, xlab = "MPG", main ="Density Plot of MPG")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

#### Figure 2. Boxplot of MPG by transmission type


```r
boxplot(mpg ~ gear, data = mtcars, xlab = "Transmission type", ylab = "Miles per gallon")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

#### Figure 3. Residuals plots


```r
par(mfrow = c(2,2))
plot(bestfit)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 
