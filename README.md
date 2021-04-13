# STA 323 & 523 :: Homework 06

## Introduction

> We should forget about small efficiencies, say about 97% of the time:
premature optimization is the root of all evil. Yet we should not pass up our
opportunities in that critical 3%. A good programmer will not be lulled into
complacency by such reasoning, he will be wise to look carefully at the critical
code; but only after that code has been identified.<br/><br/>
*Donald Knuth*

## Tasks

Choose to do two or three tasks. If you choose two tasks, each will be worth
15 points. If you do all three tasks, each will be worth 10 points.

#### Task 1 - Benchmarking

Use `bench::mark()` to evaluate the performance of the following sets of
functions. Provide a brief written summary of your results along with a
visualization or table to communicate your findings.

1. Compare `apply(X, 1, sum)` and `rowSums(X)`, where `X` is a p x p random
   normal matrix. Consider values of p = 10, 100, 1,000, and 10,000. Use 10
   iterations in your performance evaluation.

2. Compare `any(x == 55)` and `55 %in% x`, where `x` is a random integer
   vector of length n. Consider values of n = 10, 100, 1,000, and 10,000. Use
   10,000 iterations in your performance evaluation. *Hint:* `sample()`.

3. Compare `t(X) %*% X` and `crossprod(X)`, where `X` is a p x p random
   normal matrix. Consider values of p = 10, 100, 1,000. Use the
   `bench::mark()` default arguments. *Note:* use `crossprod()` from package
   `Matrix`.

4. Compare `cummean(x)` from package `cumstats` with a function you develop
   to also compute the cumulative mean for a vector `x`. Let `x` be a random
   integer vector of length n. Consider values of n = 10, 100, 1,000, 10,000,
   and 100,000. Use 10,000 iterations in your performance evaluation.
   *Hint:* `sample()`.

#### Task 2 - Parallelization

For this task you may utilize any of the parallelization functions we have
discussed.

Multicollinearity can cause serious problems in regression analysis. You are
looking for an effective way to communicate this to a client and decide to
perform a simulation using linear regression and two predictors.
In your simulation, you intend to investigate the standard errors, estimates,
and p-values as they relate to various levels of correlation between
predictors and various levels of model noise. Use parallelization to speed up
your simulation. Communicate your findings on multicollinearity in a brief
write-up.

Below is some starter code to help you think about how to set this up.
Be sure to do enough replications in your simulation for various
parameter combinations of `p` and `signma`.

```r
# parameters
n <- 100 # sample size
sigma <- 1 # error variance
p <- 0.5 # dependency parameter

# predictors and random error
X1 = rnorm(n)
X2 = rnorm(n)
e = rnorm(n, sd = sigma)

# tibble where x_1 and x_2 are independent
df_1 <- tibble(
  x_1 = X1,
  x_2 = X2,
  y   = -3 + 10 * x_1 - 2 * x_2 + e # true model form
)

# tibble where x_1 and x_2 are correlated (based on p)
df_2 <- tibble(
  x_1 = X1,
  x_2 = p * X1 + (1 - p) * X2,
  y   = -3 + 10 * x_1 - 2 * x_2 + e # true model form
)

lm(y ~ ., data = df_1) %>% broom::tidy()
lm(y ~ ., data = df_2) %>% broom::tidy()
```

*You are not required to use this code and may work with matrices instead.*

#### Task 3 - Improving performance

Assume you want to analyze some gene expression data based on `p` = 100,000
genes and `n` = 70 individuals. Assume the individuals belong to one of two
groups.

```r
p <- 100000
n <- 70

X <- matrix(rnorm(p * n, 12, 4), nrow = p, ncol = n) # p x n gene expression matrix
group_levels <- rep(0:1, each = n / 2)
```

For each variable (gene) you want to perform a two-sample t-test and only
retain the test statistic. Do not assume equal variances.
A `for`-loop with `t.test()` can accomplish this.

```r
system.time({
  t_stat <- vector(mode = "double", length = nrow(X))
  for (i in seq(nrow(X))) {
    t_stat[i] <- t.test(X[i, ] ~ group_levels)$statistic
  }
})
```

```
   user  system elapsed
 64.143   1.383  65.578
```

Improve the performance of this process without parallelization.
Profile your code after your initial attempt and briefly discuss the results.
After you arrive at your final solution, provide some further discussion on
the improvements you made since your first attempt.

A correct solution with a run time of less than one second is guaranteed to
receive full credit. Think about the linear model example provided in the
lecture slides.

## Essential details

### Deadline and submission

**The deadline to submit Homework 06 is Wednesday, April 21 at 11:59pm EST.**
Only your final commit and code in the main branch will be graded. However,
you should continue to use branches to efficiently work together as a team.

### Help

- Post your questions in the #hw6 channel on Slack. Explain your error / problem
  in as much detail as possible or give a reproducible example that generates
  the same error. Make use of the code snippet option available in Slack. You
  may also send a direct message to the instructor or TAs.

- Visit the instructor or TAs in Zoom office hours.

- The instructor and TAs will not answer any questions about this assignment
 	within six hours of the deadline.

### Academic integrity

This is a team assignment. You may communicate with other teams in the
course. As a reminder, any code you use directly or as inspiration must
be cited.

To uphold the Duke Community Standard:

  - I will not lie, cheat, or steal in my academic endeavors;
  - I will conduct myself honorably in all my endeavors; and
  - I will act if the Standard is compromised.

Duke University is a community dedicated to scholarship, leadership, and
service and to the principles of honesty, fairness, respect, and
accountability. Citizens of this community commit to reflect upon and
uphold these principles in all academic and non-academic endeavors, and
to protect and promote a culture of integrity. Cheating on exams and
quizzes, plagiarism on homework assignments and projects, lying about an
illness or absence and other forms of academic dishonesty are a breach
of trust with classmates and faculty, violate the Duke Community
Standard, and will not be tolerated. Such incidences will result in a 0
grade for all parties involved as well as being reported to the
University Judicial Board. Additionally, there may be penalties to your
final class grade. Please review Duke’s Standards of Conduct.

### Grading

| **Topic**                           | **Points** |
|-------------------------------------|-----------:|
| Task 1                              | 10 or 15   |
| Task 2                              | 10 or 15   |
| Task 3                              | 10 or 15   |
| **Total**                           |     **30** |

*A portion of the points for each task will be allocated to code style.*

*Documents that fail to knit after minimal intervention will receive a 0*.
