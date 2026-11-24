# Three Big Advancements in Raku Numerics

Anton Antonov
November 2026

## Introduction 

This blog post describes three big advancements in Raku's numerical algorithms packages:

- "Big league" Numerical Integration
- Optimization support
- Quantile regression and regression workflows

Each section below discusses the use of Large Language Models (LLMs) or Artificial Intelligence (AI) agents.

**Remark:** More detailed posts on these new(er) Raku functionalities will follow in the next few months.

**Remark:** [Here](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Markdown/LLM-ext-Three-big-advancements-in-Raku-numerics.md) 
is an extended, wordier version of blog post derived via a ["blog-writer"](https://github.com/antononcube/Python-GoogleADK-agents/tree/main/blog-writer) AI-agent. 

---

## Numerical integration

The package ["Math::NIntegrate"](https://raku.land/zef:antononcube/Math::NIntegrate) has multiple integration strategies and multiple one-dimensional integration rules, a cartesian rule that can combine them, and a family of "orbit-based" multidimensional rules.
Its framework has an object-oriented design and implementation allows the user-defined integration strategies and rules to be hooked-up to it.
It several types of criteria for stopping the integration process and allows the use of integration monitors.
Frameworks algorithms are nested depending on their type:

- A strategy decorator has a strategy
- A strategy that is not a decorator, has a rule
- A composite rule (like Cartesian Rule) has one or more component rules

In addition, the Raku package ["DSL::English::IntegrationWorkflows"](https://raku.land/zef:antononcube/DSL::English::IntegrationWorkflows) can be used to generate numerical integration commands
using natural language specs. For example:

```raku
use DSL::English::IntegrationWorkflows;

my $cmd= 'numerically integrate 1/sqrt(x) over the range [0,1] without singularity handling with up to 20 recursion levels';

ToIntegrationWorkflowCode($cmd, 'Raku::Ecosystem');
```

**Remark:** The package "DSL::English::IntegrationWorkflows" supports the generation of Wolfram Language code and translation to Bulgarian and Russian. 

### Use of LLMs

I wanted to program most of this package myself -- especially, the all elements of the OOP design, the integration strategies, and the core connections between regions, rules, and variable transformers.
For a few of the integration rules I used simple prompt code generation with LLMs. AI-agents were used to create some of the tests.
The multidimensional rules were preprogrammed from FORTRAN to Raku, but had many mistakes, so the Raku code had to be manually corrected or re-generated. 

---

## Optimization

The optimization (or Operations Research) library COIN-OR can be used in Raku via the package "COIN-OR".

The package ["Math::Optimization"](https://raku.land/zef:antononcube/Math::Optimization) is based on "COIN-OR" (and at some point "Gurobi") and provides easier to use interface:

Consider this problem: minimize $x + y$, subject to constraint $x + 2 * y ≥ 3$ and lower bounds $2 ≥ x ≥ -1$, $1 ≥ y ≥ -1$.

Here is the solution with `linear-programming` (provided by "Math::Optimization"):

```raku
use Math::Optimization;
linear-programming([1, 1], [(1, 2),], [3,], [[-1, 2], [-1, 1]])
```

### Use of LLMs

The interface packages "COIN-OR" and "Gurobi" were generated with AI-agents using fairly generic, simple prompts.
In order to generate the code "Math::Optimization" package with an AI-agent first a *complete* specification of `linear-programming` was prepared. (See ["LinearProgramming.md"](https://github.com/antononcube/Raku-Math-Optimization/blob/main/docs/LinearProgramming.md).)
A few manual and AI-assisted manipulations of "COIN-OR" and "Math::Optimization" were done in order to speed up the computations with large, sparse constraint matrices.
(That speed up made "Math::QuantileRegression" and "Math::Regressionizer" much more useful.)

---

## Quantile Regression

QuantileRegression (QR) provides multiple advantages over the other types of regression, like, Least Squares, LOESS, and others.
QR is robust to outliers, allows the finding of point- and variance anomalies, and can be used to compute conditional Cumulative Distribution Functions.  

The core Raku package ["Math::QuantileRegression"]((https://raku.land/zef:antononcube/Math::QuantileRegression) uses the packages ["Math::Splines"]((https://raku.land/zef:antononcube/Math::Splines) and ["Math::Polynomial::Chebyshev"]((https://raku.land/zef:antononcube/Math::Polynomial::Chebyshev)) to form function bases
and "Math::Optimization" to fit those bases to data.

The package ["Math::Regressionizer"](https://raku.land/zef:antononcube/Math::Regressionizer) provides monadic pipelines that allow rapid specification of regression workflows.

Here is "noisy sinusoidal" random data:

```raku
use Data::Generators;
use Data::TypeSystem;

srand(0);
my @x = 0, 0 + 1/300 ... 2;
my @y = @x.map({ sin(2 * pi * $_) }) <<+>> random-variate(NormalDistribution.new(0, 0.4), @x.elems);
my @data = @x Z @y;
deduce-type(@data):tally;
```

Here is a pipeline that fits the B-spline basis of 8 knots to the probabilities 0.02, 0.5, and 0.98:

```raku
use Math::Regressionizer;

my $obj = 
  Math::Regressionizer.new(@data)
  .echo-data-summary
  .quantile-regression(knots => 8, probs => [0.02, 0.5, 0.98])
  .outliers()
  .echo-value(with => {say "outlier counts: ", $_>>.elems} )
```

Here is plot of the data and fitted *regression quantiles*:

```raku, eval=FALSE
$obj.plot(title => "B-splines fit", template => "dark", format => 'html').take-value
```

```raku
$obj.plot(title => "B-splines fit", format => 'html').take-value
==> spurt("qr-plot.html")
```

The package ["DSL::English::QuantileRegressionWorkflows"](https://raku.land/zef:antononcube/DSL::English::QuantileRegressionWorkflows) was extended to generate Raku code. 

### Use of LLMs

I wrote semi-manually the initial versions of the package "Math::QuantileRegression" using AI-agent assistance for translation of Wolfram Language code fragments.
A fair amount of human programming was done while writing the guide ["Quantile-Regression-with-B-splines.md"](https://github.com/antononcube/Raku-Math-QuantileRegression/blob/main/docs/Quantile-Regression-with-B-splines.md) -- 
both of the code of that document and "Math::QuantileRegression". 
The initial version of the package "Math::Regressionizer" was generated from the code of the Python package ["Regressionizer"](https://pypi.org/project/Regressionizer).
Again, multiple adjustments were made while writing the [guide for that package](https://github.com/antononcube/Raku-Math-Regressionizer/blob/main/docs/Rapid-specification-of-regression-workflows.ipynb).

---

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

---

## Sources

More details about the packages can be found in these Markdown files;

- Math::NIntegrate : https://raw.githubusercontent.com/antononcube/Raku-Math-NIntegrate/refs/heads/main/README.md
- Math::QuantileRegression : https://raw.githubusercontent.com/antononcube/Raku-Math-QuantileRegression/refs/heads/main/README.md
- Math::Optimization :  https://raw.githubusercontent.com/antononcube/Raku-Math-Optimization/refs/heads/main/README.md
- Math::Regressionizer : https://raw.githubusercontent.com/antononcube/Raku-Math-Regressionizer/refs/heads/main/README.md