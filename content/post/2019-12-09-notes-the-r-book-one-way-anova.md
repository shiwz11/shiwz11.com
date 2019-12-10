---
title: NOTES_The_R_Book_One-way ANOVA
author: shiwz11
date: '2019-12-09'
slug: notes-the-r-book-one-way-anova
categories:
  - R
tags:
  - R Markdown
---

# Chapter 11 Analysis of variance

Nowadays the aim of ANOVA in R is to estimate means and standard errors of differences between means.

Comparing two means by a t test involved calculating the difference between the two means, dividing by the standard error of the differece.

The means are said to be significantly differnet when the calculated value of t is larger than the critical value.

For larger samples(n > 30) a useful rule of thumb is that a t value greater than 2 is significant.

## 11.1 One-way ANOVA

- Def:

SST:the sum of the squares of the differences between the data, y, and the overall mean.

SSA:the treatment sum of squares

$\bar{\bar{y}}$ is the the overall mean

$$
\begin{equation}
\begin{aligned}
SST &= \sum{(y - \bar{\bar{y}})^2} \\
SSE &= \sum{(y_A - \bar{y}_A)^2} + \sum{(y_B - \bar{y}_B)^2} \\
SSA &= SST - SSE
\end{aligned}
\end{equation}
$$

When differences between means are significant, then SSA will be large relative to SSE.When differences between menas are not sifnificant, then SSA will be small relative to SSE.

We convert the sums of squares into variances by dividing by their degrees of freedom

- About the degree of freedom:

In general, \emph{k} levels of any factor and hence \emph{k - 1} d.f. for treatments.

If each factor level were replicated \emph{n} times, then there would be \emph{n - 1} d.f. for error within each      level(lose one degree of freedom for each individual treatment mean estimated from the data)

\emph{k} levels, so there would be \emph{k(n - 1)} d.f. for error in the whole experiment.

total d.f. is \emph{kn - 1} cause there is \emph{kn} numbers totally
  
The individual component degree of freedom add up to the correct total:

$$
kn - 1 = k - 1 + k(n - 1) = k - 1 + kn - k
$$


\begin{table}
\caption{The divisions for turning the sums of squares into variances are conveniently carried out in an ANOVA
table:}
\begin{tabular*}
\hline
Source & SS & d.f. & MS & F & Critical F \\
\hline
Treatment & SSA & k - 1 & MSA = \frac{SSA}{K - 1} & F = \frac{MSA}{s^2} & qf(0.95, k - 1, k(n - 1)) \\
Error & SSE & k(n - 1) & s^2 = \frac{SSE}{k(n - 1)}  \\
Total & SST & kn - 1
\hline
\end{tabular*}
\end{table}

### 11.1.1 Calculation in one-way ANOVA

$$
\begin{aligned}
SST &= \sum{y^2} - \frac{(\sum y)^2}{kn} \\
SSA &= \frac{\sum C^2}{n} - \frac{(\sum y)^2}{kn} \\
\text{C:the treatment total, the sum of all the n replicates within a given level} \\
SSE &= SST - SSA
\end{aligned} 
$$

### 11.1.2 Assumptions of ANOVA

1.ramdom sampling 

2.equal variances

3.independence of errors

4.normal distribution of errors

5.additivity of treatment effects

An example:whether soil type significantly affects crops yield?

```{r}
results = read.table('yields.txt', header = T)
attach(results)
names(results)
#'sand' 'clay' 'loam'
sapply(list(sand, clay, loam), mean)

frame = stack(results)
#change the default names of the frame
names(frame = c('yield', 'soil'))
attach(frame)
head(frame)
#Check for constancy of variance across the three soil types 
tapply(yield, soil, var)
#Use the Fliger-Killeen test of homogeneity of variances
fligner.test(y ~ soil)
#use *bartlett.test(y ~ soil)* to test for non-normality
plot(yield ~ soil, col = 'green')
```

In the simplest case, we partition the total variation into just two components, explained variation and unexplained variation

SSA:Explained variantion is called the treatment sum of squares

SSE:Unexplained variation is called the error sum of squares(also known as the residual sum of squares)

```{r}
#obtain the total sum of squares bu finding the differences between the data and the overall mean
sum((yield - mean(yield))^2)
#Now the SSE
sand - mean(sand)
clay - mean(clay)
loam - mean(loam)
#we need the sum of squares of these differences
sum((sand - mean(sand))^2)
sum((clay - mean(clay))^2)
sum((loam - mean(loam))^2)
#Finally the SSE
sum(sapply(list(sand, clay, loam), function (x) {sum((x - mean(x))^2)}))
#SSA, is the amount of the variation in yield that is explained by differences between the treatment means.
SSA = SSY - SSE
```

All the work above, R can do it in a single line:

```{r}
summary(aov(yield ~ soil))
```

Next, check the assumption of the aov model

```{r}
par(mfrow = c(2, 2))
plot(aov(yield ~ soil))
```

### 11.1.4 Effect sizes

The best way to view the effect sizes graphically is to use **plot.design** (which takes a formula rather than a model object), but our example with just one factor is perhaps too simple to get full value from this(*plot.design(yield~siol)*).To see the effect sizes in tabular form use **model.tables** which takes a model obj as its argument :

```{r}
model = aov(yield~soil)
model.tables(model, se = T)
#The effects are shown as depatures from the overall mean
```

```{r}
#if we specify 'means'
model.tables(model, 'means', se = T)
#Now the three means are printed (rather than the effects) and the standard error of the difference of means is given (that is what we need for doing a t test to compare any two means)
```

Another way of looking at effect sizes is to use the `r summary.lm()` rather than `r summary.aov`

```{r}
summary.lm(model)
#In this case 'clay' is selected as the becnmark because the this factor-level name comes first in the alphabet.So the intercept is the mean yield for clay.Slopes estimated for aov are  differences between means.
```

$$
\begin{aligned}
\text{The standard error of a mean:} \\
se_{mean} &= \sqrt{\frac{s^2}{n}} \\
\text{the standard error of the difference between two means:} \\
se_{diff} = \sqrt{2 \frac{s^2}{n}}
\end{aligned}
$$

*so there it is, That is how analysis of variance works.When the means aresinificantly different, then the sum of squares computed from the individual treatmen means will be significantly smaller than the sum of squares computed from the overall mean.We judge the significance of the difference between the two sums of squares using analysis of variance*

### 11.1.5 Plots for interpreting one-way ANOVA

Two ways:

1.box-and-whisker plots;

2.barplots with error bars

```{r}
#example
comp = read.table('competition.txt', header = T)
attach(comp)
names(comp)
#'biomass' 'clipping'
plot(clipping, biomass, xlab = 'Competition treatment', ylab = 'Biomass', col = 'yellow')
#by default we get the box-and-whisker plot for five levels in one factor
```

R do not have a built-in function called error.bar to draw barplots with error bars

we write one of our own

```{r}
error_bars = function(yv, z, nn) {
  xv = barplot(yv, ylim = c(0, (max(yv) + max(z))), 
               col = 'green', names = nn, ylab = deparse(substitute(yv)))
  for (i in 1:length(xv)) {
    arrows(xv[i], yv[i] + z[i], xv[i], yv[i] - z[i], angle = 90, code = 3, length = .15)
  }
}
```

we use the standard error of a mean based on the pooled error variance from the ANOVA as the value(z)

```{r}
model = aov(biomas ~ clipping)
summary(model)
#next we need to know how many numbers we used in the calculation of each of the five means
table(clipping)

se = rep(28.75, 5)
#label the bars
labels = levels(clipping)
#work out the five mean values which will be the heights of the bars, and save them as a vector called ybar:
ybar = tapply(biomass, clipping, mean)
#At last draw thw barplot
error_bars(ybar, se, labels)
#when two error bars overlap this implies that the two means are not significantly different.
```

An alternative fraphical method is to use 95% confidence intervals for the lengths of the bars, rather than standard error of means.

```{r}
error_bars(ybar, qt(.975, 5) * se, labels)
```

Compared with the two error bars used above,a better choice is the least significant difference(*LSD*) bars.

The formula for Student's t test:

$$t = \frac{\text{a difference}}{\text{standard error of the difference}}$$

rearrange this formula to find the smallest difference that we would regard as being significant, We call this *the least sifnificant difference*:

$$
\mathrm{LSD}=\mathrm{qt}(0.975, \mathrm{df}) \times \text { standard error of a difference } \approx 2 \times s e_{\mathrm{diff}}
$$
```{r}
lsd = qt(.975, 10) * sqrt(2* 4961 / 6)
lsdbars = rep(lsd, 5) / 2
error_bars(ybar, lsdbars, labels)
#A better option might be to use box-and-whisker plots with the notch = T option to indicate significance
```

