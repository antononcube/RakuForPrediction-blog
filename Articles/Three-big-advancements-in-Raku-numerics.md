# Three big advancements in Raku numerics

**Outline draft!!"

Anton Antonov
November 2026

## Introduction 

This blog post describes three big advancements in Raku's numerical algorithms packages:

- "Big league" Numerical Integration
- Optimization support
- Quantile regression and regression workflows

(More detailed posts will follow in the next few months.)

## Numerical integration

The package "Math::NIntegrate" has 9 integration strategies and multiple one-dimensional integration rules, cartesian rule that can combine them, and a family of "orbit-based" multidimensional rules.
Its framework has an object-oriented design and implementation allows the user-defined integration strategies and rules to be hooked-up to it.
It several types of criteria for stopping the integration process and allows the use of integration monitors.
Frameworks algorithms are nested depending on their type:

- A strategy decorator has a strategy
- A strategy that is not a decorator, has a rule
- A composite rule (like Cartesian Rule) has one or more component rules

In addition, the Raku package "DSL::English::IntegrationWorkflows" can be used to generate numerical integration commands
using natural language specs. For example:

```raku
use DSL::English::IntegrationWorkflows;

my $cmd= 'numerically integrate 1/sqrt(x) over the range [0,1] without singularity handling with up to 20 recursion levels';

ToIntegrationWorkflowCode($cmd, 'Raku::Ecosystem');
```

**Remark:** The package "DSL::English::IntegrationWorkflows" supports the generation of Wolfram Language code and translation to Bulgarian and Russian. 

---

## Optimization

The optimization (or Operations Research) library COIN-OR can be used in Raku via the package "COIN-OR".

The package "Math::Optimization" is based on "COIN-OR" (and at some point "Gurobi") and provides easier to use interface:

Consider this problem: minimize $x + y$, subject to constraint $x + 2 * y ≥ 3$ and lower bounds $2 ≥ x ≥ -1$, $1 ≥ y ≥ -1$.

Here is the solution with `linear-programming` (provided by "Math::Optimization"):

```raku
use Math::Optimization;
linear-programming([1, 1], [(1, 2),], [3,], [[-1, 2], [-1, 1]])
```

---

## Quantile Regression

QuantileRegression (QR) provides multiple advantages over the other types of regression, like, Least Squares, LOESS, and others.
QR is robust to outliers, allows the finding of point- and variance anomalies, and can be used to compute conditional Cumulative Distribution Functions.  

The core Raku package "Math::QuantileRegression" uses the packages "Math::Splines" and "Math::Polynomial::Chebyshev" to form function bases
and "Math::Optimization" to fit those bases to data.

The package "Math::Regressionizer" provides monadic pipelines that allow rapid specification of regression workflows.

Here is "noisy sinusoidal" random data:

```raku
use Data::Generators;
srand(0);
my @x = 0, 0 + 1/300 ... 2;
my @y = @x.map({ sin(2 * pi * $_) }) <<+>> random-variate(NormalDistribution.new(0, 0.4), @x.elems);
my @data = @x Z @y;
```

Here is a pipeline that fits the B-spline basis of 8 knots to the probabilities 0.2, 0.5, and 0.5:

```raku
use Math::Regressionizer;

my $obj = 
  Math::Regressionizer.new(@data)
  .echo-data-summary
  .quantile-regression(knots => 8, probs => [0.2, 0.5, 0.8])
  .outliers
  .take-value
```

Here is plot of the data and fitted *regression quantiles*:

```raku
$obj.plot(title => "B-splines fit", template => "dark", format => 'html').take-value
```

The package "DSL::English::QuantileRegressionWorkflows"

## References

[AAp1] Anton Antonov, 
[Math::NIntegrate, Raku package](https://github.com/antononcube/Raku-Math-NIntegrate),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[DSL::English::IntegrationWorkflows, Raku package](https://github.com/antononcube/Raku-DSL-English-IntegrationWorkflows),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[Math::Polynomial::Chebyshev, Raku package](https://github.com/antononcube/Raku-Math-Polynomial-Chebyshev),
(2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[Math::Splines, Raku package](https://github.com/antononcube/Raku-Math-Splines),
(2024-2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov,
[COIN-OR, Raku package](https://github.com/antononcube/Raku-COIN-OR),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov,
[Math::Optimization, Raku package](https://github.com/antononcube/Raku-Math-Optimization),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov,
[Math::QuantileRegression, Raku package](https://github.com/antononcube/Raku-Math-QuantileRegression),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp8] Anton Antonov,
[Math::Regressinizer, Raku package](https://github.com/antononcube/Raku-Math-Regressionizer),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp9] Anton Antonov,
[DSL::English::QuantileRegressionWorkflows, Raku package](https://github.com/antononcube/Raku-DSL-English-QuantileRegressionWorkflows),
(2022-2026),
[GitHub/antononcube](https://github.com/antononcube).