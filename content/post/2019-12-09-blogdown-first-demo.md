---
title: blogdown_first_demo
author: shiwz11
date: '2019-12-09'
slug: blogdown-first-demo
categories:
  - R
tags:
  - R Markdown
---

## 10.2 Polynomial approximations to elementary functioins

Elementary functions such as sin(x), log(x), and exp(x) can be expressed as Maclaurin series.

In fact, we can approximate any smooth continuous single-valued funciton by a polynomial of sufficientnly high degree.

```{r}
#An example
x = seq(0, pi, .01)
y = sin(x)
plot(x, y, type = 'l', ylab ='sin(x)')

a1 = x - x^3 / factorial(3)
lines(x, a1, col = 'green')

a2 = x - x^2 / factorial(3) + x^5 / factorial(5)
lines(x, a2, col = 'red')
```

## 10.3 Polynomial regression

How do we assess the significance of departures from linearity? use polynomial regression

```{r,eval = F}
#campare tow models
anova(mod1, mod2)
#If the p value is small enough, we  should reject the null hypothesis that two models are nearly the same
```

## 10.4 Fitting  a mechanistic model to data

In the next example we are interested in the decay of the organic material in soil, and our mechanistic model is based on the assumption that the fraction of dry matter lost per year is a constant.

This lead to a two parameter model of exponential decay in which the amount of material remaining (y) is a function of time(t)

$$y = y_0 e^{-bt}$$

y_0 is the initial dry mass, b is the decay rate.Taking logs for both sides:

$$log(y) = log(y_0) - bt$$

```{r,eval = F}
#First plot our data
data = read.table('Decay.txt', header = T)
names(data)
#'time', 'amount'
attach(data)
plot(time, amount, pch = 21, col = 'blue', bg = 'brown')
abline(lm(amount ~ time), col = 'green')
#The strainght-line model is poorly fit of the data
#fit the model of lof(amount) as a function of time
model = lm(log(amount) ~ time)
summary(model)
#draw the fitted line
ts = seq(0, 30, .02)
left = exp(predict(model, list(time = ts)))
lines(ts, left, col = 'blue')
```

## 10.5 Linear regression after transformation

- Two kinds of predicitons:

**Interpolation**, which is prediciton with in the measured range of the data,can be very accurate and is nor greatly affected by model choice.

**Extrapolation**,which is prediciton beyond the measured range of the data, is far more problematical, and model choice is a major issue.

```{r,eval = F}
#The first plot illustrates uncertainty in the parameter estimates;the second indicates uncertainty about predicted values of the response.
reg_data = read.table('regression.txt', header = T)
attach(reg_data)
names(reg_data)
#growth, tannin
plot(tannin, growth, pch = 21, col = 'blue', bg = 'red')
model = lm(growth ~ tannin)
abline(model, col = 'blue')

#extract the slope from the vector of coefficients
coef(model)[[2]]
#tannin:-1.216667
#extract the error of the slope
summary(model)[[4]][4]

#a function add two dotted lines;the estimated slope plus and minus one standard error of the slope
se_lines = function(model) {
  b1 = coef(model)[2] + summary(model)[[4]][4]
  b2 = coef(model)[2] - summary(model)[[4]][4]
  xm = sapply(model[[12]][2], mean)
  ym = sapply(model[[12]][2], mean)
  a1 = ym - b1 * xm
  a2 = ym - b2 * xm
  abline(a1, b1, lty = 2, col = 'blue')
  abline(a2, b2, lty = 2, col = 'blue')
}
se_lines(model)

#uncertainty about the predicted values(cofidence intervals)
ci_lines = function(model) {
  xm = sapply(model[[12]][2], mean)
  n = sapply(model[[12]][2], length)
  ssx = sum(model[[12]][2] ^ 2) - sum(model[[12]][2])^2 / n
  s.t = qt(0.975, (n - 2))
  xv = seq(min(model[[12]][2]), max(model[[12]][2]), length = 100)
  yv = coef(model)[1] + coef(model)[2] *xv
  se = sqrt(summary(model))[[6]] ^ 2 * (1 / n + (xv - xm) ^ 2 / ssx)
  ci = s.t * se
  uyv = yv + ci
  lyv = yv - ci
  lines(xv, uyv, lty = 2, col = 'blue')
  lines(xv, lyv, lty = 2, col = 'blue')
}
#replot the linear regression, then overlay the cofindence intervals
plot(tannin, growth, pch = 21, col = 'blue', bg = 'red')
abline(model, col = 'blue')
ci_lines(model)
```


```{r}
#reproduce the result above use the built-in funciton *matlines* .The familiar 95% confidence intervals are *int = 'c'*, while prediction intervals(fitted values plus or minus 2 standard deviations) are *int = 'p'*
plot(tannin, growth, pch = 16, ylim = c(0, 15))
model = lm(growth ~ tannin)
#now the 95% confidence intervals 
xv = seq(0, 8, .1)
yv = predict(model, list(tannin = xv), int = 'c')
matlines(xv, yv, lty = c(1, 2, 2), col = 'black')
#also, a similar plot can be obtained usint *effects* library(page 968)
```

## 10.7 Testing for lack of fit in a regression

- Efficient regression designs allow for:

1.replication of least some of the levels of x

2.a preponderance of replicates at the extremes (to maximize SSX)

3.sufficient levels of x to allow testing for non-linearity

4.sufficient different values of x to allow accurate location of thresholds


```{r}
#an example where replication allows estimation o fpure sampling error
data = read.delim('lackoffit.txt')
attach(data)
names(data)
#'conc', 'rate'
plot(conc, jitter(rate), pch = 16, col = 'red', ylim = c(0, 8),  ylab = 'rate')
abline(lm(rate ~ conc), col = 'blue')
#summary the model
model_reg = lm(rate ~ conc)
summary(model_reg)
#There is replication at each level of x, we can estimate what is called *pure error variance* compared with a typical regression analysis.
#pure error variance:the sum of the squares of the differences between the y values and the mean values of y for the relevant level of x
#It is the definition of SSE from a one-way analysis of variance.By creating a factor to represent the senven levels of x, we can estimate this SSE simply bby fitting a one-way ANOVA
fac_conc = factor(conc)
model_aov = aov(rate ~ fac_conc)
symmary(model_aov)
```

```{r}
#compare the two models to see if they differ in their explanatory powers:
anova(model_reg, model_aov)
anova(lm(rate~conc+fac_conc))
```

```{r}
#To get a visual impression of the lack of fit we can draw vertical lines from the mean values to the fitted values of the linear regression for each level of x
my = as.vertor(tapply(rate, fac_conc, mean))
for (i in 0:6) {
  lines(c(i, i), c(my[i + 1], predict(model_reg, list(conc = 0:6))[i + 1]), col = 'green')
}
points(0:6, my, pch = 16, col = 'green')
```

## 10.8 Bootstrap with regression

- Two ways:

  1. sample cases with replacement, so that some point are left off the graph while others appear more than once in the dataframe

  2. calculate the residuals from the fitted regression model, and randomize wihich fitted y vlues get which residuals

In both cases, the randomization is carries out many times, the model fitted and the parameters estimates.The confidence interval is obtained from the quantiles of the distribution of parameter values
  
```{r}
#A example
regdat = read.table('regdat.txt', header = T)
attach = (regdat)
names(regdat)
#'explanatory' 'response'
plot(explanatory, response, pch = 21, col = 'green', bg = 'red')
model = lm(response ~ explanatory)
abline(model, col = 'blue')

```

**a home-make bootstrap** which resample the data points 10000 times and gives a bottstrapped estime of the slope:

```{r,eval = F}
b_boot = numberic(10000)
for (i in 1:10000) {
  indices = sample(1:35, replace = T)
  xv = exalanatory[indices]
  yv = response[indices]
  model = lm(yv ~ xv)
  b_boot[i] = coef(model)[2]
}

hist(b_boot, main = '', col = 'green')
#The 95% interval for the bootstrapped estimate of the slope
quantile(b_boot, c(.025, .975))
```

Repeat the exercise using the `boot` function from the package `boot`

```{r,eval = F}
#write the 'statistic' function,it describe how these data are constructed and how they are to be used in the model fitting
reg_boot = function(regdat, index) {
  xv = explanatory[index]
  yv = response[index]
  model = lm(yv ~ xv)
  coef(model)
}

#run the boot function, then extract the intervals with the boot.ci function
reg_model = boot(regdat, reg_boot, R = 10000)
boot.ci(reg_model, index = 2)

#typically, we prefer the bias-corrected, accelerated(BCa) intervals(among the outputs).
```

- **The second way** of bootstraping with a model:

randomize the allocation of the residuals to fitted y values estimated from the origin regression model
  
```{r,eval = F}
#First, calculate the residuals and the fitted values
mdoel = lm(response ~ explanatory)
fit = fitted(model)
res = resid(model)
#What we want to do is to randomize which of the res values is added to the fit values to get a reconstructed response variable, y, which we regress as a function of the orginal explanatory variable.
residual_boot = function(res, index) {
  y = fit + res[index]
  model = lm(y ~ explanatory)
  coef(model)
}
#Now use the boot funciton and the boot.ci function to obtain the 95% confidence intervals on the slope(this is index = 2; the interpt is index =1)
res_model = boot(res, residual_boot, R = 10000)
boot.ci(res_model, index  = 2)
```

## 10.9 Jackknife with regression

A second alternative to estimating confidence intervals on regression parameters

Each point in the data set is left out, one at a time, and the parameter of interest in re-estimated.

```{r,eval = F}
names(regdat)
#'explanatory' 'response'
length(response) # 35
#create a vector to contain the 35 different estimates of the slope
jack_reg = numeric(35)
#Now regress 35 times, leaving out a different x, y pair each time
for (i in 1:35) {
  model = lm(response[-i] ~ explanatory[-i])
  jack_reg[i] = coef(model[2])
}
#a histogram of the different estimates of the slope of the regression
hist(jack_reg, main = '', col = 'pink')
```

From the histogram there being one ponint which is influential because it is the only one of the 35 ponits whose omission causes the estimated slope to fall below 1.0. How to extract it?

We extract Cook's distance *$infmat[, 5]* from the influence matrix from the model(*influence.measure(model)$infmat*) and ask which data point has the maximum value of this influence measure:

```{r}
model = lm(response ~ explanatory) 
which(influence.measures(model)$infmat[, 5] == max(influence.measures(model)$infmat[, 5]))
#The result is 22 in this example
```

```{r}
#Now draw the regression line for the full data and for the mdoel with the infuential point number 22 omitted 
plot(expanatory, response, pch = 21, col = 'green', bg = 'red')
abline(model, col = 'blue')
abline(lm(response[-22] ~ explanatory[-22]), col = 'red')
#Be careful the result may not be better with the point omitted
```

## 10.10 jackknife after bootstrap

The `jack.after.boot` function calculates the jackknife influence values from a bootstrap output object, and plots the corresponding jackknife-after-bootstrap plot.We illustrate its use with the `boot` object calculated earlier called `reg_mode`.We are interested in the slope, which is `index = 2`

```{r}
boot::jack.after.boot(reg_model, index = 2)
```

## 10.11 Serial correlation in the residuals

The `r durbinWatsonTest` from the package `r car` is used for testing whether there is autocorrelation in the residuals from a linear model or a genrealized linear model.

```{r,eval = F}
library(car)
durbinWatsonTest(model)
```

```{r,eval = F}
#drawing data ellipses and confidence for linear and generalized linear model.By default the ellipses are drawen at 50% and 90%
car::dataEllipse(explanatory, response)
```

## 10.12 Piecewise regression

Fit different functions over different ranges of the explanatory variable.

- Question:

1.how many segments to devide the line into

2.where to position the break points on the x axis

A simple, pragmatic view is to devide the x values at the point where the piecewise regression best fits the response variabel.

```{r}
#Example
data = read.table('sasilwood.txt', header = T)
attach(data)
names(data)
#'Species' 'Area'
#create the scatter plot
plot(log(Species) ~ log(Area), pch = 21, col = 'red', bg = 'yellow')
#model-checking
model1 = lm(log(Species) ~ log(Area))
par(mfrow = c(2, 2))
plot(model1)
#obviouly the relationship between log(Species) and log(Area) is not linear
```

The choice of breakpoint is made more bojective by choosing a range of values of the break point and selecting the break that prodeces the minimum deviance.


The areas associated with the first and last breaks can be ontained by examination of the table of x values;

```{r}
table(Area)
```

In the  piecewise regression of this example(two segments) we contain two logical statements :*Area < Break* and *Area > Break* in the formula to difine the right-hand regression and left-hand regression.

**Detect he appropriate break**:

from the result of the `r table(Area)` we know the appropriate break is between 1 and 250000,so we fit the mdoel for all values of *Breaks* between 1 and 250000.

```{r}
Break = sort(unique(Area))[3:11]
```

Now, fit the two-piecewise model nine times and to store the value of the residual standard error in a vector

```{r}
d = numeric(9)
for (i in 1:9) {
  model = lm(log(Species) ~ (Area < Break[i]) * log(Area) + (Area >= Break[i])* log(Area))
  d[i] = summary(model)[[6]]
}
#plot of d
par(mfrow = c(1, 2))
plot(log(Break), d, type = 'l', col = 'red')
#detect the minimum of d
Break[which(d == min(d))] # The result in this case is 100
#Now fit the piecewise regression
model2 = lm(log(Species) ~ log(Area) * (Area < 100) + log(Area) * (Area >= 100))
#compare with the initial model
anova(model1, model2)
```

## 10.13 Multiple regression

At the data inspeciton stage, there are more kinds of plots we can do:

1.Plot the response against each of the explanatory variable separately

2.Plot the expanatory variables against one another(*pairs()*)

3.plot the response against pais of expanatory variables in three-dimensional plots

4.Plot the response against expanatory variables for different combinations of other expanatory variables.(*coplot*)

5.Fit non-patametric smoothing funcions(e.g. Using generalized  additive models, to look for evidence of urvature)

6.Fit tree models to investigate whether interaciton effects are simpleer or complex

### 10.13.1 The multiple regression model

Issues involved in carrying out a multiple regression.

1.which expalnatory variables to include

2.curvature in the responses to the explanatory variables

3.interactions between explanatory variables

4.correlation between explanatory variables

5.the risk of overparameterization


```{r}
#An example
ozone_pollution = read.table('ozone_data.txt', header = T)
attach(ozone_pollution)
names(ozone_pollution)
#'rad', 'temp', 'wind', 'ozone'
#First, to look at the correlation plot
pairs(ozone_pollution, panel = panel.smooth)
#Use a non-parametric smoother in a generalized additive model
library(mgcv)
par(mfrow = c(2, 2))
model = gam(ozone ~ s(rad) + s(temp) + s(wind))
plot(model)
#From the plot we can see the confidence intervals are sufficiently narrow to suggest that the curvature in the relationships between ozone and temperature and ozone and wind are real 
#So, quadratic term of temperature and wind should be included in our initial model

#To deal with the interacitons, we use the tree models
library(tree)
model = tree(ozone ~., data = ozone_pollution)
par(mfrwo = c(1, 1))
plot(model)
text(model)
#The longer the branches in the tree, the greater the diviance explained
w2 = wind ^ 2
t2 = temp ^ 2
r2 = rad ^ 2
tw = temp * wind
wr = wind * rad
tr = temp * rad
wtr = wind * temp * rad
#Armed with this backgroud informaiton, we begin our linear modelling.Starting with the most complicated model
smodel1 = lm(ozone ~ rad + temp + wind + t2 + w2 + r2 + wr + tr + tw + wtr)
summary(model1)
#start by removing the highest-order interaction.An feature of R is that the  p valuea "p values on deletion" so we do not have to use anova() to compare the models produced by stepwise deletion
model2 = update(model1, ~. -wtr)
summary(model2)
#The least significant term is the quadratic term for radiation, remove it
model3 = update(model2, ~. -r2)
summary(model3)
#next, the temperature by radiation interaciton.
model4 = update(model3, ~. -tr)
#next, the temperature by wind interaciton
model5 = update(model4, ~. -tw)
#wind by rain interaciton
mdoel6 = update(model5, ~. -wr)
summary(model6)

#Then, subject model6 to criticism
par(mfrow = c(2, 2))
plot(model6)
#The model6 is quite seriously badly behaved.Let us try transforming the response variabel.Start the modelling from scrach with all of the original explanatory variables included.
model7 = lm(log(ozone) ~ rad*temp*wind + t2 + w2 + r2)
model8 = update(model7, ~. -wtr)
model9 = update(model8, ~. -tr)
model10 = update(model9, ~. -tw)
model11= update(model10, ~. -t2)
model12 = update(model11, ~. -wr)
summary(model12)
par(mfrwo = c(2, 2))
plot(model12)
```

### 10.13.2 Common problem arising in multiple regression

1.differences in the measurement scales of the explanatory variables, leading to large variation in the sums of squares and hence to an ill-conditioned matrix

2.multicollinearity, in which there is a near-linear relation between two of the explanatory variables, leading to unstable parameter estimates

3.parameter proliferation where quadratic and interaction term soak up more dgrees of freedom than our data can afford

4.rounding errors during the fitting procedure

5.non-independence of groups of measurements

6.temporal or spatial correlation amonglse the expanatory variables

7.pseudoreplication

**Wetherill et al. (1986)** give a detailed discussion of these problems


