---
title: NOTES_The_R_Book_Part8
author: shiwz11
date: '2019-12-10'
slug: notes-the-r-book-part8
categories:
  - R
tags:
  - R Markdown
---


## 11.2 Factorial experiments

A factorial experiment has two or more factors, each with two or more levels, plus replication for each combination of factors levels.This means that we can investigate statistical interactions, in which *the response to one factor depends on the level of another facotr*

- Example:

```{r}
weights = read.table('growth.txt', header = T)
attach(weights)
#First inspect the data
barplot(tapply(gain, list(diet, supplement), mean), beside = T, ylim = c(0, 30), col = c('orange', 'yellow', 'cornsilk'))
labs = c('Barley', 'Oats', 'Wheat')
legend(locator(1), labs, fill = c('orange', 'yellow', 'cornsilk'))

#inspect the mean value use tapply()
tappl(gain, list(diet, supplement), mean)

#next, fit a facotrial analysis of variance
model = aov(gain ~ diet * supplement)
summary(model)
summary.lm(model)
#The parameter labelled intercept is the mean with both facotr levels set to their fitst in the alphabet.
#First simplify the model by leaving out the interaction terms 
model = aov(gain ~ diet + supplement)
summary.lm(model)

supp2 = factor(supplement)
levels(supp2)
#'agrimore' 'control' 'supergain' 'supersupp'
levels(supp2)[c(1, 4)] = 'best'
levels(supp2)[c(2, 3)] = 'worst'
levels(supp2)
#'best' 'worst'
model2 = aov(gain ~ diet + supp2)
anova(model1, model2)
summary.lm(model2)
```

## 11.3 Pseudoreplicaiton(伪重复):Nested disigns and split plots

- Two basic cases:

1.nested sampling, as when repeated measurements are taken from the same individual, or observational studies are conducted at several different spatial scales(mostly random effects);

2.split-plot analysis, as when disigned experiments have different treatments applied to plots of different sizes (mostly fixed effects)

### 11.3.1 Split-plot experiments

In a split-plot experiment, different treatments are applied to plots of different sizes.Each different plot size is associated with its own error variance.So we have as many error terms as there are different plot sizes.The analysis is presented as  a series of component ANOVA tables. one for each plot size,, in a hierarchy from thr largest plot size with the lowest replicaiton at the top, down to the smallest plot size with the greatest replication at the bottom.

```{r}
#Example
yields = read.table('splityield.txt', header = T)
attach(yields)
names(yields)
#yield, block, irrigation, density, fertilizer
model = aov(yield ~ irrigation * density * fertilizer + Error(block/irrigation/density))
summary(model)
interaction.plot(gertilizer, irrigation, yield)
interaction.plot(density, irrigation, yield)
```

We could use the `r effects` package which takes a model object (a linear model or a generalized linear model) and provides attractive trellis plots of specified interaction effects

When there are one or more missing values(NA) we use `r lme` or `r lmer` rather than `r aov`.The output of aov is not to be trustred under thest circumstances

### 11.3.2 Mixed-effects models

The explanatory variables are mixture of fixed effects and random effects:

1.fixed effects influence only the ***mean of y***

2.random effects influence only the ***variance of y***

we speak of *prediction* of random effects, rather than estimation
  
Mixed-effects model take care of this non-independence of errors by modelling the covariance structure introduced by the groupng of the data

Instead of estimating a mean of every single factor level, the random-effects model estimates the distribution of the means(usually as the standard deviation of the differences of the factor-level means around an overall mean).Thus, it economizes on the number of degrees of greedom used up by the factor levels.

Mixed-effects models are particularly useful in cases where there is temporal pseudoreplicaiton(repeated measurements) and/or spatial pseudoreplication(e.g. nested designed or split-plot experiments).These models can allow for:

1.spatial autocorrelation between neightbours

2.temporal autocorrelation across repeated measures on the same individuals

3.differences in the mean response between blocks in a field experiment

4.differences between subjects in a medical trial involving repeated measures

we want to make use of the all measurements we have taken, but cause of the pseudoreplicaiton we want to take account of both the:

1.correlation structure, used to model within-group correlaiton associated with temporal and spatial dependecies, using `r correlation` ,and

2.variance function, used to model non-constant variance in the within-group errors using `r weight`

### 11.3.4 Removing the pseudoreplicatoin

if we are interested in the fixed effects, then the best response to pseudoreplication in a data set is simply to eliminate it. Spatial pseudoreplication can be averaged away.

**Note** that we should not use `r anova` to compare different models for the fixed effects when using `r lme` or `r lmer` with `r REML`.

## 11.4 Variance components analysis

For random effects we are often more interested in the question of how much of the variantion in response variable canbe attributed to a given factor, than we are in estimating means or assessing te significance of differences between means.This procedure is called **variance components analysis**

```{r}
#An classic example of spatial pseudoreplicaiton from Snedecor and Cochran(1980)
rats = read.table('rats.txt', header = T)
attach(rats)
names(rats)
# Glycogen, Treatmen, Rat, Liver
#The factor levels are numbers , so we need to declare the explanatory variables to be categorical before we start
Treatment = factor(Treatment)
Rat = factor(Rat)
Liver = factor(Liver)
#the mean glycogen values for the six rats
(mean = tapply(Glycogen, list(Treatment, Rat), mean))
#a new variable to represent the teaments associated with each of these rats.use the gl() function('generate levels')
treat = gl(3, 1, length = 6)
#Now fit the non-pseudoreplicated model with the correct error degrees of freedom
model = aov(as.vector(means) ~ treat)
summary(model)
```

The correct analysis using `r aov` with multiple error terms.In the *Error* term we start with the largest scale(treatment), then rats within treatments, then liver bits within rats within treatments.Finally, there were replicated measurements(two preparations) made for each bit of liver

An example of three-way analysis of variance showing the difference between `r aov` and `r lm`

```{r}
daphnia = read.table('Daphnia.txt', header = T)
attach(daphnia)
names(daphnia)
#Growth.rate, Water, Detergent, Daphania
model1 = aov(Growth.rate ~ Water * Detergent * Daphnia)
summary(model1)
#The output from the same ayaysis using the linear model funciton:
model2 = lm(Growth.rate ~ Water * Detergent * Daphnia)
summay(model2)
```

In complicated designed experiments, it is easiest to summarize the effect sizes with `r pllot.design` and `r model.tables` functions.For main effects, use:

```{r}
plot.design(Growth.rate ~ Water * Detergent * Daphnia)
```

The `r model.tables` function takes the name of the fitted model object as its first argument, and we can specify whether we want the standard errors

```{r}
model.tables(model1, 'means', se = T)
```

Attractive plots of effect sizes can be obtained using the effect library

## 11.6 Multiple comparisons

we should not carry out contrasts until the analysis of variance, calculated over the whole set of samples, has indicated that there are significant differences present(i.e. until after the null hypothesis has been rejected)

When comparint the multiple means across the levels of a factor, a simple comparison using multiple t tests will inflate the probability of declaring a significant difference when there is none. This is because the intervals are calculated with a given coverage probability for each interval but the interpretation of the coverage is usually with respect to the entire family of intervals

```{r}
#example
data = read.table('Fungi.txt', header = T)
attach(data)
names(data)
#Habitat, Fungus.yield
#First we establish whether there is any variation in fungus yield to explain
model = aov(Fungus.yield ~ Habitat)
summary(model)model

#The p value here shows that some habitats produve more fungi than others. we are likely to be interested in which habitats produve significantly more fungi than others.
#two options
#1,apply the *TukeyHSD* to the model to get Tukey's honest significant differences;
#2.use the function *pairwise.t.test* to get adjusted p values for all comparisons
#Tukey's test produces a table of p values by default

TukeyHSD(model)

#plot the confidence intervals if we prefer

plot(TukeyHSD(model))

#Habitats on opposite sides of the dotted line and not overlapping it are significantly different from on another

pairwise.t.test(Fungus.yield, Habitat)

#the default method of adjustment of the p values is *holm*, other adjustment methods include *hochberg, hommel, bonferroni, BH, BY, fdr, and none*
```

## 11.7 Multivariate analysis of variance

Two or more response variables are measured in the same experiment, and we do not want to analysisi them separately, and treat the group of response variables as one multivariate response. 

The function is `r manova`

the `r manova` does not support multi-stratum analysis of variance, so the ofrmula must not include an *Error* term

```{r}
#example
data = read.table('manova.txt', header = T)
attach(data)
names(data)
#tear, gloss, opacity, rate, additive
#First, create a multivariate response variabel
Y = cbind(tear, gloss, opacity)
#Second fit the multivariate analysis of variance usint the manova function
model = manova(Y ~ rate * additive)
symmary(model)
#look at each of the three response variable separately
summay.aov(model)
```

