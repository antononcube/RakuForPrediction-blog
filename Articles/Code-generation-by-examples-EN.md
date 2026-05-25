# Code Generation by DSL Examples

The Raku package ["DSL::Examples"](https://raku.land/zef:antononcube/DSL::Examples), [AAp1], 
is "data package" with examples of DSL commands translations to programming code.

The DSL examples are suitable for 
[LLM few-shot training](https://www.prompthub.us/blog/the-few-shot-prompting-guide). 
The sub `llm-example-function` provided by 
["LLM::Functions"](https://github.com/antononcube/Raku-LLM-Functions), [AAp3],
can be effectively used to create translation functions utilizing those examples.

The utilization of such LLM-translation functions is exemplified below.
Also in the presentation ["Robust LLM pipelines (Mathematica, Python, Raku)"](https://youtu.be/QOsVTCQZq_s):
- [Short introduction](https://youtu.be/QOsVTCQZq_s?t=89)
- [Detailed explanations](https://www.youtube.com/watch?v=QOsVTCQZq_s&t=2840s)

Similar translations -- with much less computational resources -- are achieved with 
grammar-based DSL translators; see 
["DSL::Translators"](https://github.com/antononcube/Raku-DSL-Translators), [AAp2]. The package 
["LLM::Resources"](https://github.com/antononcube/Raku-LLM-Resources), [AAp4], has LLM-graphs 
for code generation that utilize the DSL examples of this package.

-----

## Usage examples

Get all examples:

```raku
use DSL::Examples;
use Data::TypeSystem;

dsl-examples()
    ==> deduce-type()
```
```
# Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 17), Assoc(Atom((Str)), Atom((Str)), 14), Assoc(Atom((Str)), Atom((Str)), 32), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 27), Assoc(Atom((Str)), Atom((Str)), 6)]), 7), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 10), Assoc(Atom((Str)), Atom((Str)), 26), Assoc(Atom((Str)), Atom((Str)), 17), Assoc(Atom((Str)), Atom((Str)), 20)]), 4), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 15), Assoc(Atom((Str)), Atom((Str)), 23), Assoc(Atom((Str)), Atom((Str)), 33), Assoc(Atom((Str)), Atom((Str)), 20)]), 4), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 15), Assoc(Atom((Str)), Atom((Str)), 10), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 6)]), 4)]), 4)
```

Tabulate all translation languages and available workflow examples:

```raku, results=asis
use Data::Translators;
dsl-examples(from => 'English').map({ $_.key X $_.value.keys }).flat(1).map({ <language workflow> Z=> $_ })».Hash.sort.Array
==> to-dataset()
==> to-html(field-names => <language workflow>)
```
<table border="1"><thead><tr><th>language</th><th>workflow</th></tr></thead><tbody><tr><td>Python</td><td>LSAMon</td></tr><tr><td>Python</td><td>QRMon</td></tr><tr><td>Python</td><td>SMRMon</td></tr><tr><td>Python</td><td>pandas</td></tr><tr><td>R</td><td>DataReshaping</td></tr><tr><td>R</td><td>LSAMon</td></tr><tr><td>R</td><td>QRMon</td></tr><tr><td>R</td><td>SMRMon</td></tr><tr><td>Raku</td><td>DataReshaping</td></tr><tr><td>Raku</td><td>LSAMon</td></tr><tr><td>Raku</td><td>SMRMon</td></tr><tr><td>Raku</td><td>TriesWithFrequencies</td></tr><tr><td>WL</td><td>ClCon</td></tr><tr><td>WL</td><td>DataReshaping</td></tr><tr><td>WL</td><td>LSAMon</td></tr><tr><td>WL</td><td>QRMon</td></tr><tr><td>WL</td><td>SMRMon</td></tr><tr><td>WL</td><td>Tabular</td></tr><tr><td>WL</td><td>TriesWithFrequencies</td></tr></tbody></table>


Note in `dsl-examples` the language to translate from is specified.
Currently, the package has DSL examples for Bulgarian, English, and Russian (being from-languages.)  

Get the examples for Latent Semantic Analysis (**LSA**) **Mon**adic pipeline segments in Python:

```raku
dsl-examples('Python', 'LSAMon')
    ==> deduce-type(:tally)
```
```
# Assoc(Atom((Str)), Atom((Str)), 15)
```

Make an LLM example function for translation of LSA workflow building commands:

```raku
use LLM::Functions;
my &llm-pipeline-segment = llm-example-function(dsl-examples()<WL><LSAMon>);
```

Run the LLM function over a list of DSL commands: 

```raku
my @commands = 
"use the dataset aAbstracts",
"make the document-term matrix without stemming",
"exract 40 topics using the method non-negative matrix factorization",
"show the topics";

@commands
.map({ .&llm-pipeline-segment })
.map({ .subst(/:i Output ':'?/):g })
.join("⟹\n")
```
```
# LSAMonUnit[aAbstracts]⟹
# LSAMonMakeDocumentTermMatrix["StemmingRules"->{},"StopWords"->Automatic]⟹
# LSAMonExtractTopics["NumberOfTopics" -> 40, Method -> "NNMF"]⟹
# LSAMonEchoTopicsTable[]
```

Same workflow specified in Bulgarian:

```raku
my &llm-pipeline-segment-bg = llm-example-function(dsl-examples(from => 'Bulgarian')<WL><LSAMon>);

my @commands = 
"използавай данните aAbstracts",
"направи документ-терм матрицата без да използаваш стъблата на думите",
"намери 40 теми ползвайки методата не-отрицателна матрична факторизация",
"покажи темите";

@commands
.map({ .&llm-pipeline-segment-bg })
.map({ .subst(/:i Output ':'?/):g })
.join("⟹\n")
```
```
# LSAMonUnit[aAbstracts]⟹
# LSAMonMakeDocumentTermMatrix["StemmingRules"->{}]⟹
# LSAMonExtractTopics["NumberOfTopics"->40,Method->"NNMF"]⟹
# LSAMonEchoTopicsTable[]
```

-----

## Implementation details

There are several ways to organize the DSL examples with respect to the from-languages:

| Type                                                                                   | Comment                                                | Currently used                      | 
|----------------------------------------------------------------------------------------|--------------------------------------------------------|-------------------------------------|
| Have a separate file for each from-langauge                                            | Convenient editing and refinement                      | Yes                                 |
| One file of all examples; from-langauge is a key for each workflow                     | Can be produces with the separate files                | No                                  |
| Keep English-only DSL examples and use dictionaries of command translations to English | Does not train the LLM directly with the from-language | Dictionaries are kept for reference |


-----

## References

### Packages

[AAp1] Anton Antonov,
[DSL::Examples, Raku package](https://github.com/antononcube/Raku-DSL-Examples),
(2024-2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[DSL::Translators, Raku package](https://github.com/antononcube/Raku-DSL-Translators),
(2020-2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[LLM::Functions, Raku package](https://github.com/antononcube/Raku-LLM-Functions), 
(2023-2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[LLM::Resources, Raku package](https://github.com/antononcube/Raku-LLM-Resources),
(2026),
[GitHub/antononcube](https://github.com/antononcube).

### Videos

[AAv1] Anton Antonov,
["Robust LLM pipelines (Mathematica, Python, Raku)"](https://youtu.be/QOsVTCQZq_s),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).