---
title: STAT 506 - Advanced Regression Analysis
framework: bootplus
layout: poststripped
mode: selfcontained
highlighter: prettify
hitheme    : twitter-bootstrap
widgets    : [mathjax]
navbar:
  title: Kevin Ferris | Final Exam
lead : >
  April 30, 2014
assets:
  css:
  - "http://fonts.googleapis.com/css?family=Raleway:300"
  - "http://fonts.googleapis.com/css?family=Oxygen"
  jshead:
  - "./lodash.js"
  - "./dygraph-combined.js"
--- 
### Number 1 Part A














Poverty rate is measured at the school level.

The number of classes the teacher has taken, the teacher's mathematical knowledge, and the number of years of experience of the teacher are measured at the class level.

SES, math score in kindergarten, sex, and minority status are measured on the individual level.

---
### Number 1 Part B

The first thing I did with these data was look at the missingness patterns because some of the scores of teacher's mathematics knowledge were missing.  On the left is the distribution of the increase in math score for both the missing and non-missing data.  The two distributions look to be fairly similar; the only noticeable difference is that all the students whose math scores decreased by 100 are from the missing data.  
<img src="assets/fig/missingPlots.png" title="plot of chunk missingPlots" alt="plot of chunk missingPlots" style="display: block; margin: auto;" />

For most of the explanatory variables, there was not much of a difference in the distribution between the missing and non-missing data.  However, on the right in the plot above it looks like the distribution of teacher's math preparation is plotted, and it looks like slightly more inexperienced teacher's are likely to have missing data.  The same pattern held for poverty rate --- poorer schools were slightly more likely to be missing.  This means that we will need to be very careful about the inferences we make for these data because it is not safe to assume that the data are missing at random. Even though we have random selection, we probably shouldn't infer the results beyond these data nor should we make causal inferences.

Because we are mostly interested in exploring the relationships between the variables and not in finding a best fitting model, I decided to use Gelman's model selection strategy: keep all variables in the model unless they have a sign we don't expect and are not significant.  With that in mind, I should lay out the expected sign for each variable:

|Variable         |Level       |Expected.Sign        |
|:----------------|:-----------|:--------------------|
|sex              |individual  |positive for female  |
|minortiy         |individual  |negative             |
|SES              |individual  |positive             |
|mathPrescore     |individual  |positive             |
|teachrXper       |class       |positive             |
|teachrMath       |class       |positive             |
|teacherMathprep  |class       |positive             |
|povertyRate      |school      |negative             |


Next, I wanted to consider what interactions would be allowed in the model.  I decided to only consider interactions of categorical and quantitative variables at the individual level.  I then made plots like the one below to assess interactions.  To make this plot, 1000 bootstrap replicates of the data were made.  A loess smoother was then fit to each replicate.  Then, the density of the fitted values at 200 equal spaced intervals of the mathprescore variable was calculated.  The intensity of the color is proportional to the density.  So, in areas where the color is more intense, we are more certain of the location of the true mean of math gain.


<iframe width = "723" height = "406" src = "vwSex_prescore.png"></iframe>

In this plot, it really appears that there is no difference in the relationship between men and women so I didn't include an interaction between pre-score and sex in the model.  There are a couple of notable features in the plot though.  It looks as though pre-score is truncated at 300 and 650.  The relationship looks to be fairly linear everywhere else, but, because of this truncation, things may not be entirely linear in the tails.  We'll have to check the residuals to make sure that things are linear.

The plot below looks for an interaction between pre-score and minority status.  Here, there does appear to be an interaction present.  It seems that the slope is more negative for minorities than for non-minorities so I will include an interaction term for minority and pre-score in the model.


<iframe width = "723" height = "406" src = "vwMinority_prescore.png"></iframe>
Neither minority status nor sex seemed to have a strong interaction with SES when I made these plots, so I did not include those interactions.

I included random intercepts for both school and class-within-school only.  I considered random intercepts, but they did not seem to provide any improvement to the model (measured using AICs) so I decided not to include any of them.  The final model I used is:

$Gain_i = \beta_0 + \beta_1 * SES_i + \beta_2 * pre_i + \beta_3 * I(minority_i == 1) + \beta_4 * I(sex_i == M) + \\ \beta_5 * I(minority_i == 1) * pre_i + \beta_6 * teacherPrep_{j[i]} + \\ \beta_7 * teacherExper_{j[i]} + \beta_8 * teacherMath_{j[i]} + \beta_9 * povertyRate_{k[i]} + \\ 
b_{1,j[i]} + b_{2,k[i]} +  \epsilon_i$

In this model $i$ denotes the student, $j[i]$ denotes the class for the $i^{th}$ student, and $k[i]$ denotes the school for the $i^{th}$ student.  $b_2$ is a random intercept for the $k^{th}$ school, $b_1$ is a random effect for the $j^{th}$ class in the $k^{th}$ school, and $\epsilon_i$ is a random error term for the $i^{th}$ student.  I centered all quantitative explanatory variables in this model.





Caterpillar plots for the fixed effects from the model are provided below.  These variables are all on different scales (in hindsight, I probably should have standardized everything) so I won't interpret the magnitude of the effect; instead, I'll just report on which variables have a positive or negative effect.  

<img src="assets/fig/catPlots.png" title="plot of chunk catPlots" alt="plot of chunk catPlots" style="display: block; margin: auto;" />


It appears as though there is not much evidence to suggest that sex, teacher's experience, teacher's math knowledge, or school's poverty rate,  have a strong positive or negative effect on student's math improvement.  For each of these terms, there is just too much uncertainty to say that they have a positive or negative effect on math improvement.  There may be some evidence that teacher's math background has a positive effect on student's improvement.

SES does tend to have a positive effect on student's math improvements.  Interestingly, the pre-score for both minorities and non-minorities has a negative relationship with math gain.  This could mean that student's who are already good don't tend to get much better.  Alternatively, it could me that we are seeing a strong regression to the mean in these data.  This would suggest that student's who did very well in kindergarten were just lucky and that luck dissipated the next year.

There is extremely strong evidence that the intercept adjustment for minorities is pretty negative.  This suggests that, on average, minorities tend to have smaller improvements than non-minorities.  I did not include a caterpillar interval for the interaction term because it is fairly small.  But there is some evidence that the slope on pre-score is more negative for minorities; this would suggest that minority students who were initially strong did not improve as much as non-minorities who were initially strong.

The variances of both the class and school effect are both one-tenth the variance of the residual error.  This suggests that, relative to the noise is the data, the variability between schools and between classes within a school is not very large.

As we noted earlier, we should look at the pattern of the residuals vs pre-score to make sure that there are no violations of linearity.  The plot is located below, and there do not appear to be any patterns so this assumption is probably okay.
<img src="assets/fig/residVsPre.png" title="plot of chunk residVsPre" alt="plot of chunk residVsPre" style="display: block; margin: auto;" />


The residuals vs fitted values plot is located below.  There do not appear to be any noticeable patterns so the constant variance assumption is probably okay.
<img src="assets/fig/residVsFitted.png" title="plot of chunk residVsFitted" alt="plot of chunk residVsFitted" style="display: block; margin: auto;" />


In the model, I assumed that all the random terms were normally distributed.  I can check these assumptions by making normal quantiles plots for each term.  These plots are located below.  There are a few outlying points in the tail of the residuals, but we have a large data set so this is not too uncommon.  Otherwise, these plots all look pretty good so the normality assumptions are probably okay.
<img src="assets/fig/normalQQ.png" title="plot of chunk normalQQ" alt="plot of chunk normalQQ" style="display: block; margin: auto;" />



To conclude, it looks as though the only strong predictors of math improvement are SES, pre-score, and minority status.  For the other variables, there is just too much uncertainty or the effect is too small to say that there is much of a relationship.  As noted before, we should be careful with our inferences for these data.  These results only hold for the 1081 students for whom we don't have missing data, and we can't say that any of the predictors cause the increase/decrease in the math improvement that we observed.

---
### Number 1 Part C

I fit the same model in JAGS using non-informative parameters.  I ran four chains, discarding the first 2000 iterations.  I then used a thinning interval of 8 for the next 2000 iterations.  This resulted in 1000 draws of the parameters.  Convergence diagnostics show that the posterior distributions had converged to stationary distributions.  Gelman's $\hat{R}$ was less than 1.02 for all the parameters and all the parameters had effective sample sizes of at least 150.  Trace plots of the parameters are provided below, and they show that the chains are mixing well.

<img src="assets/fig/jagsTracePlot1.png" title="plot of chunk jagsTracePlot" alt="plot of chunk jagsTracePlot" style="display: block; margin: auto;" /><img src="assets/fig/jagsTracePlot2.png" title="plot of chunk jagsTracePlot" alt="plot of chunk jagsTracePlot" style="display: block; margin: auto;" />



A summary of the parameters is provided in the table below.  These are now credible intervals obtained from the posterior distribution of each parameter.  This means that they have a slightly different interpretation.  For example, if we wanted to interpret the coefficient on SES, we would say that there is a 95% chance that a 1 unit increase in SES is associated with between a 2.72 and 7.79 true mean gain in math score after accounting for all other variables in the model.

|id           |      2.5%|       25%|      50%|      75%|    97.5%|
|:------------|---------:|---------:|--------:|--------:|--------:|
|preScore     |   -0.4938|   -0.4456|  -0.4200|  -0.3934|  -0.3402|
|SES          |    2.7207|    4.4301|   5.2732|   6.1785|   7.7905|
|minority     |  -11.6205|   -8.2805|  -6.7592|  -5.0921|  -1.7422|
|male         |   -1.7564|    0.3108|   1.4140|   2.6498|   4.8224|
|Interaction  |   -0.1808|   -0.1177|  -0.0846|  -0.0532|   0.0077|
|TeachPrep    |   -1.2146|    0.2818|   1.0277|   1.7920|   3.2877|
|TeachXper    |   -0.1853|   -0.0330|   0.0381|   0.1150|   0.2736|
|TeachMath    |   -0.1298|    1.0597|   1.8720|   2.7351|   4.0489|
|Poverty      |  -26.1502|  -13.7983|  -7.5987|  -0.8332|  11.8967|
|School SD    |    5.3766|    7.7167|   8.8391|   9.8604|  12.1103|
|Class SD     |    5.3254|    7.6723|   8.9047|  10.0291|  12.1741|
|Residual SD  |   25.5208|   26.3837|  26.8501|  27.3176|  28.1900|




---
### Number 2

For this problem, we are given information of the math scores of 500 students in grades 6 through 9.  The distribution of score in each grade is plotted below.  It looks as though there is a linear relationship between score and grade so I treated grade as a quantitative variable.

<img src="assets/fig/scoreBoxPlots.png" title="plot of chunk scoreBoxPlots" alt="plot of chunk scoreBoxPlots" style="display: block; margin: auto;" />


The trajectory of each child over time is plotted below.  There appears to be a lot of variability in the intercepts of each child.  Though there is not as much variability in the slopes, it still looks like it might be worth allowing the slopes to vary by student.

<img src="assets/fig/scoreLinePlot.png" title="plot of chunk scoreLinePlot" alt="plot of chunk scoreLinePlot" style="display: block; margin: auto;" />


The model used was $y_i = \beta_0 + \beta_1 * grade_i + \beta_{2,j[i]} + \beta_{3,j[i]} * grade_i + \epsilon_i$.  Here, $y_i$ is the math score for the $i^{th}$ observation.  $\beta_{2,j[i]}$ and $\beta_{3,j[i]}$ follow a multivariate normal distribution with a mean vector of 0 and a 2x2 variance-covariance matrix.  This matrix has $\sigma_{\beta2}^2$ and $\sigma_{\beta3}^2$ on the diagonal and $\sigma_{\beta2} * \sigma_{\beta3} * \rho$ on the off-diagonal.  The errors are assumed to be independent of the random effects and to be $\sim N(0, \sigma_y^2)$.




4 chains were used for the MCMC sampling, and the first 1000 iterations were discarded as a warm-up.  A thinning interval of 4 was used for the next 1000 observations.  The diagnostics for the model look pretty good.  Gelman's $\hat{R}$ are less than 1.02 for all the terms and the multivariate $\hat{R}$ is 1.02.  Traceplots for each term are provided below and show that the chains appear to be mixing well.  Overall, it looks like things have converged.

<img src="assets/fig/num2trace1.png" title="plot of chunk num2trace" alt="plot of chunk num2trace" style="display: block; margin: auto;" /><img src="assets/fig/num2trace2.png" title="plot of chunk num2trace" alt="plot of chunk num2trace" style="display: block; margin: auto;" />


Summaries of the posterior distribution for each of the terms in the model are provided below.  It does appear that math scores do tend to increase over time.  Based on this model and data, there is a 95% chance that moving from one grade to the next is associated with between a 0.97 and 1.07 increase in the true mean math score.  

|id      |      2.5%|       25%|      50%|      75%|    97.5%|
|:-------|---------:|---------:|--------:|--------:|--------:|
|b0      |   56.4652|   59.3700|  61.0144|  62.5768|  65.1235|
|b1      |   -0.4938|   -0.4456|  -0.4200|  -0.3934|  -0.3402|
|b2      |    2.7207|    4.4301|   5.2732|   6.1785|   7.7905|
|sig.b2  |  -11.6205|   -8.2805|  -6.7592|  -5.0921|  -1.7422|
|sig.b3  |   -1.7564|    0.3108|   1.4140|   2.6498|   4.8224|
|b5      |   -0.1808|   -0.1177|  -0.0846|  -0.0532|   0.0077|
|b6      |   -1.2146|    0.2818|   1.0277|   1.7920|   3.2877|
|b7      |   -0.1853|   -0.0330|   0.0381|   0.1150|   0.2736|
|b8      |   -0.1298|    1.0597|   1.8720|   2.7351|   4.0489|
|b9      |  -26.1502|  -13.7983|  -7.5987|  -0.8332|  11.8967|
|sig.a0  |    5.3766|    7.7167|   8.8391|   9.8604|  12.1103|
|sig.a1  |    5.3254|    7.6723|   8.9047|  10.0291|  12.1741|
|sig.y   |   25.5208|   26.3837|  26.8501|  27.3176|  28.1900|




---
### R Code












