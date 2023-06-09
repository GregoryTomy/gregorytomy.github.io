---
layout: single
title:  "Metropolis Hastings Algorithm"
collection: bayesian statistics
classes: wide
toc: true
toc_icon: "cog"
---

Sampling from complex probability distributions can be a challenging task, especially when they are defined on high-dimensional spaces or have multiple peaks. In this blog post, I will introduce the Metropolis-Hastings algorithm, a widely used method for generating samples from arbitrary probability distributions. To help explain how the algorithm works, I will use a toy example involving a multimodal normal distribution. We will discuss the key ideas behind the algorithm and show how to use it to generate samples from the target distribution. By the end of this post, you will have a solid understanding of the Metropolis-Hastings algorithm and its practical applications in statistics and machine learning.

# The target distribution
In Bayesian statistics and related fields, we often encounter probability distributions that we want to analyze or sample from. Target distributions can be simple or complex, and they can have various shapes and properties. Here my target distribution is a multimodal normal distribution, which has two peaks located at -2 and 2 respectively. 

$$
f(x) \sim 0.5\cdot N(-2, 1) + 0.5\cdot N(2, 1)
$$

![Alt text](/images/metrohastings/target.png)

# The proposal distribution. 
Next, we need a way to generate candidate samples that we can compare to the current sample and potentially accept as the next sample. The way we generate these candidate samples is through a proposal distribution, which is a simpler distribution that we can sample from more easily. The proposal distribution is usually chosen to be symmetric, meaning that it has the same probability density when we move from the current sample to a candidate sample as when we move from the candidate sample back to the current sample. This symmetry is important for ensuring that the algorithm satisfies a detailed balance condition and converges to the target distribution. Note, the *Hastings* part of Metropolis-Hastings is the generalizatrion of this idea wherein we relax the symmetry condition. I will stick with a normal distribution in this post.

$$
q(y|x) \sim N(x, 0.4)
$$

# The algorithm
So where are we? We have a traget to sample from a proposal to help us with it. With a lot of these algorithms, we need to provide an intial point to start at. Let that be $x_0 = 0.6$. Next, we sample from the proposal distribution $y_0 \sim q(y|x_0)$. Let this be $y_0 = 1.3$. 

![Alt text](/images/metrohastings/mh_algo2.png)

Now, the big question is whether we move to this new sample point or stay at our current point. To answer this, we use the acceptance probability which is given by:

$$
\begin{align*}
\alpha &= \min \left(1, \frac{f(y)}{f(x)}\right) \\
  &= \min \left(1, \frac{f(1.3)}{f(0.6)}\right) \\
  &= \min \left(1, \frac{0.157}{0.082}\right) \\
  &= 1
\end{align*}
$$

Thus, we move to the proposed point $x_1 = 1.3$. In practice, this condition is implemented by sampling from a uniform distribution $u \sim Unif(0, 1)$ and accepting when, 

$$
u <= \frac{f(y)}{f(x)}
$$

We repeat this a "large number of times". To understand why this works, we need to know a little bit about Markov chain theory - irreducibility, aperiodicity, recurrence, stationary distributions, detailed balance equations. For the purposes of this post, rest assured that when certain conditions are met, the simulated sequence converges to the target distribution. 

# Implementation in R

{% highlight r linenos %}
library(tidyverse)

target <- function(x){
  0.5 * dnorm(x, mean = -2, sd = 1) + 0.5 * dnorm(x, mean = 2, sd = 1)
}

x_vals <- seq(-6, 6, 0.1)

metropolis <- function(proposal_sd, iterations){
  x <- rep(0, iterations)
  
  for (t in 2:iterations) {
    # propose new x
    y <- rnorm(1, mean = x[t - 1], sd = proposal_sd)
    
    # compute acceptance probability
    alpha <- target(y) / target(x[t - 1])
    u <- runif(1)
    
    if (u <= alpha) {
      x[t] = y
    } else{
      x[t] = x[t - 1]
    }
  }
  
  return(x)
}

set.seed(123)
metro_seq <- metropolis(0.4, 100000)
df <- data.frame(x = metro_seq)

ggplot(df, aes(x)) +
  geom_histogram(aes(y = ..density..), color = "white", bins = 100, fill = "lightblue" ) +
  geom_density() +
  theme_fivethirtyeight() +
  ggtitle("Metropolis Sampling")
{% endhighlight %}

![Alt text](/images/metrohastings/mh_sampling.png)

Groovy. 

Those of you paying attention would have noticed I implemented the original Metropolis algorithm and not the Metropoilis-Hastings. As mentioned earlier, Metropolis-Hastigns is the generalization of the Metropolis algorithm, wherein the acceptance probability is a ratio of ratios,

$$
\alpha = \min \left(1, \frac{f(y)\cdot q(x|y)}{f(x)\cdot q(y|x)}\right)
$$

In the symmetric case, the new terms equal to 1 and we get the Metropolis algorithm. 

Cheers!