---
title: NOTES_The_R_Book_Part13
author: shiwz11
date: '2019-12-20'
slug: notes-the-r-book-part13
categories:
  - R
tags:
  - R Markdown
  - regression
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-20T09:25:10+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


# Chapter 16 Proportion Data

In contrast with Poisson count data, where we knew how many times an event occured, but not how many times it did not occur.

In glm, with binomial errors we must give the number of failure as well as the numbers of sucess in a two-vector response variable.

```{r}
number_of_failures = binomial_denominator - number_of_successes
y = cbind(number_of_successes, number_of_failures)
```

If we use the percentage mortality as the response variable:By calculating the percentage, we lose information on the size of the sample, n, from which the proportion was estimated.

R carries out weighted regression, using the individual sample sizes as weights, and the logit link function to ensure linearity.

## 16.1 Analyses of data on one and two proportions

For conparisons of one binomial proportion with a constant, use *binom.test*(details in page 600).For comparison of two samples of proportion data, use *prop.test*(p.365).
The methods of this chapter are required only for more complex data.

## 16.2 Count data on proportions

- Two traditional transformations of proportions data:

1.arcsin:took care of the error distribution.

2.probit:linearize the relationship between percentage mortality and log dose in a bioassay

The variance of a binomial distribution with mean np is:

$$
\begin{aligned}
s^2 = npq
\end{aligned}
$$

The variance is low when p is very high or very low, and the variance is greatest when p = q = 0.5.

As $p$ gets smaller, so the binomal distribution gets closer and closer to the Poisson distribution.

$$
s^2 = npq \approx np \qquad (q \approx 1)
$$

## 16.3 Odds

The logistic model for p as a function of $x$ is given by:

$$
p = \frac{e^{a + bx}}{1 + e^{a + bx}}
$$

The model is non-linear and strictly bounded to [0, 1]

How to linearize this model?

take the odds $p/q$ and substitute this into the formula for the logistic, we get:

$$
\begin{aligned}
\frac{p}{q} &= \frac{e^{a + bx}}{1 + e^{a + bx}}\left[1 - \frac{e^{a+bx}}{1 + e^{a + bx}}\right]^{-1} \\
\Downarrow \\
\frac{p}{q} &= \frac{e^{a+bx}}{1 + e^{a+bx}}\left[\frac{1}{1+e^{a+bx}} \right]^{-1} = e^{a+bx} \\
\Downarrow \\
ln(\frac{p}{q}) &= a + bx
\end{aligned}
$$

- GLM with binomial errors has three great anvantages here:

1.It allows for the non-constant binomial variance

2.It deals with the fact that logits for ps near 0 or 1 are infinite

3.It allows for differences between the sample sizes by weight regression.

## 16.4 Overdispersion and hypothesis testing

When we have obtained the minimal adequate model, the ***residual scaled deviance should be roughly equal to the residual degrees of freedom.***There are two possibilities:either the model misspecified, or the probability of sucess, p, is not constant with in a given tretment level. The effect of randomly varying p is pto increase the binomial variance from npq to :

$$
s^2 = npq + n(n - 1)\sigma^2
$$
To solve this, we assume the variance is not npq but $npq \phi$, where $\phi$ is an unknown ****scale parameter***($\phi > 1$).We obtain an estimate of the scale parameter by dividing the Pearson chi-squared by the degrees of freedom, and use this estimate of $\phi$ to cmopare the resulting scaled deviances.

Remember that you do not obtain exact p values with binomial errors;the chi-squared approximations are sound for large samples, but small samples may present a problem

The linear predictor is in logits(the log of the odds = ln(p/q))

You can back-transform from logits(z) to proportions(p) by p = 1/[1+1/exp(z)]

## 16.5 Applications

### 16.5.1 Logistic regression with binomial errors

- Example:

whether population density was involved in determining the fraction of males of the insects

```{r}
numbers = read.table('sexratio.txt', header = T)
attach(numbers)
head(numbers)

#Plot the data
windows(7, 4)
par(mfrow = c(1, 2))
p = males / (males+females)

plot(density, p, ylab = 'Proportion male', pch = 16, col = 'blue')
plot(log(density), p, ylab = 'Proportion male', pch = 16, col = 'blue')

#Start the modelling 
y = cbind(males, females)
#which means that *y* will interpreted in the model as the proportion of all individuals that were male.
model = glm(y ~ density, binomial)
summary(model) # Residual deviance:22.091 on 6 degrees of freedom

#Let's see the log trasformation of the explanatory variable 
model2 = glm(y ~ log(density), binomial)
summary(model2) # Residual deviance:5.6 on 6 degrees of freedom
#check the model2
plot(model2)
#draw the fitted line througn the scatterplot:
windows(7, 4)
xv = seq(0, 6, 0.01)
yv = predict(model2, list(density = exp(xv)), type = 'response')
plot(log(density), p, ylab = 'Proportion male', pch = 16, col = 'blue')
lines(xv, yv, col = 'red')
#The use of type = 'response' to back-transform from the logit scale to the s-Shaped proportion scale
```

### Estimating LD50 and LD90 from bioassay data

- Example:

We use a value of y(50%) to predict a value of x(relavant dose) and to work out a standard error on the x axis

```{r}
data = read.table('bioassay.txt', header = T)
attach(data)
names(data)
#dose, dead, batch
#The logistic regression
y = cbind(dead, batch-dead)
model = glm(y ~ log(dose), binomial)

library(MASS)
MASS::dose.p(model, p = c(0.5, 0.9, 0.95))
```

### 16.5.3 Proportion data with categorical explanatory variables

a two-way factorial analysis of deviance

```{r}
germination = read.table('germination.txt', head = T)
attach(germination)
names(germination)
#count, sample, Orobanche, extract
#The *count* is thenumber of seed that germinated out of a batch of size = *sample*.
y <- cbind(count, sample - count)
levels(Orobanche) # a73, a75
levels(extract) # bean, cucumber
```

We want to test the hypothesis that there is no interacion between Orobanche genotype and plant extract on the germination rate of the seeds.

```{r}
model = glm(y ~ Orobanche * extract, binomial)
summary(model) # Residual deviance:33.278 on 17 degrees of freedom

model = glm(y ~ Orobanche * extract, quasibinomial)
model2 = upudate(model, ~. - Orobanche:extract)

anova(model, model2, test = 'F') # p = 0.8099

#consider futher model simplification
anova(model2, test = 'F') # p_extract = 6.692e-05, p_Orobanche = 0.2887

model3 = update(model2, ~. - Orobanche)
anova(model2, model3, test = 'F') # p = 0.2457
#so the minimal adequate model is model3
#calculate the predicted values
tapply(predict(model3, type = 'response'), extract, mean) # type = 'response' makes predictions on the back-transformed scale automatically
```

## 16.6 Averaging proportions

## 16.7 Summary of modelling with propotion count data

1.Make a two-column response vector containing the successes and failures

2.Use `glm` with `family = binomial`

3.Fit the maximal model

4.Test for overdispersion

5.If you find overdispersion then use `quasibinomial` rather than `binomial` errors

6.Begin model simplification by removing interaction terms

7.Remove non-sigmificant main effects

8.Use `plot` to obtain your model-checking diagnostics

9.Back-transform using `predict` with the option `type = 'response'` to obtain means

## 16.8 Analysis of covariance with binomial data

- Example:

```{r, eval = F}
props = read.table('flowing.txt', header = T)
attach(props)
names(props)
#flowered, number, dose, variety
y = cbind(flowered, number - flowered)
pf = flowered / number
pfc = split(pf, variety)
dc = split(dose, variety)

plot(dose, pf, type = 'n', ylab  = 'Proportion flowered')

for (num in 1:5) {
  for (color in c('red', 'green', 'yellow', 'green3', 'brown')) {
    points(jitter(dc[[num]]), jitter(pfc[[num]]), pch = 21, col = 'blue', bg = color)
  }
}
  
#start modelling
model1 = glm(y ~ dose * variety, binomial)
summary(model1) # Residual deviance:51.083 on 20 degrees of freedom

#obviously, there exists overdisperison, 
#but it may be cause of poor model selection rather than extra.
#Let us investigate this by plotting the fitted curves through the scatterplot

xv = seq(0.35, 0.1)
a = list(c('A', 'red'), c('B', 'green'), c('C', 'yellow'), c('D', 'green3'), c('E', 'brown'))
for (vari in a) {
  vn = rep(vari[1], length(xv))
  yv = predict(model1, list(variety = factor(vn), dose = xv), type = 'response')
  lines(xv, yv, col = vari[2])
}

legend(locator(1), legend = c('A', 'B', 'C', 'D', 'E'), title = 'variety', lty = rep(1, 5), col = c('red', 'green', 'yellow', 'green3', 'brown'))

tapply(pf, list(dose, variety), mean)
```

In conclusion, the logistic was a poor choice for two of the five varities in this case.

## 16.9 Converting complex contingency tables to proportions

In this section we show how to remove the need for all of the nuisance variables that are involved in complex contingency tabel modelling.We can do this when the response can be restated as a binary proportion.

- Example:

```{r, eval = F}
lizards = read.table('lizards.txt', header = T)
attach(lizards)
names(lizards)
#n, sun, height, perch, time, species
head(lizards)

sorted = lizards[order(species, sun, height, perch, time)]
head(sorted)

short <- sorted[1:24]

names(short)[1] = 'Ag'
names(short)
#Ag, sun, height, perch, time, species
short = short[, -6]

sorted$n[25:48]

new_lizards = data.frame(sorted$n[25:48], short)
names(new_lizards)[1] = 'Ao'
head(new_lizards)

detach(lizards)
rm(short, sorted)
attach(new_lizards)
```

### 16.9.1 Analysing Schoener's lizards as proportion data

```{r, eval = F}
y = cbind(Ao, Ag)
model1 = glm(y ~ sun * height * perch * time, binomial)
#since there are no nuisance variables, we can use *step* directly to begin the model simplification
model2 = step(model1)
#The AIC is much too generous in leaving terms in the model, however, we are ruthless
model3 = update(model2, ~. - height:perch:time)
model4 = update(model2, ~. - sun:height:perch)
anova(model2, model3, test = 'Chi') # p = 0.07422
anova(model2, model4, test = 'Chi') # p = 0.1148

model5 = glm(y ~ (sun+height+perch+time)^2 - sun:time, binomial)
model6 = update(model5, ~. - sun:height)
anova(model5, model6, test = 'Chi') # p = 0.1252

model7 = update(model5, ~. - sun:perch) 
anova(model5, model7, test = 'Chi') # p = 0.8779

model8 = update(model5, ~. - height:perch)
anova(model5, model8, test = 'Chi') # p = 0.6242

model9 = update(model5, ~. - time:perch)
anova(model5, model9, test = 'Chi') # p = 0.9971

model10 = update(model5, ~. - time:height)
anova(model5, model10, test = 'Chi') # p = 0.6516
#so, we do not need any two-way interactions
#now, the main effects
model11 = glm(y ~ sun + height + perch + time, binomial)
summary(model11) # Residual deviance:14.205 on 17 degrees of freedom
#from the summary we know that *Mid.day* and *Morning* are not significantly different, we lump them together in a new factor called *t2*

t2 = time
levels(t2)[c(2, 3)] = 'other'
levels(t2) # Afternoon, other

model12 = glm(y ~ sun + height + perch + t2, binomial)
anova(model11, model12, test = 'Chi') # p = 0.3656

#At long last the minimal adequate model
summary(model12)
```

