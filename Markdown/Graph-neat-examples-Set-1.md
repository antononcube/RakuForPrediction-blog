# Graph Neat Examples in Raku - Set 1

Anton Antonov  
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)  
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)  
July 2024

---

## Introduction

In this blog post, we explore some neat examples of graph computations using Raku, a member of the Perl family of languages. These examples are designed to be concise, straightforward, and compelling, showcasing the capabilities of Raku and its modules.

### What is a neat example?

A neat example is a piece of code that is both concise and straightforward, producing visually or textually compelling outputs. These examples are intended to:

- Showcase Raku programming.
- Utilize functionalities from various Raku modules.
- Provide interesting perspectives on computational possibilities.

Modules featured in this post include:

- ["Graph"](https://raku.land/zef:antononcube/Graph) for graph features.
- ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3) for graph plotting using D3.js.
- ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators) for English words.
- ["Math::Nearest"](https://raku.land/zef:antononcube/Math::Nearest) for nearest neighbor computations.
- ["Text::Levenshtein::Damerau"](https://raku.land/github:ugexe/Text::Levenshtein::Damerau) for distance calculations.
- ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics) for geographical data.
- ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers) for cross-tabulation.

---

## Mod Graph

The first example involves creating a mod graph. We take integers from 0 to 99, square each, and find the remainder when divided by 74. This forms the edges of our graph.

```raku
my @redges = (^100).map({ $_.Str => (($_ ** 2) mod 74).Str });
my $gMod = Graph.new(@redges, :directed)
```

The graph is plotted using the `js-d3-graph-plot` function from the "JavaScript::D3" module, which leverages the D3.js library for visualization.

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

Next, we find the weakly connected components of the mod graph. A weakly connected component in a directed graph is a maximal subgraph where every pair of vertices is connected by an undirected path.

```raku
.say for $gMod.weakly-connected-components
```

We plot each component using subgraphs:

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

---

## Dictionary Graph

In this example, we use a dataset of approximately 85,000 English words from the "Data::Generators" package. We focus on words starting with the prefix "rac".

```raku
my $ra = Data::Generators::ResourceAccess.instance();
$ra.get-word-data().elems;
```

We generate random words and sample some of them:

```raku
random-word(4, type => 'common')
```

```raku
.say for $ra.get-word-data().pick(4)
```

We find words with the prefix "rac":

```raku
my @words = $ra.get-word-data.keys.grep({ $_.starts-with('rac'):i });
@words.elems
```

Using the Damerau-Levenshtein distance, we compute the nearest neighbors graph for these words:

```raku
my @nnEdges = nearest-neighbor-graph(@words, 2, distance-function => &dld);
```

We create and plot the directed graph:

```raku
my $gDict = Graph.new(@nnEdges, :directed)
```

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

We can also derive graph edges ad hoc without using `nearest-neighbor-graph`:

```raku
# Construct on nearest finder object over them
my &wn = nearest(@words, distance-function => &dld);

# For each word find the closest two words and make edge rules of the corresponding pairs:
my @redges = @words.map({ $_ <<=><< &wn($_, 3).flat.grep(-> $x { $x ne $_ }) }).flat;
deduce-type(@redges)
```

---

## African Centers

We build an interstate highway system connecting the geographical centers of African countries. We start by assigning coordinates to each country.

```raku
my %africaCoords = 'Algeria'=>[3.0,28.0],'Libya'=>[17.0,25.0],'Mali'=>[-4.0,17.0],'Mauritania'=>[-12.0,20.0],'Morocco'=>[-5.0,32.0],'Niger'=>[8.0,16.0],'Tunisia'=>[9.0,34.0],'WesternSahara'=>[-13.0,24.5],'Chad'=>[19.0,15.0],'Egypt'=>[30.0,27.0],'Sudan'=>[30.0,13.8],'BurkinaFaso'=>[-2.0,13.0],'Guinea'=>[-10.0,11.0],'IvoryCoast'=>[-5.0,8.0],'Senegal'=>[-14.0,14.0],'Benin'=>[2.25,9.5],'Nigeria'=>[8.0,10.0],'Cameroon'=>[12.0,6.0],'CentralAfricanRepublic'=>[21.0,7.0],'Eritrea'=>[39.0,15.0],'Ethiopia'=>[38.0,8.0],'SouthSudan'=>[30.51,6.7],'Ghana'=>[-2.0,8.0],'Togo'=>[1.1667,8.0],'GuineaBissau'=>[-15.0,12.0],'Liberia'=>[-9.5,6.5],'SierraLeone'=>[-11.5,8.5],'Gambia'=>[-16.5667,13.4667],'EquatorialGuinea'=>[10.0,2.0],'Gabon'=>[11.75,-1.0],'RepublicCongo'=>[15.0,-1.0],'DemocraticRepublicCongo'=>[25.0,0.0],'Djibouti'=>[43.0,11.5],'Kenya'=>[38.0,1.0],'Somalia'=>[49.0,10.0],'Uganda'=>[32.0,1.0],'Angola'=>[18.5,-12.5],'Burundi'=>[30.0,-3.5],'Rwanda'=>[30.0,-2.0],'Tanzania'=>[35.0,-6.0],'Zambia'=>[30.0,-15.0],'Namibia'=>[17.0,-22.0],'Malawi'=>[34.0,-13.5],'Mozambique'=>[35.0,-18.25],'Botswana'=>[24.0,-22.0],'Zimbabwe'=>[30.0,-20.0],'SouthAfrica'=>[24.0,-29.0],'Swaziland'=>[31.5,-26.5],'Lesotho'=>[28.5,-29.5];
sink records-summary(%africaCoords.values)
```

We create a graph and find its minimum spanning tree:

```raku
my @africaEdges = %('from'=>'Algeria','to'=>'Libya','weight'=>1),%('from'=>'Algeria','to'=>'Mali','weight'=>1),%('from'=>'Algeria','to'=>'Mauritania','weight'=>1),%('from'=>'Algeria','to'=>'Morocco','weight'=>1),%('from'=>'Algeria','to'=>'Niger','weight'=>1),%('from'=>'Algeria','to'=>'Tunisia','weight'=>1),%('from'=>'Algeria','to'=>'WesternSahara','weight'=>1),%('from'=>'Libya','to'=>'Niger','weight'=>1),%('from'=>'Libya','to'=>'Tunisia','weight'=>1),%('from'=>'Libya','to'=>'Chad','weight'=>1),%('from'=>'Libya','to'=>'Egypt','weight'=>1),%('from'=>'Libya','to'=>'Sudan','weight'=>1),%('from'=>'Mali','to'=>'Mauritania','weight'=>1),%('from'=>'Mali','to'=>'Niger','weight'=>1),%('from'=>'Mali','to'=>'BurkinaFaso','weight'=>1),%('from'=>'Mali','to'=>'Guinea','weight'=>1),%('from'=>'Mali','to'=>'IvoryCoast','weight'=>1),%('from'=>'Mali','to'=>'Senegal','weight'=>1),%('from'=>'Mauritania','to'=>'WesternSahara','weight'=>1),%('from'=>'Mauritania','to'=>'Senegal','weight'=>1),%('from'=>'Morocco','to'=>'WesternSahara','weight'=>1),%('from'=>'Niger','to'=>'Chad','weight'=>1),%('from'=>'Niger','to'=>'BurkinaFaso','weight'=>1),%('from'=>'Niger','to'=>'Benin','weight'=>1),%('from'=>'Niger','to'=>'Nigeria','weight'=>1),%('from'=>'Chad','to'=>'Sudan','weight'=>1),%('from'=>'Chad','to'=>'Nigeria','weight'=>1),%('from'=>'Chad','to'=>'Cameroon','weight'=>1),%('from'=>'Chad','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Egypt','to'=>'Sudan','weight'=>1),%('from'=>'Sudan','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Sudan','to'=>'Eritrea','weight'=>1),%('from'=>'Sudan','to'=>'Ethiopia','weight'=>1),%('from'=>'Sudan','to'=>'SouthSudan','weight'=>1),%('from'=>'BurkinaFaso','to'=>'IvoryCoast','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Benin','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Ghana','weight'=>1),%('from'=>'BurkinaFaso','to'=>'Togo','weight'=>1),%('from'=>'Guinea','to'=>'IvoryCoast','weight'=>1),%('from'=>'Guinea','to'=>'Senegal','weight'=>1),%('from'=>'Guinea','to'=>'GuineaBissau','weight'=>1),%('from'=>'Guinea','to'=>'Liberia','weight'=>1),%('from'=>'Guinea','to'=>'SierraLeone','weight'=>1),%('from'=>'IvoryCoast','to'=>'Ghana','weight'=>1),%('from'=>'IvoryCoast','to'=>'Liberia','weight'=>1),%('from'=>'Senegal','to'=>'GuineaBissau','weight'=>1),%('from'=>'Senegal','to'=>'Gambia','weight'=>1),%('from'=>'Benin','to'=>'Nigeria','weight'=>1),%('from'=>'Benin','to'=>'Togo','weight'=>1),%('from'=>'Nigeria','to'=>'Cameroon','weight'=>1),%('from'=>'Cameroon','to'=>'CentralAfricanRepublic','weight'=>1),%('from'=>'Cameroon','to'=>'EquatorialGuinea','weight'=>1),%('from'=>'Cameroon','to'=>'Gabon','weight'=>1),%('from'=>'Cameroon','to'=>'RepublicCongo','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'SouthSudan','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'RepublicCongo','weight'=>1),%('from'=>'CentralAfricanRepublic','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'Eritrea','to'=>'Ethiopia','weight'=>1),%('from'=>'Eritrea','to'=>'SouthSudan','weight'=>1),%('from'=>'Eritrea','to'=>'Djibouti','weight'=>1),%('from'=>'Ethiopia','to'=>'SouthSudan','weight'=>1),%('from'=>'Ethiopia','to'=>'Djibouti','weight'=>1),%('from'=>'Ethiopia','to'=>'Kenya','weight'=>1),%('from'=>'Ethiopia','to'=>'Somalia','weight'=>1),%('from'=>'SouthSudan','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'SouthSudan','to'=>'Kenya','weight'=>1),%('from'=>'SouthSudan','to'=>'Uganda','weight'=>1),%('from'=>'Ghana','to'=>'Togo','weight'=>1),%('from'=>'Liberia','to'=>'SierraLeone','weight'=>1),%('from'=>'EquatorialGuinea','to'=>'Gabon','weight'=>1),%('from'=>'Gabon','to'=>'RepublicCongo','weight'=>1),%('from'=>'RepublicCongo','to'=>'DemocraticRepublicCongo','weight'=>1),%('from'=>'RepublicCongo','to'=>'Angola','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Uganda','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Angola','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Burundi','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Rwanda','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Tanzania','weight'=>1),%('from'=>'DemocraticRepublicCongo','to'=>'Zambia','weight'=>1),%('from'=>'Djibouti','to'=>'Somalia','weight'=>1),%('from'=>'Kenya','to'=>'Somalia','weight'=>1),%('from'=>'Kenya','to'=>'Uganda','weight'=>1),%('from'=>'Kenya','to'=>'Tanzania','weight'=>1),%('from'=>'Uganda','to'=>'Rwanda','weight'=>1),%('from'=>'Uganda','to'=>'Tanzania','weight'=>1),%('from'=>'Angola','to'=>'Zambia','weight'=>1),%('from'=>'Angola','to'=>'Namibia','weight'=>1),%('from'=>'Burundi','to'=>'Rwanda','weight'=>1),%('from'=>'Burundi','to'=>'Tanzania','weight'=>1),%('from'=>'Rwanda','to'=>'Tanzania','weight'=>1),%('from'=>'Tanzania','to'=>'Zambia','weight'=>1),%('from'=>'Tanzania','to'=>'Malawi','weight'=>1),%('from'=>'Tanzania','to'=>'Mozambique','weight'=>1),%('from'=>'Zambia','to'=>'Namibia','weight'=>1),%('from'=>'Zambia','to'=>'Malawi','weight'=>1),%('from'=>'Zambia','to'=>'Mozambique','weight'=>1),%('from'=>'Zambia','to'=>'Botswana','weight'=>1),%('from'=>'Zambia','to'=>'Zimbabwe','weight'=>1),%('from'=>'Namibia','to'=>'Botswana','weight'=>1),%('from'=>'Namibia','to'=>'SouthAfrica','weight'=>1),%('from'=>'Malawi','to'=>'Mozambique','weight'=>1),%('from'=>'Mozambique','to'=>'Zimbabwe','weight'=>1),%('from'=>'Mozambique','to'=>'SouthAfrica','weight'=>1),%('from'=>'Mozambique','to'=>'Swaziland','weight'=>1),%('from'=>'Botswana','to'=>'Zimbabwe','weight'=>1),%('from'=>'Botswana','to'=>'SouthAfrica','weight'=>1),%('from'=>'Zimbabwe','to'=>'SouthAfrica','weight'=>1),%('from'=>'SouthAfrica','to'=>'Swaziland','weight'=>1),%('from'=>'SouthAfrica','to'=>'Lesotho','weight'=>1);
my @africaEdges .= map({ %(from => $_<from>, to => $_<to>, weight => geo-distance(|%africaCoords{$_<from>}, |%africaCoords{$_<to>}, units => 'km' ).round) });

my $gAfrica = Graph.new(@africaEdges, :!directed)
```

```raku
my $gAfricaTree = $gAfrica.find-spanning-tree
```

The graph is plotted, highlighting the spanning tree in orange:

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

We cross-tabulate countries and distances:

```raku
my @tbl = (%africaCoords X %africaCoords).map({ %( from => $_.head.key, to => $_.tail.key, weight => &geo-distance($_.head.value, $_.tail.value, units => 'miles').round(0.1) ) });
my @ct = cross-tabulate(@tbl, 'from', 'to', 'weight').sort(*.key);

deduce-type(@ct)
```

The table is displayed:

```raku
#% html
@ct.map({ ['from' => $_.key , |$_.value].Hash }) ==> to-html(field-names => ['from', |@ct>>.key])
```

We create a graph based on the distance table and find its spanning tree:

```raku
my $gAfricaComplete = Graph.new(@tbl)
```

```raku
$gAfricaComplete.vertex-list.combinations(2).elems + $gAfricaComplete.vertex-count
```

```raku
my $gAfricaTree2 = $gAfricaComplete.find-spanning-tree
```

The graph is plotted:

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

---

## Bulgarian Cities

Finally, we build an interstate highway system connecting cities in Bulgaria with populations over 10,000.

```raku
city-data().map(*<Country>).unique.sort
```

We filter the dataset for large cities:

```raku
my $maxPop = 10_000;
my @cities = city-data().grep({ $_<Country> eq 'Bulgaria' && $_<Population> ≥ $maxPop });
@cities.elems
```

The dataset is displayed:

```raku
#% html
@cities.head(4) 
==> to-html(field-names => <ID Country State City Latitude Longitude Elevation LocationLink>)
```

We create a complete graph and find its spanning tree:

```raku
my %coords = @cities.map({ $_<City> => ($_<Latitude>, $_<Longitude>) });
my @dsEdges = (%coords.keys X %coords.keys ).map({ %(from => $_.head, to => $_.tail, weight => geo-distance(|%coords{$_.head}, |%coords{$_.tail} )) });
my $gGeo = Graph.new(@dsEdges, :!directed)
```

```raku
my $stree = $gGeo.find-spanning-tree(method => 'kruskal')
```

The graph is plotted:

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

Thank you for exploring these graph examples in Raku with us!