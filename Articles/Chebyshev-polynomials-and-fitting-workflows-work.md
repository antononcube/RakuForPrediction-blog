# Chebyshev Polynomials and Fitting Workflows

Anton Antonov  
June 2024   
December 2024

-----

## Introduction

This post explores the use of Chebyshev polynomials in regression and curve fitting workflows. It highlights various packages that facilitate these processes, providing insights into their features and applications.

- ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts): This package is instrumental for creating scatter plots and visualizing time series data.
- ["Math::Polynomial::Chebyshev"](https://raku.land/zef:antononcube/Math::Polynomial::Chebyshev): It offers a polynomial basis with recursive and trigonometric computation methods, ensuring exact integer results for numerators and denominators.
- ["Math::Fitting"](https://raku.land/zef:antononcube/Math::Fitting): This package supports linear regression using function bases, providing functors and allowing retrieval of multiple properties.
- ["Data::TypeSystem"](https://raku.land/zef:antononcube/Data::TypeSystem): It provides a summary of data types.
- ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers): This package summarizes data values.

### TL;DR

- Chebyshev polynomials can be computed exactly.
- The "Math::Fitting" package yields functors.
- Fitting utilizes a function basis.
- Matrix formulas facilitate the computation of the fit (linear regression).
- A real-life example is demonstrated using weather temperature data. For details, see the section before the last.

-----

## Setup

Here are the packages used in this post:

```raku
use Math::Polynomial::Chebyshev;
use Math::Fitting;
use Math::Matrix;

use Data::Generators;
use Data::Importers;
use Data::Reshapers;
use Data::Summarizers;
use Data::Translators;
use Data::TypeSystem;

use JavaScript::D3;
use JavaScript::Google::Charts;
use Text::Plot;

use Hash::Merge;
```

```raku, echo=FALSE, results=NONE
my $format = 'html';
my $titleTextStyle = { color => 'DimGray', fontSize => 16 };
my $backgroundColor = 'White';
my $legendTextStyle = { color => 'DarkGray' };
my $legend = { position => "none", textStyle => {fontSize => 14, color => 'DarkGray'} };

my $hAxis = { title => 'x', titleTextStyle => { color => 'DimGray' }, textStyle => { color => 'DarkGray'}, logScale => False, format => 'decimal'};
my $vAxis = { title => 'y', titleTextStyle => { color => 'DimGray' }, textStyle => { color => 'DarkGray'}, logScale => False, format => 'decimal'};

my $annotations = {textStyle => {color => 'DarkGray', fontSize => 10}};
my $chartArea = {left => 50, right => 50, top => 50, bottom => 50, width => '90%', height => '90%'};
```

-------

## Computation Granularity

This section discusses the computation of Chebyshev polynomials using different methods and their implications on precision.

The computation over Chebyshev polynomials is supported on the interval $[-1, 1]$. The recursive and trigonometric methods are compared to understand their impact on the precision of results.

```raku
<recursive trigonometric>
==> { .map({ $_ => chebyshev-t(3, 0.3, method => $_) }) }()
```

Here we compute polynomial values over a "dense enough" grid:

```raku
my $k = 12;
my $method = 'trig'; # 'trig'
my @x = (-1.0, -0.99 ... 1.0);
say '@x.elems : ', @x.elems;

my @data  = @x.map({ [$_, chebyshev-t($k, $_, :$method)]});
my @data1 = chebyshev-t($k, @x);

say deduce-type(@data);
say deduce-type(@data1);
```

Residuals with trigonometric and recursive methods are utilized to assess precision:

```raku
sink records-summary(@data.map(*.tail) <<->> @data1)
```

-----

## Precision

The exact Chebyshev polynomial values can be computed using `FatRat` numbers, ensuring high precision in numerical computations.

```raku
my $v = chebyshev-t(100, <1/4>.FatRat, method => 'recursive')
```

The numerator and denominator of the computed result are:

```raku
say $v.numerator;
say $v.denominator;
```

-----

## Plots

This section demonstrates plotting Chebyshev polynomials using [Google Charts](https://developers.google.com/chart) via the ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts) package.

### Single Polynomial

A single polynomial can be plotted using a [Line chart](https://developers.google.com/chart/interactive/docs/gallery/linechart):

```raku
my $n = 6;
my @data = chebyshev-t(6, (-1, -0.98 ... 1).List);
```

```raku, eval=FALSE
#%html
js-google-charts('LineChart', @data, 
    title => "Chebyshev-T($n) polynomial", 
    :$titleTextStyle, :$backgroundColor, :$chartArea, :$hAxis, :$vAxis,
    width => 800, 
    div-id => 'poly1', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Chebyshev-polynomial-6.png)

### Basis

In fitting, bases of functions are crucial. The first eight Chebyshev-T polynomials are plotted to illustrate this.

```raku
my $n = 8;
my @data = (-1, -0.98 ... 1).map(-> $x { [x => $x, |(0..$n).map({ $_.Str => chebyshev-t($_, $x, :$method) }) ].Hash });

deduce-type(@data):tally;
```

The plot with all eight functions is shown below:

```raku, eval=FALSE
#%html
js-google-charts('LineChart', @data,
    column-names => ['x', |(0..$n)».Str],
    title => "Chebyshev T polynomials, 0 .. $n",
    :$titleTextStyle,
    width => 800, 
    height => 400,
    :$backgroundColor, :$hAxis, :$vAxis,
    legend => merge-hash($legend, %(position => 'right')),
    chartArea => merge-hash($chartArea, %(right => 100)),
    format => 'html', 
    div-id => "cheb$n",
    :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Chebyshev-polynomial-0-8.png)

-----

## Text Plot

Text plots provide a reliable method for visualizing data anywhere! The data is converted into a long form to facilitate plotting using ["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot).

```raku
my @dataLong = to-long-format(@data, <x>).sort(*<Variable x>);
deduce-type(@dataLong):tally
```

A sample of the data is provided:

```raku, results=asis
@dataLong.pick(8)
==> {.sort(*<Variable x>)}()
==> to-html(field-names => <Variable x Value>)
```

The text plot is presented here:

```raku
my @chebInds = 1, 2, 3, 4;
my @dataLong3 = @dataLong.grep({ $_<Variable>.Int ∈ @chebInds }).classify(*<Variable>).map({ $_.key => $_.value.map(*<x Value>).Array }).sort(*.key)».value;
text-list-plot(@dataLong3, width => 100, height => 25, title => "Chebyshev T polynomials, 0 .. $n", x-label => (@chebInds >>~>> ' : ' Z~ <* □ ▽ ❍>).join(', '))
```

-----

## Fitting

This section presents the generation of "measurements data" with noise and the fitting process using a function basis.

```raku
my @temptimelist = 0.1, 0.2 ... 20;
my @tempvaluelist = @temptimelist.map({ sin($_) / $_ }) Z+ (1..200).map({ (3.rand - 1.5) * 0.02 });
my @data1 = @temptimelist Z @tempvaluelist;
@data1 = @data1.deepmap({ .Num });

deduce-type(@data1)
```

Rescaling of x-coordinates is performed as follows:

```raku
my @data2 = @data1.map({ my @a = $_.clone; @a[0] = @a[0] / max(@temptimelist); @a });

deduce-type(@data2)
```

A summary of the data is provided:

```raku
sink records-summary(@data2)
```

The data is plotted below:

```raku, eval=FALSE
#% html
js-google-charts("Scatter", @data2, 
    title => 'Measurements data with noise',
    :$backgroundColor, :$hAxis, :$vAxis,
    :$titleTextStyle, :$chartArea,
    width => 800, 
    div-id => 'data', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Measurements-data-with-noise.png)

A function to rescale from $[0, 1]$ to $[-1, 1]$ is defined:

```raku
my &rescale = { ($_ - 0.5) * 2 };
```

The basis functions are listed:

```raku
my @basis = (^16).map({ chebyshev-t($_) o &rescale });
@basis.elems
```

**Remark:** The function composition operator `o` is utilized above. The argument is rescaled before computing the Chebyshev polynomial value.

A linear model fit is computed using these functions:

```raku
my &lm = linear-model-fit(@data2, :@basis)
```

The best fit parameters are:

```raku
&lm('BestFitParameters')
```

The plot of these parameters is shown:

```raku, eval=FALSE
#% html
js-google-charts("Bar", &lm('BestFitParameters'), 
    :!horizontal,
    title => 'Best fit parameters',
    :$backgroundColor, 
    hAxis => merge-hash($hAxis, {title => 'Basis function index'}), 
    vAxis => merge-hash($hAxis, {title => 'Coefficient'}), 
    :$titleTextStyle, :$chartArea,
    width => 800, 
    div-id => 'bestFitParams', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Best-fit-parameters.png)

It is observed from the plot that using more than 12 basis functions does not improve the fit, as coefficients beyond the 12th index are very small.

The data and the fit are plotted after preparing the plot data:

```raku
my @fit = @data2.map(*.head)».&lm;
my @plotData = transpose([@data2.map(*.head).Array, @data2.map(*.tail).Array, @fit]);
@plotData = @plotData.map({ <x data fit>.Array Z=> $_.Array })».Hash;

deduce-type(@plotData)
```

The plot is presented here:

```raku, eval=FALSE
#% html
js-google-charts('ComboChart', 
    @plotData, 
    title => 'Data and fit',
    column-names => <x data fit>,
    :$backgroundColor, :$titleTextStyle :$hAxis, :$vAxis,
    seriesType => 'scatter',
    series => {
        0 => {type => 'scatter', pointSize => 2, opacity => 0.1, color => 'Gray'},
        1 => {type => 'line'}
    },
    legend => merge-hash($legend, %(position => 'bottom')),
    :$chartArea,
    width => 800, 
    div-id => 'fit1', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Data-and-fit.png)

The residuals of the last fit are computed:

```raku
sink records-summary( (@fit <<->> @data2.map(*.tail))».abs )
```

----

## Condition Number

The [Ordinary Least Squares (OLS)](https://en.wikipedia.org/wiki/Ordinary_least_squares) fit is computed using the formula:

$$
\beta = (X^T \cdot X)^{-1} \cdot X^T \cdot y
$$

The condition number of the "normal matrix" (or "Gram matrix") $X^T \cdot X$ is examined. The design matrix is obtained first:

```raku
my @a = &lm.design-matrix();
my $X = Math::Matrix.new(@a);
$X.size
```

The Gram matrix is:

```raku
my $g = $X.transposed.dot-product($X);
$g.size
```

The condition number of this matrix is:

```raku
$g.condition
```

It is concluded that the design matrix is suitable for use.

**Remark:** For a system of linear equations in matrix form $A x = b$, the condition number of $A$, $\kappa (A)$, is defined as the maximum ratio of the relative error in $x$ to the relative error in $b$.

**Remark:** Typically, if the condition number is $\kappa (A)=10^{d}$, up to $d$ digits of accuracy may be lost in addition to any loss caused by the numerical method (due to precision issues in arithmetic calculations).

**Remark:** A very "Raku-way" to define an ill-conditioned matrix is as "almost not of full rank" or "if its inverse does not exist."

-----

## Temperature Data

The entire workflow is repeated with real-life data, specifically weather temperature data for four consecutive years in Greenville, South Carolina, USA. This location is where the [Perl and Raku Conference 2025](https://www.perl.com/article/get-ready-for-the-2025-perl-and-raku-conference/) will be held.

The time series data is ingested:

```raku
my $url = 'https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Data/dsTemperature-Greenville-SC-USA.csv';
my @dsTemperature = data-import($url, headers => 'auto');
@dsTemperature = @dsTemperature.deepmap({ $_ ~~ / ^ \d+ '-' / ?? DateTime.new($_) !! $_.Num });
deduce-type(@dsTemperature)
```

A summary of the data is shown:

```raku
sink records-summary(@dsTemperature, field-names => <Date AbsoluteTime Temperature>)
```

The plot of the data is provided:

```raku, eval=FALSE
#% html
js-google-charts("Scatter", @dsTemperature.map(*<Date Temperature>), 
    title => 'Temperature of Greenville, SC, USA',
    :$backgroundColor,
    hAxis => merge-hash($hAxis, {title => 'Time', format => 'M/yy'}), 
    vAxis => merge-hash($hAxis, {title => 'Temperature, ℃'}), 
    :$titleTextStyle, :$chartArea,
    width => 1200, 
    height => 400, 
    div-id => 'tempData', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Temperature-of-Greenville-SC-USA.png)

The fit is performed with rescaling:

```raku
my ($min, $max) = @dsTemperature.map(*<AbsoluteTime>).Array.&{ (.min, .max) }();
```

```raku
my &rescale-time = { -($max + $min) / ($max - $min) + (2 * $_) / ($max - $min)};
my @basis = (^16).map({ chebyshev-t($_) o &rescale-time });
@basis.elems
```

```raku
my &lm-temp = linear-model-fit(@dsTemperature.map(*<AbsoluteTime Temperature>), :@basis)
```

The plot of the time series and the fit is presented:

```raku
my @fit = @dsTemperature.map(*<AbsoluteTime>)».&lm-temp;
my @plotData = transpose([@dsTemperature.map({ $_<AbsoluteTime> }).Array, @dsTemperature.map(*<Temperature>).Array, @fit]);
@plotData = @plotData.map({ <x data fit>.Array Z=> $_.Array })».Hash;

deduce-type(@plotData)
```

```raku, eval=FALSE
#% html

my @ticks = @dsTemperature.map({ %( v => $_<AbsoluteTime>, f => $_<Date>.Str.substr(^7)) })».Hash[0, 120 ... *];

js-google-charts('ComboChart', 
    @plotData,
    title => 'Temperature data and Least Squares fit',
    column-names => <x data fit>,
    :$backgroundColor, :$titleTextStyle,
    hAxis => merge-hash($hAxis, {title => 'Time', :@ticks, textPosition => 'in'}), 
    vAxis => merge-hash($hAxis, {title => 'Temperature, ℃'}), 
    seriesType => 'scatter',
    series => {
        0 => {type => 'scatter', pointSize => 3, opacity => 0.1, color => 'Gray'},
        1 => {type => 'line', lineWidth => 4}
    },
    legend => merge-hash($legend, %(position => 'bottom')),
    :$chartArea,
    width => 1200, 
    height => 400, 
    div-id => 'tempDataFit', :$format,
    :png-button)
```

![](./Diagrams/Chebyshev-polynomials-and-fitting-workflows/Temperature-data-and-Least-Squares-fit.png)

-----

## Future Plans

The current capabilities of Raku in performing regression analysis for both educational and practical purposes have been demonstrated. 

Future plans include implementing computational frameworks for [Quantile Regression](https://en.wikipedia.org/wiki/Quantile_regression) in Raku. 
Additionally, the workflow code in this post can be generated using Large Language Models (LLMs), which will be explored soon.