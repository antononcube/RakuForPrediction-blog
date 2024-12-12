# Graphs in Raku

<span style="font-size: 16pt; font-style: italic; font-weight: bold">Day 12 of Raku Advent 2024 </span>

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
December 2024

-----

## Introduction

This blog post discusses the development of graph theory algorithms in Raku. Moderate number of examples is used.

**TL;DR:** Just see the mind-map and then browse the last section with a graph that resembles a snow-covered Christmas tree.

In the post:
- All computational graph features discussed are provided by ["Graph"](https://raku.land/zef:antononcube/Graph).
- Graph plotting is done with:
    - `js-d3-graph-plot`, provided by ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3).
    -  `Graph.dot`, that makes SVG images via [Graphviz](https://graphviz.org).

----- 

## Graph theory functionalities making


Because of a few "serious" projects involving conversational agents about logistics I thought that is a good idea to have geographical shortest path finding in Raku. 
(Instead of "outsourcing" those computations to some other system.)

The assumption were that:
- The fundamental graph algorithms are (relatively) easy to program.
- Just good core data structures have to be chosen
- Data and algorithms for [simple graphs](https://mathworld.wolfram.com/SimpleGraph.html) should be easy to implement.

The above assumptions turned true, in general, and while graph-programming with Raku, in particular.

The Raku package ["Graph"](https://raku.land/zef:antononcube/Graph) provides the class `Graph` that has the following classes of algorithms (as class methods):

- Finding paths cycles and flows
    - Finding shortest path, hamiltonian path, etc.
- Matching and coloring
- Graph manipulation,
    - Edge removal, vertex replacement, etc.
- Graph-to-graph combinations
    - Union, difference, etc.
- Construction of parameterized graphs
    - Complete, star, wheel, etc.
- Construction of model distribution based random graphs
    - Watts-Strogatz, Barabási–Albert, etc.

The class `Graph` uses a map of maps for the adjacency lists (i.e. vertex connections.) It _should be_ extended to have vertex- and edge tags. Having multiple edges should be supported at some point, but I consider it low priority.


Now, [Graph theory](https://en.wikipedia.org/wiki/Graph_theory) is a huge field and this inevitably produces a certain opinionated selection of which algorithms to implement first or implement at all.
It is natural to consider having (1) a set of implemented fundamental graph algorithms and (2) a framework for quicker developing of new ones. Currently, I would say, the former fairly well addressed. It is in my future plans to have user-exposed framework implementation based on Depth-First Search (DFS) and Breath-First Search (BFS) algorithms.


Graphs are naturally related to sparse matrices and many graph algorithms can be recast into sparse matrix algebra based algorithms. Because of this relationship _and_ because sparse matrices are a very useful mathematical tool I implemented the packages
["Math::SparseMatrix"](https://raku.land/zef:antononcube/Math::SparseMatrix) and
["Math::SparseMatrix::Native"](https://raku.land/zef:antononcube/Math::SparseMatrix::Native).


The current functionalities of `Graph` are demonstrated and explained in the "neat examples" videos and blog posts. Here is the videos list in order of their making:

- ["Graph neat examples in Raku (Set 1)"](https://youtu.be/5qXgqqRZHow)
- ["Sparse matrix neat examples in Raku"](https://youtu.be/kQo3wpiUu6w)
- ["Graph neat examples in Raku (Set 2)"](https://youtu.be/E7qhutQcWCY)
- ["Graph neat examples in Raku (Set 3)"](https://youtu.be/S_3e7liz4KM)
- ["Chess positions and knight's tours via graphs (in Raku)"](https://youtu.be/fwQrQyWC7R0)



Here is a mind-map of the current development status of "Graph" that should give an idea of the scope of the project:

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Graphs-in-Raku-2024/Graph-TODO-section-mind-map-light.png)


**Remark:** The mind-map above was LLM-created automatically from the [README of "Graph"](https://github.com/antononcube/Raku-Graph/blob/main/README.md) and a specially crafted prompt. The checkmark "✔️" means "implemented"; the empty circle "⭕️" means "not implemented" (yet.)


-----

## Spanning tree for large cities of USA


In relation to the Geo-motivation projects mentioned above let us consider the problem of building an interstate highway system joining the cities of USA.


Here is the available Geographical data from ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics):

```raku
city-data().map(*<Country>).unique.sort
```
```
# (Botswana Bulgaria Canada Germany Hungary Russia Spain Ukraine United States)
```

Here is a dataset of large enough cities:

```raku
my $maxPop = 200_000;
my @cities = city-data().grep({ $_<Country> eq 'United States' && $_<Population> ≥ $maxPop });
@cities.elems
```
```
# 126
```

**Remark:** Geographical data and geo-distance computation is provided by ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics).

```raku
#% html
@cities.head(4) 
==> to-html(field-names => <ID Country State City Latitude Longitude Elevation LocationLink>)
```

<table border="1"><thead><tr><th>ID</th><th>Country</th><th>State</th><th>City</th><th>Latitude</th><th>Longitude</th><th>Elevation</th><th>LocationLink</th></tr></thead><tbody><tr><td>United_States.New_York.New_York_City</td><td>United States</td><td>New York</td><td>New York City</td><td>40.6642738</td><td>-73.9385004</td><td>10</td><td>http://maps.google.com/maps?q=40.6642738,-73.9385004&amp;z=12&amp;t=h</td></tr><tr><td>United_States.California.Los_Angeles</td><td>United States</td><td>California</td><td>Los Angeles</td><td>34.0193936</td><td>-118.4108248</td><td>89</td><td>http://maps.google.com/maps?q=34.0193936,-118.4108248&amp;z=12&amp;t=h</td></tr><tr><td>United_States.Illinois.Chicago</td><td>United States</td><td>Illinois</td><td>Chicago</td><td>41.8375511</td><td>-87.6818441</td><td>179</td><td>http://maps.google.com/maps?q=41.8375511,-87.6818441&amp;z=12&amp;t=h</td></tr><tr><td>United_States.Texas.Houston</td><td>United States</td><td>Texas</td><td>Houston</td><td>29.7804724</td><td>-95.3863425</td><td>12</td><td>http://maps.google.com/maps?q=29.7804724,-95.3863425&amp;z=12&amp;t=h</td></tr></tbody></table>


Here is the corresponding weighted **complete** graph:

```raku
my %coords = @cities.map({ $_<City> => ($_<Latitude>, $_<Longitude>) });
%coords .= grep({ $_.key ∉ <Anchorage Honolulu> });
my @dsEdges = (%coords.keys X %coords.keys ).map({ %(from => $_.head, to => $_.tail, weight => geo-distance(|%coords{$_.head}, |%coords{$_.tail} )) });
my $gGeo = Graph.new(@dsEdges, :!directed)
```
```
# Graph(vertexes => 122, edges => 7503, directed => False)
```

Here is the corresponding spanning tree:

```raku
my $stree = $gGeo.find-spanning-tree(method => 'kruskal')
```
```
# Graph(vertexes => 122, edges => 121, directed => False)
```

Here is an example of finding the shortest path from Seattle to Jacksonville in the spanning tree.

```raku
my @path = $stree.find-shortest-path('Seattle', 'Jacksonville')
```
```
# [Seattle Spokane Boise City Reno Sacramento Stockton Modesto Fresno Bakersfield Santa Clarita Los Angeles Long Beach Anaheim Santa Ana Irvine Riverside Fontana San Bernardino Enterprise Henderson Glendale Phoenix Scottsdale Mesa Gilbert Tucson El Paso Albuquerque Amarillo Oklahoma City Tulsa Little Rock Memphis Nashville Huntsville Birmingham Montgomery Columbus Jacksonville]
```

And here is the plot of the spanning tree and with the shortest path highlighted:

```raku, eval=FALSE
#% js
$stree.edges
==> js-d3-graph-plot(
    vertex-coordinates => %coords.nodemap(*.reverse)».List.Hash,
    highlight => {SlateBlue => [|@path, |Graph::Path.new(@path).edge-list] },
    title => "USA, cities with population ≥ $maxPop",
    width => 1400,
    height => 700,
    :$background, :$title-color, :$edge-thickness, 
    vertex-size => 4,
    vertex-label-font-size => 12,
    vertex-label-font-family => Whatever,
    margins => {right => 200}
    )
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/USA-spanning-tree.png)

**Remark:** Using `highlight => @path` would just highlight the vertices, without the edges.

**Remark:** Would have been nice to use the spec `highlight => Graph::Path.new(@path)` instead of `highlight => [|@path, |Graph::Path.new(@path).edge-list]`. But I try to minimize the package dependencies of the Raku packages I implement, and because of that principle, "JavaScript::D3" does not know "Graph". Hence, both vertices and edges have to be given to the "highlight" option.


-----

## Visualization


Graph visualization is extremely important for development of graph software. (Meaning, it speeds up the development tremendously.)
Immediately after I implemented the first few "main" algorithms I made the corresponding Wolfram Language (WL) and Mermaid-JS representations methods of `Graph`.
Soon after, I implemented `js-d3-graph-plot` in ["JavaScript:D3"](https://raku.land/zef:antononcube/JavaScript::D3) -- [D3.js](https://d3js.org) has a very nice [_network force_](https://d3js.org/d3-force) functionalities. ("Network" is the D3.js lingo for "graph.") Since, the using of JavaScript plots rendering in Jupyter notebooks is somewhat capricious, I looked for alternatives. This lead to a "full blown" [Graphviz](https://graphviz.org) plot support in `Graph`. (More about that below.)


### JavaScript visualization


The plotting with [D3.js](https://d3js.org) (via ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3)) made me very enthusiastic to implement- and curious to see how different named and/or parameterized graphs would look visualized. So, I quickly made half a dozen of them. These graphs are have known properties, which makes them good for algorithm development testing.

Here is a map (dictionary) with most of the currently implemented parameterized graphs:

```raku
my %namedGraphs = 
    Circulant => Graph::Circulant.new(7, 3),
    Complete => Graph::Complete.new(5),
    CompleteKaryTree => Graph::CompleteKaryTree.new(3,3),
    Cycle => Graph::Cycle.new(8),
    Grid => Graph::Grid.new(4,3),
    TriangularGrid => Graph::TriangularGrid.new(3,3),
    HexagonalGrid => Graph::HexagonalGrid.new(2,2),
    Hypercube => Graph::Hypercube.new(4),
    KnightTour => Graph::KnightTour.new(6,4),
    Path => Graph::Path.new('a'..'g', :directed),
    Petersen => Graph::Petersen.new(),
    Star => Graph::Star.new(5),
    Wheel => Graph::Wheel.new(7, :!directed);

.say for %namedGraphs
```
```
# Star => Graph(vertexes => 6, edges => 5, directed => False)
# TriangularGrid => Graph(vertexes => 16, edges => 33, directed => False)
# Grid => Graph(vertexes => 12, edges => 17, directed => False)
# Complete => Graph(vertexes => 5, edges => 10, directed => False)
# Path => Graph(vertexes => 7, edges => 6, directed => True)
# Cycle => Graph(vertexes => 8, edges => 8, directed => False)
# Circulant => Graph(vertexes => 7, edges => 7, directed => False)
# Hypercube => Graph(vertexes => 16, edges => 32, directed => False)
# Petersen => Graph(vertexes => 10, edges => 15, directed => False)
# CompleteKaryTree => Graph(vertexes => 13, edges => 12, directed => False)
# Wheel => Graph(vertexes => 8, edges => 14, directed => False)
# KnightTour => Graph(vertexes => 24, edges => 44, directed => False)
# HexagonalGrid => Graph(vertexes => 16, edges => 19, directed => False)
```

```raku, eval=FALSE
#%js
%namedGraphs.pairs.sort(*.key).map({ 
    js-d3-graph-plot(
        $_.value.edges(:dataset),
        vertex-coordinates => $_.value.vertex-coordinates,
        :$background, :$title-color, :$edge-thickness, :$vertex-size,
        directed => $_.value.directed,
        title => $_.key, 
        width => 300,
        height => 300, 
        force => {charge => {strength => -200}, link => {minDistance => 40}}
    )
 }).join("\n")
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Parameterized-graphs-gallery-3-rows.png)

Of particular interest to me are the random graphs:
- Simple vertex- and edge sampling random graphs are useful for testing
- Well known random graphs with adhering to particular models are also very nice to have

The easiest way to have random graphs implementations is to have a special `Graph::Distribution` class.

Here are examples:

```raku
my %randomGraphs = 
    Barabasi-Albert => Graph::Random.new(Graph::Distribution::BarabasiAlbert.new(20,2)),
    "Price's model" => Graph::Random.new(Graph::Distribution::Price.new(14, 2, 1)),
    Random => Graph::Random.new(13,16),
    Watts-Strogatz => Graph::Random.new(Graph::Distribution::WattsStrogatz.new(20,0.07));

.say for %randomGraphs
```
```
# Watts-Strogatz => Graph(vertexes => 20, edges => 44, directed => False)
# Barabasi-Albert => Graph(vertexes => 20, edges => 34, directed => False)
# Random => Graph(vertexes => 13, edges => 16, directed => False)
# Price's model => Graph(vertexes => 14, edges => 24, directed => True)
```

```raku, eval=FALSE
#%js
%randomGraphs.pairs.sort(*.key).map({ 
    js-d3-graph-plot(
        $_.value.edges(:dataset),
        :$background, :$title-color, :$edge-thickness, :$vertex-size,
        directed => $_.value.directed,
        title => $_.key, 
        width => 400, 
        force => {charge => {strength => -200}, link => {minDistance => 40}}
    )
 }).join("\n")
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Random-model-graphs-gallery.png)


### Graphviz visualization


`Graph` has the method `dot` that translates the graph object into a [Graphviz DOT language](https://graphviz.org/doc/info/lang.html) spec.

```raku, eval=FALSE
#% html
%randomGraphs<Watts-Strogatz>.dot(:$background, engine => 'sfdp', :svg)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Watts-Strogatz-by-DOT.png)

Using the "neato" layout engine graph objects that have vertex coordinates are properly plotted:

```raku, eval=FALSE
#% html
Graph::KnightTour.new(4, 6).dot(:$background, engine => 'neato', :svg)
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Knight-tour-by-DOT.png)


Also the Graphviz DOT representations can be used to [plot chessboards](https://youtu.be/fwQrQyWC7R0) (via ["Graphviz::DOT::Chessboard"](https://raku.land/zef:antononcube/Graphviz::DOT::Chessboard)):

```raku, eval=FALSE
#% html
dot-chessboard(:4r, :6c):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Chessboard-by-DOT.png)


### How come I invested so much in Graphviz DOT support in `Graph`?

Interactive plotting with D3.js is fairly unreliable Raku-wise -- I can only use it in Visual Studio Code with Jupyter.
It was working in web browsers with the old Jupyter notebook framework. But after JupyterLab was introduced, my newest Jupyter-anything installations do not work with the JavaScript settings for plotting.
(Making static HTML pages with D3.js plots is fine.)
So, finding a more robust and universal alternative was really needed. After some discussion with a Raku-enthusiast known as "timo"
I made finding such alternatives a priority.
So, I looked for JavaScript alternatives to plot graphs first. I was also looking -- separately -- for JavaScript-based ways to use the Graphviz DOT language.
At some point I figured out that Graphviz layout engines are fairly install-able / deploy-able in many operating systems, so just using Graphviz to generate SVG, PNG, etc. is both fine and reliable.


----

## Sparse matrix representations


As it was mentioned above, graphs have a natural representation as sparse matrices. 
We can use the package ["Math::SparseMatrix"](https://raku.land/zef:antononcube/Math::SparseMatrix) 
to make those representations and "JavaScript::D3" to plot them.

Here are the sparse matrices corresponding to the parameterized graphs created and plotted above:

```raku, eval=FALSE
#% js
%namedGraphs.sort(*.key).map({
    my $g = $_.value.index-graph;
    my $m = Math::SparseMatrix.new(edge-dataset => $g.edges(:dataset), row-names => $g.vertex-list.sort(*.Int));
    js-d3-matrix-plot($m.Array, plot-label => $_.key, width => 230, margins => 30, :!tooltip, :$title-color, :$tick-labels-font-size, :$tick-labels-color)
}).join("\n")
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Sparse-matrice-for-parameterized-graphs-gallery-3-rows.png)

For more examples see the video ["Sparse matrix neat examples in Raku"](https://youtu.be/kQo3wpiUu6w).


----

## Christmas themed example


Let us finish this post with a topical example -- we make a graph that looks like a Christmas tree and we color it accordingly.


Here is the procedure outline:

- Get a regular grid graph `G`.
- Get a tree-shaped subgraph `T` of `G`.
- Pick random vertices in `T` and remove their corresponding [neighborhood graphs](https://en.wikipedia.org/wiki/Neighbourhood_(graph_theory)) from `T`.
    - Denote the new graph `X`.
- Find the vertices `v` of `X` that have degree less than 5.
- Plot `X` by highlighting `v`.
    - The highlights can be randomly selected shades of a certain color.

The procedure is followed below.

<hr width="60%" color="gray"/>


Here we create a (largish) triangular grid graph:

```raku
my $g = Graph::TriangularGrid.new(30, 30);
```
```
# Graph(vertexes => 976, edges => 2760, directed => False)
```

Here we plot a smaller version of it:

```raku, eval=FALSE
#% html
my $gSmall = Graph::TriangularGrid.new(3,3);
$gSmall.dot(:$engine, highlight => Graph::Path.new(<5 7 8 10>), :!node-labels, edge-thickness => 8, node-font-size => 30, node-width => 0.6, size => 4):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/TriangularGrid-graph-4x4.png)

We plan to take the graph under the highlighted (orange) vertices and edges and turn it into a Christmas tree. (Or something that resembles it.)


**Remark:** Note that with `Graph.dot` we can use a graph object as a highlight spec.


Derive rectangular area boundaries:

```raku
my ($min-y, $max-y) = $g.vertex-coordinates.values.map(*.tail).Array.&{ (.min, .max)};

my $bottom-min-x = $g.vertex-coordinates.values.grep(*.tail == $min-y).map(*.head).min;
my $bottom-max-x = $g.vertex-coordinates.values.grep(*.tail == $min-y).map(*.head).max;

my $top-min-x = $g.vertex-coordinates.values.grep(*.tail == $max-y).map(*.head).min;
my $top-max-x = $g.vertex-coordinates.values.grep(*.tail == $max-y).map(*.head).max;
```
```
# 150.68842025849298
```

Top left node:

```raku
my $top-vertex = $g.vertex-coordinates.grep({ $_.value.head == $top-min-x && $_.value.tail == $max-y })».key.head
```
```
# 255
```

"Weak" diagonal equation:

```raku
my ($x1, $y1) = ($bottom-max-x, $min-y);
my ($x2, $y2) = |$g.vertex-coordinates{$top-vertex};

my $k = ($y1 - $y2)/($x1 - $x2);
my $n = -(($x2*$y1 - $x1*$y2)/($x1 - $x2));

sub y-diag(Numeric:D $x) { $k * $x +$n }
```
```
# &y-diag
```

Get vertices under the diagonal:

```raku
my @focus-vertexes = $g.vertex-coordinates.grep({  $_.value.tail ≤ y-diag($_.value.head) + 0.001 })».key;
@focus-vertexes.elems
```
```
# 499
```

Get the "pyramid" from the filtered vertexes:

```raku
my $c-tree = $g.subgraph(@focus-vertexes); 
```
```
# Graph(vertexes => 498, edges => 1395, directed => False)
```

Remove random parts (using neighborhood graphs):

```raku
my $c-tree2 = $c-tree.difference( $c-tree.neighborhood-graph($c-tree.vertex-list.pick(36).grep({ $_ ne $top-vertex }), d => 1) );
```
```
# Graph(vertexes => 498, edges => 975, directed => False)
```

Plot with highlights:

```raku, eval=FALSE
#%html
my @vs = $c-tree2.vertex-degree(:p).grep(*.value≤4)».key;
@vs = @vs.map({ <Snow White GhostWhite>.pick => $_ });
my %highlight = @vs.classify(*.key).nodemap({ $_».value });

$c-tree2.dot( 
    :%highlight,
    node-width => 1.7,
    node-height => 1.7,
    node-fill-color => 'Red', 
    node-shape => 'circle',
    graph-size => 6,
    edge-color => 'Green',
    edge-width => 28,
    node-font-size => 380,
    pad => 2,
    node-labels => {$top-vertex => '⭐️', |(($g.vertex-list (-) $top-vertex).keys X=> '')},
    engine => 'neato'
):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Articles/Diagrams/Graphs-in-Raku-2024/Christmas-tree-graph-art.png)


Here are the special graph functionalities used to make the plot above:

- Construction of [triangular grid graph](https://mathworld.wolfram.com/TriangularGridGraph.html)
- [Subgraph](https://mathworld.wolfram.com/Subgraph.html) taking
- [Neighborhood graphs](https://mathworld.wolfram.com/NeighborhoodGraph.html)
- [Graph difference](https://mathworld.wolfram.com/GraphDifference.html)
- Graph plotting via Graphviz DOT using:
    - Customized styling of various elements
    - Vertex coordinates
    - Specified vertex labels (see the top of the tree)
- Graph highlighting
    - Multiple sets of vertices and edges with different colors can be specified


-----

## Conclusion

Graph theory is both very mathematical and very computer-science-ish. It has diverse applications across different fields. So, I urge you to start thinking how to use graphs in all of your Raku and non-Raku projects! (And leverage ["Graph"](https://raku.land/zef:antononcube/Graph).)
