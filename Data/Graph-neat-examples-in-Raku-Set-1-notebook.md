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

### ***Set 1***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
July 2024


----

## Introduction


**What is a neat example?** : Concise or straightforward code that produces compelling visual or textual outputs.


**Maybe:** We know *neat* when we see it?


The neat examples:

- Showcase Raku programming.
- Use functionalities of different Raku modules.
- Give interesting perspectives on what is computationally possible.


Showcased:
- All computational graph features discussed here are provided by ["Graph"](https://raku.land/zef:antononcube/Graph).   
- Graph plotting -- with `js-d3-graph-plot` -- is provided by ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3).
- ≈ 85,000 English words are provided ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators).
- Nearest neighbors computations are provided ["Math::Nearest"](https://raku.land/zef:antononcube/Math::Nearest).
- The function `dld` is provided by ["Text::Levenshtein::Damerau"](https://raku.land/github:ugexe/Text::Levenshtein::Damerau).
- Geographical data and `geo-distance` are provided by ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics).
- The function `cross-tabulate` is provided by ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers).


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
use Graph::Hypercube;
use Graph::KnightTour;
use Graph::Star;
use Graph::Path;
use Graph::Petersen;
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

use JavaScript::D3;
use WWW::MermaidInk;
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
my $stroke-color = 'SlateGray';
my $tooltip-color = 'LightBlue';
my $tooltip-background-color = 'none';
my $background = '1F1F1F';
my $color-scheme = 'schemeTableau10';
my $edge-thickness = 3;
my $vertex-size = 6;
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

--------

## Mod graph


Make square-&-mod edges and corresponding graph:

```raku
my @redges = (^100).map({ $_.Str => (($_ ** 2) mod 74).Str });
my $gMod = Graph.new(@redges, :directed)
```

Plot mod graph:

```raku
#%js
js-d3-graph-plot(
    $gMod.edges(:dataset),
    :$background, :$title-color, 
    edge-thickness => 3, 
    vertex-size => 4,
    vertex-color => 'SlateBlue',
    directed => $gMod.directed,
    title => 'Mod 74 graph', 
    width => 800,
    height => 600, 
    force => {charge => {strength => -100}, y => {strength => 0.12}, link => {minDistance => 4}}
    )
```

Find graphs *weakly* connected components:

```raku
.say for $gMod.weakly-connected-components
```

Plot the components (using `.subgraph`):

```raku
#% js
$gMod.weakly-connected-components.pairs.map({
    js-d3-graph-plot(
        $gMod.subgraph($_.value).edges(:dataset),
        title => ($_.key + 1).Str,
        :$background, :$title-color, :$edge-thickness, vertex-size => 4,
        vertex-color => 'SlateBlue',
        vertex-label-color => 'none',
        force => { charge => {strength => -70}},
    )
}).join("\n")
```

-----

## Dictionary graph


Take all English words from the package ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators):

```raku
my $ra = Data::Generators::ResourceAccess.instance();
$ra.get-word-data().elems;
```

Those are used by `random-word`:

```raku
random-word(4, type => 'common')
```

Here is a sample:

```raku
.say for $ra.get-word-data().pick(4)
```

Find words with the prefix "rac":

```raku
my @words = $ra.get-word-data.keys.grep({ $_.starts-with('rac'):i });
@words.elems
```

Compute the corresponding nearest neighbors graph using the two closest neighbors for each word:

```raku
my @nnEdges = nearest-neighbor-graph(@words, 2, distance-function => &dld);
```

Make the corresponding directed graph:

```raku
my $gDict = Graph.new(@nnEdges, :directed)
```

Plot it:

```raku
#%js
js-d3-graph-plot(
    $gDict.edges(:dataset),
    highlight => <raccoon racoon>,
    :$background, :$title-color, :$edge-thickness, 
    vertex-size => 5,
    vertex-color => 'Blue',
    directed => $gDict.directed,
    title => "«rac» graph", 
    width => 1200,
    height => 650, 
    force => {charge => {strength => -400}, y => {strength => 0.2}, link => {minDistance => 6}}
)
```

**Remark:** We can derive graph's edges ad hoc, without using `nearest-neighbor-graph`:

```raku
# Construct on nearest finder object over them
my &wn = nearest(@words, distance-function => &dld);

# For each word find the closest two words and make edge rules of the corresponding pairs:
my @redges = @words.map({ $_ <<=><< &wn($_, 3).flat.grep(-> $x { $x ne $_ }) }).flat;
deduce-type(@redges)
```

**Remark:** `nearest` is provided by ["Math::Nearest"](https://raku.land/zef:antononcube/Math::Nearest).


------

## African centers


Build an interstate highway system joining the geographical centers of all African countries:


Assign coordinates of the geographical centers and summarize them:

```raku
my %africaCoords = 'Algeria'=>[3.0,28.0],'Libya'=>[17.0,25.0],'Mali'=>[-4.0,17.0],'Mauritania'=>[-12.0,20.0],'Morocco'=>[-5.0,32.0],'Niger'=>[8.0,16.0],'Tunisia'=>[9.0,34.0],'WesternSahara'=>[-13.0,24.5],'Chad'=>[19.0,15.0],'Egypt'=>[30.0,27.0],'Sudan'=>[30.0,13.8],'BurkinaFaso'=>[-2.0,13.0],'Guinea'=>[-10.0,11.0],'IvoryCoast'=>[-5.0,8.0],'Senegal'=>[-14.0,14.0],'Benin'=>[2.25,9.5],'Nigeria'=>[8.0,10.0],'Cameroon'=>[12.0,6.0],'CentralAfricanRepublic'=>[21.0,7.0],'Eritrea'=>[39.0,15.0],'Ethiopia'=>[38.0,8.0],'SouthSudan'=>[30.51,6.7],'Ghana'=>[-2.0,8.0],'Togo'=>[1.1667,8.0],'GuineaBissau'=>[-15.0,12.0],'Liberia'=>[-9.5,6.5],'SierraLeone'=>[-11.5,8.5],'Gambia'=>[-16.5667,13.4667],'EquatorialGuinea'=>[10.0,2.0],'Gabon'=>[11.75,-1.0],'RepublicCongo'=>[15.0,-1.0],'DemocraticRepublicCongo'=>[25.0,0.0],'Djibouti'=>[43.0,11.5],'Kenya'=>[38.0,1.0],'Somalia'=>[49.0,10.0],'Uganda'=>[32.0,1.0],'Angola'=>[18.5,-12.5],'Burundi'=>[30.0,-3.5],'Rwanda'=>[30.0,-2.0],'Tanzania'=>[35.0,-6.0],'Zambia'=>[30.0,-15.0],'Namibia'=>[17.0,-22.0],'Malawi'=>[34.0,-13.5],'Mozambique'=>[35.0,-18.25],'Botswana'=>[24.0,-22.0],'Zimbabwe'=>[30.0,-20.0],'SouthAfrica'=>[24.0,-29.0],'Swaziland'=>[31.5,-26.5],'Lesotho'=>[28.5,-29.5];
sink records-summary(%africaCoords.values)
```

Make a corresponding graph:

```raku
my @africaEdges = %('from'=>'Algeria','to'=>'Libya','weight'=>1),%('from'=>'Algeria','to'=>'Mali','weight'=>1),%('from'=>'Algeria','to'=>'Mauritania','weight'=>1),%('from'=>'Algeria','to'=>'Morocco','weight'=>1),%('from'=>'Algeria','to'=>'Niger','weight'=>1),%('from'=>'Algeria','to'=>'Tunisia','weight'=>1),%('from'=>'Algeria','to'=>'WesternSahara','weight'=>1),%('from'=>'Libya','to'=>'Niger','weight'=>1),%('from'=>'Libya','to'=>'Tunisia','weight'=>1),%('from'=>'Libya','to'=>'Chad','weight'=>1),%('from'=>'Libya','to'=>'Egypt','weight'=>1),%('from'=>'Libya','to'=>'Sudan','weight'=>1),%('from'=>'Mali','to'=>'Mauritania','weight'=>1),%('from'=>'Mali','to'=>'Niger','weight'=>1),%('from'=>'Mali','to'=>'BurkinaFaso','weight'=>1),%('from'=>'Mali','to'=>'Guinea','weight'=>1),%('from'=>'Mali','to'=>'IvoryCoast','weight'=>1),%('from'=>'Mali','to'=>'Senegal','weight'=>1),%('from'=>'Mauritania','to'=>'WesternSahara','weight'=>1),%('from'=>'Mauritania','to'=>'Senegal','weight'=>1),%('from'=>'Morocco','to'=>'WesternSahara','weight'=>1),%('from'=>'Niger','to'=>'Chad','weight'=>1),%('from'=>'Niger','to'=>'BurkinaFaso','weight'=>1),%('from'=>'Niger','to'=>'Benin','weight'=>1),%('from'=>'Niger','to'=>'Nigeria','weight'=>1),%('from'=>'Chad','to'=>'Sudan','weight'=>1),%('from'=>'Chad','to'=>'Nigeria','weight'=>1),%('from'=>'Chad','to'=>'Cameroon','weight'=>1),%('from'=>'Chad','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Egypt','to'=>'Sudan','weight'=>1),%('from'=>'Sudan','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Sudan','to'=>'Eritrea','weight'=>1),%('from'=>'Sudan','to'=>'Ethiopia','weight'=>1),%('from'=>'Sudan','to'=>'SouthSudan','weight'=>1),%('from'=>'BurkinaFaso','to'=>'IvoryCoast','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Benin','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Ghana','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Togo','weight'=>1),%('from'=>'Guinea','to'=>'IvoryCoast','weight'=>1),%('from'=>'Guinea','to'=>'Senegal','weight'=>1),%('from'=>'Guinea','to'=>'GuineaBissau','weight'=>1),%('from'=>'Guinea','to'=>'Liberia','weight'=>1),%('from'=>'Guinea','to'=>'SierraLeone','weight'=>1),%('from'=>'IvoryCoast','to'=>'Ghana','weight'=>1),%('from'=>'IvoryCoast','to'=>'Liberia','weight'=>1),%('from'=>'Senegal','to'=>'GuineaBissau','weight'=>1),%('from'=>'Senegal','to'=>'Gambia','weight'=>1),%('from'=>'Benin','to'=>'Nigeria','weight'=>1),%('from'=>'Benin','to'=>'Togo','weight'=>1),%('from'=>'Nigeria','to'=>'Cameroon','weight'=>1),%('from'=>'Cameroon','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Cameroon','to'=>'EquatorialGuinea','weight'=>1),%('from'=>'Cameroon','to'=>'Gabon','weight'=>1),%('from'=>'Cameroon','to'=>'RepublicCongo','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'SouthSudan','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'RepublicCongo','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'Eritrea','to'=>'Ethiopia','weight'=>1),%('from'=>'Eritrea','to'=>'SouthSudan','weight'=>1),%('from'=>'Eritrea','to'=>'Djibouti','weight'=>1),%('from'=>'Ethiopia','to'=>'SouthSudan','weight'=>1),%('from'=>'Ethiopia','to'=>'Djibouti','weight'=>1),%('from'=>'Ethiopia','to'=>'Kenya','weight'=>1),%('from'=>'Ethiopia','to'=>'Somalia','weight'=>1),%('from'=>'SouthSudan','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'SouthSudan','to'=>'Kenya','weight'=>1),%('from'=>'SouthSudan','to'=>'Uganda','weight'=>1),%('from'=>'Ghana','to'=>'Togo','weight'=>1),%('from'=>'Liberia','to'=>'SierraLeone','weight'=>1),%('from'=>'EquatorialGuinea','to'=>'Gabon','weight'=>1),%('from'=>'Gabon','to'=>'RepublicCongo','weight'=>1),%('from'=>'RepublicCongo','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'RepublicCongo','to'=>'Angola','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Uganda','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Angola','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Burundi','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Rwanda','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Tanzania','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Zambia','weight'=>1),%('from'=>'Djibouti','to'=>'Somalia','weight'=>1),%('from'=>'Kenya','to'=>'Somalia','weight'=>1),%('from'=>'Kenya','to'=>'Uganda','weight'=>1),%('from'=>'Kenya','to'=>'Tanzania','weight'=>1),%('from'=>'Uganda','to'=>'Rwanda','weight'=>1),%('from'=>'Uganda','to'=>'Tanzania','weight'=>1),%('from'=>'Angola','to'=>'Zambia','weight'=>1),%('from'=>'Angola','to'=>'Namibia','weight'=>1),%('from'=>'Burundi','to'=>'Rwanda','weight'=>1),%('from'=>'Burundi','to'=>'Tanzania','weight'=>1),%('from'=>'Rwanda','to'=>'Tanzania','weight'=>1),%('from'=>'Tanzania','to'=>'Zambia','weight'=>1),%('from'=>'Tanzania','to'=>'Malawi','weight'=>1),%('from'=>'Tanzania','to'=>'Mozambique','weight'=>1),%('from'=>'Zambia','to'=>'Namibia','weight'=>1),%('from'=>'Zambia','to'=>'Malawi','weight'=>1),%('from'=>'Zambia','to'=>'Mozambique','weight'=>1),%('from'=>'Zambia','to'=>'Botswana','weight'=>1),%('from'=>'Zambia','to'=>'Zimbabwe','weight'=>1),%('from'=>'Namibia','to'=>'Botswana','weight'=>1),%('from'=>'Namibia','to'=>'SouthAfrica','weight'=>1),%('from'=>'Malawi','to'=>'Mozambique','weight'=>1),%('from'=>'Mozambique','to'=>'Zimbabwe','weight'=>1),%('from'=>'Mozambique','to'=>'SouthAfrica','weight'=>1),%('from'=>'Mozambique','to'=>'Swaziland','weight'=>1),%('from'=>'Botswana','to'=>'Zimbabwe','weight'=>1),%('from'=>'Botswana','to'=>'SouthAfrica','weight'=>1),%('from'=>'Zimbabwe','to'=>'SouthAfrica','weight'=>1),%('from'=>'SouthAfrica','to'=>'Swaziland','weight'=>1),%('from'=>'SouthAfrica','to'=>'Lesotho','weight'=>1);
my @africaEdges .= map({ %(from => $_<from>, to => $_<to>, weight => geo-distance(|%africaCoords{$_<from>}, |%africaCoords{$_<to>}, units => 'km' ).round) });

my $gAfrica = Graph.new(@africaEdges, :!directed)
```

Find the (minimum) spanning tree:

```raku
my $gAfricaTree = $gAfrica.find-spanning-tree
```

```raku
#% js
js-d3-graph-plot(
    $gAfrica.edges,
    highlight => [|$gAfricaTree.vertex-list, |$gAfricaTree.edges],
    vertex-coordinates => %africaCoords,
    title => 'Africa geographical centers',
    highlight-color => 'Orange',
    width => 700,
    height => 700,
    vertex-label-color => 'DimGray',
    margins => {right => 100, top => 80},
    :$background, :$title-color, :$edge-thickness, :$vertex-size,
    )
```

### Distances


Cross-tabulate countries and distances:

```raku
my @tbl = (%africaCoords X %africaCoords).map({ %( from => $_.head.key, to => $_.tail.key, weight => &geo-distance($_.head.value, $_.tail.value, units => 'miles').round(0.1) ) });
my @ct = cross-tabulate(@tbl, 'from', 'to', 'weight').sort(*.key);

deduce-type(@ct)
```

**Remark:** `cross-tabulate` is provided by ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers).


Show table:

```raku
#% html
@ct.map({ ['from' => $_.key , |$_.value].Hash }) ==> to-html(field-names => ['from', |@ct>>.key])
```

Graph based on the distance table:

```raku
my $gAfricaComplete = Graph.new(@tbl)
```

Edge count verification:

```raku
$gAfricaComplete.vertex-list.combinations(2).elems + $gAfricaComplete.vertex-count
```

Spanning tree:

```raku
my $gAfricaTree2 = $gAfricaComplete.find-spanning-tree
```

Plot graph:

```raku
#% js
js-d3-graph-plot(
    $gAfricaTree2.edges,
    highlight => [|$gAfricaTree2.vertex-list, |$gAfricaTree2.edges],
    vertex-coordinates => %africaCoords,
    title => 'Africa geographical centers',
    highlight-color => 'Orange',
    width => 700,
    height => 700,
    vertex-label-color => 'Silver',
    margins => {right => 100, top => 80},
    :$background, :$title-color, :$edge-thickness, :$vertex-size,
    )
```

-----

## Bulgarian cities


Build an interstate highway system joining the cities of Bulgaria.

```raku
city-data().map(*<Country>).unique.sort
```

Here is a dataset of large enough cities:

```raku
my $maxPop = 10_000;
my @cities = city-data().grep({ $_<Country> eq 'Bulgaria' && $_<Population> ≥ $maxPop });
@cities.elems
```

**Remark:** Geographical data and geo-distance computation is provided by ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics).

```raku
#% html
@cities.head(4) 
==> to-html(field-names => <ID Country State City Latitude Longitude Elevation LocationLink>)
```

Here is the corresponding weighted **complete** graph:

```raku
my %coords = @cities.map({ $_<City> => ($_<Latitude>, $_<Longitude>) });
my @dsEdges = (%coords.keys X %coords.keys ).map({ %(from => $_.head, to => $_.tail, weight => geo-distance(|%coords{$_.head}, |%coords{$_.tail} )) });
my $gGeo = Graph.new(@dsEdges, :!directed)
```

Here is the corresponding spanning tree:

```raku
my $stree = $gGeo.find-spanning-tree(method => 'kruskal')
```

```raku
#% js
$stree.edges
==> js-d3-graph-plot(
    vertex-coordinates => %coords.nodemap(*.reverse)».List.Hash,
    title => "Bulgaria, cities with population ≥ $maxPop",
    width => 1400,
    height => 700,
    :$background, :$title-color, :$edge-thickness, 
    vertex-size => 4,
    vertex-label-font-size => 8,
    vertex-label-font-family => Whatever,
    margins => {right => 200}
    )
```
