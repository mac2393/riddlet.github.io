---
title: "Simulating outcomes"
layout: post
---

I recently described in this space how to generate correlated predictors. What I didn't show was how to take those correlated predictors and use them to generate some observed outcomes. Once we've learned how to do this, we can take this general idea and begin toying around with the structure of the data to see how it changes (or doesn't change) our estimation process. 

So, after having generated our predictors, we can then determine what the function is that will generate our observed values.

For instance, if we have three predictors, $$X_{1}$$, $$X_{2}$$, $$X_{3}$$, we could define a linear equation to generate an outcome:

####EQN 1:    
$$
Y = \beta_{0} + \beta_{1}X_{1} + \beta_{2}X_{2} + \beta_{3}X_{3} + \epsilon
$$

Changing the value of any $$\beta_{i}$$ will change the relationship between $$X_{i}$$ and Y. As before, $$\epsilon$$ is a 'noise' parameter that adds variability to the linear relationship.

As an example, let's set:

$$
\beta_{0} = 1,  \\
\beta_{1} = .5,  \\
\beta_{2} = 2,  \\
\beta_{3} = -1.5  \\
\epsilon \sim N(0, 5) \\
$$

Thus, our linear equation for Y is:

####EQN 2:  
$$
Y = 1 + .5X_{1} + 2X_{2} - 1.5X_{3} + \epsilon
$$

And recall that we can define the relationships between our predictors with a variance-covariance matrix as follows:

$$\left[\begin{array}
{rrr}
Var_{X_{1}}   & Cov_{X_{1},X_{2}} & Cov_{X_{1},X_{3}} \\
Cov_{X_{1},X_{2}} & Var_{X_{2}}   & Cov_{X_{2},X_{3}} \\
Cov_{X_{1},X_{3}}  & Cov_{X_{2},X_{3}}  & Var_{X_{3}}
\end{array}\right]
$$

As last time, we will set $$X_{1}$$ and $$X_{2}$$ to not be correlated, $$X_{1}$$ and $$X_{3}$$ will be slightly correlated, and $$X_{2}$$ and $$X_{3}$$ to have a strong correlation.


{% highlight r %}
library(compiler)
library(corpcor)
library(MASS)

set.seed(42)
n<-100

#correlations between predictors
VCV <- matrix(c(1, 0, .2,
                0, 1, .7,
                .2, .7, 1), nrow=3, ncol=3)
rownames(VCV) <- c('X1', 'X2', 'X3')
VCV
{% endhighlight %}



{% highlight text %}
##    [,1] [,2] [,3]
## X1  1.0  0.0  0.2
## X2  0.0  1.0  0.7
## X3  0.2  0.7  1.0
{% endhighlight %}



{% highlight r %}
#Generate predictors
dat <- as.data.frame(mvrnorm(n = n, mu = rep(0, 3), Sigma = VCV))
{% endhighlight %}

Now we need to indicate the parameters for the linear function described above. We can store these parameters in a vector, and then multiply the values in that vector by the observed values for $$X_{1}$$, $$X_{2}$$, and $$X_{3}$$, and use the `rnorm` function to draw random observations from a normal distribution defined by the Equation 2 above.


{% highlight r %}
params <- c(.5, 2, -1.5)
y <- rnorm(n=dim(dat)[1], mean=(1 + as.matrix(dat) %*% params), 5)
{% endhighlight %}

Let's break that second line down a little bit. The function `rnorm` generates $$n$$ random numbers from a normal distribution described by a mean and a standard deviation. To use it, we need to give it the mean, the standard deviation, and $$n$$.

We set $$n$$ to be the length of our predictors dataframe, `dim(dat)[1]`. For the mean, we convert that same dataframe to a matrix `as.matrix(dat)`, and use the matrix multiplication operator `%*%` to multiply this by our parameter vector. You'll notice we also add a constant of 1, because we specified that the intercept should be 1 (see Equation 2). In other words, the code inside the argument for the mean is the r translation of the linear portion of Equation 2, $$1 + .5X_{1} + 2X_{2} - 1.5X_{3}$$. We also stated that the noise would come from a normal distribution, centered on zero with a standard deviation of five. We added this in the sd argument of `rnorm`, and appears as a 5 at the very end.

To check how well the simulated data fits the function that generated it, we can fit a linear model. We should recover something close to out parameters indicated in Equation 2.


{% highlight r %}
summary(lm(y~dat$X1+dat$X2+dat$X3))
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = y ~ dat$X1 + dat$X2 + dat$X3)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.9720 -2.9335 -0.5192  3.0941 11.6400 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)   
## (Intercept)   1.1575     0.4457   2.597  0.01088 * 
## dat$X1        0.4409     0.4949   0.891  0.37523   
## dat$X2        1.6325     0.6124   2.666  0.00902 **
## dat$X3       -1.4410     0.6769  -2.129  0.03581 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.434 on 96 degrees of freedom
## Multiple R-squared:  0.07374,	Adjusted R-squared:  0.04479 
## F-statistic: 2.547 on 3 and 96 DF,  p-value: 0.06042
{% endhighlight %}

Pretty close!

Having created a set of steps that will simulate data, we can now use these steps to investigate if there is any systematic problem when we, for example, change the degree to which two predictors are correlated.

