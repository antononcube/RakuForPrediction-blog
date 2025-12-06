# Robust code generation combining grammars and LLMs

Anton Antonov  
RakuForPrediction at WordPress   
October, December 2025

----

## Introduction

This document 
([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Robust-code-generation-combining-grammars-and-LLMs.ipynb)) 
discusses different combinations of Grammar-Based Parser-Interpreters (GBPI) and Large Language Models (LLMs) to generate executable code from Natural Language Computational Specifications (NLCM). We have the _soft_ assumption that the NLCS adhere to a certain relatively small Domain Specific Language (DSL) or use terminology from that DSL. Another assumption is that the target software packages are not necessarily well-known by the LLMs, i.e. direct LLM requests for code using them would produce meaningless results.

We want to do such combinations because:

- GBPI are fast, precise, but with a narrow DSL scope
- LLMs can be unreliable and slow, but with a wide DSL scope 

Because of GBPI and LLMs are complementary technologies with similar and overlapping goals the possible combinations are many. 
We concentrate on two of the most straightforward designs: 
(1) judged parallel race of methods execution, and 
(2) using LLMs as a fallback method if grammar parsing fails. 
We show [asynchronous programming](https://en.wikipedia.org/wiki/Asynchrony_(computer_programming)) implementations for both designs using the package [LLM::Graph](https://raku.land/zef:antononcube/LLM::Graph).

The Machine Learning (ML) package ["ML::SparseMatrixRecommender"](https://raku.land/zef:antononcube/ML::SparseMatrixRecommender) is used to demonstrate that the generated code is executable. 

The rest of the document is structured as follows:

- Initial grammar-LLM combinations
    - Assumptions, straightforward designs, and trade-offs
- Comprehensive combinations enumeration (attempt)
    - Tabular and morphological analysis breakdown
- Three methods for parsing ML DSL specs into Raku code 
    - One grammar-based, two LLM-based
- Parallel execution with an LLM judge
    - Straightforward, but computationally wasteful and expensive
- Grammar-to-LLM fallback mechanism
    - The easiest and most robust solution
- Concluding comments and observations


### TL;DR

- Combining grammars and LLMs produces robust translators.
- Three translators with different faithfulness and coverage are demonstrated and used.
- Two of the simplest, yet effective, combinations are implemented and demonstrated.
    - Parallel race and grammar-to-LLM fallback.
- Asynchronous implementations with LLM-graphs are a very good fit!
    - Just look at the LLM-graph plots (and be done reading.)


----

## Initial Combinations and Associated Assumptions

The goal is to combine the core features of Raku with LLMs to achieve robust parsing and interpretation of computational workflow specifications.

Here are some example combinations of these approaches:

1. A few methods, both grammar-based and LLM-based, are initiated in parallel. Whichever method produces a correct result first is selected as the answer. 
   - This approach assumes that when the grammar-based methods are effective, they will finish more quickly than the LLM-based methods.

2. The grammar method is invoked first; if it fails, an LLM method (or a sequence of LLM methods) is employed.

3. LLMs are utilized at the grammar-rule level to provide matching objects that the grammar can work with.

4. If the grammar method fails, an LLM normalizer for user commands is invoked to generate specifications that the grammar can parse.

5. It is important to distinguish between declarative specifications and those that prescribe specific steps. 
   - For a workflow given as a list of steps the grammar parser may successfully parse most steps, but LLMs may be required for a few exceptions.

The main trade-off in these approaches is as follows:

- Grammar methods are challenging to develop but can be very fast and precise.
   - Precision can be guaranteed and rigorously tested.
  
- LLM methods are quicker to develop but tend to be slower and can be unreliable, particularly for less popular workflows, programming languages, and packages.

Also, combinations based on LLM tools (aka LLM external function calling) are not considered because LLM-tools invocation is to unpredictable and unreliable.

---

## Comprehensive breakdown (attempt)

This section has a "concise" table that expands the combinations list above into the main combinatorial strategies for **Raku core features × LLMs** for robust parsing and interpretation of workflow specifications. The table is not an exhaustive list of such combinations, but illustrates their diversity and, hopefully, can give ideas for future developments.

A few summary points (for table's content/subject):

- **Grammar (Raku regex/grammar)**

    - **Pros:** fast, deterministic, validated, reproducible
    - **Cons:** hard to design for large domains, brittle for natural language inputs

- **LLMs**

    - **Pros:** fast to prototype, excellent at normalization/paraphrasing, flexible
    - **Cons:** slow, occasionally wrong, hallucination risk, inconsistent output formats

- **Conclusion:**
    - The most robust systems combine *grammar precision* with *LLM adaptability*, typically by putting grammars first and using LLMs for repair, normalization, expansions, or semantic interpretation (i.e. "fallback".)


### Table: Combination Patterns for Parsing Workflow Specifications


| **#**  | **Combination Pattern**                               | **Description**                                                                                                             | **Pros**                                                                                        | **Cons / Trade-offs**                                                      |
| ------ | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **1**  | **Parallel Race: Grammar + LLM**                      | Launch Raku grammar parsing and one or more LLM interpreters in parallel; whichever yields a valid parse first is accepted. | • Fast when grammar succeeds<br>• Robust fallback<br>• Reduces latency unpredictability of LLMs | • Requires orchestration<br>• Need a validator for LLM output              |
| **2**  | **Grammar-First, LLM-Fallback**                       | Try grammar parser first; if it fails, invoke LLM-based parsing or normalization.                                           | • Deterministic preference for grammar<br>• Testable correctness when grammar succeeds          | • LLM fallback may produce inconsistent structures                         |
| **3**  | **LLM-Assisted Grammar (Rule-Level)**                 | Individual grammar rules delegate to an LLM for ambiguous or context-heavy matching; LLM supplies tokens or AST fragments.  | • Handles complexity without inflating grammar<br>• Modular LLM usage                           | • LLM behavior may break rule determinism<br>• Harder to reproduce         |
| **4**  | **LLM Normalizer → Grammar Parser**                   | When grammar fails, LLM rewrites/normalizes input into a canonical form; grammar is applied again.                          | • Grammar remains simple<br>• Leverages LLM skill at paraphrasing                               | • Quality depends on normalizer<br>• Feedback loops possible               |
| **5**  | **Hybrid Declarative vs Procedural Parsing**          | Grammar extracts structural/declarative parts; LLM interprets procedural/stepwise parts or vice versa.                      | • Each subsystem tackles what it’s best at<br>• Reduces grammar complexity                      | • Harder to maintain global consistency<br>• Requires AST stitching logic  |
| **6**  | **Grammar-Generated Tests for LLM**                   | Grammar used to generate examples and counterexamples; LLM outputs are validated against grammar constraints.               | • Powerful for verifying LLM outputs<br>• Reduces hallucinations                                | • Grammar must encode constraints richly<br>• Validation pipeline required |
| **7**  | **LLM as Adaptive Heuristic for Grammar Ambiguities** | When grammar yields multiple parses, LLM chooses or ranks the “most plausible” AST.                                         | • Improves disambiguation<br>• Good for underspecified workflows                                | • LLM can pick syntactically impossible interpretations                    |
| **8**  | **LLM as Semantic Phase After Grammar**               | Grammar creates an AST; LLM interprets semantics, fills in missing steps, or resolves vague ops.                            | • Clean separation of syntax vs semantics<br>• Grammar ensures correctness                      | • Semantic interpretation may drift from syntax                            |
| **9**  | **Self-Healing Parse Loop**                           | Grammar fails → LLM proposes corrections → grammar retries → if still failing, LLM creates full AST.                        | • Iterative and robust<br>• Captures user intent progressively                                  | • More expensive; risk of oscillation                                      |
| **10** | **Grammar Embedding Inside Prompt Templates**         | Raku grammar definitions serialized into the prompt, guiding the LLM to conform to the grammar (soft constraints).          | • Faster than grammar execution in some cases<br>• Encourages consistent structure              | • Weak guarantees<br>• LLM may ignore grammar                              |
| **11** | **LLM-Driven Grammar Induction or Refinement**        | LLM suggests new grammar rules or transformations; developer approves; Raku grammar evolves over time.                      | • Faster grammar evolution<br>• Useful for new workflow languages                               | • Requires human QA<br>• Risk of regressing accuracy                       |
| **12** | **Raku Regex Engine as LLM Guardrail**                | Raku regex or token rules used to validate or filter LLM results before accepting them.                                     | • Lightweight constraints<br>• Useful for quick prototyping                                     | • Regex too weak for complex syntax                                        |


### Diversity reasons

- The diversity of combinations in the table above arises because Raku grammars and LLMs occupy fundamentally different but highly complementary positions in the parsing spectrum. 
- Raku grammars provide determinism, speed, verifiability, and structural guarantees, but they require upfront design and struggle with ambiguity, informal input, and evolving specifications. 
- LLMs, in contrast, excel at normalization, semantic interpretation, ambiguity resolution, and adapting to fluid or poorly defined languages, yet they lack determinism, may hallucinate, and are slower. 
- When these two technologies meet, every architectural choice — **who handles syntax, who handles semantics, who runs first, who validates whom, who repairs or refines** — defines a distinct strategy. 
- Hence, the design space naturally expands into many valid hybrid patterns rather than a single "best" pipeline.
- That said, the fallback pattern implemented below can be considered the "best option" from certain development perspectives because it is simple, effective, and has fast execution times.


See the corresponding 
[Morphological Analysis table](./Diagrams/Robust-code-generation-combining-grammars-and-LLMs/Grammar-LLM-combintations-morphological-analysis-table.md) 
which correspond to this taxonomy mind-map:

![](./Diagrams/Robust-code-generation-combining-grammars-and-LLMs/Grammar-LLM-combinations-taxonomy-mind-map.png)

----

## Setup

Here are the packages used in this document (notebook):


```raku
use DSL::Translators;
use DSL::Examples;
use ML::NLPTemplateEngine;

use LLM::Graph;
```

Here are LLM-models access configurations:


```raku
sink my $conf41-mini = llm-configuration('ChatGPT', model => 'gpt-4.1-mini', temperature => 0.45);
sink my $conf41 = llm-configuration('ChatGPT', model => 'gpt-4.1', temperature => 0.45);
sink my $conf51 = llm-configuration('ChatGPT', model => 'gpt-5.1', reasoning-effort => 'none');
sink my $conf-gemini20-flash = llm-configuration('Gemini', model => 'gemini-2.0-flash');
```

----

## Three DSL translations

This section demonstrates the use of three different translation methods:

1. Grammar-based parser-interpreter of computational workflows
2. LLM-based translator using few-shot learning with relevant DSL examples
3. Natural Language Processing (NLP) interpreter using code templates and LLMs to fill-in the corresponding parameters

The translators are ordered according of their faithfulness, most faithful first. 
It can be said that at the same time, the translators are ordered according to their coverage -- widest coverage is by the last.

### Grammar-based

Here a recommender pipeline specified with natural language commands is translated into Raku code of the package ["ML::SparseMatrixRecommender"](https://raku.land/zef:antononcube/ML::SparseMatrixRecommender) using a sub of the package [ "DSL::Translators"](https://github.com/antononcube/Raku-DSL-Translators):


```raku
'
create from @dsData; 
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value
'
==> ToDSLCode(to => 'Raku', format => 'CODE')
==> {.subst('.', "\n."):g}()
```

```
# my $obj = ML::SparseMatrixRecommender
# .new
# .create-from-wide-form(@dsData)
# .apply-term-weight-functions(global-weight-func => "IDF", local-weight-func => "None", normalizer-func => "Cosine")
# .recommend-by-profile(["passengerSex:male", "passengerClass:1st"])
# .join-across(@dsData, on => "id" )
# .echo-value()
```


For more details of the grammar-based approach see the presentations:
- ["Raku for Prediction presentation at The Raku Conference 2021"](https://www.youtube.com/watch?v=flPz4lFyn8M)
- ["Simplified Machine Learning Workflows Overview (Raku-centric)"](https://www.youtube.com/watch?v=p3iwPsc6e74)

### Via LLM examples

LLM translations can be done using a set of from-to rules. 
This is the so-called *few shot learning* of LLMs. The package ["DSL::Examples"](https://raku.land/zef:antononcube/DSL::Examples) has a collection of such examples for different computational workflows. 
(Mostly ML at this point.) The examples are hierarchically organized by programming language and workflow name; 
see the resource file ["dsl-examples.json"](https://github.com/antononcube/Raku-DSL-Examples/blob/main/resources/dsl-examples.json), or execute `dsl-examples`.

Here is a table that shows the known DSL translation examples in ["DSL::Examples"](https://raku.land/zef:antononcube/DSL::Examples):


```raku
#% html
dsl-examples().map({ $_.key X ($_.value.keys Z $_.value.values».elems) }).flat(1).map({ <language workflow examples-count> Z=> $_.flat })».Hash.sort(*<language workflow>).Array
==> to-dataset()
==> to-html(field-names => <language workflow examples-count>)
```

<table border="1"><thead><tr><th>language</th><th>workflow</th><th>examples-count</th></tr></thead><tbody><tr><td>Python</td><td>LSAMon</td><td>15</td></tr><tr><td>Python</td><td>QRMon</td><td>23</td></tr><tr><td>Python</td><td>SMRMon</td><td>20</td></tr><tr><td>R</td><td>LSAMon</td><td>17</td></tr><tr><td>R</td><td>QRMon</td><td>26</td></tr><tr><td>R</td><td>SMRMon</td><td>20</td></tr><tr><td>Raku</td><td>SMRMon</td><td>20</td></tr><tr><td>WL</td><td>ClCon</td><td>20</td></tr><tr><td>WL</td><td>LSAMon</td><td>17</td></tr><tr><td>WL</td><td>QRMon</td><td>27</td></tr><tr><td>WL</td><td>SMRMon</td><td>20</td></tr></tbody></table>

Here is the definition of an LLM translation function that uses examples:


```raku
my &llm-pipeline-segment = llm-example-function(dsl-examples()<Raku><SMRMon>);
```

Here is a recommender pipeline specified with natural language commands:


```raku
my $spec = q:to/END/;
new recommender;
create from @dsData; 
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value;
classify by profile passengerSex:female, and passengerClass:1st on the tag passengerSurvival;
echo value
END

sink my @commands = $spec.lines;
```

Translate to Raku code line-by-line:


```raku
@commands
.map({ .&llm-pipeline-segment })
.map({ .subst(/:i Output \h* ':'?/, :g).trim })
.join("\n.")
```

```
# ML::SparseMatrixRecommender.new;
# .create(@dsData)
# .apply-term-weight-functions('IDF', 'None', 'Cosine')
# .recommend-by-profile({'passengerSex.male' => 1, 'passengerClass.1st' => 1})
# .join-across(@dsData, on => 'id')
# .echo-value()
# .classify-by-profile('passengerSurvival', {'passengerSex.female' => True, 'passengerClass.1st' => True})
# .echo-value()
```

Or translate by just calling the function over the whole `$spec`:


```raku
&llm-pipeline-segment($spec)
```

```
# ML::SparseMatrixRecommender.new;  
# create(@dsData);  
# apply-term-weight-functions('IDF', 'None', 'Cosine');  
# recommend-by-profile({'passengerSex.male' => 1, 'passengerClass.1st' => 1});  
# join-across(@dsData, on => 'id');  
# echo-value();  
# classify-by-profile('passengerSurvival', [{'passengerSex.female' => 1, 'passengerClass.1st' => 1}]);  
# echo-value()
```


**Remark:** That latter call is faster, but it needs additional processing for "monadic" workflows.

### By NLP Template Engine

Here a "free text" recommender pipeline specification is translated to Raku code using the sub `concretize` of the package ["ML::NLPTemplateEngine"](https://raku.land/zef:antononcube/ML::NLPTemplateEngine):


```raku
'create a recommender with dfTitanic; apply the LSI functions IDF, None, Cosine; recommend by profile 1st and male'
==> concretize(lang => "Raku", e => $conf41-mini)
```

```
# my $smrObj = ML::SparseMatrixRecommender.new
# .create-from-wide-form(dfTitanic, item-column-name='id', :add-tag-types-to-column-names, tag-value-separator=':')
# .apply-term-weight-functions('IDF', 'None', 'Cosine')
# .recommend-by-profile(["1st"], 12, :!normalize)
# .join-across(dfTitanic)
# .echo-value();
```


The package ["ML::NLPTemplateEngine"](https://raku.land/zef:antononcube/ML::NLPTemplateEngine) uses a Question Answering System (QAS) implemented in ["ML::FindTextualAnswer"](https://raku.land/zef:antononcube/ML::FindTextualAnswer). 
A QAS can be implemented in different ways, with different conceptual and computation complexity. 
Currently, "ML::FindTextualAnswer" has only an LLM based implementation of QAS.

For more details of the NLP template engine approach see the presentations:

- ["NLP Template Engine, Part 1"](https://www.youtube.com/watch?v=a6PvmZnvF9I)
- ["Natural Language Processing Template Engine"](https://www.youtube.com/watch?v=IrIW9dB5sRM)

-----

## Parallel race: Grammar + LLM


In this section we implement the first, most obvious, and conceptually simplest combination of grammar-based- with LLM-based translations:

- All translators -- grammar-based and LLM-based are run in parallel 
- An LLM judge selects the one that adheres best to the given specification

The implementation of this strategy with an LLM graph (say, by using ["LLM::Graph"](https://raku.land/zef:antononcube/LLM::Graph)) is straightforward.

Here is such a LLM graph that:
- Runs all three translation methods above
- There is a judge that picks which on of the LLM methods produced better result
- Judge's output is used to make a Markdown report


```raku
my %rules =
    dsl-grammar => { 
        eval-function => sub ($spec, $lang = 'Raku') { ToDSLCode($spec, to => $lang, format => 'CODE') }
    },

    llm-examples => { 
        llm-function => 
            sub ($spec, $lang = 'Raku', $split = False) { 
                my &llm-pipeline-segment = llm-example-function(dsl-examples(){$lang}<SMRMon>);
                return do if $split {
                    note 'with spec splitting...';
                    my @commands = $spec.lines;
                    @commands.map({ .&llm-pipeline-segment }).map({ .subst(/:i Output \h* ':'?/, :g).trim }).join("\n.")
                } else {
                    note 'no spec splitting...';
                    &llm-pipeline-segment($spec).subst(";\n", "\n."):g
                }
            },
    },

    nlp-template-engine => {
        llm-function => sub ($spec, $lang = 'Raku') { concretize($spec, :$lang) }
    },

    judge => sub ($spec, $lang, $dsl-grammar, $llm-examples, $nlp-template-engine) {
            [
                "Choose the generated code that most fully adheres to the spec:\n",
                $spec,
                "\nfrom the following $lang generation results:\n\n",
                "1) DSL-grammar:\n$dsl-grammar\n",
                "2) LLM-examples:\n$llm-examples\n",
                "3) NLP-template-engine:\n$nlp-template-engine\n",
                "and copy it:"
            ].join("\n\n")
    },
    
    report => {
            eval-function => sub ($spec, $lang, $dsl-grammar, $llm-examples, $nlp-template-engine, $judge) {
                [
                    '# Best generated code',
                    "Three $lang code generations were submitted for the spec:",
                    '```text',
                    $spec,
                    '```',
                    'Here are the results:',
                    to-html( ['dsl-grammar', 'llm-examples', 'nlp-template-engine'].map({ [ name => $_, code => ::('$' ~ $_)] })».Hash.Array, field-names => <name code> ).subst("\n", '<br/>'):g,
                    '## Judgement',
                    $judge.contains('```') ?? $judge !! "```$lang\n" ~ $judge ~ "\n```"
                ].join("\n\n")
            }
    }        
;

my $gBestCode = LLM::Graph.new(%rules)
```

```
# LLM::Graph(size => 5, nodes => dsl-grammar, judge, llm-examples, nlp-template-engine, report)
```


Here is a recommender workflow specification:


```raku
my $spec = q:to/END/;
make a brand new recommender with the data @dsData;
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value;
END

$gBestCode.eval(:$spec, lang => 'Raku', :split)
```

```
#    with spec splitting...
#   LLM::Graph(size => 5, nodes => dsl-grammar, judge, llm-examples, nlp-template-engine, report)
```

Here the LLM-graph result -- which is a Markdown report -- is rendered:

```raku
#% markdown
$gBestCode.nodes<report><result>
```

### Best generated code

Three Raku code generations were submitted for the spec:

```text

make a brand new recommender with the data @dsData;
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value;


```

Here are the results:

<table border="1"><thead><tr><th>name</th><th>code</th></tr></thead><tbody><tr><td>dsl-grammar</td><td></td></tr><tr><td>llm-examples</td><td>ML::SparseMatrixRecommender.new(@dsData)<br/>.apply-term-weight-functions(&#39;IDF&#39;, &#39;None&#39;, &#39;Cosine&#39;)<br/>.recommend-by-profile({&#39;passengerSex.male&#39; =&gt; 1, &#39;passengerClass.1st&#39; =&gt; 1})<br/>.join-across(@dsData, on =&gt; &#39;id&#39;)<br/>.echo-value()</td></tr><tr><td>nlp-template-engine</td><td>my $smrObj = ML::SparseMatrixRecommender.new<br/>.create-from-wide-form([&quot;passengerSex:male&quot;, &quot;passengerClass:1st&quot;]set, item-column-name=&#39;id&#39;, :add-tag-types-to-column-names, tag-value-separator=&#39;:&#39;)<br/>.apply-term-weight-functions(&#39;IDF&#39;, &#39;None&#39;, &#39;Cosine&#39;)<br/>.recommend-by-profile([&quot;passengerSex:male&quot;, &quot;passengerClass:1st&quot;], 12, :!normalize)<br/>.join-across([&quot;passengerSex:male&quot;, &quot;passengerClass:1st&quot;]set)<br/>.echo-value();</td></tr></tbody></table>

#### Judgement

```Raku
ML::SparseMatrixRecommender.new(@dsData)
.apply-term-weight-functions('IDF', 'None', 'Cosine')
.recommend-by-profile({'passengerSex.male' => 1, 'passengerClass.1st' => 1})
.join-across(@dsData, on => 'id')
.echo-value()
```


### LLM-graph visualization 

Here is a visualization of the LLM graph defined and run above:


```raku
#% html
$gBestCode.dot(engine => 'dot', :9graph-size, node-width => 1.7, node-color => 'grey', edge-color => 'grey', edge-width => 0.4, theme => 'default'):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Robust-code-generation-combining-grammars-and-LLMs/LLM-graph-parallel-run-with-judge-light.png)

For details on LLM-graphs making and their visualization representations see blog posts:

- ["LLM::Graph"](https://rakuforprediction.wordpress.com/2025/08/23/llmgraph/)
- ["LLM::Graph plots interpretation guide"](https://rakuforprediction.wordpress.com/2025/09/12/llmgraph-plots-interpretation-guide/)

---

## Fallback: DSL-grammar to LLM-examples

Instead of having DSL-grammar- and LLM computations running in parallel, 
we can make an LLM-graph in which the LLM computations are invoked if the DSL-grammar parsing-and-interpretation fails. 
In this section we make such a graph.

Before making the graph let us also generalize it to work with other ML workflows, not just recommendations. 
The function `ToDSLCode` (of the package "DSL::Translators") has an ML workflow classifier based on prefix trees; see [AA1]. 

Let us make an LLM function with a similar functionality. 
I.e. an LLM-function that classifies a natural language computation specification into workflow labels used by "DSL::Examples". 
Here is such a function using the sub `llm-classify` provided by ["ML::FindTextualAnswer"](https://raku.land/zef:antononcube/ML::FindTextualAnswer):


```raku
# Natural language labels to be understood by LLMs
my @mlLabels = 'Classification', 'Latent Semantic Analysis', 'Quantile Regression', 'Recommendations';

# Map natural language labels to workflow names in "DSL::Examples"
my %toMonNames = @mlLabels Z=> <ClCon LSAMon QRMon SMRMon>; 

# Change the result of &llm-classify result into workflow names
my &llm-ml-workflow = -> $spec { my $res = llm-classify($spec, @mlLabels, request => 'which of these workflows characterizes it'); %toMonNames{$res} // $res };

# Example invocation
&llm-ml-workflow($spec)
```

```
# SMRMon
```


In addition, we have to specify a pipeline "separator" for the different programming languages: 


```raku
sink my %langSeparator = Python => "\n.", Raku => "\n.", R => "%>%\n", WL => "⟹\n";
```

Here is the LLM-graph:


```raku
my %rules =
    dsl-grammar => { 
        eval-function => sub ($spec, $lang = 'Raku') { 
            my $res = ToDSLCode($spec, to => $lang, format => 'CODE'); 
            my $checkStr = 'my $obj = ML::SparseMatrixRecommender.new';
            return do with $res.match(/ $checkStr /):g { 
                $/.list.elems > 1 ?? $res.subst($checkStr) !! $res 
            }
        }
    },

    workflow-name => {
        llm-function => sub ($spec) { &llm-ml-workflow($spec) }
    },

    llm-examples => { 
        llm-function => 
            sub ($spec, $workflow-name, $lang = 'Raku', $split = False) {
                my &llm-pipeline-segment = llm-example-function(dsl-examples(){$lang}{$workflow-name});
                return do if $split {
                    my @commands = $spec.lines;
                    @commands.map({ .&llm-pipeline-segment }).map({ .subst(/:i Output \h* ':'?/, :g).trim }).join(%langSeparator{$lang})
                } else {
                    &llm-pipeline-segment($spec).subst(";\n", %langSeparator{$lang}):g
                }
            },
        test-function => sub ($dsl-grammar) { !($dsl-grammar ~~ Str:D && $dsl-grammar.trim.chars) }
    },
    
    code => {
            eval-function => sub ($dsl-grammar, $llm-examples) {
                $dsl-grammar ~~ Str:D && $dsl-grammar.trim ?? $dsl-grammar !! $llm-examples
            }
    }   
;

my $gRobust = LLM::Graph.new(%rules):!async
```

```
# LLM::Graph(size => 4, nodes => code, dsl-grammar, llm-examples, workflow-name)
```


Here the LLM graph is run over a spec that can be parsed by DSL-grammar (notice the very short computation time):


```raku
my $spec = q:to/END/;
create from @dsData; 
apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value;
END

$gRobust.eval(:$spec, lang => 'Raku', :!split)
```

```
# LLM::Graph(size => 4, nodes => code, dsl-grammar, llm-examples, workflow-name)
```

Here is the obtained result:


```raku
$gRobust.nodes<code><result>
```

```
# my $obj = ML::SparseMatrixRecommender.new.create-from-wide-form(@dsData).apply-term-weight-functions(global-weight-func => "IDF", local-weight-func => "None", normalizer-func => "Cosine").recommend-by-profile(["passengerSex:male", "passengerClass:1st"]).join-across(@dsData, on => "id" ).echo-value()
```


Here is a spec that cannot be parsed by the DSL-grammar interpreter -- note that there is just a small language change in the first line:


```raku
my $spec = q:to/END/;
new recommender with @dsData, please; 
also apply LSI functions IDF, None, Cosine; 
recommend by profile for passengerSex:male, and passengerClass:1st;
join across with @dsData on "id";
echo the pipeline value;
END

$gRobust.eval(:$spec, lang => 'Raku', :!split)
```

```
#    Cannot parse the command; error in rule recommender-object-phrase:sym<English> at line 1; target 'new recommender with @dsData, please' position 16; parsed 'new recommender', un-parsed 'with @dsData, please' .
#
#    LLM::Graph(size => 4, nodes => code, dsl-grammar, llm-examples, workflow-name)
```


Nevertheless, we obtain a correct result via LLM-examples:


```raku
$gRobust.nodes<code><result>
```

```
# ML::SparseMatrixRecommender.new(@dsData)
# .apply-term-weight-functions('IDF', 'None', 'Cosine')
# .recommend-by-profile({'passengerSex.male' => 1, 'passengerClass.1st' => 1})
# .join-across(@dsData, on => 'id')
# .echo-value();
```


Here is the corresponding graph plot:


```raku
#% html
$gRobust.dot(engine => 'dot', :9graph-size, node-width => 1.7, node-color => 'grey', edge-color => 'grey', edge-width => 0.4, theme => 'default'):svg
```

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Robust-code-generation-combining-grammars-and-LLMs/LLM-graph-fallback-LLM-examples-light.png)

Let us specify another workflow -- for ML-classification with Wolfram Language -- and run the graph: 


```raku
my $spec = q:to/END/;
use the dataset @dsData;
split the data into training and testing parts with 0.8 ratio;
make a nearest neighbors classifier;
show classifier accuracy, precision, and recall;
echo the pipeline value;
END

$gRobust.eval(:$spec, lang => 'WL', :split)
```

```
# LLM::Graph(size => 4, nodes => code, dsl-grammar, llm-examples, workflow-name)
```



```raku
$gRobust.nodes<code><result>
```

```
#  ClConUnit[dsData]⟹
#    ClConSplitData[0.8]⟹
#    ClConMakeClassifier["NearestNeighbors"]⟹
#    Function[{v,c},ClConUnit[v,c]⟹ClConClassifierMeasurements[{"Accuracy","Precision","Recall"}]⟹ClConEchoValue]⟹
#    ClConEchoValue
```


-----

## Concluding comments and observations

- Using LLM graphs gives the ability to impose desired orchestration and collaboration between deterministic programs and LLMs.

    - By contrast, the "inversion of control" of LLM-tools is "capricious."

- LLM-graphs are both a generalization of LLM-tools, and a lower level infrastructural functionality than LLM-tools.

- The LLM-graph for the parallel-race translation is very similar to the LLM-graph for comprehensive document summarization described in [AA4].

- The expectation that DSL examples would provide both fast and faithful results is mostly confirmed in ≈20 experiments.

- Using the NLP template engine is also fast because LLMs are harnessed through QAS.

- The DSL examples translation had to be completed with a workflow classifier.

    - Such classifiers are also part of the implementations of the other two approaches.

    - The grammar-based one uses a deterministic classifier, [AA1].

    - The NLP template engine uses an LLM classifier.

- An interesting extension of the current work is to have a grammar-LLM combination in which when the grammar fails whe LLM "normalizes" the specs until the grammar can parse them.

    - Currently, LLM-graph does not support graphs with cycles, hence this approach "can wait" (or be implemented by other means.)

- Multiple DSL examples can be efficiently derived by random sentence generation with the different grammars.

    - Similar to the DSL commands classifier making approach taken in [AA1].

- LLMs can be also used to improve and extend the DSL grammars. 

    - And it is interesting to consider automating that process, instead of doing it via human supervision.

----

## References

### Articles, blog posts

[AA1] Anton Antonov, ["Fast and compact classifier of DSL commands"](https://rakuforprediction.wordpress.com/2022/07/31/fast-and-compact-classifier-of-dsl-commands/), (2022), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/).

[AA2] Anton Antonov, ["Grammar based random sentences generation, Part 1"](https://rakuforprediction.wordpress.com/2023/01/23/grammar-based-random-sentences-generation-part-1/), (2023), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/).

[AA3] Anton Antonov, ["LLM::Graph"](https://rakuforprediction.wordpress.com/2025/08/23/llmgraph/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/).

[AA4] Anton Antonov, ["Agentic-AI for text summarization"](https://rakuforprediction.wordpress.com/2025/09/02/agentic-ai-for-text-summarization/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/).

[AA5] Anton Antonov, ["LLM::Graph plots interpretation guide"](https://rakuforprediction.wordpress.com/2025/09/12/llmgraph-plots-interpretation-guide/), (2025), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/).

### Packages

[AAp1] Anton Antonov, [DSL::Translators, Raku package](https://github.com/antononcube/Raku-DSL-Translators), (2020-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, [ML::FindTextualAnswer, Raku package](https://github.com/antononcube/Raku-ML-FindTextualAnswer/), (2023-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov, [MLP::NLPTemplateEngine, Raku package](https://github.com/antononcube/Raku-ML-NLPTemplateEngine/), (2023-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov, [DSL::Examples, Raku package](https://github.com/antononcube/Raku-DSL-Examples), (2024-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov, [LLM::Graph, Raku package](https://github.com/antononcube/Raku-LLM-Graph), (2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov, [ML::SparseMatrixRecommender, Raku package](https://github.com/antononcube/Raku-ML-SparseMatrixRecommender/), (2025), [GitHub/antononcube](https://github.com/antononcube).

### Videos

[AAv1] Anton Antonov, ["NLP Template Engine, Part 1"](https://www.youtube.com/watch?v=a6PvmZnvF9I), (2021), [YouTube/@AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov, ["Natural Language Processing Template Engine"](https://www.youtube.com/watch?v=IrIW9dB5sRM), (2023), [YouTube/@WolframResearch](https://www.youtube.com/@WolframResearch).

[WRIv1] Wolfram Research, Inc., [“Live CEOing Ep 886: Design Review of LLMGraph”](https://www.youtube.com/watch?v=ewU83vHwN8Y), (2025), [YouTube/@WolframResearch](https://www.youtube.com/@WolframResearch).


