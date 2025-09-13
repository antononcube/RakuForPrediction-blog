# LLM::Graph plots interpretation guide

Anton Antonov   
August 2025


----

## Introduction


This document ([notebook](https://github.com/antononcube/Raku-LLM-Graph/blob/main/docs/Graph-plots-interpretation-guide.ipynb)) provides visual dictionaries for the interpretation of graph-plots of LLM-graphs, [AAp1, AAp2].

The "orthogonal style" LLM-graph plot is used in ["Agentic-AI for text summarization"](https://rakuforprediction.wordpress.com/2025/09/02/agentic-ai-for-text-summarization/), [AA1].

---

## Setup

```raku
use LLM::Graph;
```

----

## LLM graph


Node specs:

```raku
sink my %rules =
        poet1 => "Write a short poem about summer.",
        poet2 => "Write a haiku about winter.",
        poet3 => sub ($topic, $style) {
            "Write a poem about $topic in the $style style."
        },
        poet4 => {
                llm-function => {llm-synthesize('You are a famous Russian poet. Write a short poem about playing bears.')},
                test-function => -> $with-russian { $with-russian ~~ Bool:D && $with-russian || $with-russian.Str.lc ∈ <true yes> }
        },
        judge => sub ($poet1, $poet2, $poet3, $poet4) {
            [
                "Choose the composition you think is best among these:\n\n",
                "1) Poem1: $poet1",
                "2) Poem2: $poet2",
                "3) Poem3: {$poet4.defined && $poet4 ?? $poet4 !! $poet3}",
                "and copy it:"
            ].join("\n\n")
        },
        report => {
            eval-function => sub ($poet1, $poet2, $poet3, $poet4, $judge) {
                [
                    '# Best poem',
                    'Three poems were submitted. Here are the statistics:',
                    to-html( ['poet1', 'poet2', $poet4.defined && $poet4 ?? 'poet4' !! 'poet3'].map({ [ name => $_, |text-stats(::('$' ~ $_))] })».Hash.Array, field-names => <name chars words lines> ),
                    '## Judgement',
                    $judge
                ].join("\n\n")
            }
        }
    ;
```

**Remark:** This is a documentation example -- I want to be seen that `$poet4` can be undefined. That hints that the corresponding sub is not always evaluated. 
(Because of the result of the corresponding test function.)

Make the graph:

```raku
my $gBestPoem = LLM::Graph.new(%rules)
```

Now, to make the execution quicker, we assign the poems (instead of LLM generating them):

```raku
# Poet 1
my $poet1 = q:to/END/;
Golden rays through skies so blue,
Whispers warm in morning dew.
Laughter dances on the breeze,
Summer sings through rustling trees.

Fields of green and oceans wide,
Endless days where dreams abide.
Sunset paints the world anew,
Summer’s heart in every hue.
END

# Poet 2
my $poet2 = q:to/END/;
Silent snowflakes fall,
Blanketing the earth in white,
Winter’s breath is still.
END

# Poet 3
my $poet3 = q:to/END/;
There once was a game on the ice,  
Where players would skate fast and slice,  
With sticks in their hands,  
They’d score on the stands,  
Making hockey fans cheer twice as nice!
END

# Poet 4
sink my $poet4 = q:to/END/;
В лесу играют медведи —  
Смех разносится в тиши,  
Тяжело шагают твердо,  
Но в душе — мальчишки.

Плюшевые лапы сильны,  
Игривы глаза блестят,  
В мире грёз, как в сказке дивной,  
Детство сердце охраняет.
END

sink my $judge = q:to/END/;
The 3rd one.
END
```

----

## Graph evaluation


Evaluate the LLM graph with input arguments and intermediate nodes results:

```raku
$gBestPoem.eval(topic => 'Hockey', style => 'limerick', with-russian => 'yes', :$poet1, :$poet2, :$poet3, :$poet4)
#$gBestPoem.eval(topic => 'Hockey', style => 'limerick', with-russian => 'yes')
```

Here is the final result (of the node "report"):

```raku
#% markdown
$gBestPoem.nodes<report><result>
```

# Best poem

Three poems were submitted. Here are the statistics:

<table border="1"><thead><tr><th>name</th><th>chars</th><th>words</th><th>lines</th></tr></thead><tbody><tr><td>poet1</td><td>259</td><td>42</td><td>9</td></tr><tr><td>poet2</td><td>81</td><td>12</td><td>3</td></tr><tr><td>poet4</td><td>209</td><td>33</td><td>9</td></tr></tbody></table>

## Judgement

I think Poem1 is the best among these. Here it is:

Golden rays through skies so blue,  
Whispers warm in morning dew.  
Laughter dances on the breeze,  
Summer sings through rustling trees.

Fields of green and oceans wide,  
Endless days where dreams abide.  
Sunset paints the world anew,  
Summer’s heart in every hue.

-----

## Default style


Here is the [Graphviz DOT](https://graphviz.org) visualization of the LLM graph:

```raku
#% html
$gBestPoem.dot(engine => 'dot', :9graph-size, node-width => 1.2, node-color => 'grey', edge-width => 0.8):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/LLM-graph-plots-interpretation-guide/Default-style.svg)

Here are the node spec-types:

```raku
$gBestPoem.nodes.nodemap(*<spec-type>)
```

Here is a dictionary of the shapes and the corresponding node spec-types:


| Graph node / edge                    | DOT shape / form                                               | SVG preview                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain text                           | `shape=egg`                                                     | <svg width="120" height="60" viewBox="0 0 120 60" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="egg"><path d="M60 6 C78 6 98 22 98 38 C98 50 80 56 60 56 C40 56 22 50 22 38 C22 22 42 6 60 6 Z" fill="SteelBlue" stroke="Grey" stroke-width="2"/></svg>                                                                                                                                                                    |
| LLM function (functor)                        | `shape=ellipse`                                                 | <svg width="120" height="60" viewBox="0 0 120 60" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="ellipse"><ellipse cx="60" cy="30" rx="45" ry="22" fill="SteelBlue" stroke="Grey" stroke-width="2"/></svg>                                                                                                                                                                                                                  |
| Evaluation function, wrapped<br>LLM-submit   | `shape=ellipse` + cluster with `style=dashed` | <svg width="120" height="60" viewBox="0 0 120 60" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="ellipse inside dashed rectangle"><rect x="6" y="6" width="108" height="48" fill="none" stroke="Grey" stroke-width="2" stroke-dasharray="6 4"/><ellipse cx="60" cy="30" rx="40" ry="18" fill="SteelBlue" stroke="Grey" stroke-width="2"/></svg>                                                                            |
| Evaluation function<br>no LLM-submit         | `shape=box`                                                     | <svg width="120" height="60" viewBox="0 0 120 60" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="box"><rect x="18" y="12" width="84" height="36" fill="SteelBlue" stroke="Grey" stroke-width="2"/></svg>                                                                                                                                                                                                                    |
| Input argument                       | `shape=parallelogram`                                           | <svg width="120" height="60" viewBox="0 0 120 60" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="parallelogram"><polygon points="28,12 108,12 92,48 12,48" fill="SteelBlue" stroke="Grey" stroke-width="2"/></svg>                                                                                                                                                                                                          |
| Dependency between nodes  | edge with default (solid) style                    | <svg width="160" height="40" viewBox="0 0 160 40" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="solid arrow"><defs><marker id="arrowSolid" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto"><polygon points="0 0, 10 3.5, 0 7" fill="SteelBlue"/></marker></defs><line x1="10" y1="20" x2="150" y2="20" stroke="SteelBlue" stroke-width="2" marker-end="url(#arrowSolid)"/></svg>                           |
| Conditional dependency between nodes | edge with `style=dashed`                                        | <svg width="160" height="40" viewBox="0 0 160 40" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="dashed arrow"><defs><marker id="arrowDashed" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto"><polygon points="0 0, 10 3.5, 0 7" fill="SteelBlue"/></marker></defs><line x1="10" y1="20" x2="150" y2="20" stroke="SteelBlue" stroke-width="2" stroke-dasharray="6 4" marker-end="url(#arrowDashed)"/></svg> |



-----

## Specified shapes


Here different node shapes are specified and the edges are additionally styled:

```raku
#% html
$gBestPoem.dot(
    engine => 'dot', :9graph-size, node-width => 1.2, node-color => 'Grey', 
    edge-color => 'DimGrey', edge-width => 0.8, splines => 'ortho',
    node-shapes => {
        Str => 'note', 
        Routine => 'doubleoctagon', 
        :!RoutineWrapper, 
        'LLM::Function' => 'octagon' 
    }
):svg
```

Similar visual effect is achieved with the option spec `theme => 'ortho'`:

```raku
$gBestPoem.dot(node-width => 1.2, theme => 'ortho'):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/LLM-graph-plots-interpretation-guide/Ortho-style.svg)


**Remark:** The option "theme" takes the values "default", "ortho", and `Whatever`. 


Here is the corresponding dictionary:


| Graph node / edge                    | DOT form / style                       | SVG preview                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------ | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain text                           | `shape=note`                           | <svg width="140" height="70" viewBox="0 0 140 70" xmlns="http://www.w3.org/2000/svg" aria-label="note"><path d="M20,10 h80 l20,20 v30 h-100 z" fill="#4682B4" stroke="#808080" stroke-width="2"/><line x1="100" y1="10" x2="120" y2="30" stroke="#808080" stroke-width="2"/></svg>                                                                                                                                                 |
| LLM function (functor)               | `shape=octagon`                        | <svg width="140" height="70" viewBox="0 0 140 70" xmlns="http://www.w3.org/2000/svg" aria-label="octagon"><polygon points="40,10 100,10 120,25 120,45 100,60 40,60 20,45 20,25" fill="#4682B4" stroke="#808080" stroke-width="2"/></svg>                                                                                                                                                                                           |
| Evaluation function, wrapped<br>LLM-submit  | `shape=doubleoctagon`                  | <svg width="140" height="70" viewBox="0 0 140 70" xmlns="http://www.w3.org/2000/svg" aria-label="doubleoctagon"><polygon points="28,12 112,12 128,26 128,44 112,58 28,58 12,44 12,26" fill="none" stroke="#808080" stroke-width="2"/><polygon points="36,18 104,18 116,27 116,43 104,52 36,52 24,43 24,27" fill="SteelBlue" stroke="#808080" stroke-width="2"/></svg>                                                                |
| Evaluation function<br>no LLM-submit | `shape=box`                            | <svg width="140" height="70" viewBox="0 0 140 70" xmlns="http://www.w3.org/2000/svg" aria-label="box"><rect x="28" y="17" width="84" height="36" fill="#4682B4" stroke="#808080" stroke-width="2"/></svg>                                                                                                                                                                                                                          |
| Input argument                       | `shape=parallelogram`                  | <svg width="140" height="70" viewBox="0 0 140 70" xmlns="http://www.w3.org/2000/svg" aria-label="parallelogram"><polygon points="32,17 120,17 104,53 16,53" fill="#4682B4" stroke="#808080" stroke-width="2"/></svg>                                                                                                                                                                                                               |
| Dependency between nodes  | edge with default (solid) style.       | <svg width="180" height="50" viewBox="0 0 180 50" xmlns="http://www.w3.org/2000/svg" aria-label="solid arrow"><defs><marker id="arrowSolidGray" markerWidth="12" markerHeight="8" refX="10" refY="4" orient="auto"><polygon points="0 0, 10 4, 0 8" fill="#808080"/></marker></defs><line x1="10" y1="25" x2="170" y2="25" stroke="#808080" stroke-width="2.5" marker-end="url(#arrowSolidGray)"/></svg>                           |
| Conditional dependency between nodes | edge with `style=dashed`               | <svg width="180" height="50" viewBox="0 0 180 50" xmlns="http://www.w3.org/2000/svg" aria-label="dashed arrow"><defs><marker id="arrowDashedGray" markerWidth="12" markerHeight="8" refX="10" refY="4" orient="auto"><polygon points="0 0, 10 4, 0 8" fill="#808080"/></marker></defs><line x1="10" y1="25" x2="170" y2="25" stroke="#808080" stroke-width="2.5" stroke-dasharray="6 4" marker-end="url(#arrowDashedGray)"/></svg> |



---

## References

### Articles, blog posts

[AA1] Anton Antonov,
["Agentic-AI for text summarization"](https://rakuforprediction.wordpress.com/2025/09/02/agentic-ai-for-text-summarization/),
(2025),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

### Packages

[AAp1] Anton Antonov, 
[LLM::Graph, Raku package](https://github.com/antononcube/Raku-LLM-Graph),
(2025),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, 
[Graph, Raku package](https://github.com/antononcube/Raku-Graph),
(2024-2025),
[GitHub/antononcube](https://github.com/antononcube).
