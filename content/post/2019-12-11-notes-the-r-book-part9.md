---
title: NOTES_The_R_Book_Part9
author: shiwz11
date: '2019-12-11'
slug: notes-the-r-book-part9
categories:
  - R
tags:
  - R Markdown
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-11T09:10:51+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

# Chapter 12 Analysis of Covariance

ANCOVA combines elements from regression and ANOVA.The response variabel is continous and there is atleast one continuous explanatory variable and at least on categorical exlanatory variable.

1.Fit tow or more linear regression of y and x (one for each level of the factor)

2.Estimate different slopes and intercepts for each level

3.Use mdoel simplification (deletion tests) to eliminate unnecessary parameters

Suppose we are modelling weight as a funciton of sex and age.

$$
weight_{male} = a_{male} + b_{male} \times age \\
weight_{female} = a_{female} + b_{female} \times age
$$

```{r}
#An example
regrowth = read.table('ipomopsis.txt', header = T)
attach(regrowth)
names(regrowth)
#Root, Fruit, Grazing
#Inspect the data with a plot of fruit production against root size for each of the two treatments separately
plot(Root, Fruit, pch = 16, col = c('blue', 'red')[as.numeric(Grazing)])
levels(Grazing)
#Grazed, Ungrazed
#Now draw linear regression line for the two grazing treatments separately, using *abline*
abline(lm(Fruit[Grazing == 'Grazed'] ~ Root[Grazing == 'Grazed']), col = 'blue')
abline(lm(Fruit[Grazing == 'Ungrazed'] ~ Root[Grazing == 'Ungrazed']), col = 'red')
#we get a highly counterintuitive result
tapply(Fruit, Grazing, mean)
```

We do the analysis of covarance by hand.

First, work out the overall totals based on all 40 data points:

```{r}
#The famous five
sum(Root);sum(Root^2)
sum(Fruit); sum(Fruit^2)
sum(Root * Fruit)

sum(Root[Grazing == 'Grazed']); sum(Root[Grazing == 'Grazed']^2)
sum(Root[Grazing == 'Ungrazed']); sum(Root[Grazing == 'Ungrazed'] ^ 2)
sum(Fruit[Grazing == 'Grazed']); sum(Fruit[Grazing == 'Grazed'] ^ 2)
sum(Fruit[Grazing == 'Ungrazed']); sum(Fruit[Grazing == 'Ungrazed'] ^ 2)
#Finally, the sums of productss
sum(Root[Grazing == 'Grazed'] * Fruit[Grazing == 'Grazed'])
sum(Root[Grazing == 'Ungrazed'] * Fruit[Grazing == 'Ungrazed'])
```

```{r}
ancova = lm(Fruit ~ Grazing * Root)
summary(ancova)
anova(ancova)
```

The next step is to delete the non-significant interaction from the model.

Firt, do this manually

```{r}
ancova2 = update(ancova, ~ . - Grazing:Root)
#compare the two models
anova(ancova, ancova2)
```

Next, compare whether or not grazing had a significant effect on fruit producion once we control for initial root size

```{r}
ancova3 = update(ancova2, ~ .  - Grazing)
anova(ancova2, ancova3)
summary(ancova2)
```

funcion `r step(model)` causes all the terms to be tested to see whether they are needed in the minimal adequate model.The criterion used is *Akaike's information criterion(AIC)*.

The AIC adds 2 times the number of parameters in the model to the deviance(to penalize it).Deviance, is twice the log-likelihood of the current model.AIC is a measure of *lack of fit*.big AIC is bad , small AIC is good

```{r}
step(ancova)
```

## 12.3 ANCOVA with tow factors and one contiuous covariate

```{r}
Gain = read.table('Gain.txt', header = T)
attach(Gain)
names(Gain)
#Weight, Sex, Age, Genotype, Score
#Begin by fitting the maximal model with 24 parameters
m1 = lm(Weight ~ Sex * Age * Genotype)
summary(m1)
#Thereo os only one intercept and one slope intheis entire outplut.Everything else is either a difference between intercepts, or a difference between slopes
#Firstly let's check the step to see how far it gets with mdoel simplification
m2 = step(m1)
summary(m2)
```

```{r}
newGenotype = Genotype
levels(newGenotype)
#CloneA, CloneB, CloneC, CloneD, CloneE, CloneF
levels(newGenotype)[c(3, 5)] = 'ClonesCandE'
levels(newGenotype)[c(2, 4)] = 'ClonesBandD'
levels(newGenotype)
#CloneA, ClonesBanD, ClonesCandE, CloneF
m3 = lm(Weight ~ Sex + Age + newGenotype)
anova(m2, m3)
summary(m3)
plot(Age, Weight, type = 'n')
colours = c('green', 'red', 'black', 'blue')
lines = c(1, 2)
symbols = c(16, 17)
points(Age, Weight, pch = symbols[as.numeric(Sex)], col = colours[as.numeric(newGenotype)])
xv = c(1, 5)
for (i in 1:2) {
  for (j in 1:4) {
    a = coef(m3)[1] + (i > 1) * coef(m3)[2] + (j > 1) * coef(m3)[j + 2]
    b = coef(m3)[3]
    yv = a + b * xv
    liens(xv, yv, lty = line[i], col = colours[j])
  }
}

```

Other fucntions to be considered in plotting the results of ANCOVA are `r split` and `r augPred`

## 12.4 Contrasts and the parmeters of ANCOVA models

When the factor are unordered, then T takes the factor level that comes first in the alphabet as the estimate and the others are expressed as differences
  
```{r}
#example
Ancovacontrasts = read.table('Ancovacontrasts.txt', header = T)
attach(Ancovacontrasts)
names(Ancovacontrasts)
#weight, sex, age
#Firstly, work out the two regressions separately so that we know the values of the two slopes and the two intercepts
lm(weight[sex == 'male'] ~ age[sex == 'male'])
lm(weight ~ age, subset = (sex == 'female'))
#Secondly, do an overall regression, ignoring gender
lm(weight ~ age)
#Thirdly, carry out the covariance and compre the output, in R, the default method used is treatmet contrast.
options(contrasts = c('contr.treatment', 'contr.poly'))
model1 = lm(weight ~ age*sex)
summary(model1)
#Now turn to the analysis using Helemert contrasts
options(contrasts = c('contr.helmert', 'contr.poly'))
model2 = lm(weight ~ age * sex)
summary(model2)

```

The advantage of Helmert contrasts is in hypothesis testing in more complicated models than this, because it is easy to see which terms we need to retain in a simplified model by inspecting their significance  levels in the summary.lm tabe. The disadvantage is that it is much harder to reconstruct the slopes and the intercepts from the estimated parameters values

Finally, the sum contrasts

```{r}
options(contrasts = c('contr.sum', 'contr.poly'))
model3 = lm(weight ~ age * sex)
summary(model)
```

## 12.5 Order mattters in summary.aov

Whenever the $x$ values are different in different factor levels, and/or there is different replication in differen factor levels, then SSX and SSXY will vary from level to level and this will affect the way the sum of squares is distributed across the main effects.
(more details in page.555)

