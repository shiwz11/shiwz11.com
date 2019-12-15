---
title: NOTES_The_R_Book_Part12
author: shiwz11
date: '2019-12-15'
slug: notes-the-r-book-part12
categories:
  - R
tags:
  - R Markdown
  - regression
subtitle: ''
summary: ''
authors: []
lastmod: '2019-12-15T10:14:16+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

# Chapter 15 Count Data in Tables

## 15.1 A two-class table of counts

Pearson's chi-squared:

$$
\chi^2 = \sum \frac{(\text{observed - expected})^2}{\text{expected}}
$$

```{r}
#An built-in function for this test
observed = c(29, 18)
chisq.test(observed)
#binom.test
binom.test(observed)
```

## Sample size for count data

## A four-class table of counts

- Mendel's famous peas test:

whether these data depart significantly from the 9:3:3:1

```{r}
observed <- c(315, 101, 108, 32)
expected <- 556 * c(9, 3, 3, 1) / 16
#quantify the difference between them and ask how likely such a differencce is to arise by chance alone
chisq.test(observed, p = c(9, 3, 3, 1), rescale.p = T)
#Pearson's chi-squared value
sum((observed - expected) ^ 2/ expected)
```

```{r}
#The p_value comes from the right-hand tail of the cumulative probability function of the chi-squared distribution
pchisq(0.470024, 3, lower.tail = F)
```

## 15.4 Two-by-two contingency tables

When there are two explanatory variables and both have just two levels, we have the famous $2 \times 2$ contingency table

```{r}
observed = matrix(observed, nrow = 2)
observed
#Fisher's exact test
fisher.test(observed)
```

```{r}
#Pearson's chi-squared test with Yates' continuity correction
chisq.test(observed)
```

## 15.5 Using log-linear models for simple contingency tables

deviance for a log-linear model of count data

$$
\begin{aligned}
\text{deviance} = 2\sum O\ ln(\frac{O}{E})
\end{aligned}
$$
An example:

29 males and 18 females,we want to know if the sex ratio was significantly male-biased:

```{r}
observed <- c(29, 18)
summary(glm(observed ~ 1, poisson))
```


```{r}
pchisq(2.5985, 1, lower.tail = F)
#we accept the null hypothesis that the sex ratio is 50:50
```

```{r}
#Mendel's peas
observed = c(315, 101, 108, 32)
shape = factor(c('round', 'round', 'wrinkled', 'wrinkled'))
colour = factor(c('yellow', 'green', 'yellow', 'green'))

model1 = glm(observed ~ shape * colour, poisson)
model2 = glm(observed ~ shape + colour, poisson)
anova(model1, model2, test = 'Chi')

summary(model2)
```

## 15.6 The danger of contingency tables

- Example:

```{r}
induced = read.table('induced.txt', header = T)
attach(induced)
names(induced)
#Tree, Aphid, Caterpillar, Count
#start by fitting a saturated model is always the best place to start modelling complex contingency tables
model = glm(Count ~ Tree*Aphid*Caterpillar, family = poisson)

#use the *update* to remove the three-way interaction and use *anova* to test whether the three-way interaciton is significant or not
model2 = update(model, ~, - Tree:Aphid:Caterpillar)
anova(model, model2, test = 'Chi')

#The main question:if there is an interaction between aphid attach and leaf hoiling?
model3 = updata(model2, ~ . - Aphid:Caterpillar)
anova(model3, model2, test = 'Chi')
#The result shows no hint of an interaction(p = 0.954)
```

```{r}
#A wrong example
wrong = glm(Count ~ Aphid * Caterpillar, family = poisson)
wrong1 = update(wrong, ~ . - Aphid:Caterpillar)
anova(wrong, wrong1, test = 'Chi')
#The result shows the Aphid by Caterpillar interaction is highly significant(p = 0.01)
```

Main effects are meaningless in contingency tabels(they do nothing more than constrain the marginal totals), as are the model summaries.

Always test for overdispersion

## 15.7 Quasi-Poisson and negative binomial models compared

An example for data on red blood cell counts:

```{r}
data = read.table('bloodcells.txt', header = T)
attach(data)
names(data)
#count

gender = factor(rep(c('female', 'male'), c(5000, 5000)))
#The idea is to test the significance of the differencce in mean cell counts for the two genders
tapply(count, gender, mean)

#The simplest  log-linear model - a GLM with Poisson errors
model = glm(count ~ gender, poisson)
summary(model)
```

```{r}
#Check for overdispersion, and repeat the modelling using quasi-Poisson errors instead
model = glm(count ~ gender, quasipoisson)
summary(model)

#a GLM with negative binomial errors.
library(MASS)
model = glm.nb(count ~ gender)
summary(model)
```

## 15.8 A contingency table of intermediate complexity

Example:a three-dimensional table of count data from college records

```{r}
numbers = c(24, 30, 29, 41, 14, 31, 36, 35)
#The question is whether the relationship between gender and discipline varies between freshmen and sophomores(that is, we want to know the significance of the three-way interaction between year, discipline and gender)

dim(numbers) = c(2, 2, 2)
numbers
dimnames(numbers)[[3]] <- list('male', 'female')
dimnames(numbers)[[2]] <- list('arts', 'science')
dimnames(numbers)[[1]] <- list('freshman', 'sophomore')
#To see it as a flat tabel
ftable(numbers)
#The dimnames are the factor levels(e.g. male or female), not the names(e.g. gender) of the factors
#transfor into dataframe
as.data.frame.table(numbers)
```

```{r}
frame = as.data.frame.table(numbers)
names(frame) = c('year', 'discipline', 'gender', 'count')

#start by fitting a saturated model with eight parameters(the model generates the observed count exactly, so the deviance is zero and there are no degrees fo freedom)
model1 = glm(count ~ year * discipline * gender, poisson, data = frame)
model2 = update(model1, ~. - year:discipline:gender)
anova(model1, model2, test = 'Chi')
```

The interaciton is not significant(p = 0.079), indecating similar gender by discipline relationship in the two year group.

## 15.9 Schoener's lizards:A complex contingency table

- Example:

We are interested in whether lizards show any niche separation across various ecological factors and, in particular, whether there are any interacitons - for example, whether they show different habitat separation at different times of day:

```{r}
lizards = read.table('lizards.txt', header = T)
attach(lizards)
names(lizards)
#n, sun, height, perch, time, species
#begin by fitting a saturated model
model1 = glm(n ~ sun * height * perch * time * species, poisson)
model2 = update(model1, ~ . - sun:height:perch:time:species)
anova(model1, model2)
#When the change in deviance is so close to zero, R does not print a p_value

model3 = update(model2, ~. - sun:height:perch:species)
anova(model2, model3, test = 'Chi') # p = 0.0998

model4 = update(model2, ~. - sun:height:time:species)
anova(model2, model4, test = 'Chi') # p = 0.8019 

model5 = update(model2, ~. - sun:perch:time:species)
anova(model2, model5, test = 'Chi') # p = 0.667

model6 = update(model2, ~. - height:perch:time:species)
anova(model2, model6, test = 'Chi') # p = 0.1997
#none of the four-way interactions involving species need be retained

#next handle the three-way interaction involving species, speed this up by using *step*
#we do not allow *step* to remove any interactions that do not involve species(cause these are the essential nuisance varianles).We do this with the **lower** argument

model7 = step(model1, lower =~ sun*height*perch*time)

#as a result, *step* is too forgiving, and has left two of the four-way interactions involving species in the model.we take both of them out and start *step* off again with this simpler starting point.

model8 <- glm(n ~ sun * height * perch * time + (species + sun + height + perch + time) ^ 3, family = poisson)

model9 = step(model8, lower = ~sun*height*perch*time)

model10 = update(model9, ~. - sun:height:species)
anova(model9, model10, test = 'Chi') # p = 0.1362 drop it

#Now, the two-way interactions
model11 <- update(model10, ~. - sun:species)
model12 <- update(model10, ~. - height:species)
model13 <- update(model10, ~. - perch:species)
model14 <- update(model10, ~. - time:species)
anova(model10, model11, test = 'Chi') # p = 0.0056 retain
anova(model10, model12, test = 'Chi') # p = 2.634e-06 retain
anova(model10, model13, test = 'Chi') # p = 0.0003 retain
anova(model10, model14, test = 'Chi') # p = 0.003 retain

#a summary table of the counts
ftable(tapply(n, list(species, sun, height, perch, time), sum))

#The only remaining question for model sumplification is whether we need to keep all three levels for time of day.
tod = factor(1 + (time == 'Afternoon'))
model15 = update(model10, ~. - species:time + species:tod)
anova(model10, model15, test = 'Chi') # p = 0.3656
#keep time in the model but as a two-level factor
```

## 15.10 Plot methods for contingency tables

The R function called *assocplot* produces a Cohen-Friendly association plot indicating deviations from independence of rows and columns in a two-dimensional contingency table

```{r}
data(HairEyeColor)
(x = margin.table(HairEyeColor, c(1, 2)))
assocplot(x, main = 'Relation between hair and eye color')

#The same data plotted as a mosaic plot
mosaicplot(HairEyeColor, shade = T)
```

data ***UCBAdmissions*** describes the admissions policy of different departments at the University of California at Berkeley in relation to gender:

```{r}
data(UCBAdmissions)
head(UCBAdmissions)
str(UCBAdmissions)
#The object is a table with three dimensions:two levels of status, two levels of gender and six departments.
x = aperm(UCBAdmissions, c(2, 1, 3))
names(dimnames(x)) <- c('Sex', 'Admit?', 'Department')
ftable(x)

fourfoldplot(x, margin = 2)
```

Here we use `r gl` to generate factor levels for department, sex and admission, then fit a saturated contingency table model for the counts, x, We then use `r anova` with `r test = 'Chi'` to assess the significance of the three-way interaction.

```{r}
dept = gl(6, 4)
sex = gl(2, 1, 24)
admit = gl(2, 2, 24)
model1 = glm(as.vector(x) ~ dept*sex*admit, family = poisson)
model2 = update(model1, ~. - dept:sex:admit)
anova(model1, model2, test = 'Chi')
```

```{r}
#Another way to do the same test
admissions = as.data.frame(UCBAdmissions)
admissions
```

`r xtabs` creates a contingency tabel from cross-classifying factors using a formula interface

```{r}
xtabs(Freq ~ Gender + Dept, admissions)
```

```{r}
#*summary* option with *xtabs* calculates the total number of applications, and tests for independece
#The dot '.' means "fit all the expanatory variables"
summary(xtabs(Freq ~ ., admissions))
```

```{r}
#Turn the counts into proportions
xtabs(Freq ~ Admit + Dept + Gender, admissions)
#Tis table is three-dimensional, we need to specify three subscripts separated by two commas.
#*colSums* means column totals
females <- colSums(xtabs(Freq ~ Admit + Dept + Gender, admissions)[,,2])
females
#extract the numbers of femeles admitted to each department, which is the top row [1, ] of the lower half-table [,,2]
admitted_females <- xtabs(Freq ~ Admit + Dept + Gender, admissions)[,,2][1,]
#At last, the proportion of female applicats admitted by departments is:
(female_success <- admitted_females/females)
```

## 15.11 Graphics for count data:Spine plots and spinograms

```{r}
data = read.table('spino.txt', header = T)
attach(data)
head(data)
#specify the order of the factor levels
condition = factor(condition, c('much.worse', 'worse', 'no.change', 'better', 'much.better'))
treatment = factor(treatment, c('placebo', 'drug.A', 'drug.B'))
#use *spineplot* (either in (x, y) format or in (y ~ x) format)
spineplot(condition ~ ttreatment)

#counts for the data
table(condition, treatment)
as.data.frame.table(table(condition, treatment))

aggregate(data, data, length)
#*aggregate* leaves out rows from the dataframe when there were zero cases.so there are only 14 rows in the dataframe, not the 15 we want for doning the statistics

#start by fitting a saturated model
new = as.data.frame.table(table(condition, treatment))
model1 = glm(Freq ~ condition * treatment, poisson, data = new)
model2 = glm(Freq ~ condition + treatment, poisson, data = new)
anova(model1, model2, test = 'Chi') # p = 0.05661
```

In spinogram, the response is categorical but the explanatory variable is continuous.

```{r}
wasps <- read.table('para.txt', header = T)
attach(wasps)
head(wasps) 

table(density, fate)
spineplot(fate ~ density)
#if we want a smooth plot we use *cdplot* to draw a conditional density plot
cdplot(fate ~ density)

#model, logistic regression
model1 = glm(fate ~ density, binomial)
model2 = glm(fate ~ log(density), binomial)
AIC(model1, model2) # model1:144.3978, model2:143.0790
#so we pick the log-transformed explanatory variable
summary(model2)
#plot the data and the regression line
plot(as.numeric(fate) - 1 ~ log(density), pch = 16, ylab = 'parasitized')
xv = seq(0, 4.5, 0.01)
yv = -1 / (1 + 1 / exp(-4.023 + 1.3062 * xv))
lines(xv, yv)

#choose four bins in an example like this, averaging the four lowest density classes, and using the counts data from the three highest classes
den = c(3.75, 16, 32, 64)
pd = c(3/15, 5/16, 20/32, 52/64)
#redraw the axes leaving out the 0s and 1s (type = 'n') then add the logistic trend line and overlay the empirical frequencies as larger solid circles(cex = 2)
plot(as.numeric(fate) - 1 ~ log(density), pch = 16, ylab = 'parasitized', type = 'n')
lines(xv, yv)
points(log(den), pd, pch = 16, cex = 2)

#add error bars (eb) to show plus and minus one standard error of the estimated proportion
eb = sqrt(pd * (1 - pd) / den)
for (i in 1:4) {
  lines(c(log(den)[i], log(den)[i]), c(pd[i]+eb[i], pd[i]-eb[i]))
}
```

