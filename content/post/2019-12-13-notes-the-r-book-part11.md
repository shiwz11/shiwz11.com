---
title: NOTES_The_R_Book_Part11
author: shiwz11
date: '2019-12-13'
slug: notes-the-r-book-part11
categories:
  - R
tags:
  - regression
  - R Markdown
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-13T10:46:51+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

# Chapter 14 Count Data

## 14.1 A regression with Possion errors

A example:whether or not proximity ot the reactor affects the number of cancer cases

```{r}
clusters <- read.table('clusters.txt', header = T)
attach(clusters)
names(clusters)
#Cancers, Distances
model1 = glm(Cancers ~ Distance, family = poisson)
summary(model1)
```

Actually the residual deviance should be the equals to degrees of freedom, however in this case, we can see the First is far bigger that the second value, which indicates that we have overidspersion.We compensate for the overdispersion by refitting the model using ***quasi-Possion*** rather than Possion errors:

```{r}
model2 = glm(Cancers ~ Distance, family = quasipoisson)
summary(model2)
```

The GLM with Poisson error uses the log link, so the parameter estimates and the predicitons from the model (the 'linear predictor') are in logs, and need to be antilogged `r exp(yv)` before the (non-significant) fitted line is drawn.

```{r}
xv <- seq(0, 100)
yv <- predict(model2, list(Distance = xv))

plot(Cancers ~ Distance, pch = 21, col = 'red', bg = 'orange')
lines(xv, exp(yv), col = 'blue')
```

## 14.2 Analysis of deviance with count data

- Another example:

```{r}
count = read.table('cellcounts.txt' header = T)
attach(count)
names(count)
#cells, smoker, age, sex, weight  
```

```{r}
table(cells)
```

```{r}
#tabulating the main effect means
tapply(cells, smoker, mean)
tapply(cells, weight, mean)
tapply(cells, sex, mean)
tapply(cells, age, mean)
```

```{r}
model1 = glm(cells ~ smoker * sex * age * weight, poisson)
summary(model1)
```

```{r}
#refit the model using quasi-Poisson, because the residual deviance is much greter than the residual degrees of freedom, which indecates overdispersion.
model2 = glm(cells ~ smoker * sex * age * weight, quasipoisson)
summary(model2)
```

There are NAs in the coefficints table, because, there were too few subjets to assess the four-way interaction.

```{r}
model3 = update(model2, ~. -smoker:sex:age:weight)
summary(model3)
```

```{r}
#At last the minimal adequate model look something like this
newWt <- weight
levels(newWt)[c(1, 3)] <- 'not'
summary(model5)

tapply(cells, list(smoker, weight), mean)
```

```{r}
barplot(tapply(cells, list(smoker, weight), mean), col = c('wheat2', 'wheat4'), 
        beside = T, ylab = 'damaged cells', xlab = 'body mass')
legend(1.2, 3.4, c('non-smoker', 'smoker'), fill = c('wheat2', 'wheat4'))
```

## 14.3 Analysis of covariance with count data

```{r}
library(tidyverse)
library(readr)
species <- read.csv('species.txt', header = T)
names(species)
#pH, Biomass, Species
ggplot(species, aes(x = Biomass, y = Species, color = pH)) + geom_point()
```

```{r}
model1 = glm(Species ~ Biomass * pH, poisson)
summary(model1)
```

```{r}
model2 = glm(Species ~ Biomass + pH, poisson)
anova(model1, modle2, test = 'Chi')
```

```{r}
levels(pH) # high, low, mid

pred_line = function(ph, color) {
  pHs = factor(rep(ph, 101))
  xv = seq(0, 10, 0.1)
  yv = predict(model1, list(Biomass = xv, pH = pHs))
  lines(xv, exp(yv), col = color)
}

pred_line('high', 'red')
pred_line('low', 'green')
pred_line('mid', 'blue')
```

## 14.4 Frequency distributions

```{r}
case_book = read.table('cases.txt', header = T)
attach(cases_book)
names(cases_book) #cases

(frequencies <- table(cases_bood$cases))
```

```{r}
#Compare our distributon(frequencies) with the standard Poisson distribution
#draw two distributins side by side
mean(case_bood$cases) #1.775
windows(7, 4)
par(mfrow = c(1, 2))

barplot(frequencies, ylab = 'Frequency', xlab = 'Cases', col = 'green4', main = 'Cases')
barplot(dpois(0:10, 1.775) * 80, names = as.character(0:10), ylab = 'Frequency', xlab = 'Cases', col = 'green3', main = 'Poisson')

var(case_book$cases) / mean(case_book$cases) #299483 much greter than 1
```

Obviously, the data are not poisson, a good candidate distribution where the variance-mean ratio is this big (around 3.0) is the ***negative binomial distribution***.

- A two parameter distribution:

First:mean number of cases

Second:the clumping parameter, k, small values of k(<1) show high aggregation, while large values of k (>5) show randomness.

We can get an approximate estimate of the magnitude of k from:

$$
\begin{aligned}
\hat{k} = \frac{\bar{x}^2}{s^2 - \bar{x}}
\end{aligned}
$$
```{r}
mean(case_book$cases) ^ 2 / (var(case_bood$cases) - mean(case_book$cases)) #0.8898003

exp <- dnbinom(0:10, 1, mu = 1.775) * 80
both = numeric(22)
both[1:22 %% 2 != 0] <- frequencies
both[1:22 %% 2 == 0] <- exp

labels <- character(22)
labels[1:22 %% 2 == 0] <- as.character(0:10)

windows(7, 7)
barplot(both, col = rep(c('red4', 'blue4'), 11), names = labels, ylab = 'Frequency', xlab = 'Cases')
legend(locator(1), c('observed', 'expected'), fill = c('red4', 'blu4'))
```

We can calculate ***Pearson's chi-squared*** $\sum{(O - E)^2 / E}$ based on the number of comparisons that have expected frequency greater than **4**.

```{r}
exp
#create an upper interval called *5+* for '5 or more'
cs = factor(0:10)
levels(cs)[6:11] <- '5+'
levels(cs)

ef = as.vector(tapply(exp, cs, sum))
of = as.vector(tapply(frequencies, cs, sum))

sum((of - ef) ^ 2 / ef) #3.594145
(p_value = pchisq(3.594145, 3, lower.tail = F))
#H0:the observed and expected distributions are not signigicantly different
```

In conclusion, a negative binomial description of these data is reasonable

## 14.5 Overdispersion in log-linear models

```{r}
library(MASS)
data(quine)
names(quine)
```

```{r}
model1 = glm(Days ~ Eth * Sex * Age * Lrn, poisson, data = quine)
summary(model1)
```
```{r}
#use quasi-Poisson errors to account for the overdispersion
model2 = glm(Days ~ Eth * Sex * Age * Lrn, quasipoisson, data = quine)
summary(model2)
```

***NAs*** are not estimated cause of missing factor-level combinations, as indicated by the zeros in the following table:

```{r}
attach(quine)
ftable(table(Eth, Sex, Age, Lrn))
```

Akaike's information citerion is not defined for this model, so we cannot use the `r step` or `r stepAIC` function.

Remember to do F tests (not Chi-squared) cause of the overdispersion.

```{r}
#The last step for the model simplification
model4 = update(model3, ~. - Age:Lrn)
anova(model3, model4, test = 'F')
#the p_value = 0.3598 so we do not need the interaction 
#so , the minimal adequate model with quasi-Poisson errors
summary(model4)

ftable(tapply(Days, list(Eth, Sex, Lrn), mean))
```

## 14.6 Negative binomial errors

```{r}
model_nb1 = glm.nb(Days ~Eth*Sex*Age*Lrn)
summary(model_nb1, cor = F)
```

The $\theta$ in the output is the parameter ***k*** in the negative binomial model

In the negative binomial model we can use `r stepAIC` to simplify the model automatically

```{r}
model_nb2 = stepAIC(model_nb1)
summary(model_nb2, cor = F)
```

```{r}
#we continue the process by hand
model_nb3 = update(model_nb2, ~. - Sex:Age:Lrn)
anova(model_nb3, model_nb2)
```

```{r}
model_nb4 = update(model_nb3, ~. - Eth:Age:Lrn)
anova(model_nb3, model_nb4)

model_nb5 = update(model_nb4, ~. - Age:Lrn)
anova(model_nb4, model_nb5)

summary(model_nb5, cor = F)
par(mfrow = c(2, 2))
plot(model_nb5)
```

In conclusion: we get the same minimal adequate model as we obtained with quasi-Poisson, but the significance levels are higher.

The combination of low P values and ability to use `r stepAIC` makes `r glm.nb` a very useful modelling function for count data.

