# Grammar based random sentences generation -- Part 1

## Introduction

In this article we discuss the generation of random sentences using [Raku grammars](https://docs.raku.org/language/grammars).

The generations discussed are done with the package functions and scripts of 
["Grammar::TokenProcessing"](https://raku.land/zef:antononcube/Grammar::TokenProcessing), [AAp1].

The random sentence generator in "Grammar::TokenProcessing" is limited, it does not parse all possible Raku grammars. 
But since I need it for my 
[Domain Specific Language (DSL) parser-interpreters work](https://raku.land/?q=DSL%3A%3AEnglish),
it covers a fairly large (if common) grammar ground.

### Preliminary examples

Let us generate a few random sentences using the front page example at [raku.org](https://raku.org):

```perl6
grammar Parser {
    rule  TOP  { I <love> <lang> }
    token love { '♥' | ':O=' | love }
    token lang { < Raku Perl Rust Go Python Ruby > }
}

use Grammar::TokenProcessing;

.say for random-sentence-generation(Parser) xx 6;
```
```
# I ♥ Go
# I love Perl
# I ♥ Go
# I love Raku
# I love Go
# I :O= Perl
```

Here is an example of random sentence generation using the DSL package
[DSL::English::LatentSemanticAnalysis](https://raku.land/zef:antononcube/DSL::English::LatentSemanticAnalysisWorkflows):

```perl6
use DSL::English::LatentSemanticAnalysisWorkflows::Grammar;

.say for random-sentence-generation(DSL::English::LatentSemanticAnalysisWorkflows::Grammar) xx 6;
```
```
# include the setup code
# echo the current pipeline value
# render using topics the QUERY_TEXT("papa papaverine symbol uptick")
# compute the doc word matrix using stop words and using stop words & stop words & using no stop words & NA stemming
# compute and show some statistics with the document word matrix
# render by topics the VAR_NAME("OqgVO")
```

------

## Motivation

The ability to design and implement grammars using the Object-Oriented Programming (OOP) paradigm
is one of the most distinguishing features of Raku. (I think it is "the most.")

Real-life grammars -- especially developed with OOP -- can have multiple component definitions (say, roles)
in multiple files. Figuring out what sentences a grammar can handle is most likely not an easy task. 

Here are some additional reasons to have grammar-based random sentence generation:

- Quick review of what kind of sentences a grammar can parse
- Grammar adequacy evaluation
- Handling un-parsable statements by showing examples of similar (random) statements
- Development of classifiers of grammars or grammar rules
- Generation of random expressions with a particular structure

**Remark:** The article [AA1] and the presentation [AAv1] discuss making grammar classifiers
using random sentences generated with the grammars the classifiers are for.

------

## Not using RakuAST

I evaluate maturity of parser-making systems by their ability to generate random sentences from grammars.
Surprisingly, with almost all systems that is not easy. (That list includes Raku.)

I was told that making a random sentence generator with
[RakuAST](https://news.perlfoundation.org/post/2022-02-raku-ast-grant)
would be easy. But until "RakuAST" is released, I am interested in having "current Raku" solution(s).

------

## Simple grammars

### Love-hate of languages

Consider a version of the "love-hate of languages" grammar in which:

- The `<love>` token has a quantifier

- More standard alternation specification is used

```perl6
grammar Parser2 {
    rule  TOP  { I <love> ** 1..3 <lang> }
    token love { '♥' | '︎:O=' | love }
    token lang { Raku | Perl | Rust | Go | Python | Ruby }
}

.say for random-sentence-generation(Parser2) xx 6;
```
```
#ERROR: Unrecognized regex metacharacter '︎ (must be quoted to match literally)
#ERROR: Unrecognized regex metacharacter : (must be quoted to match literally)
#ERROR: Malformed regex
# Nil
```

Now let us move the quantifier in the token `love`:

```perl6
grammar Parser3 {
    rule  TOP  { I <love>  <lang> }
    token love { [ '♥' | ':O=' | love ] ** 1..3 }
    token lang { Raku | Perl | Rust | Go | Python | Ruby }
}

.say for random-sentence-generation(Parser3) xx 6;
```
```
# I love love Go
# I ♥ ♥ ♥ Perl
# I :O= :O= Perl
# I love love Perl
# I :O= :O= :O= Go
# I :O= Raku
```

### ISBN parser

Here is an [International Standard Book Number (ISBN)](https://en.wikipedia.org/wiki/ISBN) grammar:

```perl6
grammar ISBN {
    token TOP {  <tenner>  |  <niner-xray>  }
    token tenner { [ <digit> <:Pd>? ] ** 10   }
    token niner-xray { [ <digit> <:Pd>? ] ** 9 X }
}
```
```
# (ISBN)
```

In order to generate "good looking" random sentences with that grammar we have to:

- Define a random digit generator that is simpler than the default one
- Use an empty string as a separator

```perl6
use Data::Generators;
use Data::Reshapers;

my %randomTokenGenerators = default-random-token-generators();
%randomTokenGenerators{'<:Pd>'} = { '-' };
%randomTokenGenerators{'<digit>'} = -> { random-string(chars => 1, ranges => "0" .. "9") };

.say for random-sentence-generation(ISBN, random-token-generators => %randomTokenGenerators, sep => '') xx 6;
```
```
# 734950311X
# 4-9-5-7-6-7-0-2-1-X
# 29-5-07-12-7-27-
# 5-6-2-0-2-2-7-8-5-X
# 64-7-42-9-92-86
# 1653-8-0-3-3-90
```

Here is how the random ISBNs look with the default settings:

```perl6
.say for random-sentence-generation(ISBN) xx 6;
```
```
# DIGIT(5) - DIGIT(9) - DIGIT(6) - DIGIT(2) - DIGIT(9) - DIGIT(0) - DIGIT(4) - DIGIT(5) - DIGIT(0) - X
# DIGIT(4) - DIGIT(1) - DIGIT(6) - DIGIT(7) - DIGIT(7) - DIGIT(3) - DIGIT(4) - DIGIT(4) - DIGIT(7) - X
# DIGIT(3) DIGIT(6) DIGIT(9) DIGIT(8) DIGIT(0) DIGIT(8) DIGIT(2) DIGIT(3) DIGIT(3) X
# DIGIT(0) DIGIT(3) DIGIT(8) - DIGIT(2) DIGIT(7) - DIGIT(2) - DIGIT(3) - DIGIT(9) - DIGIT(1) - DIGIT(9)
# DIGIT(8) DIGIT(3) - DIGIT(0) - DIGIT(0) DIGIT(8) DIGIT(0) DIGIT(6) DIGIT(6) - DIGIT(4) - DIGIT(8) -
# DIGIT(0) DIGIT(8) DIGIT(6) DIGIT(3) DIGIT(5) DIGIT(0) DIGIT(5) DIGIT(9) DIGIT(3) X
```

------

## Default random token generators

Here are the *keys* of the default random token generators used by `random-sentence-generation`:

```perl6
.say for default-random-token-generators().keys;
```
```
# <ws>
# <:Pd>
# <.ws>?
# <list-separator>
# <dataset-name>
# <query-text>
# <mixed-quoted-variable-name>
# <number>
# <function-name>
# <list-separator>?
# <alnum>
# <digit>
# <integer-value>
# <integer>
# <variable-name>
# <number-value>
# <quoted-variable-name>
# <regex-pattern>
# <code-expr>
# <shell-expr>
# <raku-module-name>
# <.alnum>
# <.ws>
# <.digit>
# <wl-expr>
```

As it was demonstrated in the previous section, generation rules can be changed and new ones added.

------

## "Advanced" DSL grammars

My primary "target" grammars are DSL grammars for computational workflows. Here are the main ones:

- `DSL::English::ClassificationWorkflows::Grammar`
- `DSL::English::DataQueryWorkflows::Grammar`
- `DSL::English::LatentSemanticAnalysisWorkflows::Grammar`
- `DSL::English::QuantileRegressionWorkflows::Grammar`
- `DSL::English::RecommenderWorkflows::Grammar`

The main, first level rules of those grammars have names that finish with "-command".
Here are the commands in the classification workflows grammar:

```perl6
use DSL::English::ClassificationWorkflows;
my $focusGrammar = DSL::English::ClassificationWorkflows::Grammar;
my %focusRules = $focusGrammar.^method_table;
to-pretty-table(%focusRules.keys.grep({ $_.ends-with('command') }).sort.rotor(3):partial, align=>'l');
```
```
# +--------------------------------------+----------------------------------+--------------------------+
# | 0                                    | 1                                | 2                        |
# +--------------------------------------+----------------------------------+--------------------------+
# | classifier-ensemble-creation-command | classifier-measurements-command  | classifier-query-command |
# | classifier-testing-command           | data-load-command                | data-outliers-command    |
# | data-summary-command                 | dimension-reduction-command      | dsl-module-command       |
# | dsl-spec-command                     | dsl-translation-target-command   | echo-command             |
# | ensemble-by-single-method-command    | find-outliers-command            | make-classifier-command  |
# | make-classifier-simple-command       | make-classifier-thorough-command | pipeline-command         |
# | remove-outliers-command              | roc-diagrams-command             | roc-plots-command        |
# | setup-code-command                   | show-outliers-command            | split-data-command       |
# | user-id-spec-command                 | user-spec-command                | workflow-command         |
# +--------------------------------------+----------------------------------+--------------------------+
```

Here we generate sentences with `<split-data-command>`:

```perl6
.say for random-sentence-generation($focusGrammar, '<split-data-command>') xx 6;
```
```
# partition dataset by split NUMBER(284.48)
# divide data frame with validation NUMBER(213.58) and using data ratio NUMBER(276.87) using validation NUMBER(30.46)
# split dataset with the split NUMBER(279.5)
# partition the time series data by validation fraction NUMBER(45.42) using validation data ratio NUMBER(193.92) and validation ratio NUMBER(23.7) , and training data split NUMBER(212.71) using method label proportional method
# partition the data frame
# divide the dataset by split ratio NUMBER(87.77)
```

Here we generate sentences with `<recommend-data-command>` of the recommender workflows grammar:

```perl6
use DSL::English::RecommenderWorkflows;
my $focusGrammar = DSL::English::RecommenderWorkflows::Grammar;
.say for random-sentence-generation($focusGrammar, '<recommend-by-profile-command>', max-iterations => 100) xx 6;
```
```
# compute most relevant INTEGER(25) top profile recommendation
# most relevant most relevant recommendation using the profile VAR_NAME("1PdoY") → NUMBER(174.37) and VAR_NAME("Hh8EB") : VAR_NAME("7nMIV") -> NUMBER(268.92) , VAR_NAME("I9JsK") -> NUMBER(8.38)
# recommend for profile with ‘ VAR_NAME("Nw3ng") : VAR_NAME("QFk9T") ’ = NUMBER(238.79) and ‘ VAR_NAME("LuY2g") : VAR_NAME("5nRjo") ’ : NUMBER(239.2) and VAR_NAME("kmBRa") : VAR_NAME("6BxzP") = NUMBER(216.24) , “ VAR_NAME("CGgB1") : VAR_NAME("f2QhV") ” -> NUMBER(72.25)
# what are recommendations by profile VAR_NAME("Hjk0w") : VAR_NAME("SEF5Y") , VAR_NAME("VYTEZ") , “ VAR_NAME("dMVgn") : VAR_NAME("sIXP0") “
# recommend using the profile for VAR_NAME("qYAYJ") : VAR_NAME("SFosy") → NUMBER(90.81)
# recommend using the profile VAR_NAME("U3V8s") : VAR_NAME("KK7OL") = NUMBER(206.76) and VAR_NAME("3o8fD") → NUMBER(100.22) and VAR_NAME("P85U4") : NUMBER(131.88) and VAR_NAME("ZLvxF") : VAR_NAME("7FgKZ") → NUMBER(144.24) and VAR_NAME("Wrmsz") : VAR_NAME("7NeBP") → NUMBER(178.39) , VAR_NAME("G2vAM") → NUMBER(122.21)
```

**Remark:** The grammars, generally, parse a larger set of sentences than the grammatically correct ones.
Hence, in some (or many) cases the generated sentences might look "strange" or "non-linear."

------

## Other natural languages

Here is how we generate commands with DSLs based on other languages (Bulgarian):

```perl6
use DSL::Bulgarian::QuantileRegressionWorkflows::Grammar;
.say for random-sentence-generation(DSL::Bulgarian::QuantileRegressionWorkflows::Grammar, syms => <Bulgarian English>) xx 6;
```
```
# покажи  текущ конвейерен контекст функция
# ползвай квантила регресия обект VAR_NAME("Ntqo9")
# създай DATASET_NAME("uRBnF")
# покажи чертежи за  грешка
# движещ съпоставка WL_EXPR("Sqrt[3]") чрез NUMBER(291.64)
# множество горе код
```

**Remark:** The package "DSL::Bulgarian", [AAp2], reuses the English-based grammars for computational workflows. 
It just provides Bulgarian tokens that replace English tokens -- the assumption is that short Bulgarian and English commands
have the same structure, [AA2]. Hence, a fair amount of the Bulgarian random sentences are "wrong." As mentioned in 
the previous section, correct Bulgarian sentences are, of course, also parsed.

------

## CLI

The package "Grammar::TokenProcessing" provides a Command Line Interface (CLI) script. Here is an example:

```shell
random-sentence-generation DSL::English::QuantileRegressionWorkflows::Grammar -n=10
```
```
# do a QuantileRegression with the knots from NUMBER(245.95) to NUMBER(276.63) with step NUMBER(60.66) , using INTEGER(57) interpolation degree , using INTEGER(156) interpolation degree , INTEGER(159) the knots
# rescale the axes
# display diagram with dates
# WL_EXPR("Sqrt[3]")
# display plot the error
# use object VAR_NAME("H7MoH")
# echo data summary
# compute anomalies with the outlier identifier VAR_NAME("SKEak")
# summarize the data
# display graph of the error
```

------

## "Leftover" comments 

- I decided I cannot wait for RakuAST in order to advance the development of my DSL projects, so, I actively program "workarounds".
  One such workaround is the implementation of random sentences generator.

- Moritz Lenz remarks in "Parsing with Perl 6 Regexes and Grammars", [ML1], that grammars are Swiss-army chain saw for parsing.
  Hence, being able to randomly -- and quickly -- unfold the components of such chain saw would give a nice overview of 
  its Swiss-army-ness.

- Jonathan Worthington discussed random sentences generation with Raku (Perl6) grammars 7-9 years. 
  Unfortunately, I cannot find that presentation. (When I do I will post a link here.)

------

## References

### Articles, books

[AA1] Anton Antonov,
["Fast and compact classifier of DSL commands"](https://rakuforprediction.wordpress.com/2022/07/31/fast-and-compact-classifier-of-dsl-commands/),
(2021),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["DSL::Bulgarian"](https://rakuforprediction.wordpress.com/2022/12/31/dslbulgarian/),
(2021),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[ML1] Moritz Lenz,
"Parsing with Perl 6 Regexes and Grammars: A Recursive Descent into Parsing",
2017,
Apress; 1st ed. edition.
ISBN-10 : 1484232275.
ISBN-13 : 978-1484232279.


### Packages

[AAp1] Anton Antonov,
[Grammar::TokenProcessing Raku package](https://raku.land/zef:antononcube/Grammar::TokenProcessing),
(2021-2022),
[raku.land/antononcube](https://raku.land/zef:antononcube).

[AAp2] Anton Antonov,
[DSL::Bulgarian Raku package](https://raku.land/zef:antononcube/DSL::Bulgarian),
(2021-2022),
[raku.land/antononcube](https://raku.land/zef:antononcube).


### Presentations

[AA1] Anton Antonov,
["Natural Language Processing Template Engine"](https://www.youtube.com/watch?v=IrIW9dB5sRM),
(2022),
[Wolfram Technology Conference 2022]().



