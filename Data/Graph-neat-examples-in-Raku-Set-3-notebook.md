---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.1
  kernelspec:
    display_name: RakuChatbook
    language: raku
    name: raku
---

# Graph neat examples in Raku

<span style="font-size: 16pt; font-style: italic; font-weight: bold">Set 3</span>

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
November 2024


-----

## Introduction


**What is a neat example?** : Concise or straightforward code that produces compelling visual or textual outputs.


**Maybe:** We know *neat* when we see it?


The neat examples:

- Showcase Raku programming.
- Use functionalities of different Raku modules.
- Give interesting perspectives on what is computationally possible.


Showcased:
- All computational graph features discussed here are provided by ["Graph"](https://raku.land/zef:antononcube/Graph).   
- Graph plotting with:
    - `js-d3-graph-plot`, provided by ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3).
    -  `Graph.dot`, that makes SVG images via [Graphviz](https://graphviz.org).


------

## Setup


Here are loaded the packages used in the rest of notebook:

```raku
use Graph;

use Graph::Circulant;
use Graph::Complete;
use Graph::CompleteKaryTree;
use Graph::Cycle;
use Graph::Grid;
use Graph::HexagonalGrid;
use Graph::Hypercube;
use Graph::KnightTour;
use Graph::Nested;
use Graph::Path;
use Graph::Petersen;
use Graph::Star;
use Graph::TriangularGrid;
use Graph::Wheel;

use Graph::Distribution;
use Graph::Random;

use Data::Reshapers;
use Data::Summarizers;
use Data::Generators;
use Data::TypeSystem;
use Data::Translators;
use Data::Geographics;

use Math::DistanceFunctions;
use Math::Nearest;
use Text::Levenshtein::Damerau;

use Hash::Merge;
use FunctionalParsers;
use FunctionalParsers::EBNF;
use EBNF::Grammar;
use Graphviz::DOT::Grammar;

use JavaScript::D3;
use WWW::MermaidInk;

use paths;
```

### JavaScript


Here we prepare the notebook to visualize with JavaScript:

```raku
#% javascript
require.config({
     paths: {
     d3: 'https://d3js.org/d3.v7.min'
}});

require(['d3'], function(d3) {
     console.log(d3);
});
```

Verification:

```raku
#% js
js-d3-list-line-plot(10.rand xx 40, background => 'none', stroke-width => 2)
```

Here we set a collection of visualization variables:

```raku
my $title-color = 'Ivory';
my $tick-labels-color = 'Ivory';
my $stroke-color = 'SlateGray';
my $tooltip-color = 'LightBlue';
my $tooltip-background-color = 'none';
my $background = '#1F1F1F';
my $color-scheme = 'schemeTableau10';
my $edge-thickness = 3;
my $vertex-size = 6;
my $engine = 'neato';
my $mmd-theme = q:to/END/;
%%{
  init: {
    'theme': 'forest',
    'themeVariables': {
      'lineColor': 'Ivory'
    }
  }
}%%
END
my %force = collision => {iterations => 0, radius => 10},link => {distance => 180};
my %force2 = charge => {strength => -30, iterations => 4}, collision => {radius => 50, iterations => 4}, link => {distance => 30};

my %opts = :$background, :$title-color, :$edge-thickness, :$vertex-size;
```

----

## Nested graphs


Basic nested graph example:

```raku
#% html

my $g1 = Graph::Nested.new({$_ ** 2}, 2, 3, :directed);

my $g2 = Graph::Nested.new({"f($_)"}, 'x', 3, :directed);

$g1.dot(:$background, engine => 'dot', vertex-shape => 'ellipse', vertex-width => 0.75, :svg)
~
$g2.dot(:$background, engine => 'dot', vertex-shape => 'ellipse', vertex-width => 0.75, :svg)
```

### Binary tree


More interesting version of the basic graph above:

```raku
my $g = Graph::Nested.new({["f($_)", "g($_)"]}, 'x', 3, :directed)
```

```raku
#%html
$g.dot(node-width => 0.75, node-height => 0.3, node-shape => 'rect', engine => 'dot'):svg
```

```raku
#%js
$g.edges ==>
    js-d3-graph-plot(
        width => 800,
        :$background, 
        :$title-color,
        vertex-size => 6,
        vertex-fill-color => 'SlateBlue',
        vertex-label-font-size => 18,
        edge-thickness => 3,
        directed => $g.directed,
        force => {charge => {strength => -600, iterations => 4}, collision => {radius => 20, iterations => 4}, link => {distance => 60}}
    )
```

### Mod graph


Mod graph using a range of numbers:

```raku
my $g = Graph::Nested.new({($_.Int ** 2 + 1) mod 10}, ^10, :directed)
```

```raku
#%html
$g.dot(node-width => 0.3, node-height => 0.3, node-shape => 'circle', engine => 'sfdp'):svg
```

### Range graph


Similar idea using an integer range:

```raku
my $g = Graph::Nested.new({^$_}, '9', 2, :directed)
```

```raku
#%html
$g.dot(node-width => 0.4, node-height => 0.4, node-font-size => 18, engine => 'dot', size => (6, 6)):svg
```

```raku
$g.undirected-graph.is-complete
```

```raku
say 'in  : ', $g.vertex-in-degree(:pairs);
say 'out : ', $g.vertex-out-degree():p;
```

```raku
#%js
$g.edges ==>
    js-d3-graph-plot(
        #vertex-coordinates => Graph::Cycle.new(10).vertex-coordinates,
        :directed,
        width => 600,
        edge-thickness => 3,
        vertex-size => 5,
        vertex-color => 'SlateBlue',
        :$background, 
        :$title-color,
        force => {charge => {strength => -400, iterations => 4}, collision => {radius => 50, iterations => 4}, link => {distance => 60}}
    )
```

----

## File system graphs


### RakuMode notebooks


Get a list of file paths from a certain directory (using the package ["paths"](https://raku.land/zef:lizmat/paths)):

```raku
my @paths = paths($*HOME ~ "/MathFiles/RakuForPrediction");
my @paths2 = @paths>>.subst($*HOME.Str)>>.split("/", :skip-empty);
@paths2.elems
```

Make the edges of the corresponding graph (tree) using the split file names: 

```raku
my @edges = @paths2.map({ $_.rotor(2 => -1).map({ $_.head => $_.tail }) }).map(*.Slip).unique(:as({.Str}));
my $g = Graph.new(@edges, :directed)
```

Plot the graph using the Graphviz DOT language:

```raku
#%html
my $preamble = q:to/END/;
fontcolor = "Ivory";
fontsize = "12";
labelloc = "t";
label = "Directory paths";
graph [size="8,8!"];

bgcolor="#1F1F1F";
node [style=filled, fixedsize=true, shape=circle, color="Black", fillcolor="SlateBlue", penwidth=1, fontsize=4, fontcolor="White", labelloc=c, width=0.08, height=0.08];
edge [color="SteelBlue", penwidth=0.6, arrowsize=0.4];
END

$g.dot(:$preamble, engine=>'twopi'):svg;
```

### Raku-doc files


Here is a larger graph:

```raku
my @paths = paths($*HOME ~ "/Downloads/doc/doc");
my @paths2 = @paths>>.subst($*HOME.Str)>>.split("/", :skip-empty);
my @edges = @paths2.map({ $_.rotor(2 => -1).map({ $_.head => $_.tail }) }).map(*.Slip).unique(:as({.Str}));
my $g2 = Graph.new(@edges, :directed)
```

Graph plot via [D3.js `d3-force`](https://d3js.org/d3-force):

```raku
#%js
$g2.edges ==>
    js-d3-graph-plot(
        width => 1100,
        height => 400,
        :$background, 
        :$title-color,
        vertex-size => 3,
        vertex-fill-color => 'SlateBlue',
        vertex-label-font-size => 10,
        vertex-label-color => 'none',
        edge-thickness => 1,
        directed => $g.directed,
        force => {charge => {strength => -50, iterations => 1}, y => {strength => 0.4}, collision => {radius => 2, iterations => 1}, link => {distance => 1}}
    )
```
