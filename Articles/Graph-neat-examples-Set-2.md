# Graph Neat Examples in Raku

### ***Set 2***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
July, November 2024

## Introduction

In this blog post, I will walk you through some neat examples of graph manipulation and visualization using Raku.
These examples showcase the capabilities of Raku and its modules in handling graph-related tasks.

All computational graph features discussed here are provided by the ["Graph"](https://raku.land/zef:antononcube/Graph) module.
The graphs are visualized using [D3.js](https://d3js.org) (via ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3)) and
[Graphviz DOT](https://graphviz.org/doc/info/lang.html) (via `Graph.dot`),
providing both interactive and static visualizations.


> **What is a neat example?**
>
> Concise or straightforward code that produces compelling visual or textual outputs. In this context, neat examples:
>
> - Showcase Raku programming.
> - Use functionalities of different Raku modules.
> - Give interesting perspectives on what is computationally possible.

Here is the link to the related presentation recording ["Graph neat examples in Raku (Set 2)](https://youtu.be/E7qhutQcWCY):

[![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-YouTube-thumbnail-small.png)](https://youtu.be/E7qhutQcWCY)

------

## Triangular grid graph

Here is a triangular grid graph:

```raku
#% js
use Graph::TriangularGrid;
use JavaScript::D3;

my $g = Graph::TriangularGrid.new(4, 4);
my @highlight = ($g.vertex-list Z=> $g.vertex-degree).classify(*.value).map({ $_.value».key });


js-d3-graph-plot( $g.edges(:dataset),
    :@highlight,
    background => 'none', 
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

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-openning-triangular-grid-graph.png)

Here are the corresponding adjacency- and incidence matrices:

```raku
#% js
use Math::SparseMatrix;
use Data::Reshapers;

my ($amat, $imat) = $g.adjacency-matrix, $g.incidence-matrix;

my %opts = grid-lines => {width => 0.25, color => 'DimGray'}, title-color => 'Silver', color-palette => 'Inferno', :!tooltip, background => '#1F1F1F', height => 300;
#my %opts2 = grid-lines => {width => 0.25, color => 'DimGray'}, title-color => 'Gray', color-palette => 'Greys', :!tooltip, background => 'White', height => 300;

js-d3-matrix-plot($amat, plot-label => 'Adjacency matrix', |%opts, width => 300+80, margins => {:2left, :40top, :2bottom, :80right})
~ "\n" ~ 
js-d3-matrix-plot($imat, plot-label => 'Incidence matrix', |%opts, width => 600, margins => {:2left, :40top, :2bottom, :2right})
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-openning-triangular-grid-graph-matrixes.png)

------

## Setup

The setup for these examples is the same as in the first set, so it is skipped here.

------

## Bipartite Graph Coloring

Let's start with a simple example: bipartite graph coloring.
Although this might seem basic, it's a good warm-up exercise.
Here, we're creating a grid graph.

```raku
my $gg = Graph::Grid.new(6, 16);
```

The method `is-bipartite` checks if the graph is bipartite, meaning it can be colored using two colors such that no two adjacent vertices share the same color.

```raku
$gg.is-bipartite
```

We can show the bipartite coloring of the grid graph using the following code:

```raku
$gg.bipartite-coloring
```

To prepare the graph for highlighting, we classify the vertices based on their bipartite coloring:

```raku
my %highlight = <SlateBlue Orange> Z=> $gg.bipartite-coloring.classify(*.value).nodemap(*».key).values;
.say for %highlight
```

Finally, we plot the grid graph as a bipartite graph:

```raku
#%js
$gg.edges(:dataset) ==>
    js-d3-graph-plot(
        :%highlight,
        vertex-coordinates => $gg.vertex-coordinates,
        background => '#1F1F1F', 
        title-color => 'Silver',
        edge-thickness => 5,
        vertex-size => 12,
        vertex-label-color => 'none',
        directed => $gg.directed,
        title => 'Grid graph bipartite coloring', 
        width => 1000,
        height => 400, 
        margins => {top => 80},
        force => {charge => {strength => -300}, x => {strength => 0.12}, link => {minDistance => 4}}
    )
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-grid-graph-bipartite.png)

-----

## Highlight Connected Components

In this example, we create a grid graph with randomly directed edges and find its connected components.

```raku
my $g = Graph::Grid.new(10, 20, :!directed);
my $g2 = $g.directed-graph(method => 'random');
```

Finding connected components in a directed graph is more complex than in an undirected graph. The connected components are the subsets of the graph where each vertex is reachable from any other vertex in the same subset.

```raku
my @components = $g2.connected-components.grep(*.elems - 1);
@components».elems;
```

We highlight these connected components in the graph using:

```raku
#% js
$g2.edges(:dataset) ==> 
js-d3-graph-plot(
    vertex-coordinates => $g2.vertex-coordinates,
    highlight => @components,
    directed => $g2.directed,
    background => '#1F1F1F', 
    title-color => 'Silver', 
    width => 1000, 
    height => 500, 
    vertex-label-color => 'none',
    vertex-fill-color => 'SteelBlue',
    edge-thickness => 3,
    edge-color => 'Gray',
    vertex-size => 14,
    arrowhead-size => 4,
    force => {charge => {strength => -160, iterations => 2}, collision => {radius => 1, iterations => 1}, link => {minDistance => 1}}
)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-grid-graph-components.png)

### Visualize via ***Graphviz DOT***

We can also visualize the graph using the Graphviz DOT language:

```raku
my $g3 = Graph::TriangularGrid.new(8, 16, scale => 0.3, :!directed);
$g3 = $g3.directed-graph(method => 'random', flip-threshold => 0.25);
```

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

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-triangular-grid-graph-components.png)

**Remark:** Note that the shape of graph vertices (nodes) is randomly selected.

**Remark:** The method `.dot` takes graph vertex styling options with both prefixes "node-" and "vertex-".
Graphviz DOT uses "node" in its specs.

------

## Collage of Star Graphs

In this final example, we create a collage of star graphs.

1. Make 40 different [star graphs](https://en.wikipedia.org/wiki/Star_(graph_theory)), each with its own vertex-name prefix.
2. Make a single _big_ graph with the edges of the 40 graphs.
3. Select a set of colors.
4. For each color, pick a graph at random.
5. Plot the big graph with highlighted edges and vertices of the randomly selected graphs.

```raku
my @graphs = (^40).map({ Graph::Star.new(n => (8..16).pick, prefix => "$_-") });
my $bigGraph = Graph.new( @graphs.map(*.edges).flat )
```

We can't use graph disjoined union because we need to keep the vertex prefixes for random highlights. Instead, we use a normal union:

```raku
my $bigGraph = reduce({ $^a.union($^b) }, |@graphs)
```

To get a range of colors, we use:

```raku
my @colors = (^14).map: { sprintf "#%02X%02X%02X", 250 - $_*10, 128 - $_*5, 114 + $_*10 };
```

Finally, we plot the collage of graphs:

```raku
#%js
$bigGraph.edges(:dataset) ==>
js-d3-graph-plot(
    highlight => (@colors Z=> @graphs.pick(@colors.elems).map({ [|$_.vertex-list, |$_.edges] })).Hash,
    background => '#1F1F1F', 
    title-color => 'Silver',
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

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-star-graphs-collage.png)

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

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graph-neat-examples-Set-2/Graph-neat-examples-in-Raku-Set-2-star-graphs-collage-DOT.png)

----

## Conclusion

These examples demonstrate the power and flexibility that Raku *currently* has for graph manipulation and visualization.
I would say, the dark mode, dynamic Javascript plot of the star graphs collage is one of the prettiest graph plots I have created!
(Just random, not tied to anything significant...)
