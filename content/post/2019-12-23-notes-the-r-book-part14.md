---
title: NOTES_The_R_Book_Part14
author: shiwz11
date: '2019-12-23'
slug: notes-the-r-book-part14
categories:
  - R
tags:
  - R Markdown
  - regression
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-23T10:26:08+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

# Chapter 17 Binary Response Variables

Binary analysis would be useful when at least one of our explanatory variables is continuous
Bernoulli distribution:

$$
\begin{aligned}
P(y) = p^y (1 - p)^{(1 - y)}
\end{aligned}
$$

Do I have unique values of one or more expanatory variables for each and every individual case?

If the answer is "yes", then analysis with a binary response variable is likely to be fruitful.

If no, then there is nothing to be gained, and we should reduce our data ba aggregating the counts to the resolution at which each 
count does have a unique set of explanatory variables.

In order to carry out modelling on a binary response variable we take the follwing steps:

1.Create a single vector containing 0s and 1s as the response variable(独热编码，采用`factor()`即可)

2.Use *glm* with *gamily = binomial*

3.Consider changing the link function from the default logit to complementary log-log

4.Fit the model in the usual way.

5.Test significance by deletion of terms from the maximal model, and compare the change in deviance with chi-squared

We can fit non-parametric smoother to bianry response variables using generalized additive models instead of carrying our parametric logistic regression.

## 17.1 Incidence funcitons

Example:

```{r, eval = F}
island = read.table('isolation.txt', header = T)
attach(island)
names(island)
#incidence, area, isolation

model1 = glm(incidence ~ area * isolation, binomial)
model2 = glm(incidence ~ area + isolation, binomial)
anova(model1, model2, test = 'Chi') # p = 0.6981
#accept the simpler model
summary(model2)
#plot the model
modela = glm(incidence ~ area, binomial)
modeli = gln(incidence ~ islation, binomial)

windows(7, 4)
par(mfrow = c(1, 2))
xv = seq(0, 9, 0.01)
yv = predict(modela, list(area = xv), type = 'response')
plot(area, incidence)
lines(xv, yv, col = 'red')

xv2 = seq(0, 10, 0.01)
yv2 = predict(modeli, list(isolation = xv2), type = 'response')
plot(isolation, incidence)
lines(xv2, yv2, col = 'red')
```

## 17.2 Graphical tests of the fit of the logistic to data

Example:

The response is occupation of territories and the expanatory variable is resource availability in each territory

```{r, eval = F}
occupy = read.table('occupation.txt', header = T)
attach(occupy)
names(occupy)
#resources, occupied

plot(resources, occupied, type = 'n')
rug(jitter(resources[occupied == 0]))
rug(jitter(resources[occupied == 1]), side = 3)

#fit the logistic regression and draw the line
model = glm(occupied ~ resources, binomial)
xv = 0:1000
yv = predict(model, list(resources = xv), type = 'response')
lines(xv, yv, col = 'red')
#cut up the ranked values on the x axis into five categories and then work out the mean and the standard error of the proportions in each group
cutr = cut(resources, 5)
tapply(occupied, cutr, sum)

#use *table* functino to count the number of cases in each bin
table(cutr)
#the empirical parobabilities 
probs = tapply(occupied, cutr, sum) / table(cutr)
probs = as.vector(probs)
resmeans = tapply(resources, cutr, mean)
resmeans = as.vector(resmeans)
points(resmeans, probs, pch = 16, cex = 2, col = 'blue')
#add a measure of unreliability to the points, the srandard error of abinomial proportion will do:
se = sqrt(probs*(1-probs)/table(cutr))
#draw lines up and down from each point indicating one standard error
up = probs + as.vector(se)
down = probs - as.vector(se)
for (i in 1:5) {
  lines(c(resmeans[i], resmeans[i]), c(up[i], down[i]), col = 'blue')
}

```

## 17.3 ANCOVA with a binary response variable

Example:

response variable is paraisite infection, the explanatory variables are weight and sex and age

```{r, eval = F}
infection = read.table('infeection.txt', header = T)
attach(infection)
names(infection)
#infected, age, weight, sex
windows(7, 4)
par(mfrow = c(1, 2))
plot(infected, weight, xlab = 'Infection', ylab = 'Weight', col = 'green')
plot(infected, age, xlab = 'Infection', ylab = 'Age', col = 'green4')
#to see the relationship between infection and gender(both categorical variables)
table(infected, sex)

model = glm(infected ~ age * weight * sex, family = binomial)
summary(model)

model2 = step(model)
summary(model2)

model3 = update(model2, ~. - age:weight)
anova(model2, model3, test = 'Chi') # p = 0.09526
model4 = update(model2, ~. - age:sex)
anova(model2, model4, test = 'Chi') # p = 0.1552

model5 = glm(infected ~ age + weight + sex, falimy = binomial)
summary(model5)

#whether there is any evidence of non-linearity in the response of infection to weight or age?
model6 = glm(infected ~ age + weight + sex + I(weight^2) + I(age^2), binomial)
summary(model6)
#both relationships are significantly curvilinear.
#Let us start with generalized additive model
library(mgcv)
model7 = gam(infected ~ sex + s(age) + s(weight), binomial)
windows(7, 4)
par(mfrow = c(1, 2))
plot.gam(model7)
#the results are excellent at showing the humped relationship between infection and age, and highlighting the possibility of a threshold at at weight \approx 8 in the relation ship between weight and infection.
#we try a piecewise linear fit for weight.
model8 = glm(infected ~ sex + age + I(age^2) + I((weight-12)*(weight>12)), family = binomial)
#I((weight-12)*(weight>12)) means regress infection on the value of *weight - 12*, but only do this when *weight > 12* is true
summary(model8)

#Finally, the minimal adequate model
model9 = glm(infected ~ age + I(age^2) + I((weight - 12) * (weight > 12)), binomial)
summary(model9)
```

Conclusion:

there is a humped relationship between infecitno and age, and a threshold effect of weight on infection. The effect of sex is marginal, but might repay futhur investigation(p = 0.0071)

## 17.4 Binary response with pseudoreplication

Question:whether the two treatments singnificantly reduced bacterial infection?

```{r}
library(MASS)
names(bacteria)
```

```{r}
table(bacteria$y)
table(bacteria$y, bacteria$trt)
```

There is temporal pseodoreplication(repreted measures on the same patients) so we cannot use `glm`.So we use the generalized mixed models function `lmer`.

`lmer` is in the package `lme4`.The random effects apper in the same formula as the fixed effects, but defined by the round brackets and the "given" operator `|` to separate the continuous random effect(*week*) from the catrgorical random effect (patient *ID*).

```{r}
library(lme4)
model1 = glmer(y ~ trt + (week|ID), family = binomial, data = bacteria)
summary(model1)
```

We can simplify the model by removing the dependence of infection on week, retaining only the intercept as a random effect *+(1|ID)*

```{r}
model2 = glmer(y ~ trt + (1|ID), family = binomial, data = bacteria)
anova(model1, model2)
```

The simpler `model2` is significantly wore (p = 0.01047) so we accept model1(it has lower AIC than model2), retaining the random effect for week.

```{r}
#combine the drug levels
bacteria$drugs = factor(1 + (bacteria$trt != 'placebo'))
table(bacteria$y, bacteria$drugs)

model3 = glmer(y ~ drugs + (week|ID), family = binomial, data = bacteria)
summary(model3)
```

Another way to get rid of the pseudoreplication is to restrict the analysis to the patients that were there at the end of the experiment.We just use `subset = (week == 11)` and this removes all the pseusoreplication cause no subjects were measured twice within any week.

```{r}
any(table(bacteria$ID, bacteria$week) > 1)
```

Then the model will became a GLM with a binary response variable and a single explanatory variable.

```{r}
model = glm(y ~ trt, binomial, subset = (week == 11), data = bacteria)
summary(model)
```

```{r}
bacteria$drugs = factor(1 + (bacteria$trt == 'placebo'))
table(bacteria$drugs[bacteria$week == 11])
```

```{r}
model = glm(y ~ drugs, binomial, subset = (week == 11), data = bacteria)
summary(model)
```

```{r}
dss = data.frame(table(bacteria$trt, bacteria$ID))
names(dss) = c('trt', 'ID', 'Freq')
head(dss)
```

Find out the treatments of the patients that scored `Freq > 0`

```{r}
tss = dss[dss[, 3] > 0, ]$trt
ys = table(bacteria$y, bacteria$ID)
yv = cbind(ys[2, ], ys[1, ])

model = glm(yv ~ tss, binomial)
summary(model)
```

```{r}
#correct for overdispersion
model = glm(yv ~ tss, quasibinomial)
summary(model)
```

aggregate two drug levels into one

```{r}
tss2 = factor(1 + (tss == 'placebo'))
model = glm(yv ~ tss2, quasibinomial)
summary(model)
```

