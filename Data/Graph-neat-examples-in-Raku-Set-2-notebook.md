
# Graph neat examples in Raku

### ***Set 2***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
July, November 2024   

```raku
#% js
 use Graph::TriangularGrid;
 use JavaScript::D3;
 
 my $g = Graph::TriangularGrid.new(4, 4);
 my @highlight = ($g.vertex-list Z=> $g.vertex-degree).classify(*.value).map({ $_.value».key });


  js-d3-graph-plot( $g.edges(:dataset),
        :@highlight,
        background => '#1F1F1F', 
        edge-thickness => 3,
        vertex-size => 10,
        vertex-label-color => 'none',
        width => 1000,
        height => 300, 
        margins => 5,
        edge-color => 'SteelBlue',
        force => {charge => {strength => -300}, y => {strength => 0.2}, link => {minDistance => 4}}
    ) 
```

```raku
#% js
  use Math::SparseMatrix;
  use Data::Reshapers;

  my ($amat, $imat) = $g.adjacency-matrix, $g.incidence-matrix;
  
  my %opts = grid-lines => {width => 0.25, color => 'DimGray'}, title-color => 'Silver', color-palette => 'Inferno', :!tooltip, background => '#1F1F1F', height => 300;

  js-d3-matrix-plot($amat, plot-label => 'Adjacency matrix', |%opts, width => 300+80, margins => {:2left, :40top, :2bottom, :80right})
  ~ "\n" ~ 
  js-d3-matrix-plot($imat, plot-label => 'Incidence matrix', |%opts, width => 600, margins => {:2left, :40top, :2bottom, :2right})
```

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
- Graph plotting with:
    - `js-d3-graph-plot`, provided by ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3).
    -  `Graph.dot`, that makes SVG images via [Graphviz](https://graphviz.org).

--------

## Setup

*(Setup is the same as in the first set of examples. Hence, skipped here.)*

--------

## Bipartite graph coloring


*Maybe this example is not that neat, since it might see too regular or basic.*    
*But it is a good warm up.*


Make grid graph:

```raku
my $gg = Graph::Grid.new(6, 16);
```

```raku
$gg.is-bipartite
```

Show bipartite coloring of the grid graph:

```raku
$gg.bipartite-coloring
```

Prepare graph highlight code:

```raku
my %highlight = <SlateBlue Orange> Z=> $gg.bipartite-coloring.classify(*.value).nodemap(*».key).values;

.say for %highlight
```

Plot grid graph as a bipartite graph:

```raku
#%js
$gg.edges(:dataset) ==>
    js-d3-graph-plot(
        :%highlight,
        vertex-coordinates => $gg.vertex-coordinates,
        :$background, :$title-color,
        edge-thickness => 5,
        vertex-size => 12,
        vertex-label-color => 'none',
        directed => $g.directed,
        title => 'Grid graph bipartite coloring', 
        width => 1000,
        height => 400, 
        margins => {top => 80},
        force => {charge => {strength => -300}, x => {strength => 0.12}, link => {minDistance => 4}}
        )
```

------

## Highlight connected components


Make a grid graph with randomly directed edges:

```raku
my $g = Graph::Grid.new(10, 20, :!directed);
my $g2 = $g.directed-graph(method => 'random');
```

Find the connected components in the graph:

```raku
my @components = $g2.connected-components.grep(*.elems - 1);
@components».elems;
```

Highlight the connected components in a graph:

```raku
#% js
$g2.edges(:dataset) ==> 
js-d3-graph-plot(
    vertex-coordinates => $g2.vertex-coordinates,
    highlight => @components,
    directed => $g2.directed,
    :$background, 
    :$title-color, 
    width => 1000, 
    height => 500, 
    vertex-label-color => 'none',
    vertex-fill-color => 'SteelBlue',
    :$edge-thickness,
    edge-color => 'Gray',
    vertex-size => 14,
    arrowhead-size => 4,
    force => {charge => {strength => -160, iterations => 2}, collision => {radius => 1, iterations => 1}, link => {minDistance => 1}}
)
```

### Visualize via ***Graphviz DOT***


Make a triagular grid graph:

```raku
my $g3 = Graph::TriangularGrid.new(8, 16, scale => 0.3, :!directed);
$g3 = $g3.directed-graph(method => 'random', flip-threshold => 0.25);
```

Visualize using the [Graphviz DOT language](https://graphviz.org/doc/info/lang.html):

```raku
#% html
$g3.dot( 
    highlight => $g3.connected-components.grep(*.elems - 1),
    :!node-labels,
    node-shape => <hexagon triangle>.pick, 
    node-height => 0.7, 
    node-width => 0.7, 
    edge-thickness => 4, 
    edge-color => 'Gray',
    size => '10,6!',
    engine => 'neato' 
):svg
```

---------

## Collage of star graphs


1. Make 40 different [star graphs](https://en.wikipedia.org/wiki/Star_(graph_theory)), each with its own vertex-name prefix.
2. Make a single _big_ graph with the edges of the 40 graphs 
3. Select a set of colors
4. For each color pick at random a graph
5. Plot the big graph with highlighted edges and vertexes of the graph random selection

```raku
my @graphs = (^40).map({ Graph::Star.new(n => (8..16).pick, prefix => "$_-") });
my $bigGraph = Graph.new( @graphs.map(*.edges).flat )
```

**Remark:** Graph disjoined union _cannot_ be used, 
because we have to keep the vertex prefixes in order to be able to do random highlights.
Normal union still applies (but it is slower.) For example:

```
my $bigGraph = reduce({ $^a.union($^b) }, |@graphs)
```


Get a range of colors:

```raku
my @colors = (^14).map: { sprintf "#%02X%02X%02X", 250 - $_*10, 128 - $_*5, 114 + $_*10 };
```

Plot the collage of graphs:

```raku
#%js
$bigGraph.edges(:dataset) ==>
js-d3-graph-plot(
    highlight => (@colors Z=> @graphs.pick(@colors.elems).map({ [|$_.vertex-list, |$_.edges] })).Hash,
    :$background, :$title-color,
    edge-thickness => 2, 
    vertex-stroke-color => 'LightSteelBlue',
    vertex-size => 8,
    vertex-label-color => 'none',
    directed => False,
    title => 'Collage graph', 
    width => 1200,
    height => 700, 
    force => {charge => {strength => -40}, y => {strength => 0.2}, collision => {radius => 12}, link => {minDistance => 4}}
    )
```

### Visualize via Graphviz DOT

```raku
#% html
my $preamble = q:to/END/;
label = "Collage graph";
labelloc = "b";
fontcolor = "Gray";
fontsize = 42;
bgcolor = "#1F1F1F";
graph [size="9,9!"];
node [label="", shape=circle, style="filled", fillcolor="SteelBlue", color="Ivory", penwidth=3, width=0.65, height=0.65];
edge [color="SteelBlue", penwidth=4]
END

$bigGraph.dot(
    highlight => (@colors Z=> @graphs.pick(@colors.elems).map({ [|$_.vertex-list, |$_.edges] }) ).Hash,
    :$preamble,
    engine => 'neato'):svg
```
