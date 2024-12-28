# Sparse matrix neat examples

### ...in Raku

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)      
October 2024 

```raku
#%js
    use Math::SparseMatrix;
    use Math::SparseMatrix::Utilities;
    use JavaScript::D3;
    
    my $nrow = 5;
    my $ncol = 24;
    my $density = 0.35;
    my $tol = 0.001;
    my $type = 'CSR';

    my $mRand = generate-random-sparse-matrix($nrow, $ncol, :$density, :$type, :$tol);
    js-d3-matrix-plot(($mRand).Array, width => 1000, margins => 1, tick-labels-font-size => 10, color-palette => 'Purples')
```

----

## Introduction


In this notebook we consider a few examples of sparse matrices utilization.

1. Random graph 
    - Adjacency matrix of random graph `G`
        - From a model of social interactions
    - Over-imposed adjacency matrices with `G` and a shortest path in `G`

2. Movie-actor bipartite graph 
    - Ingesting data for relationships of actors starring in movies.
    - Sparse matrix algebra can help doing certain information retrieval tasks

3. Sparse matrices visualization discussion


Support of sparse matrix linear algebra is a sign of maturity of the corresponding systems for technological computations.


| Language                  | Initial Introduction      | Confirmed Update          |
|---------------------------|---------------------------|---------------------------|
| MATLAB                    | 1992                      | ~                         |
| Mathematica / Wolfram Language | 2003                 | updated 2007              |
| Python                    | maybe since 2004          | updated 2006              |
| R                         | maybe since 2011          | updated 2014              |



-----

## Setup

```raku
use Math::SparseMatrix :ALL;
use Math::SparseMatrix::Utilities;

use Data::Importers;
use Data::Reshapers;
use Data::Summarizers;
use Data::Generators;

use Graph;
use Graph::Distribution;
use Graph::Random;
use Graph::Path;

use Hash::Merge;
use JavaScript::D3;
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
my $title-color = 'Silver';
my $stroke-color = 'SlateGray';
my $tooltip-color = 'LightBlue';
my $tooltip-background-color = 'none';
my $tick-labels-font-size = 10;
my $tick-labels-color = 'Silver';
my $tick-labels-font-family = 'Helvetica';
my $background = '#1F1F1F';
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

------

## Random graph matrix


Here is a random graph using the Watts-Strogatz model:

```raku
#% js
my $gl = Graph::Random.new: Graph::Distribution::WattsStrogatz.new(20,0.06);

my $gp = Graph::Path.new: $gl.find-shortest-path('0','12'), :directed;

my $grPlot = 
js-d3-graph-plot(
    $gl.edges(:dataset),
    highlight => [|$gp.vertex-list, |$gp.edge-list],
    background => '1F1F1F', 
    title-color => 'Silver', 
    edge-thickness => 3,
    vertex-size => 6,
    width => 600,
    force => {charge => {strength => -260, iterations => 2}, y => {strength => 0.2}, collision => {radius => 6, iterations => 10}, link => {distance => 4}}
    )
```

Here is the corresponding matrix:

```raku
#% js
my $m = Math::SparseMatrix.new(edge-dataset => $gl.edges(:dataset), row-names => $gl.vertex-list.sort(*.Int));
say $m;
$m.Array ==> js-d3-matrix-plot(width => 400, margins => 15, :$tick-labels-font-size)
```

Here the plots:
- Graph matrix
- Shortest path matrix
- Sum of the corresponding matrices

```raku
#% js
my $m2 = Math::SparseMatrix.new(edge-dataset => $gp.edges(:dataset), row-names => $m.row-names);

my $m3 = $m.add($m2.multiply(0.75));

# Vusualize
 my %opts = width => 350, margins => {top => 30, left => 16, right => 16, bottom => 16}, :$tick-labels-font-size, :$tick-labels-color, :$title-color, :!tooltip, color-palette => 'Inferno';
 [
   js-d3-matrix-plot($m.Array, |%opts, title => 'Graph'),
   js-d3-matrix-plot($m2.Array, |%opts, title => 'Shortest path graph'),
   js-d3-matrix-plot($m3.Array, |%opts, title => 'Sum')
 ].join("\n")
```

Here is "plain" print of the element-sum matrix:

```raku
$m3.print
```

Let us compare the graph and the "sum matrix" side-by-side:

```raku
#% js
[
    $grPlot,
    js-d3-matrix-plot($m3.Array, margins => 16, :$tick-labels-font-size, :$tick-labels-color, width => 400, color-palette => 'Inferno')
].join("\n")
```

-----

## Ingest data movie-actor data


Here we ingest a CSV file with movie data:

```raku
my $file = $*CWD ~ '/Sparse-matrices/dsMovieRecords.csv';
my @dsMovieRecords = data-import($file, 'csv', headers => 'auto');

deduce-type(@dsMovieRecords)
```

Tabular form of the movie data:

```raku
#% html
my @field-names = <Movie Actor Genre1 Genre2 Genre3 BoxOffice>;
@dsMovieRecords ==> to-html(:@field-names)
```

Summary:

```raku
#% html
records-summary(@dsMovieRecords, :8max-tallies, :!say) 
==> to-html(:@field-names)
```

-----

## Bipartite graph


Here we make a graph based on the movie-actor relationships:

```raku
my @rules = @dsMovieRecords.map({ $_<Movie> => $_<Actor> });
my $g = Graph.new(@rules) 
```

The graph is bi-partite:

```raku
$g.is-bipartite
```

Here is the coloring:

```raku
.say for $g.bipartite-coloring.classify(*.value)
```

```raku
#% js

$g.edges(:dataset) 
==> js-d3-graph-plot(
        highlight => @dsMovieRecords.map(*<Actor>).List,
        :$background, 
        title-color => 'silver',  
        width => 1000, 
        :$edge-thickness,
        :$vertex-size,
        vertex-color => 'Red',
        vertex-label-font-size => 12,
        vertex-label-color => 'Grey',
        vertex-label-font-family => 'Helvetica',
        :!directed,
        force => {charge => {strength => -680, iterations => 2}, collision => {radius => 10, iterations => 1}, link => {minDistance => 10}}
    )
```

------

## Sparse matrix


Here we make the sparse matrix for movie-actor starring relationship:

```raku
my @allVertexNames = [|@dsMovieRecords.map(*<Movie>).unique.sort, |@dsMovieRecords.map(*<Actor>).unique.sort];
my %h = @allVertexNames Z=> ^@allVertexNames.elems;
```

Note the row- and column names: the sorted movie titles are first and are followed by the sorted actor names:

```raku
.say for @allVertexNames
```

Here we make the sparse matrix of the bi-partite graph:

```raku
my $m = Math::SparseMatrix.new(edge-dataset => $g.edges(:dataset))
#my $m = Math::SparseMatrix.new(edge-dataset => $g.edges(:dataset), row-names => @allVertexNames)
```

```raku
#%js
$m.Array ==> js-d3-matrix-plot(width=>400)
```

It is not obvious that the matrix represents bipartite graph, hence we "restructure" it by using pre-arranged movie-actor row- and column-names:

```raku
$m = $m[@allVertexNames; @allVertexNames]
```

Now the matrix plot clearly shows the corresponding graph is bipartite:

```raku
#%js
$m.Array ==> js-d3-matrix-plot(width=>400)
```

Instead of a matrix plot we can make an HTML "pretty print" of the sparse matrix:

```raku
#% html

$m
.to-html(:v)
.subst('<td>1</td>', '<td><b>●</b></td>', :g)
```

----

## Fundamental information retrieval operation


- Get row / vector corresponding to an actor 
- Transpose it

```raku
#%html
my $m-actor = $m['Orlando Bloom'].transpose;
$m-actor.to-html.subst('<td>0</td>','<td> </td>'):g
```

Multiply the incidence matrix with the actor-vector:

```raku
#% html
$m.dot($m-actor).to-html
```

-----

## Matrix plot (*details*)


Two ways to plot sparse matrices.


### Via tuples


Essentially, using a heatmap plot spec:

```raku
#% js
my @ds3D = $m.tuples.map({ <x y z tooltip>.Array Z=> [|$_.Array, "⎡{$m.row-names[$_[0]]}⎦ : ⎡{$m.column-names[$_[1]]}⎦ : {$_.tail}"] })».Hash;
js-d3-matrix-plot(
    @ds3D, 
    :$tooltip-background-color, 
    :$tooltip-color, 
    :$background, 
    width => 400)
```

Here is the corresponding ("coordinates") list plot:

```raku
#%js
$m.tuples
==> js-d3-list-plot( :$background, width => 400, :!grid-lines)
```

### As dense matrix

```raku
#%js
$m.Array
==> js-d3-matrix-plot(width => 400)
```

### Larger sparse matrix


Large sparse matrix:

```raku
my $gLarge = Graph::Random.new: Graph::Distribution::WattsStrogatz.new(100,0.1);
my $mLarge = Math::SparseMatrix.new(edge-dataset => $gLarge.edges(:dataset));
```

Corresponding graph:

```raku
#% js
$mLarge.tuples
==> js-d3-list-plot( :$background, width => 600, height => 600, :!grid-lines)
```

**Remark:** The list plot might be much more useful for large matrices with (relatively) high density.


Tuples dataset:

```raku
#%js
$mLarge.tuples(:dataset)
==> {rename-columns($_, (<i j x> Z=> <x y z>).Hash)}()
==> js-d3-matrix-plot(:$background, width => 600)
```

### Random dense matrix


Another example with a dense matrix:

```raku
#%js
my @a = random-real(10, 48) xx 12;
@a = rand > 0.5 ?? @a.map(*.sort) !! @a.&transpose.map(*.sort.Array).&transpose;
say "dimensions : ", dimensions(@a);
js-d3-matrix-plot(@a, width => 1600, margins => 1, :$tick-labels-font-size, color-palette => <Turbo Plasma Warm Inferno Oranges>.pick, :$background)
```

-------

## Reference


### Articles

[AA1] Anton Antonov,
["RSparseMatrix for sparse matrices with named rows and columns"](https://mathematicaforprediction.wordpress.com/2015/10/08/rsparsematrix-for-sparse-matrices-with-named-rows-and-columns/),
(2015),
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).


### Packages

[AAp1] Anton Antonov,
[Math::SparseMatrix Raku package](https://github.com/antononcube/Raku-Math-SparseMatrix),
(2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[Math::SparseMatrix::Native Raku package](https://github.com/antononcube/Raku-Math-SparseMatrix-Native),
(2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[Graph Raku package](https://github.com/antononcube/Raku-Graph),
(2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[JavaScript::D3 Raku package](https://github.com/antononcube/Raku-JavaScript-D3),
(2022-2024),
[GitHub/antononcube](https://github.com/antononcube).
