# Pi Day 2026: Formulas, Series, and Plots for π

***...Explored in Raku***


## Introduction

- Happy Pi Day! Today (3/14) we celebrate the most famous mathematical constant: π ≈ 3.141592653589793…
- π is irrational and transcendental, appears in circles, waves, probability, physics, and even random walks.
- Raku (with its built-in `π` constant, excellent rational support, lazy lists, and unicode operators) makes experimenting with π especially enjoyable.
- In this blog post ([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Pi-day-celebration.ipynb)) we explore a selection of beautiful formulas and clever algorithms — with short Raku snippets to try yourself.


---

## 0. Setup

```raku
use Math::NumberTheory;

use Image::Markup::Utilities;
use Graphviz::DOT::Chessboard;

use Data::Reshapers;
use Data::Importers;
use Data::Summarizers;
use Data::TypeSystem;
use Statistics::Distributions;

use JavaScript::D3;
use JavaScript::D3::Utilities;
```

### D3.js

```raku
#%javascript
require.config({
     paths: {
     d3: 'https://d3js.org/d3.v7.min'
}});

require(['d3'], function(d3) {
     console.log(d3);
});
```

```raku
my $title-color = 'Ivory';
my $stroke-color = 'SlateGray';
```

----

## 1. Continued fraction approximation

The built-in Raku constant `pi` (or `π`) is fairly low precision:

```raku
say π.fmt('%.25f')
```

```
# 3.1415926535897930000000000

```

One way to remedy that is to use continued fractions. For example, using the (first) sequence line of On-line Encyclopedia of Integer Sequences (OEIS) [A001203](https://oeis.org/A001203) produces $\pi$ with precision 56:

```raku
my @s = 3, 7, 15, 1, 292, 1, 1, 1, 2, 1, 3, 1, 14, 2, 1, 1, 2, 2, 2, 2, 1, 84, 2, 1, 1, 15, 3, 13, 1, 4, 2, 6, 6, 99, 1, 2, 2, 6, 3, 5, 1, 1, 6, 8, 1, 7, 1, 2, 3, 7, 1, 2, 1, 1, 12, 1, 1, 1, 3, 1, 1, 8, 1, 1, 2, 1, 6, 1, 1, 5, 2, 2, 3, 1, 2, 4, 4, 16, 1, 161, 45, 1, 22, 1, 2, 2, 1, 4, 1, 2, 24, 1, 2, 1, 3, 1, 2, 1;

my $pi56 = from-continued-fraction(@s».FatRat.List);
```

```
# 3.14159265358979323846264338327950288419716939937510582097
```

Here we verify the precision using Wolfram Language:

```raku
"wolframscript -code 'N[Pi, 100] - $pi56'"
andthen .&shell(:out)
andthen .out.slurp(:close)
```

```
# 0``56.

```

More details can be found in Wolfram MathWorld page ["Pi Continued Fraction"](https://mathworld.wolfram.com/PiContinuedFraction.html), [EW1].

---

## 2. Continued fraction terms plots

It is interesting to consider the plotting the terms of continued fraction terms of $\pi$. 

First we ingest the more "pi-terms" from OEIS [A001203](https://oeis.org/A001203) (20k terms):

```raku
my @ds = data-import('https://oeis.org/A001203/b001203.txt').split(/\s/)».Int.rotor(2);
my @terms = @ds».tail;
@terms.elems
```

```
# 20000
```

Here is the summary:

```raku
sink records-summary(@terms)
```

```
# +-------------------+
# | numerical         |
# +-------------------+
# | 1st-Qu => 1       |
# | Median => 2       |
# | Min    => 1       |
# | Max    => 20776   |
# | Mean   => 12.6809 |
# | 3rd-Qu => 5       |
# +-------------------+

```

Here is an array plot of the first 128 terms of the continued fraction approximating $\pi$: 

```raku
#% html
my @mat = |@terms.head(128)».&integer-digits(:2base);
my $max-digits = @mat».elems.max;
@mat .= map({ [|(0 xx ($max-digits - $_.elems)), |$_] });
dot-matrix-plot(transpose(@mat), size => 10):svg
```

<img src="https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Pi-day-celebration/pi-continued-fraction-terms-128.svg" alt="cell 26 output 1 svg 1">

Next, we show the Pareto principle manifestation of for the continued fraction terms. First we observe that the terms a distribution similar to [Benford's law](https://en.wikipedia.org/wiki/Benford%27s_law):

```raku
#% js
my @tally-pi = tally(@terms).sort(-*.value).head(16) <</>> @terms.elems;
my @terms-b = random-variate(BenfordDistribution.new(:10base), 2_000);
my @tally-b = tally(@terms-b).sort(-*.value).head(16) <</>> @terms-b.elems;

js-d3-bar-chart(
    [
        |@tally-pi.map({ %( x => $_.key, y => $_.value, group => 'π') }),
        |@tally-b.map({ %( x => $_.key, y => $_.value, group => 'Benford') })
    ],
    plot-label => "Pi continued fraction terms vs. Benford's law",
    :$title-color,
    :$background)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Pi-day-celebration/continued-fraction-terms-vs-benford-law.png)

Here is the Pareto principle plot -- ≈5% of the unique term values correspond to ≈80% of the terms:

```raku
#% js
js-d3-list-line-plot(
    pareto-principle-statistic(@terms), 
    plot-label => "Pareto principle for Pi continued fraction terms",
    :$title-color,
    :$background,
    stroke-width => 5,
    :grid-lines
)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Pi-day-celebration/pareto-principle-for-continued-fraction-terms.png)

----

## 3. Classic Infinite Series

Many ways to express π as an infinite sum — some converge slowly, others surprisingly fast.

**Leibniz–Gregory series** (1671/ Madhava earlier)

$$
\pi = 4 \sum_{n=0}^{\infty} \frac{(-1)^n}{2n+1} = 4 \left(1 - \frac{1}{3} + \frac{1}{5} - \frac{1}{7} + \cdots \right)
$$

Raku implementation:

```raku
sub pi-leibniz($n) {
    4 * [+] map { ($_ %% 2 ?? 1 !! -1) / (2 * $_.FatRat + 1) }, 0 ..^ $n
}

my $piLeibniz = pi-leibniz(1_000);

```

```
# 3.140592653839792925963596502869395970451389330779724489367457783541907931239747608265172332007670207231403885276038710899938066629552214564551237742887150050440512339302537072825852760246628025562008569471700451065826106184744099667808080815231833582150382088582680381403109153574884416966097481526954707518119416184546424446286573712097944309435229550466609113881892172898692240992052089578302460852737674933105951137782047028552762288434104643076549100475536363928011329215789260496788581009721784276311248084584199773204673225752150684898958557383759585526225507807731149851003571219339536433193219280858501643712664329591936448794359666472018649604860641722241707730107406546936464362178479780167090703126423645364670050100083168338273868059379722964105943903324595829044270168232219388683725629678859726914882606728649659763620568632099776069203461323565260334137877
```

Verify with Wolfram Language (again):

```raku
"wolframscript -code 'N[Pi, 1000] - $piLeibniz'"
andthen .&shell(:out)
andthen .out.slurp(:close)

```

```
# 0.0009999997500003124990468804101069137457800685953813316074868087659084750464613903628624933344468607507442012372435957471557779799983676671741216652413310670097717633994014482797068843763209293683733949571105246001073399437315485659787023500393683269952664783407799672073451730733289766411627676143190170688871901310417504352343472507313097822801348174259812461294383576501282255293320573736390507566953823372598084541710451575646196441776884620419989739868435988215560226699634643944393571732901648535224252778564568698813481691942445398382321447961013581765450314094451433257488134554789312362119127197096255015508964981938348939634299427016185291166077991789832457000391430384345864301012094787564513168296884836572908139343349914753559067119302375546012674466754025439337797828275123441323706889161647325404643015739928625446327760529354858619847096864378775046150095875332083814206`866.9999998914263

```

**Nilakantha series** (faster convergence):

$$
\pi = 3 + \sum_{n=1}^{\infty} \frac{(-1)^{n+1} \cdot 4}{(2n)(2n+1)(2n+2)}
$$

Raku:

```raku
sub pi-nilakantha($n) {
    3 + [+] map {
        ($_ %% 2 ?? -1 !! 1 ) * 4 / ((2 * $_.FatRat) * (2 * $_ + 1) * (2 * $_ + 2))
    }, 1 .. $n
}

pi-nilakantha(1_000);
```

```
# 3.141592653340542051900128736253203567152539255317954874674304859504426172618558702218695071137605738966036069683335561974900086119307836254205910905806190030949758215864755464129701335459521079534522811851010296642538249613529207613335816447914992502190861349451746347920350033634355181084537761886275546599078437173552420948534950023442771396391252038722980428723971632669306434394851189528826699233048019261441283970866004550291393472342649870962106821115715774722114776992400455398838055772839725805047379519366309217982783671029012753365224924699602163737619311405432798527164991008945233085366633073462699045511265528492985424805854418596455931463431855615794431867539190155631617285217459790661344075940516099637034367441911754544671168909454186231972510120715400925996293656987342326715209388299050131213232932065481743222390684073879385764855135985734675127240826
```

```raku
"wolframscript -code 'N[Pi, 1000] - {pi-nilakantha(1_000)}'"
andthen .&shell(:out)
andthen .out.slurp(:close)
```

```
# 2.492511865625146470262993170446301440571509463006397328033902336676502964093397542045113290161120168299467446721937584902427459775194485023222910865005258868371830569758583091634278753958591525778006790233951965149464406204509667173562094069547872174717140006901042930138582125227224872549981779879281691420357539329859709393864826953241123291696099312873588978795140321265329574231148123763850380089244756210870876358259884602691432341564412533466641880274525748933522694923898824101722233247862938430743474251032377239173824713719657774802587799651512278117434946495873936658732345840384989271054280997522952992927859179519351905771812472580365852854305215626073594297394335070622294682621979440155960293849264965925064052689586058618557742867347012292845974469817562940723894061008869302959743282421139762371607790858626606111558761663330350242684627115120605875332083814206`860.3966372344514*^-10

```

---

## 3. Beautiful Products

[**Wallis product** (1655)](https://en.wikipedia.org/wiki/Wallis_product) — elegant infinite product:

$$\frac{\pi}{2} = \prod_{n=1}^{\infty} \frac{(2n)^2}{(2n-1)(2n+1)} = \frac{2}{1} \cdot \frac{2}{3} \cdot \frac{4}{3} \cdot \frac{4}{5} \cdot \frac{6}{5} \cdot \frac{6}{7} \cdots$$

Raku running product:

```raku
my $p = 2.0;
for 1 .. 1_000 -> $n {
    $p *= (2 * $n) * (2 * $n) / ( (2 * $n - 1 ) * ( 2 * $n + 1) );
    say "$n → {$p / $piLeibniz} relative error" if $n %% 100;
}
```

```
# 100 → 0.9978331595460779 relative error
# 200 → 0.9990719099195204 relative error
# 300 → 0.9994865459690567 relative error
# 400 → 0.9996941876848563 relative error
# 500 → 0.9998188764663584 relative error
# 600 → 0.9999020455903246 relative error
# 700 → 0.9999614733132168 relative error
# 800 → 1.0000060557070767 relative error
# 900 → 1.0000407377794782 relative error
# 1000 → 1.000068487771041 relative error

```

----

## 4. Very Fast Modern Series — [Chudnovsky Algorithm](https://en.wikipedia.org/wiki/Chudnovsky_algorithm)

One of the fastest-converging series used in record computations:

$$
\frac{1}{\pi} = 12 \sum_{k=0}^{\infty} \frac{(-1)^k (6k)! (13591409 + 545140134k)}{(3k)! (k!)^3 640320^{3k + 3/2}}
$$

Each term adds roughly 14 correct digits. Cannot be implemented easily in Raku, since Raku does not have bignum `sqrt` and `power` operations.

---

## 5. Spigot Algorithms — Digits “Drip” One by One

[Spigot algorithms](https://en.wikipedia.org/wiki/Spigot_algorithm) compute decimal digits using only integer arithmetic — no floating-point errors accumulate.

The classic [**Rabinowitz–Wagon spigot**](http://stanleyrabinowitz.com/download/spigot-revised.pdf) (based on a transformed Wallis product) produces base-10 digits sequentially.

Simple (but bounded) version outline in Raku:

```raku
sub spigot-pi($digits) {
    my $len = (10 * $digits / 3).floor + 1;
    my @a = 2 xx $len;

    my @result;
    for 1..$digits {
        my $carry = 0;
        for $len-1 ... 0 -> $i {
            my $x = 10 * @a[$i] + $carry * ($i + 1);
            @a[$i] = $x % (2 * $i + 1);
            $carry = $x div (2 * $i + 1);
        }
        @result.push($carry div 10);
        @a[0] = $carry % 10;
        # (handle carry-over / nines adjustment in full impl)
    }
    @result.head(1).join('.') ~ @result[1..*].join
}

spigot-pi(50);
```

```
# 314159265358979323846264338327941028841971693993751
```

```raku
"wolframscript -code 'N[Pi, 100] - {spigot-pi(50).FatRat / 10e49.FatRat}'"
andthen .&shell(:out)
andthen .out.slurp(:close)
```

```
# 2.3969628881355243801510070603398913366797194459230781640628621`41.37966130996076*^-16

```

---

## 6. [BBP Formula](https://mathworld.wolfram.com/BBPFormula.html) — Hex Digits Without Predecessors

[Bailey–Borwein–Plouffe (1995) formula](https://en.wikipedia.org/wiki/Bailey–Borwein–Plouffe_formula) lets you compute the nth hexadecimal digit of π directly (without earlier digits):

$$
\pi = \sum_{k=0}^{\infty} \left[ \frac{1}{16^k} \left( \frac{4}{8k+1} - \frac{2}{8k+4} - \frac{1}{8k+5} - \frac{1}{8k+6} \right) \right]
$$

Very popular for distributed π-hunting projects. The best known [digit-extraction algorithm](https://mathworld.wolfram.com/Digit-ExtractionAlgorithm.html).

Raku snippet for partial sum (base 16 sense):

```raku
sub bbp-digit-sum($n) {
    [+] (0..$n).map: -> $k {
        my $r = 1/16**$k;
        $r * (4/(8*$k+1) - 2/(8*$k+4) - 1/(8*$k+5) - 1/(8*$k+6))
    }
}
say bbp-digit-sum(100).base(16).substr(0,20);
```

```
# 3.243F6B

```

---

## 7. (Instead of) Conclusion

- π contains (almost surely) every finite sequence of digits — your birthday appears infinitely often.
- [The Feynman point](https://en.wikipedia.org/wiki/Six_nines_in_pi): six consecutive 9s starting at digit 762.
- Memorization world record > 100,000 digits.
- π appears in the normal distribution, quantum mechanics, random walks, Buffon's needle problem (probability ≈ 2/π).


Let us plot a random walk using the terms of continued fraction of Pi -- the 20k or OEIS [A001203](https://oeis.org/A001203) -- to determine directions:

```raku
#% js
my @path = angle-path(@terms)».reverse».List;
my &pi-path-map = { 
    given @terms[$_] // 0 { 
        when $_ ≤ 100 { 0 }
        when $_ ≤ 1_000 { 1 }
        default { 2 } 
    } 
}
@path = @path.kv.map( -> $i, $p {[|$p, &pi-path-map($i).Str]});
my %opts = color-scheme => 'Observable10', background => '#1F1F1F', :!axes, :!legends, stroke-width => 2;
js-d3-list-line-plot(@path, :800width, :500height, |%opts)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Pi-day-celebration/random-walk-continued-fraction-terms-dark-mode.png)

In the plot above the blue segments correspond to origin terms ≤ 100, yellow segments to terms between 100 and 1000, and red segment for origin terms greater than 1000. 

---

## References

[EW1] Eric Weisstein, ["Pi Continued Fraction"](https://mathworld.wolfram.com/PiContinuedFraction.html), [Wolfram MathWorld](https://mathworld.wolfram.com).