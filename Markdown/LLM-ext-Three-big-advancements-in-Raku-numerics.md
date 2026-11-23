# Three Big Advancements in Raku Numerics

**Anton Antonov**  
**November 2026**

## Introduction

Raku’s numerical ecosystem is developing from a collection of individual routines into a set of reusable frameworks. This article surveys three developments:

1. **Extensible numerical integration** with `Math::NIntegrate`;
2. **Optimization support** through `COIN-OR` and `Math::Optimization`; and
3. **Quantile regression and regression workflows** through `Math::QuantileRegression` and `Math::Regressionizer`.

“Big advancements” is used here in an infrastructural sense. The claim is not that these packages outperform mature numerical ecosystems in every application, nor that independent benchmarks have established new numerical-method records. The advancement is the increasing availability of extensible APIs, higher-level workflows, and connections between numerical algorithms.

A second theme is the role of large language models (LLMs) and AI agents in developing numerical software. In my experience, they helped with wrappers, translations, repetitive code, and parts of the test suites. They did not replace mathematical design, numerical validation, or maintenance.

The examples below are based on package documentation and development work available in November 2026. They should be rerun in a pinned environment before being used as reproducibility claims. Such an environment should record:

- the Raku version;
- package versions or Git commits;
- the operating system and architecture;
- native-library versions;
- the selected solver or numerical backend; and
- the exact returned values and diagnostics.

More detailed articles about the individual packages will follow.

---

## Numerical integration

[`Math::NIntegrate`](https://github.com/antononcube/Raku-Math-NIntegrate) is a Raku numerical-integration library whose main function, `nintegrate`, is architecturally modeled on Wolfram Language’s `NIntegrate`.

The important development is not simply the presence of several quadrature algorithms. It is the attempt to provide an extensible framework in which integration rules, strategies, regions, variable transformations, stopping criteria, and monitors can be combined.

### Rules, composite rules, and strategies

The package documentation describes one-dimensional rules such as:

- Clenshaw–Curtis;
- Gauss–Kronrod;
- Newton–Cotes; and
- trapezoidal rules.

It also describes multidimensional and composite approaches, including Cartesian, double-exponential, Monte Carlo, and quasi-Monte Carlo methods. A Cartesian rule can combine component rules across dimensions or regions.

A **rule** supplies an approximation scheme. A **strategy** controls how that rule is applied—for example, how regions are subdivided, how estimates are adapted, or how termination is monitored. A **composite rule** is itself made from one or more component rules.

The framework’s README summarizes its nesting model in this way:

- a strategy decorator has a strategy;
- a strategy that is not a decorator has a rule; and
- a composite rule, such as a Cartesian rule, has one or more component rules.

This distinction is useful because it separates numerical approximation from the control logic surrounding it.

The README also discusses several object-oriented design patterns, including Strategy, Composite, Decorator, Template Method, Builder, Interpreter, and Abstract Factory. These descriptions characterize the framework’s architecture; they should not be interpreted as a claim that every pattern corresponds to a separately documented public class or that every algorithm is available in every release.

The package supports multiple stopping criteria and integration monitors. Depending on the selected method, a computation may be controlled by accuracy goals, precision goals, recursion limits, region counts, or monitoring logic. User-defined rules and strategies can also be connected to the framework when they satisfy the relevant interfaces.

The inventory of algorithms is version-sensitive. In particular, the README contains different indications about the status of adaptive Monte Carlo. It is therefore safer to describe `Math::NIntegrate` as having a growing collection of rules and strategies rather than to repeat an unqualified count such as “nine strategies.” Users should consult the algorithm inventory for the installed version.

### Natural-language integration workflows

[`DSL::English::IntegrationWorkflows`](https://github.com/antononcube/Raku-DSL-English-IntegrationWorkflows) translates English-like integration specifications into Raku or Wolfram Language code.

For example:

```raku
use DSL::English::IntegrationWorkflows;

my $cmd =
    'numerically integrate sin(x) over the range [0, pi] '
    ~ 'with up to 20 recursion levels';

my $generated =
    ToIntegrationWorkflowCode($cmd, 'Raku::Ecosystem');
```

The intended result is generated workflow code. The generated text should be inspected before it is evaluated. The exact syntax and return contract should be checked against the installed version of the DSL package.

The package documentation also describes translation into Bulgarian and Russian. These features make the package a code-generation and language-interface layer; they do not remove the need to check the generated numerical method.

A smooth integral provides a simple analytical reference value:

$$
\int_0^\pi \sin(x)\,dx = 2.
$$

The value \(2\) is an analytical reference, not a numerical output from the package. A reproducible numerical test should execute the generated or directly specified workflow and compare its result with this reference value, while recording the selected rule, strategy, tolerance, error estimate, and recursion or region counts.

A suitable acceptance check is:

$$
\left|I_{\text{numerical}}-2\right| < \varepsilon,
$$

for a stated tolerance \(\varepsilon\). The actual result must be generated in the pinned test environment rather than inferred from the analytical answer.

### Use of LLMs

I wanted to implement most of `Math::NIntegrate` myself, especially:

- the object-oriented architecture;
- integration strategies;
- the connections between regions, rules, and variable transformations; and
- the framework for composing strategies and rules.

For some integration rules, I used simple LLM-assisted code generation. AI agents also helped create parts of the test suite.

The multidimensional rules were initially translated from FORTRAN to Raku. That translation contained many errors. The resulting code required manual correction, additional regeneration, and numerical testing. Numerical translation is not merely a change of syntax: indexing conventions, normalization factors, endpoint treatment, and precision assumptions all have to be checked.

The central advancement is therefore the framework: rules and strategies can be inspected, combined where supported, monitored, and extended.

---

## Optimization

Raku can access parts of the [COIN-OR](https://www.coin-or.org/) optimization ecosystem through the package [`COIN-OR`](https://github.com/antononcube/Raku-COIN-OR).

COIN-OR is an ecosystem containing several optimization projects and native solvers. The Raku binding should not be understood as a binding to every COIN-OR project or as a guarantee that every solver backend is available on every platform. The native functionality exposed depends on the package version, installed libraries, and platform configuration.

[`Math::Optimization`](https://github.com/antononcube/Raku-Math-Optimization) provides a higher-level Raku interface. The supplied package documentation specifically demonstrates linear programming.

Consider:

$$
\text{minimize } x+y
$$

subject to

$$
x+2y\geq 3,
$$

with bounds

$$
-1\leq x\leq 2,
\qquad
-1\leq y\leq 1.
$$

The documented call is:

```raku
use Math::Optimization;

linear-programming(
    [1, 1],
    [(1, 2),],
    [3,],
    [[-1, 2], [-1, 1]]
);
```

The README shows the result as:

```text
[1 1]
```

This vector is mathematically correct. It satisfies the constraint and bounds, and its objective value is \(2\). The README displays the variable vector rather than documenting a separate status object. Consequently, production code should consult the installed version’s return-value contract and should not assume that the vector alone provides solver status, error information, or infeasibility diagnostics.

An independently checkable validation is:

$$
x+2y\geq3,\qquad
-1\leq x\leq2,\qquad
-1\leq y\leq1,
$$

together with

$$
|x+y-2|<\varepsilon.
$$

The actual tolerance and solver status should be recorded in a pinned test.

### Native-library prerequisites

`Math::Optimization` requires the Raku `COIN-OR` package. The binding in turn depends on native COIN-OR libraries. Installation details can vary with operating system, architecture, compiler, library paths, and the solver components installed on the system.

A reproducible optimization report should identify:

- the Raku version;
- the versions or commits of `COIN-OR` and `Math::Optimization`;
- the operating system and architecture;
- the native COIN-OR libraries and versions;
- the backend or solver selected by the binding; and
- the solution and any status or diagnostic information actually exposed by that installation.

The example above is a documented interface example, not a claim that one native configuration is universally portable.

### Use of LLMs

The Raku interfaces to `COIN-OR` and, at an earlier stage, `Gurobi` were generated with AI-agent assistance using relatively generic prompts. This is my account of the development process, not a capability claim established by the package README. Gurobi-related code should not be assumed to be part of the currently supported public interface.

For `Math::Optimization`, I first prepared a more complete specification of `linear-programming`. That specification is available in [`LinearProgramming.md`](https://github.com/antononcube/Raku-Math-Optimization/blob/main/docs/LinearProgramming.md). It defines intended conventions for objective coefficients, constraints, bounds, and returned values.

Manual and AI-assisted changes were later made to the optimization packages, including changes intended to improve work with large and sparse constraint matrices. In my workflows, those changes made optimization-backed packages such as `Math::QuantileRegression` and `Math::Regressionizer` more useful. This is an experience report, not a benchmark claim. Performance should be measured with the target data, matrix sparsity pattern, native libraries, and Raku installation.

The practical lesson is that an AI agent can accelerate wrapper development, but the mathematical contract should be written first and the generated implementation tested against independently understood examples.

---

## Quantile regression and regression workflows

Quantile regression models conditional quantiles rather than only the conditional mean. For a predictor value \(x\), it can estimate, for example, the 20th, 50th, and 80th conditional quantiles of a response \(y\).

This is useful when the response distribution is asymmetric or when its conditional spread changes with \(x\). Quantile-regression results can also support anomaly and conditional-distribution workflows, but those applications require explicit interpretation.

In particular:

- independently fitted quantiles can cross;
- fitted quantiles are not automatically a calibrated conditional cumulative distribution function;
- outlier detection requires an explicit residual, tail-probability, leverage, or domain-specific rule; and
- a good-looking plot does not establish predictive accuracy.

### Package relationships

[`Math::QuantileRegression`](https://github.com/antononcube/Raku-Math-QuantileRegression) describes itself as providing basic quantile-regression routines in Raku and requires the Raku `COIN-OR` package. Its README cites quantile-regression literature, the R `quantreg` package, Wolfram Language resources, and the Raku quantile-regression workflow DSL.

The accompanying B-spline guide describes a broader workflow involving basis construction and optimization. That guide, rather than the short package README alone, is the source for the detailed implementation narrative. In the associated development work, `Math::Splines`, `Math::Polynomial::Chebyshev`, and optimization components were used for basis and fitting tasks. The exact dependencies and public APIs should be checked for the release being used; they should not be inferred solely from the package name.

[`Math::Regressionizer`](https://github.com/antononcube/Raku-Math-Regressionizer) provides a higher-level, pipeline-like interface for regression workflows. Its README documents operations including:

- data summaries and rescaling;
- quantile regression;
- least-squares fitting;
- outlier-related workflows;
- conditional-CDF reconstruction; and
- plotting data, fits, residuals, outliers, and CDF-related results.

The README describes `quantile-regression` as using a B-spline function basis, with knots and interpolation order as parameters. The documented default interpolation order is 3. In the example below, `knots => 8` requests the package’s eight-knot configuration; it should not be interpreted as eight coefficients or a particular number of basis functions unless that is explicitly defined by the installed package documentation.

Conditional-CDF reconstruction should be understood as a workflow feature, not a guarantee that arbitrary independently fitted quantiles form a valid distribution. Ordering, monotonicity, and calibration must be checked.

### Example data

The documented examples use seeded noisy-sinusoid data:

```raku
use Data::Generators;

srand(0);

my @x = 0, 0 + 1/300 ... 2;

my @y =
    @x.map({ sin(2 * pi * $_) })
    <<+>>
    random-variate(
        NormalDistribution.new(0, 0.4),
        @x.elems
    );

my @data = @x Z @y;
```

This code follows the documented workflow pattern. The import and distribution-construction syntax should be checked against the installed `Data::Generators` release before execution. A fixed seed guarantees reproducibility only when the random-number implementation and package versions are also fixed.

### Regressionizer workflow

To preserve the workflow object for plotting, retain it before extracting its value:

```raku
use Math::Regressionizer;

my $workflow =
    Math::Regressionizer.new(@data)
    .echo-data-summary
    .quantile-regression(
        knots => 8,
        probs => [0.2, 0.5, 0.8]
    )
    .outliers;

my $plot-result = $workflow.plot(
    title    => "B-splines fit",
    template => "dark",
    format   => "html"
);

my $value = $workflow.take-value;
```

The workflow requests regression quantiles for probabilities \(0.2\), \(0.5\), and \(0.8\). The README calls the resulting curves **regression quantiles**.

This example deliberately separates three objects or values:

- `$workflow`, the retained pipeline object;
- `$plot-result`, the result returned by `.plot`; and
- `$value`, the result returned by `.take-value`.

The precise return types and whether either operation mutates the workflow are package-version details. They should be verified in the pinned environment rather than assumed from method names. The README documents that plotting can be exported to HTML and uses [`JavaScript::Google::Charts`](https://github.com/antononcube/Raku-JavaScript-Google-Charts).

A reproducible plotting test should record whether `$plot-result` is an HTML string, a chart object, or another package-defined value, and should confirm that the resulting HTML is valid.

### Regression validation

A minimal validation should record:

- package versions and commit identifiers;
- the number of observations;
- the knot and probability settings;
- the fitted-value or coefficient representation;
- the number and representation of reported outliers; and
- the type and content of the plotting result.

On a grid of predictor values, the fitted quantiles should be checked for ordering:

$$
\widehat q_{0.2}(x)
\leq
\widehat q_{0.5}(x)
\leq
\widehat q_{0.8}(x).
$$

If this inequality fails, the result exhibits quantile crossing. It should not be interpreted as an ordered conditional distribution without additional processing.

Other checks should include:

- calibration of predicted quantiles against observed coverage;
- monotonicity of any reconstructed conditional CDF;
- residual and leverage diagnostics;
- sensitivity of outlier labels to the chosen probabilities and thresholds; and
- performance on a held-out or otherwise fixed evaluation set.

For the noisy sinusoid, the fitted median can be compared with the known generating signal. Because the data contain noise, exact equality is not expected. A reproducible report should generate and record a mean absolute error or root-mean-square error rather than inventing one from the source code.

### Use of LLMs

I wrote the initial versions of `Math::QuantileRegression` semi-manually, with AI-agent assistance for translating selected Wolfram Language code fragments. Considerable human programming was required to connect basis construction, optimization, fitted functions, and workflow-level results.

A substantial amount of manual and AI-assisted work also went into the guide [*Quantile Regression with B-splines*](https://github.com/antononcube/Raku-Math-QuantileRegression/blob/main/docs/Quantile-Regression-with-B-splines.md). Writing the guide exposed ambiguities in both the implementation and the API.

The initial version of `Math::Regressionizer` was generated from the Python package [`Regressionizer`](https://pypi.org/project/Regressionizer/). It was subsequently adjusted while writing the [regression-workflow guide](https://github.com/antononcube/Raku-Math-Regressionizer/blob/main/docs/Rapid-specification-of-regression-workflows.ipynb).

These experiences suggest a useful division of labor. LLMs and AI agents can translate code, generate repetitive structures, and suggest tests. Human work remains essential for API design, numerical validation, cross-language reconciliation, documentation, and maintenance.

The package [`DSL::English::QuantileRegressionWorkflows`](https://github.com/antononcube/Raku-DSL-English-QuantileRegressionWorkflows) extends the natural-language workflow approach to quantile-regression workflows. Generated code should be inspected and tested rather than treated as an unquestioned statistical specification.

---

## Limitations and conclusion

The packages described here are active projects, not replacements for the mature numerical ecosystems of Fortran, C++, Python, R, or Wolfram Language. Performance depends on the algorithm, data size, precision, native libraries, and Raku implementation. Numerical stability requires problem-specific testing. APIs and algorithm inventories may change.

For reproducible numerical work:

1. pin package versions or Git commits;
2. record the Raku, operating-system, and architecture details;
3. record native solver and library versions;
4. preserve generated workflow code;
5. compare results with analytical examples or trusted implementations;
6. report tolerances, error estimates, solver status, and fitted-result diagnostics; and
7. distinguish documented behavior from personal development experience.

Within those limits, the three developments form a connected set of reusable numerical infrastructure:

- `Math::NIntegrate` supplies extensible integration rules and strategies;
- `COIN-OR` and `Math::Optimization` provide an optimization interface documented here primarily through linear programming;
- `Math::QuantileRegression` provides basic quantile-regression routines and is associated with optimization-backed fitting workflows; and
- `Math::Regressionizer` turns regression operations into composable pipelines.

The common advancement is not merely the addition of algorithms. It is the construction of reusable interfaces around algorithms: rules that can be combined with supported strategies, solver interfaces that can support higher-level statistical work, and pipelines that make analyses easier to specify.

LLMs and AI agents contributed to parts of this work, especially wrappers, translations, repetitive code, and tests. The resulting numerical software still depends on clear specifications, careful implementation, reproducible environments, and independent verification.

---

## References

[AAp1] Anton Antonov. [*Math::NIntegrate*, Raku package](https://github.com/antononcube/Raku-Math-NIntegrate). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov. [*DSL::English::IntegrationWorkflows*, Raku package](https://github.com/antononcube/Raku-DSL-English-IntegrationWorkflows). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov. [*Math::Polynomial::Chebyshev*, Raku package](https://github.com/antononcube/Raku-Math-Polynomial-Chebyshev). 2024. GitHub: [antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov. [*Math::Splines*, Raku package](https://github.com/antononcube/Raku-Math-Splines). 2024–2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov. [*COIN-OR*, Raku package](https://github.com/antononcube/Raku-COIN-OR). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov. [*Math::Optimization*, Raku package](https://github.com/antononcube/Raku-Math-Optimization). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov. [*Math::QuantileRegression*, Raku package](https://github.com/antononcube/Raku-Math-QuantileRegression). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp8] Anton Antonov. [*Math::Regressionizer*, Raku package](https://github.com/antononcube/Raku-Math-Regressionizer). 2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp9] Anton Antonov. [*DSL::English::QuantileRegressionWorkflows*, Raku package](https://github.com/antononcube/Raku-DSL-English-QuantileRegressionWorkflows). 2022–2026. GitHub: [antononcube](https://github.com/antononcube).

[AAp10] Anton Antonov. [*LinearProgramming.md*](https://github.com/antononcube/Raku-Math-Optimization/blob/main/docs/LinearProgramming.md). Documentation for *Math::Optimization*. 2026.

[AAp11] Anton Antonov. [*Quantile Regression with B-splines*](https://github.com/antononcube/Raku-Math-QuantileRegression/blob/main/docs/Quantile-Regression-with-B-splines.md). Documentation for *Math::QuantileRegression*. 2026.

[AAp12] Anton Antonov. [*Rapid specification of regression workflows*](https://github.com/antononcube/Raku-Math-Regressionizer/blob/main/docs/Rapid-specification-of-regression-workflows.ipynb). Documentation for *Math::Regressionizer*. 2026.

[AAp13] Anton Antonov. [*Data::Generators*, Raku package](https://github.com/antononcube/Raku-Data-Generators). Accessed November 2026.

[AAp14] Anton Antonov. [*JavaScript::Google::Charts*, Raku package](https://github.com/antononcube/Raku-JavaScript-Google-Charts). Accessed November 2026.

## Package documentation

- [`Math::NIntegrate` README](https://raw.githubusercontent.com/antononcube/Raku-Math-NIntegrate/refs/heads/main/README.md)
- [`DSL::English::IntegrationWorkflows` README](https://raw.githubusercontent.com/antononcube/Raku-DSL-English-IntegrationWorkflows/refs/heads/main/README.md)
- [`COIN-OR` repository](https://github.com/antononcube/Raku-COIN-OR)
- [`Math::Optimization` README](https://raw.githubusercontent.com/antononcube/Raku-Math-Optimization/refs/heads/main/README.md)
- [`Math::QuantileRegression` README](https://raw.githubusercontent.com/antononcube/Raku-Math-QuantileRegression/refs/heads/main/README.md)
- [`Math::Regressionizer` README](https://raw.githubusercontent.com/antononcube/Raku-Math-Regressionizer/refs/heads/main/README.md)
- [`Math::Splines` repository](https://github.com/antononcube/Raku-Math-Splines)
- [`Math::Polynomial::Chebyshev` repository](https://github.com/antononcube/Raku-Math-Polynomial-Chebyshev)
- [`Data::Generators` repository](https://github.com/antononcube/Raku-Data-Generators)
- [`JavaScript::Google::Charts` repository](https://github.com/antononcube/Raku-JavaScript-Google-Charts)