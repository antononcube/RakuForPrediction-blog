# Jacobian conjecture counterexample by LLM

Anton Antonov  
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
July 2026

### Introduction

The [Jacobian conjecture](https://en.wikipedia.org/wiki/Jacobian_conjecture) was (very) recently disproved by a [counterexample found by Anthropic's Fable 5](https://x.com/__alpoge__/status/2079028340955197566). [MS1]. This notebook demonstrates that counterexample using some of the built-in Raku functions and dedicated packages that facilitate symbolic computations, like ["CortexJS"](https://raku.land/zef:antononcube/CortexJS), [AAp1], ["LaTeX::Grammar"](https://raku.land/zef:antononcube/LaTeX::Grammar), [AAp2], ["Proc::ZMQed"](https://raku.land/zef:antononcube/Proc::ZMQed), [AAp3], and ["WWW::WolframAlpha"](https://raku.land/zef:antononcube/WWW::WolframAlpha), [AAp4]. We implement and use a simple Raku `find-instance` to find three different points which the counterexample polynomial 3-dimensional (3D) transformation maps into the same 3D point.

In the comments of [this X-post](https://x.com/__alpoge__/status/2079028340955197566) (and New Scientist article, [MS1]) it is pointed out that the counterexample found using [Anthropic's Fable 5](https://www.anthropic.com/claude/fable) Large Language Model (LLM).

**Remark:** The points found in this notebook are different than those in the[ X-post](https://x.com/__alpoge__/status/2079028340955197566) .

---

## Theory

In this section we provide a theoretical formulation of the Jacobian conjecture. For more a mathematically elaborated formulation see the Wikipedia article ["Jacobian conjecture"](https://en.wikipedia.org/wiki/Jacobian_conjecture), [Wk1].

Suppose $u$ and $v$ are polynomials of the variables $x$ and $y$. Consider the 2-dimensional (2D) transformation (i.e., a vector valued function) $F(x,y)\text{:=}(u[x,y], v[x,y])$. The Jacobian conjectures says that if the Jacobian determinant:

$$J = \frac{\partial u}{\partial x} \frac{\partial v}{\partial y} - \frac{\partial u}{\partial y} \frac{\partial v}{\partial x}$$

is a non-zero constant, then there must exist another pair of polynomials that invert transformation with the polynomials in $u$ and $v$.

Below we show that the conjecture is not true using the 3D transformation polynomials proclaimed in [this X post](https://x.com/__alpoge__/status/2079028340955197566), [MS1].

**Remark:** This notebook is the Raku version of the Wolfram Language (WL) notebook ["Jacobian conjecture counterexample by LLM"](https://community.wolfram.com/groups/-/m/t/3766129), [AAn1].

**Remark:** The Wolfram Desktop or Wolfram Engine is installed then the WL symbolic computations can be done via Raku-chatbook's `bash` magic cell using `wolframscript`.

----

## Setup

Here are the packages used in this notebook:

```raku
use CortexJS;
use LaTeX::Grammar;
use Proc::ZMQed;
use WWW::WolframAlpha;

use Data::TypeSystem;
```

**Remark:** The package ["WWW::WolframAlpha"](https://raku.land/zef:antononcube/WWW::WolframAlpha) is available in any [Raku-chatbook session](https://rakuforprediction.wordpress.com/2024/05/18/chatbook-new-magic-cells/#wolframalpha), [AA1].

Here we define a sub to render matrices and vectors via LaTeX in Markdown magic cells of Raku-chatbook:

```raku
sub latex-matrix(@data) {
    ['$$', '\\begin{bmatrix}', @data.map({ $_».subst('$',:g).join(' & ') }).join(' \\\\' ~ "\n"), '\\end{bmatrix}', '$$'].join("\n")
}
```


---

## Polynomial mapping and Jacobian matrix determinant

In this section the Anthropic's Fable 5 found counterexample is exemplified using corresponding definitions and invocations.

Definition of the counterexample polynomial mapping as a list of LaTeX strings:

```raku
#%markdown
my @poly-map =
     '(1 + x y)^3 z + y^2 (1 + x y) (4 + 3 x y)', 
     'y + 3 x (1 + x y)^2 z + 3 x y^2 (4 + 3 x y)', 
     '2 x - 3 x^2 y - x^3 z';

latex-matrix(@poly-map)
```

The Jacobian matrix:

```raku
#%markdown
my @jacobian-matrix=
cross(@poly-map, <x y z>)
.map({ 'D_' ~ $_.tail ~ '{' ~ $_.head ~ '}' })
.map({ $_
    ==> parse-latex()
    ==> evaluate()
    ==> to-latex() })
.rotor(3);

latex-matrix(@jacobian-matrix)
```

The Jacobian matrix -- via [`wolframscript`](https://www.wolfram.com/wolframscript/) and the Wolfram Function Repository (WFR) entry [JacobianMatrix](https://resources.wolframcloud.com/FunctionRepository/resources/JacobianMatrix)):

```raku
#% markdown
my $proc = run 'wolframscript', '-code', "ResourceFunction[\"JacobianMatrix\"][{"\{@poly-map.join(', ')\}"}, \{x, y, z\}]//Simplify//Flatten", :out;
my $res = $proc.out.slurp(:close);
latex-matrix($res.split(/<[{},]>/, :skip-empty)».trim.rotor(3))
```

The determinant is a constant:

```raku
'\\det' ~ latex-matrix(@jacobian-matrix).lines[1 .. *-2].join("\n")
==> parse-latex()
==> evaluate()
==> simplify()
```

```
# -2
```

We can directly find the determinant via WFR's [JacobianDeterminant](https://resources.wolframcloud.com/FunctionRepository/resources/JacobianDeterminant):

```raku
#% markdown
my $proc = run 'wolframscript', '-code', "ResourceFunction[\"JacobianDeterminant\"][{"\{@poly-map.join(', ')\}"}, \{x, y, z\}]", :out;
my $res = $proc.out.slurp(:close);
```

Here we verify that the expressions of the Jacobian matrix computed with "CortexJS" are the same as the ones computed with Wolfram Language:

```raku
my @cortex-js-exprs = @jacobian-matrix.flat(:hammer).map({ latex-interpret($_.trim.substr(1, $_.chars-2), actions => 'WL') });
my $proc = run 'wolframscript', 
    '-code', 
    "Simplify[ResourceFunction[\"JacobianMatrix\"][{"\{@poly-map.join(', ')\}"}, \{x, y, z\}] 
    - 
    Partition[\{{@cortex-js-exprs.join(', ')}\},3]]", 
    :out;
my $res = $proc.out.slurp(:close);
```

```
# {{0, 0, 0}, {0, 0, 0}, {0, 0, 0}}
```

----

## Finding the points mapped into the same point

In the Wolfram notebook ["Jacobian conjecture counterexample by LLM"](https://community.wolfram.com/groups/-/m/t/3766129), [AAn1], the Wolfram Language `FindInstance` was use to find points for which the polynomial map gives the same result. I.e., $p_i\text{:=}\left(x_i,y_i,z_i\right), i\in [1,2,3]$ ∧ $p_i\text{!=}p_j\land i\neq j$.

In this section with Raku we do a more exhaustive point search using the results in [AAn1] as guidance.

Searching all three points is too slow. We use a two-stage computation: we find first a solution with two different points, then we find a third point that is different that previously found two.

Define a function that computes the polynomial mapping using the LaTeX strings of the (symbolic) definition above:

```raku
use MONKEY-SEE-NO-EVAL;
my $poly-map-expr = '[' ~ @poly-map.map({ latex-interpret($_, actions => 'RakuAST').DEPARSE.subst('x', '$x', :g).subst('y', '$y', :g).subst('z', '$z', :g) }).join(', ') ~ ']';
my &poly-map = EVAL('sub ($x, $y, $z) {' ~ $poly-map-expr ~ '}');
```

Verify definition of `&poly-map`:

```raku
&poly-map(1.1, 0.2, 1/2)
```

```
# [1.135332 3.27098 0.8085]
```

Search grid:

```raku
my $b = 6;
#my @grid3d = (-$b, -$b + 1 ... $b) X (-$b, -$b + 1/2 ... $b) X (-$b, -$b + 1/100 ... $b);
my @zs = (([\+] -1/36 xx 2_000).reverse, [\+] 1/36 xx 2_000).flat.grep(-$b ≤ * ≤ $b);
#my @grid3d = (-$b, -$b + 1 ... $b) X (-1, -1/2, 0, 1/2, 1) X @zs;
my @grid3d = (-$b, 0, $b) X (-1, -1/2, 0, 1/2, 1) X @zs;
say @grid3d.elems;
```

```
# 6480
```

```raku
deduce-type(@grid3d):tally
```

```
# Tuple([Tuple([Atom((Int)), Atom((Int)), Atom((Rat))]) => 3888, Tuple([Atom((Int)), Atom((Rat)), Atom((Rat))]) => 2592], 6480)
```

Pre-compute polynomial mapping values:

```raku
my %poly-map-values = @grid3d.race(:4degree).map({ $_.Str => &poly-map(|$_) });
%poly-map-values.elems
```

```
# 6480
```

Search for two point mapped into one:

```raku
my $k = 0;
my $degree = 4;
my @found;

@grid3d.race(:$degree, batch => ceiling(@grid3d.elems / $degree)).map( -> @p {
  
  my @mp = |%poly-map-values{@p.Str};
  
  if $k %% 500 { say (:$k) }
  
  $k++;

  for @grid3d -> @candidate {

    my @mpc = |%poly-map-values{@candidate.Str};

    if (@mpc <<->> @mp)».abs.max ≤ 0.0001 && @p ne @candidate {
      @found.push( %(:@p, :@candidate, :@mp) );
      say @found.tail;
    }   
  }
});

@found.elems

# ≈90 minutes
```

```
# k => 0
# k => 0
# k => 0
# k => 0
# {candidate => (-6 0 0.055556), mp => [0.055556 -1 0], p => (6 -0.5 0.305556)}
# {candidate => (0 -1 -3.944444), mp => [0.055556 -1 0], p => (6 -0.5 0.305556)}
# k => 500
# k => 500
# k => 1000
# k => 1000
# k => 1000
# {candidate => (6 0.5 -0.194444), mp => [0.555556 3 0], p => (-6 1 0.555556)}
# k => 1500
# k => 2000
# k => 2000
# {candidate => (-6 0.5 0.305556), mp => [0.055556 1 0], p => (6 0 0.055556)}
# {candidate => (0 1 -3.944444), mp => [0.055556 1 0], p => (6 0 0.055556)}
# {candidate => (-6 0 0.055556), mp => [0.055556 -1 0], p => (0 -1 -3.944444)}
# {candidate => (6 -0.5 0.305556), mp => [0.055556 -1 0], p => (0 -1 -3.944444)}
# {candidate => (6 -1 0.555556), mp => [0.555556 -3 0], p => (-6 -0.5 -0.194444)}
# k => 2500
# {candidate => (-6 0.5 0.305556), mp => [0.055556 1 0], p => (0 1 -3.944444)}
# {candidate => (6 0 0.055556), mp => [0.055556 1 0], p => (0 1 -3.944444)}
# k => 3000
# k => 3500
# {candidate => (-6 1 0.555556), mp => [0.555556 3 0], p => (6 0.5 -0.194444)}
# k => 4000
# {candidate => (0 -1 -3.944444), mp => [0.055556 -1 0], p => (-6 0 0.055556)}
# {candidate => (6 -0.5 0.305556), mp => [0.055556 -1 0], p => (-6 0 0.055556)}
# k => 4500
# k => 5000
# {candidate => (-6 -0.5 -0.194444), mp => [0.555556 -3 0], p => (6 -1 0.555556)}
# k => 5500
# k => 6000
# {candidate => (0 1 -3.944444), mp => [0.055556 1 0], p => (-6 0.5 0.305556)}
# {candidate => (6 0 0.055556), mp => [0.055556 1 0], p => (-6 0.5 0.305556)}
```

```
# 16
```

Direct assignment -- for faster setup of (consecutive, repeated) experiments:

```raku
sink my @found = 
{:candidate($(-6, 0, <1/18>)), :mp($[<1/18>, -1.0, 0.0]), :p($(6, -0.5, <11/36>))}, 
{:candidate($(0, -1, <-71/18>)), :mp($[<1/18>, -1.0, 0.0]), :p($(6, -0.5, <11/36>))}, 
{:candidate($(6, 0.5, <-7/36>)), :mp($[<5/9>, 3.0, 0.0]), :p($(-6, 1, <5/9>))}, 
{:candidate($(-6, 0.5, <11/36>)), :mp($[<1/18>, 1.0, 0.0]), :p($(6, 0, <1/18>))}, 
{:candidate($(0, 1, <-71/18>)), :mp($[<1/18>, 1.0, 0.0]), :p($(6, 0, <1/18>))}, 
{:candidate($(-6, 0, <1/18>)), :mp($[<1/18>, -1.0, 0.0]), :p($(0, -1, <-71/18>))}, 
{:candidate($(6, -0.5, <11/36>)), :mp($[<1/18>, -1.0, 0.0]), :p($(0, -1, <-71/18>))}, 
{:candidate($(6, -1, <5/9>)), :mp($[<5/9>, -3.0, 0.0]), :p($(-6, -0.5, <-7/36>))}, 
{:candidate($(-6, 0.5, <11/36>)), :mp($[<1/18>, 1.0, 0.0]), :p($(0, 1, <-71/18>))}, 
{:candidate($(6, 0, <1/18>)), :mp($[<1/18>, 1.0, 0.0]), :p($(0, 1, <-71/18>))}, 
{:candidate($(-6, 1, <5/9>)), :mp($[<5/9>, 3.0, 0.0]), :p($(6, 0.5, <-7/36>))}, 
{:candidate($(0, -1, <-71/18>)), :mp($[<1/18>, -1.0, 0.0]), :p($(-6, 0, <1/18>))}, 
{:candidate($(6, -0.5, <11/36>)), :mp($[<1/18>, -1.0, 0.0]), :p($(-6, 0, <1/18>))}, 
{:candidate($(-6, -0.5, <-7/36>)), :mp($[<5/9>, -3.0, 0.0]), :p($(6, -1, <5/9>))}, 
{:candidate($(0, 1, <-71/18>)), :mp($[<1/18>, 1.0, 0.0]), :p($(-6, 0.5, <11/36>))}, 
{:candidate($(6, 0, <1/18>)), :mp($[<1/18>, 1.0, 0.0]), :p($(-6, 0.5, <11/36>))};
```

Here we find all records for the gathered co-domain points and keeping the groups with more than three different points:

```raku
my %classes = @found.classify(*<mp>.Str).grep(*.value.elems ≥ 3).sort(*.key);
deduce-type(%classes)
```

```
# Struct([0.055556 -1 0, 0.055556 1 0], [Array, Array])
```

Combine the points of each record group into one array of _unique_ points:

```raku
my @point-groups = %classes.values.map({ $_.map(*<p candidate>).map(*.Slip).unique(with => {(@^a <<->> @^b)».abs.max ≤ 0.0001}) });
@point-groups».elems
```

```
# (3 3)
```

Check the polynomial mapping gives the same co-domain point per group:

```raku
@point-groups.map(*.map({ &poly-map(|$_) })).map({ [eqv] $_})
```

```
# (True True)
```

---

## References

### Articles, blog posts

[AA1] Anton Antonov, ["Chatbook New Magic Cells"](https://rakuforprediction.wordpress.com/2024/05/18/chatbook-new-magic-cells/), (2024), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com). 

[MS1] Matthew Sparkes, ["AI's solution to 87-year-old riddle takes mathematicians by surprise"](https://www.newscientist.com/article/2580374-ais-solution-to-87-year-old-riddle-takes-mathematicians-by-surprise/), (2026), New Scientist.

[MW1] MathWorld entry, ["Jacobian conjecture"](https://mathworld.wolfram.com/JacobianConjecture.html).

[Wk1] Wikipedia entry, ["Jacobian conjecture"](https://en.wikipedia.org/wiki/Jacobian_conjecture).

### Notebooks

[AAn1] Anton Antonov, ["Jacobian conjecture counterexample by LLM"](https://community.wolfram.com/groups/-/m/t/3766129), (2026), [Wolfram Community](https://community.wolfram.com). 

### Packages 

[AAp1] Anton Antonov, ["CortexJS"](https://github.com/antononcube/Raku-CortexJS), (2026), [GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, ["LaTeX::Grammar"](https://github.com/antononcube/Raku-LaTeX-Grammar), (2026), [GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov, ["Proc::ZMQed"](https://github.com/antononcube/Raku-Proc-ZMQed), (2022), [GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov, ["WWW::WolframAlpha"](https://github.com/antononcube/Raku-WWW-WolframAlpha), (2024), [GitHub/antononcube](https://github.com/antononcube).