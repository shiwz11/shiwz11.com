---
title: NOTES_The_R_Book_Part10
author: shiwz11
date: '2019-12-12'
slug: notes-the-r-book-part10
categories:
  - R
tags:
  - R Markdown
  - regression
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-12T10:17:46+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

# Chapter 13 Generalized Linear Models

- Consider using GLMs(glims) when the variable is :

1.count data expressed as proportions(e.g. logistic regressions)

2.count data that are not proportions(e.g. log-linear models of counts)

3.binary response variables(e.g. dead or alive)

4.data on time to death where the variance increase faster than linearly with the mean(e.g. time data with gamma errors)

- A GLM has three important properties:

1.the error structure

2.the linear predictor

3.the link function

## 13.1 Error structure

- Example

1.errors that are strongly skewed

2.errors that are kurtotic

3.errors that are strictly bounded(as in proportions)

4.errors that connot lead to negative fitted values(as in count)


- A GLM allows the specification of a variety of different error distributions:

1.Poisson errors, useful with count data

2.binomial errors, useful with data on proportions

3.gamma errors, useful with data showing a constant coefficient of variation

4.exponential error, useful with data on time to death(survival analysis)

The *error structure* is defined by means of the ***family*** directive.

- example;

```{r}
#the response variable y has Possion errors
glm(y ~ z, family = poisson)
#The response variable is binary, and the model has binomial errors
glm(y ~ z, family = binomial)
```

## 13.2 Linear predictor

The linear predictor $\eta$, is a linear sum of the effects of one or more explanatory variables, $x_j$

$$
\begin{aligned}
\eta_i = \sum_{j = 1}^{p}{x_{ij}{\beta_j}}
\end{aligned}
$$

where the $xs$ are the values of the $p$ diferent explanatory variables, and the $\beta s$ are the(usually) unknown parameters to be estimated from the data. The right-hand side of the equation is called the ***lienar structure*** 






```{r}
model2 = aov(Glycongon ~ Treatment + Error(Treatment/Rat/Liver))
summary(model2)
```

```{r}
library(tidyverse)
```

## 13.3 Link function

The `Link function` relates the mean value of y to its linear predictor, In symbols, this means that

$$\eta = g(\mu)$$
### 13.3.1 Canonical link functions

## 13.4 Proportion data and bibomal errors

- Proportion data have three important properties that affect the way the data should be analysed:

1.The data are strictly bounded

2.the variance is non-constant

3.errors are non-normal

for proportion data, therefore, the variance increases with the mean up to a maximum(when the probability of sucess is 0.5) then declines again towards zero as the mean approaches 1.The variance-mean relationship is humped, rather than constant as assumed in the classical tests

we use a generalized linear model and sepcify that the error family is binomial like this

```{r}
glm(y ~ x, family = binomial)
```

## 13.5 Count data and Posson errors

- count data have a number of properties that need to be considered during modelling:

1.Count data are bounded below(cannot have counts less than zero)

2.Variance is not constant(Varianve increases with the mean)

3.Errors are not normally distributed

4.The fact that the data are whole numbers(integers) affects the error distribution

Deal with thest issues by using a `GLM`

```{r}
glm(y ~ x, poisson)
#The moDel is fitted with a log link(to ensure that the fitted values are bounded below) and Poisson errors (to accont for the non-normality)
```

If, having fitted the minimal adequate model, we discover that the residual deviance is greater than the residual degrees of freedom, then we have contravened an important assumption of the model.This is called overdispersion, and we can correct for it by specifying ***quasipoisson*** errors like this:

```{r}
glm(y ~ x, quasipoisson)
```

## 13.6 Deviance:Measuring the goodness of fit of a GLM

The measure of discrepancy in a GLM to assess the goodness of fit of the data is called the *deviance*.
Deviance is defined as -2 times the difference in log-likelihood between the current model and a saturated model.
Minimizing the deviance is the same as maximizing the likelikood.

## 13.7 Quasi-likelihood

Some cases, we may not be uneasy about specifying the precise form of the error distribution.
One very simple and robust alternative known as quasi-likelihood, introduved by <font color=red>Wedderburn(1974)</font>, which uses only the most elementary information about the response variable, namely the variance-mean relationship(see *Taylor's power law, p.262*). 

Quasi-likelihood frees us from the need to specify a particular distribution, and requires us only to specify the mean-variance relationship up to a proportionality constant, which can be estimated from the data:

$$
\operatorname{var}\left(y_{i}\right) \propto v\left(\mu_{i}\right)
$$

An example of the principle at work compares quasi-likelihood with maximum likelihood in the case of Poisson errors(***full details are in McCulloch and Searle, 2001***).

mximum quasi-likelihood (*MQL*) equations for $\beta$ are

$$
\frac{\partial}{\partial \beta} \sum\left(y_{i} \log \mu_{i}-\mu_{i}\right)=0
$$

MQL esitmates are generally robust and efficient.

take the original GLM density function and find the derivative of the log-likelihood with respect  to the mean:

$$
\frac{\partial l_{i}}{\partial \mu_{i}}=\frac{\partial l_{i}}{\partial \theta_{i}} \frac{\partial \theta_{i}}{\partial \mu_{i}}=\frac{y_{i} b^{\prime}\left(\theta_{i}\right)}{a_{i}(\phi)} \frac{1}{b^{\prime \prime}\left(\theta_{i}\right)}=\frac{y_{i}-\mu_{i}}{\operatorname{var}\left(y_{i}\right)}
$$
The quasi-likelihood Q is defined as:

$$
\begin{aligned}
Q(y, \mu) &= \int_{y}^{\mu} \frac{y-\mu}{\phi V(\mu)} \mathrm{d} \mu \\
var(y) &= \phi V(\mu)  \text{:The variance of y}
\end{aligned}
$$

The scale parameter:$\phi$ is estimated from the generalized Pearson statistic rather than from the residual deviance (as when correcting for over dispersion with Poisson or binomial errors):

$$
\begin{aligned}
\hat{\phi} = \frac{\sum_i \left\{(y_i - \hat{\mu_i}) / V_i(\hat{\mu_i}) \right\}}{n - p} = \frac{\chi ^ 2}{n  - p}
\end{aligned}
$$

For normally distributed data, the residual sum of squares SSE is chi-squared distributed

## 13.8 The quasi famuly of models

```{r}
#example
data = read.table('timber.txt', header = T)
attach(data)\
head(data)
#model1 uses the cube root link, model2 = uses the log link
model1 = glm(volume ~ girth + height, family = quasi(link = power(0.3333)))
model2 = glm(volume ~ girth + height, family = quasi(link = log))
#we can compare them use the anova()
anova(model1, model2)
#anova just tell us the difference of the deviance between the two models
#assess the constancy-of-variance and normality-of-errors assumptions of the two models
plot(model1)
plot(model2)
#conclusion:model1 behaved much better in terms of its assumptions as well as having the lower of deviance
```

## 13.9 Generalized additive models(GAMs)

If we had a three-variable multiple regression on count data and we want to smooth all three explanatory variables, we woul write:

```{r}
model = gam(y ~ s(w) + s(x) + s(z), poisson)
```

These are hierarchical models, so the inclusion of a high-order interaction (such as A:B:C) necessarily implies the inclusion of all the lower-order terms marginal to it(i.e. A:B, A:C and B:C, along with main effects for A , B, C)

## 13.10 Offsets

For GLMs, it is necessary to specify the offset;this is held constant while other explanatory variables are evaluated.
Example of the volume:

$$
v = \frac{g^2}{4\pi} h
$$

Taking logarithms gives:

$$
log(v) = log(\frac{1}{4\pi}) + 2log(g) + log(h)
$$

```{r}
#if we do a mulitple linear regression of log(v) on log(h) and log(g) we would get estimated slopes of 1.0 for log(h) and 2.0 for log(g)
data = read.delim('timber.txt')
attach(data)
names(data)
#volumn, girth, height
girth = girth / 100
model1 = glm(log(volumn) ~ log(girth) + log(height))
summary(model1)
#Now we shall use offset to specfy the theoretical response of log(v) log(h), i.e. a slope of 1.0 rather than the estimated 1.11714
model2 = glm(log(volumn) ~ log(girth) + offset(log(height)))
summary(model2)
#Let us try including the theoretical slope(2.0) for log(g) in the offset as well
model3 = glm(log(volumn) ~ 1 + offset(log(height) + 2 * log(girth)))
summary(model3)
#The AIC is smaler step by step
#put the entire model into the offset and informing GLM not to estimate an intercept from the data(y ~ -1):
model4 = glm(log(volumn) ~ offset(log(1/(4*pi)) + log(height) + 2 * log(girth)) - 1)
summary(model4)
#The AIC is smaller
```

## 13.11 Residuals

For Possion errors, the standardized residuals are:

$$
\frac{y - fitted \ values}{\sqrt{fitted \ values}} 
$$
For binomial errors:

$$
\frac{y - fitted \ values}{\sqrt{fitted \ values \times \left[1 - \frac{\text{fitted values}}{\text{binomial denominator}}\right]}} 

$$

binomial denominator:the size of the sample from which the y successes were drawen.

gamma errors:

$$
\frac{y - \text{fitted values}}{\text{fitted values}}
$$


In general, we can use several kinds of standardized residuals:

$$
\begin{aligned}
\text{standarized residuals} = (y - \text{fitted values})\sqrt{\frac{\text{prior weight}}{\text{scale parameter} \times \text{variance function}}}
\end{aligned}
$$

**prior weight**:optionally specified to give individual data points more or less influedce.

**scale parameter**:the degree of overidpersion

**variance function**: the relationship between the variance and the mean

### 13.11.1 Misspecified error structure

R has no direct facility for specifying negatice binomial errors, we can use quasi-likelihood to specify the variance function in a GLM with *family = quasi(see page 564)*

### 13.11.2 Misspecified link function

## 13.12 Overdispersion

- Two general techniques avaliable to us:

1.use F tests with an empirical scale parameter instead of chi-squared

2.use quasi-likelihood to specify a more appropriate variance function

Overdispersion meas that something we did not measure turned out to have an important impact on the results. If we did not measure this factor, then we have no confidence that our randomization process took care of it properly and we may have introduced an important bias into the results.

## 13.13 Bootstrapping a GLM

- Two ways of using bootstrapping

1.Fit the model lots of times by selecting cases for inclusion at random with replacement, so that some data point a excluded and others apper more than once in a particular model fit.

2.Fit the model once and calculate the residuals and the fitted values, then shuffle the residuals lots of times and add them to the fitte values in different permutations, fitting the model to the many different data sets.

```{r}
library(boot)
model_boot <- function (data, indices) {
  sub_data <- data[indices, ]
  model <- glm(log(volume) ~ log(girth) + log(height), data = sub_data)
  coef(model)
}
#run the bootstrap for 2000 resamplings using the boot function
glim_boot <- boot(tree, model_boot, R = 2000)
glim_boot
```

- Another way:

raw residuals:`y - fitted(model)`

The new `y` value:`fitted(model) + sample(y - fitted(model))`

```{r}
#A home-made version:
model <- glm(log(volume) ~ log(girth) + log(height))
yhat = fitted(model)
residuals = log(volume) - yhat
coefs = numeric(6000)
coefs = matrix(coefs, nrow = 2000)
#shuffle the residuals 2000 times to get different vectors of y values
for (i in 1:2000) {
  y = yhat + sample(residuals)
  boot_model = glm(y ~ log(girth) + log(height))
  coefs[i, ] = coef(boot_model)
}
#Extracting the means and standard deviations of the coefs
apply(coefs, 2, mean)
apply(coefs, 2, sd)

model = glm(log(volume) ~ log(girth) + log(height))
yhat = fitted(model)
resid = resid(model)

res_data = data.frame(resid, girth, height)
#write the statistic funciton to do the work within boot
bf = function(res_data, i) {
  y = yhat + res_data[i, 1]
  nd = data.frame(y, girth, height)
  model = glm(y ~ log(girth) + log(height), data = nd)
  coef(model)
}
#in order to shuffle the residuals rather than sample them with replacement, we specify *sim = 'permutation'*
boot(res_data, bf, R = 2000, sim = 'permutation')

perms = boot(res_data, bf, R = 2000, sim = 'permutation')
boot.ci(perms, index = 1)
boot.ci(perms, index = 2)
```

## 13.14 Binomial GLM with ordered categorical variables

An example using the dataframe *esoph*
There are too few cases to fit a full factorial of **agegp*tobgpacco*alcgp**, so we start with a maximal model that a main effect for age and an interaction between tobacco and alcohol:

```{r}
model1 = glm(cbind(ncases, ncontrols) ~ agegp + alcgp * tobgp, binomial, data = esoph)
summary(model1)
```

```{r}
model2 = glm(cbind(ncases, ncontrols) ~ agegp + alcgp + tobgp, binomial, data = esoph)
anova(model1, model2)
```

```{r}
#The critical value of chi-squared with 9 d.f.
qchisq(.95, 9)
summary(model2)
```

```{r}
#loot at the data as proportions
p = esoph$ncases / (esoph$ncases + esoph$ncontrols)
#plot this against the three explanatory variables
par(mfrow = c(2, 2))
plot(p ~ alcgp, col = 'red', data = esoph)
plot(p ~ tobgp, col = 'blue', data = esoph)
plot(p ~ agegp, col = 'green', data = esoph)
```

The tobacco response (blue) could probably be simplified by combining the two intermediate smoking rates:

```{r}
esoph[['tob2']] = esoph$tobgp
levels(esoph$tob2)[2:3] <- '10-30'
levels(esoph$tob2)

esoph[['age2']] <- esoph$agegp
levels(esoph$age2)[4:6] <- '55+'
levels(esoph$age2)[1:2] <- 'under45'
levels(esoph$age2)
```

```{r}
#start the model again with these new ordered factors
model3 = glm(cbind(ncases, ncontrols) ~ age2 * alcgp * tob2, binomial, data = esoph)
model4 = step(model3)
model5 = update(model4, ~. - age2:alcgp)
anova(model4, model5, test = 'Chi')
```

```{r}
esoph[['alc3']] <- esoph$alcgp
levels(esoph$alc3)[2:3] <- '40-119'
esoph$alc3 = factor(esoph$alc3, ordered = F)
levels(esoph$alc3)

esoph[['age3']] <- esoph$age2
esoph[['tob3']] <- esoph$tob2
esoph$age3 <- factor(esoph$age2, ordered = F)
esoph$tob3 <- factor(esoph$tob2, ordered = F)
```

```{r}
model6 = glm(cbind(ncases, ncontrols) ~ age3 * alc3 * tob3, binomial, data = esoph)
model7 = step(model6)
summary(model7)
```

```{r}
model8 = update(model7, ~. - age3:alc3)
anova(model7, model8, test = 'Chi')
```

```{r}
#to visualize the interaction, we plot the predict means of the logits by age and alcohol consumption???
barplot(tapply(predict(model7), list(esoph$age3, esoph$alc3), mean), beside = T)
```

