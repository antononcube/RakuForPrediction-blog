# Monadic programming examples

Anton Antonov   
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com)  
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
November, December 2025

----

## Introduction

This document ([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Monadic-programming-examples.ipynb)) provides examples of monadic pipelines for computational workflows in Raku. It expands on the blog post ["Monad laws in Raku"](https://rakuforprediction.wordpress.com/2025/11/16/monad-laws-in-raku/), [AA2], ([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Monad-laws-in-Raku.ipynb)), by including practical, real-life examples.

### Context

As mentioned in [AA2], here is a list of the applications of monadic programming we consider:

1. Graceful failure handling

2. Rapid specification of computational workflows

3. Algebraic structure of written code

**Remark:** Those applications are discussed in [AAv5] (and its future Raku version.)

As [a tools maker for Data Science (DS) and Machine Learning (ML)](https://raku-advent.blog/2025/12/02/day-2-doing-data-science-with-raku/), [AA3],
I am very interested in Point 1; but as a "simple data scientist" I am mostly interested in Point 2.

That said, a large part of my Raku programming has been dedicated to rapid and reliable code generation for DS and ML by leveraging the algebraic structure of corresponding software monads, i.e. Point 3. (See [AAv2, AAv3, AAv4].) For me, first and foremost, ***monadic programming pipelines are just convenient interfaces to computational workflows***. Often I make software packages that allow "easy", linear workflows that can have very involved computational steps and multiple tuning options.

### Dictionary

- **Monadic programming**   
  A method for organizing computations as a series of steps, where each step generates a value along with additional information about the computation, such as possible failures, non-determinism, or side effects. See [Wk1].

- **Monadic pipeline**   
  Chaining of operations with a certain syntax. Monad laws apply loosely (or strongly) to that chaining.

- **Uniform Function Call Syntax (UFCS)**  
  A feature that allows both free functions and member functions to be called using the same `object.function()` method call syntax.

- **Method-like call**   
  Same as UFCS. A Raku example: `[3, 4, 5].&f1.$f2`.

---

## Setup

Here are loaded packages used in this document (notebook):


```raku
use Data::Reshapers;
use Data::TypeSystem;
use Data::Translators;

use DSL::Translators;
use DSL::Examples;

use ML::SparseMatrixRecommender;
use ML::TriesWithFrequencies;

use Hilite::Simple;
```

---

## Prefix trees

Here is a list of steps:

- Make a prefix tree (trie) with frequencies by splitting words into characters over `@words2`
- Merge the trie with another trie made over `@words3`
- Convert the node frequencies into probabilities
- Shrink the trie (i.e. find the "prefixes")
- Show the tree-form of the trie

Let us make a small trie of pet names (used by Raku or Perl fans):


```raku
sink my @words1 = random-pet-name(*)».lc.grep(/ ^ perl /);
sink my @words2 = random-pet-name(*)».lc.grep(/ ^ [ra [k|c] | camel ] /);
```

Here we make a trie (prefix tree) for those pet names using the feed operator and the functions of ["ML::TriesWithFrequencies"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies), [AAp5]:


```raku
@words1 ==> 
trie-create-by-split==>
trie-merge(@words2.&trie-create-by-split) ==>
trie-node-probabilities==>
trie-shrink==>
trie-say
```

    TRIEROOT => 1
    ├─camel => 0.10526315789473684
    │ ├─ia => 0.5
    │ └─o => 0.5
    ├─perl => 0.2631578947368421
    │ ├─a => 0.2
    │ ├─e => 0.2
    │ └─ita => 0.2
    └─ra => 0.631578947368421
      ├─c => 0.75
      │ ├─er => 0.2222222222222222
      │ ├─he => 0.5555555555555556
      │ │ ├─al => 0.2
      │ │ └─l => 0.8
      │ │   └─  => 0.5
      │ │     ├─(ray ray) => 0.5
      │ │     └─ray => 0.5
      │ ├─ie => 0.1111111111111111
      │ └─ket => 0.1111111111111111
      └─k => 0.25
        ├─i => 0.3333333333333333
        └─sha => 0.6666666666666666


Using `andthen` and the `Trie` class methods (but skipping node-probabilities calculation in order to see the counts):


```raku
@words1
andthen .&trie-create-by-split
andthen .merge( @words2.&trie-create-by-split )
# andthen .node-probabilities
andthen .shrink
andthen .form
```




    TRIEROOT => 19
    ├─camel => 2
    │ ├─ia => 1
    │ └─o => 1
    ├─perl => 5
    │ ├─a => 1
    │ ├─e => 1
    │ └─ita => 1
    └─ra => 12
      ├─c => 9
      │ ├─er => 2
      │ ├─he => 5
      │ │ ├─al => 1
      │ │ └─l => 4
      │ │   └─  => 2
      │ │     ├─(ray ray) => 1
      │ │     └─ray => 1
      │ ├─ie => 1
      │ └─ket => 1
      └─k => 3
        ├─i => 1
        └─sha => 2



---- 

## Data wrangling

One appealing way to show that monadic pipelines result in clean and readable code, is to demonstrate their use in Raku through data wrangling operations. (See the "data packages" loaded above.)
Here we get the Titanic dataset, show its structure, and show a sample of its rows:


```raku
#% html
my @dsTitanic = get-titanic-dataset();
my @field-names = <id passengerClass passengerSex passengerAge passengerSurvival>;

say deduce-type(@dsTitanic);

@dsTitanic.pick(6) 
==> to-html(:@field-names)
```

    Vector(Assoc(Atom((Str)), Atom((Str)), 5), 1309)





<table border="1"><thead><tr><th>id</th><th>passengerClass</th><th>passengerSex</th><th>passengerAge</th><th>passengerSurvival</th></tr></thead><tbody><tr><td>960</td><td>3rd</td><td>male</td><td>30</td><td>died</td></tr><tr><td>183</td><td>1st</td><td>female</td><td>30</td><td>survived</td></tr><tr><td>1043</td><td>3rd</td><td>female</td><td>-1</td><td>survived</td></tr><tr><td>165</td><td>1st</td><td>male</td><td>40</td><td>survived</td></tr><tr><td>891</td><td>3rd</td><td>male</td><td>20</td><td>died</td></tr><tr><td>806</td><td>3rd</td><td>male</td><td>-1</td><td>survived</td></tr></tbody></table>



Here is an `andthen` data wrangling monadic pipeline, the lines of which have the following interpretations:

- Initial pipeline value (the dataset)
- Rename columns
- Filter rows (with age greater or equal to 10)
- Group by the values of the columns "sex" and "survival"
- Show the structure of the pipeline value
- Give the sizes of each group as a result


```raku
@dsTitanic 
andthen rename-columns($_,  {passengerAge => 'age', passengerSex => 'sex', passengerSurvival => 'survival'})
andthen $_.grep(*<age> ≥ 10).List
andthen group-by($_, <sex survival>)
andthen {say "Dataset type: ", deduce-type($_); $_}($_)
andthen $_».elems
```

    Dataset type: Struct([female.died, female.survived, male.died, male.survived], [Array, Array, Array, Array])





    {female.died => 88, female.survived => 272, male.died => 512, male.survived => 118}



**Remark:** The `andthen` pipeline corresponds to the R pipeline in the next section.

Similar result can be obtained via cross-tabulation and using a pipeline with the feed (`==>`) operator:


```raku
@dsTitanic
==> { .grep(*<passengerAge> ≥ 10) }()
==> { cross-tabulate($_, 'passengerSex', 'passengerSurvival') }()
==> to-pretty-table()
```




    +--------+----------+------+
    |        | survived | died |
    +--------+----------+------+
    | female |   272    |  88  |
    | male   |   118    | 512  |
    +--------+----------+------+



Tries with frequencies can be also used for finding this kind of (*deep*) contingency tensors (not just some shallow tables):


```raku
@dsTitanic
andthen $_.map(*<passengerSurvival passengerSex passengerClass>)
andthen .&trie-create
andthen .form
```




    TRIEROOT => 1309
    ├─died => 809
    │ ├─female => 127
    │ │ ├─1st => 5
    │ │ ├─2nd => 12
    │ │ └─3rd => 110
    │ └─male => 682
    │   ├─1st => 118
    │   ├─2nd => 146
    │   └─3rd => 418
    └─survived => 500
      ├─female => 339
      │ ├─1st => 139
      │ ├─2nd => 94
      │ └─3rd => 106
      └─male => 161
        ├─1st => 61
        ├─2nd => 25
        └─3rd => 75



**Remark:** This application of Tries with frequencies can be leveraged in making [mosaic plots](https://en.wikipedia.org/wiki/Mosaic_plot). (See this [`MosaicPlot` implementation in Wolfram Language](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MosaicPlot/), [AAp8].)

----

## Data wrangling code with multiple languages and packages

Let us demonstrate the *rapid specification of workflows* application by generating data wrangling code from natural language commands. Here is a natural language workflow spec (each row corresponds to a pipeline segment):


```raku
sink my $commands = q:to/END/;
use dataset dfTitanic;
rename columns passengerAge as age, passengerSex as sex, passengerClass as class;
filter by age ≥ 10;
group by 'class' and 'sex';
counts;
END
```

### Grammar based interpreters

Here is a table with the generated codes for different programming languages according to the spec above (using ["DSL::English::DataQueryWorkflows"](https://raku.land/zef:antononcube/DSL::English::DataQueryWorkflows), [AAp3]):


```raku
#% html
my @tbl = <Python R Raku WL>.map({ %( language => $_, code => ToDSLCode($commands, format=>'code', target => $_) ) });
to-html(@tbl, field-names => <language code>, align => 'left').subst("\n", '<br>', :g)
```




<table border="1"><thead><tr><th>language</th><th>code</th></tr></thead><tbody><tr><td align=left>Python</td><td align=left>obj = dfTitanic.copy()<br>obj = obj.assign( age = obj[&quot;passengerAge&quot;], sex = obj[&quot;passengerSex&quot;], class = obj[&quot;passengerClass&quot;] )<br>obj = obj[((obj[&quot;age&quot;]&gt;= 10))]<br>obj = obj.groupby([&quot;class&quot;, &quot;sex&quot;])<br>obj = obj.size()</td></tr><tr><td align=left>R</td><td align=left>dfTitanic %&gt;%<br>dplyr::rename(age = passengerAge, sex = passengerSex, class = passengerClass) %&gt;%<br>dplyr::filter(age &gt;= 10) %&gt;%<br>dplyr::group_by(class, sex) %&gt;%<br>dplyr::count()</td></tr><tr><td align=left>Raku</td><td align=left>$obj = dfTitanic ;<br>$obj = rename-columns( $obj, %(&quot;passengerAge&quot; =&gt; &quot;age&quot;, &quot;passengerSex&quot; =&gt; &quot;sex&quot;, &quot;passengerClass&quot; =&gt; &quot;class&quot;) ) ;<br>$obj = $obj.grep({ $_{&quot;age&quot;} &gt;= 10 }).Array ;<br>$obj = group-by($obj, (&quot;class&quot;, &quot;sex&quot;)) ;<br>$obj = $obj&gt;&gt;.elems</td></tr><tr><td align=left>WL</td><td align=left>obj = dfTitanic;<br>obj = Map[ Join[ KeyDrop[ #, {&quot;passengerAge&quot;, &quot;passengerSex&quot;, &quot;passengerClass&quot;} ], &lt;|&quot;age&quot; -&gt; #[&quot;passengerAge&quot;], &quot;sex&quot; -&gt; #[&quot;passengerSex&quot;], &quot;class&quot; -&gt; #[&quot;passengerClass&quot;]|&gt; ]&amp;, obj];<br>obj = Select[ obj, #[&quot;age&quot;] &gt;= 10 &amp; ];<br>obj = GroupBy[ obj, {#[&quot;class&quot;], #[&quot;sex&quot;]}&amp; ];<br>obj = Map[ Length, obj]</td></tr></tbody></table>



Executing the Raku pipeline (by replacing `dfTitanic` with `@dsTitanic` first):


```raku
my $obj = @dsTitanic;
$obj = rename-columns( $obj, %("passengerAge" => "age", "passengerSex" => "sex", "passengerClass" => "class") ) ;
$obj = $obj.grep({ $_{"age"} >= 10 }).Array ;
$obj = group-by($obj, ("class", "sex")) ;
$obj = $obj>>.elems
```




    {1st.female => 132, 1st.male => 149, 2nd.female => 96, 2nd.male => 149, 3rd.female => 132, 3rd.male => 332}



That is not monadic, of course -- see the monadic version above.

----

## LLM generated (via DSL examples)

Here we define an LLM-examples function for translation of natural language commands into code using DSL examples (provided by ["DSL::Examples"](https://raku.land/zef:antononcube/DSL::Examples), [AAp6]):


```raku
my sub llm-pipeline-segment($lang, $workflow-name = 'DataReshaping') { llm-example-function(dsl-examples(){$lang}{$workflow-name}) };
```




    &llm-pipeline-segment



Here is the LLM translated code:


```raku
my $code = llm-pipeline-segment('Raku', 'DataReshaping')($commands)
```




    use Data::Reshapers; use Data::Summarizers; use Data::TypeSystem
    my $obj = @dfTitanic;
    $obj = rename-columns($obj, %(passengerAge => 'age', passengerSex => 'sex', passengerClass => 'class'));
    $obj = $obj.grep({ $_{'age'} >= 10 }).Array;
    $obj = group-by($obj, ('class', 'sex'));
    $obj = $obj>>.elems;



Here the translated code *is turned into monadic code* by string manipulation:


```raku
my $code-mon =$code.subst(/ $<lhs>=('$' \w+) \h+ '=' \h+ (\S*)? $<lhs> (<-[;]>*) ';'/, {"==> \{{$0}\$_{$1} \}()"} ):g;
$code-mon .= subst(/ $<printer>=[note|say] \h* $<lhs>=('$' \w+) ['>>'|»] '.elems' /, {"==> \{$<printer> \$_>>.elems\}()"}):g;
```




    use Data::Reshapers; use Data::Summarizers; use Data::TypeSystem
    my $obj = @dfTitanic;
    ==> {rename-columns($_, %(passengerAge => 'age', passengerSex => 'sex', passengerClass => 'class')) }()
    ==> {$_.grep({ $_{'age'} >= 10 }).Array }()
    ==> {group-by($_, ('class', 'sex')) }()
    ==> {$_>>.elems }()



**Remark:** It is believed that the string manipulation shown above provides insight into how and why monadic pipelines make imperative code simpler.

----

## Recommendation pipeline

Here is a computational specification for creating a recommender and obtaining a profile recommendation:


```raku
sink my $spec = q:to/END/;
create from @dsTitanic; 
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsTitanic on "id";
echo the pipeline value;
END
```

Here is the Raku code for that spec given as an HTML code snipped with code-highlights:


```raku
#%html
ToDSLCode($spec, default-targets-spec => 'Raku', format => 'code')
andthen .subst('.', "\n.", :g)
andthen hilite($_)
```




<div style="font-weight: 500 .nohighlights { background; none color: inherit" class="raku-code"><div><pre class="nohighlights" style="font-size: 1em; font-family: monospace"><span style="color: #008c7e" class="rainbow-keyword">my</span>&nbsp;<span style="color: #2458a2" class="rainbow-name_scalar">$obj</span>&nbsp;<span style="color: #1ca24f" class="rainbow-operator">=</span>&nbsp;ML::SparseMatrixRecommender<br>.<span style="color: #489fdc" class="rainbow-routine">new</span><br>.create-from-wide-form(<span style="color: #B01030" class="rainbow-name_array">@dsTitanic</span>)<br>.apply-term-weight-functions(global-weight-func&nbsp;<span style="color: #1ca24f" class="rainbow-operator">=&gt;</span>&nbsp;<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">IDF</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #1ca24f" class="rainbow-operator">,</span>&nbsp;local-weight-func&nbsp;<span style="color: #1ca24f" class="rainbow-operator">=&gt;</span>&nbsp;<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">None</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #1ca24f" class="rainbow-operator">,</span>&nbsp;normalizer-func&nbsp;<span style="color: #1ca24f" class="rainbow-operator">=&gt;</span>&nbsp;<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">Cosine</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span>)<br>.recommend-by-profile([<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">passengerSex:male</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #1ca24f" class="rainbow-operator">,</span>&nbsp;<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">passengerClass:1st</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span>])<br>.join-across(<span style="color: #B01030" class="rainbow-name_array">@dsTitanic</span><span style="color: #1ca24f" class="rainbow-operator">,</span>&nbsp;on&nbsp;<span style="color: #1ca24f" class="rainbow-operator">=&gt;</span>&nbsp;<span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span><span style="color: #369ec6" class="rainbow-string">id</span><span style="color: #1d90d2" class="rainbow-string_delimiter">&quot;</span>&nbsp;)<br>.echo-value()</pre></div></div>



Here we execute a slightly modified version of the pipeline (based on ["ML::SparseMatrixRecommender"](https://raku.land/zef:antononcube/ML::SparseMatrixRecommender), [AAp7]):


```raku
sink my $obj = ML::SparseMatrixRecommender.new
.create-from-wide-form(@dsTitanic)
.apply-term-weight-functions("IDF", "None", "Cosine")
.recommend-by-profile(["passengerSex:male", "passengerClass:1st"])
.join-across(@dsTitanic, on => "id" )
.echo-value(as => {to-pretty-table($_, )} )
```

    +----------------+-----+--------------+-------------------+----------+--------------+
    | passengerClass |  id | passengerAge | passengerSurvival |  score   | passengerSex |
    +----------------+-----+--------------+-------------------+----------+--------------+
    |      1st       |  10 |      70      |        died       | 1.000000 |     male     |
    |      1st       | 101 |      50      |      survived     | 1.000000 |     male     |
    |      1st       | 102 |      40      |        died       | 1.000000 |     male     |
    |      1st       | 107 |      -1      |        died       | 1.000000 |     male     |
    |      1st       |  11 |      50      |        died       | 1.000000 |     male     |
    |      1st       | 110 |      40      |      survived     | 1.000000 |     male     |
    |      1st       | 111 |      30      |        died       | 1.000000 |     male     |
    |      1st       | 115 |      20      |        died       | 1.000000 |     male     |
    |      1st       | 116 |      60      |        died       | 1.000000 |     male     |
    |      1st       | 119 |      -1      |        died       | 1.000000 |     male     |
    |      1st       | 120 |      50      |      survived     | 1.000000 |     male     |
    |      1st       | 121 |      40      |      survived     | 1.000000 |     male     |
    +----------------+-----+--------------+-------------------+----------+--------------+


----

## Functional parsers (multi-operation pipelines)

In can be said that the package ["FunctionalParsers"](https://raku.land/zef:antononcube/FunctionalParsers), [AAp4], implements multi-operator monadic pipelines for the creation of parsers and interpreters. "FunctionalParsers" achieves that using special infix implementations.


```raku
use FunctionalParsers :ALL;
my &p1 = {1} ⨀ symbol('one');
my &p2 = {2} ⨀ symbol('two');
my &p3 = {3} ⨀ symbol('three');
my &p4 = {4} ⨀ symbol('four');
my &pH = {10**2} ⨀ symbol('hundred');
my &pT = {10**3} ⨀ symbol('thousand');
my &pM = {10**6} ⨀ symbol('million');
sink my &pNoun = symbol('things') ⨁ symbol('objects');
```

Here is a parser -- all three monad operations (`⨁`, `⨂`, `⨀`) are used:


```raku
# Parse sentences that have (1) a digit part, (2) a multiplier part, and (3) a noun
my &p = (&p1 ⨁ &p2 ⨁ &p3 ⨁ &p4) ⨂ (&pT ⨁ &pH ⨁ &pM) ⨂ &pNoun;

# Interpreter:
# (1) flatten the parsed elements
# (2) multiply the first two elements and make a sentence with the third element
sink &p = { "{$_[0] * $_[1]} $_[2]"} ⨀ {.flat} ⨀ &p 
```

Here the parser is applied to different sentences:


```raku
['three million things', 'one hundred objects', 'five thousand things']
andthen .map({ &p($_.words.List).head.tail })
andthen (.say for |$_)
```

    3000000 things
    100 objects
    Nil


The last sentence is not parsed because the parser `&p` knows only the digits from 1 to 4.

----

## References

### Articles, blog posts

[Wk1] Wikipedia entry: [Monad (functional programming)](https://en.wikipedia.org/wiki/Monad_(functional_programming)), URL: [https://en.wikipedia.org/wiki/Monad_(functional_programming)](https://en.wikipedia.org/wiki/Monad_(functional_programming)) .

[Wk2] Wikipedia entry: [Monad transformer](https://en.wikipedia.org/wiki/Monad_transformer), URL: [https://en.wikipedia.org/wiki/Monad_transformer](https://en.wikipedia.org/wiki/Monad_transformer) .

[H1] Haskell.org article: [Monad laws,](https://wiki.haskell.org/Monad_laws) URL: [https://wiki.haskell.org/Monad_laws](https://wiki.haskell.org/Monad_laws).

[SH2] Sheng Liang, Paul Hudak, Mark Jones, ["Monad transformers and modular interpreters",](http://haskell.cs.yale.edu/wp-content/uploads/2011/02/POPL96-Modular-interpreters.pdf) (1995), Proceedings of the 22nd ACM SIGPLAN-SIGACT symposium on Principles of programming languages. New York, NY: ACM. pp. 333--343. doi:10.1145/199448.199528.

[PW1] Philip Wadler, ["The essence of functional programming"](https://page.mi.fu-berlin.de/scravy/realworldhaskell/materialien/the-essence-of-functional-programming.pdf), (1992), 19'th Annual Symposium on Principles of Programming Languages, Albuquerque, New Mexico, January 1992.

[RW1] Hadley Wickham et al., [dplyr: A Grammar of Data Manipulation](https://github.com/tidyverse/dplyr), (2014), [tidyverse at GitHub](https://github.com/tidyverse), URL: [https://github.com/tidyverse/dplyr](https://github.com/tidyverse/dplyr) .
(See also, [http://dplyr.tidyverse.org](http://dplyr.tidyverse.org) .)

[AA1] Anton Antonov, ["Monad code generation and extension"](https://mathematicaforprediction.wordpress.com/2017/06/23/monad-code-generation-and-extension/), (2017), [MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).

[AA2] Anton Antonov, ["Monad laws in Raku"](https://rakuforprediction.wordpress.com/2025/11/16/monad-laws-in-raku/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov, ["Day 2 – Doing Data Science with Raku"](https://raku-advent.blog/2025/12/02/day-2-doing-data-science-with-raku/), (2025), [Raku Advent Calendar at WordPress](https://raku-advent.blog/).

### Packages, paclets

[AAp1] Anton Antonov, [MonadMakers](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MonadMakers/), Wolfram Language paclet, (2023), [Wolfram Language Paclet Repository](https://resources.wolframcloud.com/PacletRepository/).

[AAp2] Anton Antonov, [StatStateMonadCodeGeneratoreNon](https://github.com/antononcube/R-packages/tree/master/StateMonadCodeGenerator), R package, (2019-2024),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp3] Anton Antonov, [DSL::English::DataQueryWorkflows](https://github.com/antononcube/Raku-DSL-English-DataQueryWorkflows), Raku package, (2020-2024),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp4] Anton Antonov, [FunctionalParsers](https://github.com/antononcube/Raku-FunctionalParsers), Raku package, (2023-2024),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp5] Anton Antonov, [ML::TriesWithFrequencies](https://github.com/antononcube/Raku-ML-TriesWithFrequencies), Raku package, (2021-2024),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp6] Anton Antonov, [DSL::Examples](https://github.com/antononcube/Raku-DSL-Examples), Raku package, (2024-2025),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp7] Anton Antonov, [ML::SparseMatrixRecommender](https://github.com/antononcube/Raku-ML-SparseMatrixRecommender), Raku package, (2025),
[GitHub/@antononcube](https://github.com/antononcube/).

[AAp8] Anton Antonov, [MosaicPlot](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MosaicPlot/), Wolfram Language paclet, (2023), [Wolfram Language Paclet Repository](https://resources.wolframcloud.com/PacletRepository/).

### Videos

[AAv1] Anton Antonov, [Monadic Programming: With Application to Data Analysis, Machine Learning and Language Processing](https://www.youtube.com/watch?v=_cIFA5GHF58), (2017), Wolfram Technology Conference 2017 presentation. [YouTube/WolframResearch](https://www.youtube.com/@WolframResearch).

[AAv2] Anton Antonov, [Raku for Prediction](https://www.youtube.com/watch?v=frpCBjbQtnA), (2021), [The Raku Conference 2021](https://www.youtube.com/@therakuconference6823).

[AAv3] Anton Antonov, [Simplified Machine Learning Workflows Overview](https://www.youtube.com/watch?v=Xy7eV8wRLbE), (2022), Wolfram Technology Conference 2022 presentation. [YouTube/WolframResearch](https://www.youtube.com/@WolframResearch).

[AAv4] Anton Antonov, [Simplified Machine Learning Workflows Overview (Raku-centric)](https://www.youtube.com/watch?v=p3iwPsc6e74), (2022), Wolfram Technology Conference 2022 presentation. [YouTube/@AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv5] Anton Antonov, [Applications of Monadic Programming, Part 1, Questions & Answers](https://www.youtube.com/watch?v=Xz5B4B0kVco), (2025), [YouTube/@AAA4prediction](https://www.youtube.com/@AAA4prediction).
