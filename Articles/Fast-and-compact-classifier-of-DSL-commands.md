# Fast and compact classifier of DSL commands
***AKA "Fairly fast and accurate trie-based classifier of DSL commands"***  

Anton Antonov   
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
July 2022


## Introduction

In this (computational Markdown) document we show how to derive with Raku a fast, accurate, and compact 
Machine Learning (ML) classifier that classifies natural language commands made within
the Domain Specific Languages (DSLs) of a certain set of computational ML workflows.

For example, such classifier should classify the command *"calculate document term matrix"*
as a Latent Semantic Analysis (LSA) workflow command. (And, say, give it the label "LatentSemanticAnalysis".) 

The primary motivation for making a DSL-classifier is to speed up the parsing of specifications that
belong to a (somewhat) large collection of computational workflow DSLs. 
(For example, the Raku package [AAp5] has twelve workflows.)

**Remark:** Such classifier is used in the Mathematica package provided by the 
["NLP Template Engine" project](https://github.com/antononcube/NLP-Template-Engine), [AAr2, AAv1]. 

**Remark:** This article can be seen as an extension of the article
["Trie-based classifiers evaluation"](https://rakuforprediction.wordpress.com/2022/07/07/trie-based-classifiers-evaluation/),
[AA2].

### General classifier making workflow

Here is a mind-map that summarizes the methodology of ML classifier making, [AA1]:

![](https://raw.githubusercontent.com/antononcube/SimplifiedMachineLearningWorkflows-book/master/Diagrams/Making-competitions-classifiers-mind-map.png)

### Big picture flow chart

Here is a "big picture" flow-chart that *encompasses* the procedures outlined and implemented in this documents:

![](https://github.com/antononcube/NLP-Template-Engine/raw/main/Documents/Diagrams/General/Computation-workflow-type-classifier-making.png)

Here is a narration of the flow chart:

1. Get a set of computational workflows as an input
 
2. If the textual data is sufficiently large

   1. Make a classifier
   
   2. Evaluate classifier's measurements 
   
   3. If the classifier is good enough export it
      
      - Finish
   
   4. Else
      
      - Go to Step 2

3. Else

    1. If specifications can be automatically generated:
       - Generate specifications and store them in a database
    2. Else
       - Manually write specifications and store them in a database
    3. Go to Step 2  
    

### DSL specifications

Here are examples of computational DSL specifications for the workflows 
*Classification*, *Latent Semantic Analysis*, and *Quantile Regression*:

```shell
dsl-translation WL "
DSL MODULE Classification;
use the dataset dfGoods;
split data with ratio 0.8;
make a logistic regression classifier;
show accuracy, precision;
"
```
```
# ClConUnit[ dfGoods ] \[DoubleLongRightArrow]
# ClConSplitData[ "TrainingFraction" -> 0.8 ] \[DoubleLongRightArrow]
# ClConMakeClassifier[ "LogisticRegression" ] \[DoubleLongRightArrow]
# ClConClassifierMeasurements[ {"Accuracy", "Precision"} ] \[DoubleLongRightArrow] ClConEchoValue[]
```

```shell
dsl-translation R "
DSL MODULE LatentSemanticAnalysis;
use aDocs;
create document-term matrix;
apply LSI functions IDF, Frequency, and Cosine;
extract 36 topics with the method NNMF and max steps 12;
show topics table 
"
```
```
# LSAMonUnit(aDocs) %>%
# LSAMonMakeDocumentTermMatrix() %>%
# LSAMonApplyTermWeightFunctions(globalWeightFunction = "IDF", localWeightFunction = "None", normalizerFunction = "Cosine") %>%
# LSAMonExtractTopics( numberOfTopics = 36, method = "NNMF",  maxSteps = 12) %>%
# LSAMonEchoTopicsTable()
```

```shell
dsl-translation R "
DSL MODULE QuantileRegression;
use dfStocksVolume;
summarize data;
computed quantile regression with 30 knots and order 2;
show date list plot 
"
```
```
# QRMonUnit( data = dfStocksVolume) %>%
# QRMonEchoDataSummary() %>%
# QRMonQuantileRegression(df = 30, probabilities = c(2)) %>%
# QRMonPlot( datePlotQ = TRUE)
```

### Problem formulation

**Definition:** We refer to the number of characters a parser could not parse as *parsing residual*.

**Definition:** If the parsing residual is 0 then we say that the parser "exhausted the specification" or "parsed the specification completely."

**Assumptions:** It is assumed that:

- We have two or more DSL parsers.
- For each parser we can obtain a *parsing residual*.

**Problem:** For a given DSL specification order the available DSL parsers according to how likely
each of them is to parse the given DSL specification completely.

------

## Procedures outlines

In this section we outline: 

- The brute force DSL parsing procedure

- The modification of the brute force procedure by using a DSL-classifier

- The derivation of a DSL-classifier

- Possible applications of Association Rule Learning algorithms

### Inputs

- A computational DSL specification
- A list of available DSL parsers

### Brute force DSL parsing

1. Randomly shuffle the available DSL parsers.
2. Attempt parsing with each of the available DSL parsers.
3. If any parser gives a zero residual then stop the loop and use that parser as "the work parser." 
4. The parser that gives the smallest residual is chosen as "the work parser."

### Parsing with the help of a DSL-classifier

1. Apply the DSL classifier to the given spec and order the DSL parsers according to the obtained classification probabilities.
2. Do the "Brute force DSL parsing" steps 2, 3, and 4.

### Derivation of a DSL-classifier

1. For each of the DSLs generate at least a few hundred random commands using their grammars.
   - Label each command with the DSL it was generated with. 
   - Export to a JSON file and / or CSV file.
2. Ingest the DSL commands data into a hash (dictionary or association.)
3. Do basic data analysis.
   - Summarize the textual data
   - Split the commands into words
   - Remove stop words, random words, words with (too many) special symbols
   - Find, summarize, and display word frequencies
4. Split the data into training and testing parts.
   - Do stratified splitting, per label.
5. Turn each command into a trie phrase:
   - Split the command into words
   - Keep frequent enough words (as found in step 3)
   - Sort the words and append the DSL label 
6. Make a trie with trie commands of the training data part.
7. Evaluate the trie classifier over the trie commands of the testing data part.
8. Show classification success rates and confusion matrix. 

**Remark:** The trie classifiers are made with the Raku package
["ML::TriesWithFrequencies"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies), [AAp9].

### Association rules study

1. Create Association Rule Learning (ARL) baskets of words.
   - Put all words to lower case
   - Filter words using different criteria:
     - Remove stop words
     - Keep dictionary words
     - Remove words that have special symbols or are random strings
2. Find frequent sets that include the DSL labels.
3. Examine frequent sets.
4. Using the frequent sets create and evaluate trie classifier.

**Remark:** ARL algorithm Apriori can be implemented via Tries. See for example the Apriori
implementation in the Raku package
["ML::AssociationRuleLearning"](https://raku.land/zef:antononcube/ML::AssociationRuleLearning), [AAp7].

------

## Load packages

Here we load the Raku packages used below:

```perl6
use ML::AssociationRuleLearning;
use ML::TriesWithFrequencies;
use Lingua::StopwordsISO;

use Data::Reshapers;
use Data::Summarizers;
use Data::Generators;
use Data::ExampleDatasets;
```
```
# (Any)
```

**Remark:** All packages are available at [raku.land](https://raku.land).

------

## Obtain textual data

In this section we show how we obtain the textual data and do rudimentary pre-processing.

Read the text data -- the labeled DSL commands -- from a CSV file (using `example-dataset` from 
["Data::ExampleDatasets"](https://github.com/antononcube/Raku-Data-ExampleDatasets), [AAp2]):

```perl6
my @tbl = example-dataset('https://raw.githubusercontent.com/antononcube/NLP-Template-Engine/main/Data/RandomWorkflowCommands.csv');
@tbl.elems
```
```
# 5220
```

Show summary of the data (using `records-summary` from 
["Data::Summarizers"](https://github.com/antononcube/Raku-Data-Summarizers), [AAp4]):

```perl6
records-summary(@tbl)
```
```
# +---------------------------------------+-------------------------------+
# | Command                               | Workflow                      |
# +---------------------------------------+-------------------------------+
# | summarize data                => 27   | Classification         => 870 |
# | summarize the data            => 25   | QuantileRegression     => 870 |
# | train                         => 16   | Recommendations        => 870 |
# | graph                         => 14   | LatentSemanticAnalysis => 870 |
# | net regression                => 13   | RandomTabularDataset   => 870 |
# | extract statistical thesaurus => 13   | NeuralNetworkCreation  => 870 |
# | drill                         => 13   |                               |
# | (Other)                       => 5099 |                               |
# +---------------------------------------+-------------------------------+
```

Make a list of pairs:

```perl6
my @wCommands = @tbl.map({ $_<Command> => $_<Workflow>}).List;
say @wCommands.elems
```
```
# 5220
```

Show a sample of the pairs:

```perl6
srand(33);
.say for @wCommands.pick(12).sort
```
```
# calculate the consumption profile of the 7dxahjsy => Recommendations
# give the matrix dimensions => Recommendations
# how many nets in repository => NeuralNetworkCreation
# list the names of models => NeuralNetworkCreation
# list the names of models assign boolean decoder using n2pr n2pr chain with dot layer how many networks how many nets in repository set decoder tokens by ndwax6 ndwax6 => NeuralNetworkCreation
# load the text collection 9ze2l17vmn => LatentSemanticAnalysis
# make a random-driven data set for max number of values 574 => RandomTabularDataset
# modify boolean columns to boolean => Classification
# set decoder scalar with 1tnclgp 1tnclgp => NeuralNetworkCreation
# split the into 100.413 percent train data summarize data into 387.633 percent train create an classifier using n4x for 118.329 percent of available records => Classification
# verify that CorrectlyClassifiedExamples is equal to 825.482 percent => Classification
# what is the number of nets in repository => NeuralNetworkCreation
```

**Remark:** The labeled DSL commands ingested above were generated using the grammars of the project
[ConversationalAgents at GitHub](https://github.com/antononcube/ConversationalAgents), [AAr1],
and the function `GrammarRandomSentences` of the Mathematica package 
["FunctionalParsers.m"](https://github.com/antononcube/MathematicaForPrediction/blob/master/FunctionalParsers.m), [AAp12]. 

**Remark:** Currently it is very hard to generate random sentences using grammars in Raku.
That is also true for other grammar systems. (If that kind of functionality exists, it is usually added
much latter in development phase.) I am hopeful that the
[Raku AST project](https://news.perlfoundation.org/post/2022-02-raku-ast-grant)
is going to greatly facilitate grammar-based random sentence generation.

------

## Word tallies

In this section we analyze the words presence in the DSL commands.

Here we get more than 80,000 English dictionary words (using the function `random-word` from 
["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators), [AAp1]):

```perl6
my %dictionaryWords = Set(random-word(Inf)>>.lc);
%dictionaryWords.elems
```
```
# 83599
```

**Remark::** The set `%dictionaryWords` is most likely a subset of the generally "known English words." 
(And in this document we are fine with that.)  

Here we:

1. Split into words the key (i.e. command) of each of the data pairs
2. Flatten into one list of words 
3. Trim each word and turn into lower case
4. Find word tallies (using the function `tally` from ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers), [AAp4].)

```perl6
my %wordTallies = @wCommands>>.key.map({ $_.split(/ \s | ',' /) }).&flatten>>.trim>>.lc.&tally;
%wordTallies.elems
```
```
# 4090
```

Show summary of the word tallies:

```perl6
records-summary(%wordTallies.values.List)
```
```
# +---------------------+
# | numerical           |
# +---------------------+
# | Max    => 2828      |
# | Mean   => 11.189731 |
# | Median => 1         |
# | 3rd-Qu => 2         |
# | Min    => 1         |
# | 1st-Qu => 1         |
# +---------------------+
```

Here we filter the word tallies to be only with words that are:
- Have frequency ten or higher
- Have at least two characters
- Dictionary words 
- Not English stop words (using the function `stopwords-iso` from ["Lingua::StopwordsISO"](https://raku.land/cpan:ANTONOV/Lingua::StopwordsISO), [AAp6])

```perl6
my %wordTallies2 = %wordTallies.grep({ $_.value ≥ 10 && $_.key.chars > 1 && $_.key ∈ %dictionaryWords && $_.key ∉ stopwords-iso('English')});
%wordTallies2.elems
```
```
# 173
```

Instead of checking for dictionary words -- or in conjunction -- we can filter the word tallies to be only 
with words that are made of letters and dashes:

```perl6
my %wordTallies3 = %wordTallies2.grep({ $_.key ~~ / ^ [<:L> | '-']+ $ /});
%wordTallies3.elems
```
```
# 172
```

Here we tabulate the most frequent words (in descending order):

```perl6
my @tbls = do for %wordTallies3.pairs.sort(-*.value).rotor(40) { to-pretty-table(transpose([$_>>.key, $_>>.value])
.map({ %(<word count>.Array Z=> $_.Array) }), align => 'l', field-names => <word count>).Str }
to-pretty-table([%( ^@tbls.elems Z=> @tbls),], field-names => (0 ..^ @tbls.elems)>>.Str, align => 'l', :!header, vertical-char => ' ', horizontal-char => ' ');
```
```
# +                         +                            +                            +                         +
#   +-------------+-------+   +----------------+-------+   +----------------+-------+   +-------------+-------+  
#   | word        | count |   | word           | count |   | word           | count |   | word        | count |  
#   +-------------+-------+   +----------------+-------+   +----------------+-------+   +-------------+-------+  
#   | data        | 1432  |   | workflow       | 121   |   | latent         | 54    |   | dependent   | 26    |  
#   | tabular     | 513   |   | normal         | 120   |   | generator      | 52    |   | cosine      | 26    |  
#   | set         | 485   |   | recommendation | 120   |   | extend         | 52    |   | curve       | 26    |  
#   | create      | 444   |   | form           | 118   |   | verify         | 51    |   | class       | 25    |  
#   | generate    | 416   |   | term           | 112   |   | fit            | 48    |   | input       | 24    |  
#   | pipeline    | 341   |   | variable       | 110   |   | summary        | 45    |   | count       | 24    |  
#   | frame       | 331   |   | history        | 109   |   | step           | 44    |   | basis       | 23    |  
#   | values      | 330   |   | partition      | 107   |   | add            | 43    |   | method      | 23    |  
#   | context     | 329   |   | semantic       | 106   |   | plot           | 43    |   | minutes     | 22    |  
#   | display     | 256   |   | thesaurus      | 105   |   | repository     | 39    |   | inverse     | 22    |  
#   | names       | 254   |   | explain        | 100   |   | rescale        | 38    |   | false       | 21    |  
#   | matrix      | 243   |   | wide           | 99    |   | assert         | 38    |   | maximum     | 21    |  
#   | layer       | 242   |   | filter         | 95    |   | reduction      | 37    |   | minute      | 20    |  
#   | neural      | 226   |   | network        | 94    |   | sum            | 36    |   | temporal    | 19    |  
#   | random      | 225   |   | recommend      | 93    |   | modify         | 36    |   | sequence    | 19    |  
#   | max         | 224   |   | model          | 87    |   | image          | 35    |   | total       | 19    |  
#   | profile     | 215   |   | load           | 87    |   | remove         | 35    |   | hours       | 19    |  
#   | compute     | 191   |   | format         | 86    |   | drill          | 35    |   | steps       | 19    |  
#   | train       | 183   |   | extract        | 85    |   | symbolic       | 34    |   | element     | 18    |  
#   | standard    | 177   |   | transform      | 84    |   | terms          | 34    |   | absolute    | 18    |  
#   | arbitrary   | 176   |   | chain          | 83    |   | divide         | 34    |   | categorical | 18    |  
#   | batch       | 174   |   | frequency      | 82    |   | binary         | 33    |   | hour        | 17    |  
#   | size        | 173   |   | function       | 82    |   | chart          | 33    |   | map         | 17    |  
#   | calculate   | 172   |   | current        | 81    |   | tabulate       | 33    |   | audio       | 17    |  
#   | randomized  | 169   |   | decoder        | 80    |   | reduce         | 33    |   | collection  | 16    |  
#   | regression  | 167   |   | list           | 78    |   | classification | 32    |   | fraction    | 16    |  
#   | loss        | 166   |   | ensemble       | 77    |   | histogram      | 31    |   | validating  | 16    |  
#   | classifier  | 163   |   | initialize     | 74    |   | interpolation  | 30    |   | squared     | 16    |  
#   | assign      | 160   |   | entropy        | 74    |   | characteristic | 30    |   | true        | 15    |  
#   | driven      | 158   |   | object         | 73    |   | receiver       | 30    |   | validation  | 15    |  
#   | column      | 158   |   | dimension      | 70    |   | roc            | 30    |   | probability | 14    |  
#   | chance      | 158   |   | analysis       | 69    |   | ingest         | 30    |   | ctc         | 14    |  
#   | min         | 157   |   | summarize      | 65    |   | moving         | 30    |   | testing     | 14    |  
#   | time        | 147   |   | retrieve       | 64    |   | operating      | 30    |   | ramp        | 14    |  
#   | consumption | 146   |   | apply          | 61    |   | idf            | 30    |   | scalar      | 14    |  
#   | echo        | 138   |   | percent        | 61    |   | boolean        | 30    |   | measurement | 13    |  
#   | document    | 133   |   | cross          | 59    |   | equal          | 30    |   | day         | 13    |  
#   | item        | 130   |   | normalization  | 59    |   | axis           | 28    |   | synonym     | 13    |  
#   | word        | 128   |   | graph          | 56    |   | split          | 27    |   | density     | 12    |  
#   | series      | 124   |   | statistical    | 55    |   | degree         | 27    |   | naive       | 12    |  
#   +-------------+-------+   +----------------+-------+   +----------------+-------+   +-------------+-------+  
# +                         +                            +                            +                         +
```

------

## Data split

In this section we split the data into training and testing parts. The split is stratified per DSL.

Here we:
- Categorize the DSL commands according to their DSL label 
- Tabulate the corresponding number of commands per label

```perl6
srand(83);
my %splitGroups = @wCommands.categorize({ $_.value });
to-pretty-table([%splitGroups>>.elems,])
```
```
# +-----------------------+----------------+-----------------+----------------------+------------------------+--------------------+
# | NeuralNetworkCreation | Classification | Recommendations | RandomTabularDataset | LatentSemanticAnalysis | QuantileRegression |
# +-----------------------+----------------+-----------------+----------------------+------------------------+--------------------+
# |          870          |      870       |       870       |         870          |          870           |        870         |
# +-----------------------+----------------+-----------------+----------------------+------------------------+--------------------+
```

Here each category is:
- Randomly shuffled 
- Split into training and testing parts with the ratio 0.75 (using the function `take-drop` from ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers), [AAp3])
- The corresponding number of elements are tabulated

```perl6
my %split = %splitGroups.map( -> $g { $g.key => %( ['training', 'testing'] Z=> take-drop($g.value.pick(*), 0.75)) });
to-pretty-table(%split.map({ $_.key => $_.value>>.elems }))
```
```
# +------------------------+----------+---------+
# |                        | training | testing |
# +------------------------+----------+---------+
# | Classification         |   653    |   217   |
# | LatentSemanticAnalysis |   653    |   217   |
# | NeuralNetworkCreation  |   653    |   217   |
# | QuantileRegression     |   653    |   217   |
# | RandomTabularDataset   |   653    |   217   |
# | Recommendations        |   653    |   217   |
# +------------------------+----------+---------+
```

Here we aggregate the training and testing parts for each category and show the corresponding sizes: 

```perl6
my %split2;
for %split.kv -> $k, $v { 
	%split2<training> = %split2<training>.append(|$v<training>); 
	%split2<testing> = %split2<testing>.append(|$v<testing>);
};
%split2>>.elems
```
```
# {testing => 1302, training => 3918}
```

Here we show a sample of commands from the training part:

```perl6
.raku.say for %split2<training>.pick(6)
```
```
# "initialize the neural network 7tmq" => "NeuralNetworkCreation"
# "get from context gz9evj" => "QuantileRegression"
# "consider xj4nacw2 data" => "Classification"
# "display chart with dates" => "QuantileRegression"
# "show the tag types" => "Recommendations"
# "get dataset with id 6p0ydgtb9" => "QuantileRegression"
```

------

## Trie creation

Here we obtain the unique DSL commands labels:

```perl6
my @labels = unique(@wCommands>>.value)
```
```
# [Classification LatentSemanticAnalysis NeuralNetworkCreation QuantileRegression RandomTabularDataset Recommendations]
```

Here we make a "known words" set using the "frequent enough" words of the training data:

```perl6
%wordTallies = %split2<training>>>.key.map({ $_.split(/ \s | ',' /) }).&flatten>>.trim>>.lc.&tally;
%wordTallies2 = %wordTallies.grep({ $_.value ≥ 12 && $_.key.chars > 1 && $_.key ∉ stopwords-iso('English')});
%wordTallies3 = %wordTallies2.grep({ $_.key ~~ / ^ [<:L> | '-']+ $ /});
%wordTallies3.elems
```
```
# 206
```

```perl6
my %knownWords = Set(%wordTallies3);
%knownWords.elems
```
```
# 206
```

Here we define a sub that converts commands into trie-phrases: 

```perl6
multi make-trie-basket(Str $command, %knownWords) {
	$command.split(/\s | ','/)>>.trim>>.lc.grep({ $_ (elem) %knownWords }).unique.sort.Array
}

multi make-trie-basket(Pair $p, %knownWords) {
   make-trie-basket($p.key, %knownWords).append($p.value)
}
```
```
# &make-trie-basket
```

Here is an example invocation of `make-trie-basket`:

```perl6
my $rb = %split2<training>.pick;
say $rb.raku;
say make-trie-basket($rb, %knownWords).raku;
```
```
# "set encoder Class by 4ndjyqve 4ndjyqve" => "NeuralNetworkCreation"
# ["class", "encoder", "set", "NeuralNetworkCreation"]
```

Here we convert all training data commands into trie-phrases:

```perl6
my $tStart = now;

my @training = %split2<training>.map({ make-trie-basket($_, %knownWords) }).Array;

say "Time to process traning commands: {now - $tStart}."
```
```
# Time to process traning commands: 0.336231422.
```

Here we make the trie:

```perl6
$tStart = now;

my $trDSL = @training.&trie-create.node-probabilities;

say "Time to make the DSL trie: {now - $tStart}."
```
```
# Time to make the DSL trie: 0.445799211.
```

Here are the trie node counts:

```perl6
$trDSL.node-counts
```
```
# {Internal => 5259, Leaves => 1785, Total => 7044}
```

Here is an example classification of a command:

```perl6
$trDSL.classify(make-trie-basket('show the outliers', %knownWords), prop => 'Probabilities'):!verify-key-existence
```
```
# {Classification => 0.5862068965517241, QuantileRegression => 0.4137931034482759}
```

------

## Confusion matrix

In this section we put together the confusion matrix of derived trie classifier over the testing data.

First we define a sub that gives the actual and predicted DSL-labels for a given training rule:

```perl6
sub make-cf-couple(Pair $p) {
    my $query = make-trie-basket($p.key, %knownWords);
    my $lbl = $trDSL.classify($query, :!verify-key-existence);
    %(actual => $p.value, predicted => ($lbl ~~ Str) ?? $lbl !! 'NA', command => $p.key)
}
```
```
# &make-cf-couple
```

Here we classify all commands in the testing data part:

```perl6
my $tStart = now;

my @actualPredicted = %split2<testing>.map({ make-cf-couple($_) }).Array;

my $tEnd = now;
say "Total time to classify {%split2<testing>.elems} tests with the DSL trie: {$tEnd - $tStart}.";
say "Time per classification: {($tEnd - $tStart)/@actualPredicted.elems}."
```
```
# Total time to classify 1302 tests with the DSL trie: 0.608161339.
# Time per classification: 0.00046709780261136716.
```

Here is the confusion matrix (using `cross-tabulate` of 
["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers), [AAp3]):

```perl6
my $ct = cross-tabulate(@actualPredicted, "actual", "predicted");
to-pretty-table($ct, field-names=>@labels.sort.Array.append('NA'))
```
```
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----+
# |                        | Classification | LatentSemanticAnalysis | NeuralNetworkCreation | QuantileRegression | RandomTabularDataset | Recommendations | NA |
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----+
# | Classification         |      170       |           9            |                       |         18         |          9           |        8        | 3  |
# | LatentSemanticAnalysis |       3        |          195           |                       |         3          |          6           |        10       |    |
# | NeuralNetworkCreation  |                |                        |          199          |         2          |                      |        2        | 14 |
# | QuantileRegression     |       25       |           13           |           6           |        150         |          5           |        16       | 2  |
# | RandomTabularDataset   |                |                        |                       |                    |         216          |        1        |    |
# | Recommendations        |       3        |           12           |           1           |         14         |          12          |       167       | 8  |
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----+
```

Here are the corresponding fractions:

```perl6
my $ct2 = $ct.map({ $_.key => $_.value <</>> $_.value.values.sum });
to-pretty-table($ct2, field-names=>@labels.sort.Array.append('NA'))
```
```
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----------+
# |                        | Classification | LatentSemanticAnalysis | NeuralNetworkCreation | QuantileRegression | RandomTabularDataset | Recommendations |    NA    |
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----------+
# | Classification         |    0.783410    |        0.041475        |                       |      0.082949      |       0.041475       |     0.036866    | 0.013825 |
# | LatentSemanticAnalysis |    0.013825    |        0.898618        |                       |      0.013825      |       0.027650       |     0.046083    |          |
# | NeuralNetworkCreation  |                |                        |        0.917051       |      0.009217      |                      |     0.009217    | 0.064516 |
# | QuantileRegression     |    0.115207    |        0.059908        |        0.027650       |      0.691244      |       0.023041       |     0.073733    | 0.009217 |
# | RandomTabularDataset   |                |                        |                       |                    |       0.995392       |     0.004608    |          |
# | Recommendations        |    0.013825    |        0.055300        |        0.004608       |      0.064516      |       0.055300       |     0.769585    | 0.036866 |
# +------------------------+----------------+------------------------+-----------------------+--------------------+----------------------+-----------------+----------+
```

Here is the diagonal of the confusion matrix:

```perl6
to-pretty-table( @labels.map({ $_ => $ct2.Hash{$_;$_} }) )
```
```
# +------------------------+----------+
# |                        |    0     |
# +------------------------+----------+
# | Classification         | 0.783410 |
# | LatentSemanticAnalysis | 0.898618 |
# | NeuralNetworkCreation  | 0.917051 |
# | QuantileRegression     | 0.691244 |
# | RandomTabularDataset   | 0.995392 |
# | Recommendations        | 0.769585 |
# +------------------------+----------+
```

By examining the confusion matrices we can conclude that the classifier is accurate enough.
(We examine the diagonals of the matrices and the most frequent confusions.)

By examining the computational timings we conclude that the classifier is both accurate and fast enough.

**Remark:** We addition to the confusion matrix we can do compute the Top-K query statistics -- not done here.
(Top-2 query statistic is answering the question: "Is the expected label one of the top 2 most probable labels?")

Here we show a sample of confused (misclassified) commands:

```perl6
srand(883);
to-pretty-table(@actualPredicted.grep({ $_<actual> ne $_<predicted> }).pick(12).sort({ $_<command> }), field-names=><actual predicted command>, align=>'l')
```
```
# +------------------------+------------------------+-----------------------------------------------------------------------------+
# | actual                 | predicted              | command                                                                     |
# +------------------------+------------------------+-----------------------------------------------------------------------------+
# | NeuralNetworkCreation  | NA                     | ConstantArrayLayer                                                          |
# | Classification         | RandomTabularDataset   | consider data zow6 data                                                     |
# | QuantileRegression     | LatentSemanticAnalysis | display current pipeline context                                            |
# | Classification         | QuantileRegression     | echo summary                                                                |
# | QuantileRegression     | Classification         | find and echo the time series top the outliers by quantile 246.654 quantile |
# | Classification         | Recommendations        | find and echo variable importance                                           |
# | QuantileRegression     | RandomTabularDataset   | get data with id m1lyaov                                                    |
# | QuantileRegression     | Classification         | give summary                                                                |
# | Recommendations        | NeuralNetworkCreation  | join the recommendations over the c9u8j4l2qo                                |
# | LatentSemanticAnalysis | Recommendations        | put in context as w1ku8pxev                                                 |
# | Recommendations        | Classification         | show pipeline context keys                                                  |
# | Recommendations        | NA                     | suggest via esv4zaunp -> 538.697                                            |
# +------------------------+------------------------+-----------------------------------------------------------------------------+
```

**Remark:** We observe that a certain proportion of the misclassified commands are ambiguous -- they do not belong to only one DSL.

-----

## Association rules

In this section we go through the association rules finding outlined above. 

**Remark:** The found frequent sets can be used for ML "feature engineering."
They can also be seen as a supplement or alternative to the ML classification "importance of variables" investigations.

**Remark:** We do not present the trie classifier making and accuracy results with frequent sets, 
but I can (bravely) declare that experiments with trie classifiers made with the words 
of found frequent sets produce very similar results as the trie classifiers with (simple) word-tallies.

Here we process the "word baskets" made from the DSL commands and append corresponding DSL workflow labels:

```{perl6, eval=FALSE}
my $tStart = now;

my @baskets = @wCommands.map({ ($_.key.split(/\s | ','/)>>.trim.grep({ $_.chars > 0 && $_ ~~ /<:L>+/ && $_ ∈ %dictionaryWords && $_ ∉ stopwords-iso('English')})).Array.append($_.value) }).Array;

say "Number of baskets: {@baskets.elems}";

say "Time to process baskets {now - $tStart}."
```

```{perl6, eval=FALSE}
# Number of baskets: 5220
# Time to process baskets 17.622710602.
```

Here is a sample of the baskets:

```{perl6, eval=FALSE}
.say for @baskets.pick(6) 
```

```{perl6, eval=FALSE}
# [transform symbolic numeric Classification]
# [echo data time series data graph QuantileRegression]
# [verify Classification]
# [compute moving average QuantileRegression]
# [calculate rescale add context time series data default step QuantileRegression]
# [partition LatentSemanticAnalysis]
```

Here is a summary of the basket sizes:

```{perl6, eval=FALSE}
records-summary(@baskets>>.elems)
```

```{perl6, eval=FALSE}
# +--------------------+
# | numerical          |
# +--------------------+
# | Min    => 1        |
# | 3rd-Qu => 5        |
# | Max    => 37       |
# | 1st-Qu => 2        |
# | Mean   => 4.081418 |
# | Median => 3        |
# +--------------------+
```

Here we find frequent sets of words (using the function `frequent-sets` from 
["ML::AssociationRuleLearning"](https://raku.land/zef:antononcube/ML::AssociationRuleLearning), [AAp7]):

```{perl6, eval=FALSE}
my $tStart = now;

my @freqSets = frequent-sets(@baskets.grep({ 3 < $_.elems }).Array, min-support => 0.005, min-number-of-items => 2, max-number-of-items => 6):counts;

say "\t\tNumber of frequent sets: {@freqSets.elems}.";

my $tEnd = now;
say "Timing: {$tEnd - $tStart}."
```

```{perl6, eval=FALSE}
# Number of frequent sets: 5110.
# Timing: 138.428897789.
```

Here is a sample of the found frequent sets:

```{perl6, eval=FALSE}
.say for @freqSets.pick(12)
```

```{perl6, eval=FALSE}
# (form frame max tabular values) => 17
# (LatentSemanticAnalysis item matrix word) => 41
# (RandomTabularDataset chance data frame tabular values) => 12
# (create data random values) => 12
# (data max randomized set tabular) => 12
# (generate max names) => 18
# (matrix term) => 70
# (RandomTabularDataset data generate max min names) => 13
# (RandomTabularDataset arbitrary form frame) => 12
# (arbitrary data randomized) => 13
# (RandomTabularDataset chance data driven min set) => 18
# (create set tabular) => 47

```

------

## Conclusion

In this section we discuss assumptions, alternatives, and "final" classifier deployment.

### Hopes

It is hoped that the classifier created with the procedures above is going to be adequate in the 
"real world." This is largely dependent on the quality of the training data. 

The data presented and used above use grammar-rules generated commands and those commands are generalized by:
- Removing the sequential order of the words
- Using only frequent enough, dictionary words

### Using a recommender instead

We also experimented with a Recommender-based Classifier (RC) -- 
the accuracy results with RC were slightly better (4±2%) than the trie-based classifier,
but RC is ≈10 times slower. We plan to discuss RC training and results in a subsequent article.

### Final result
 
Since we find the performance of the trie-based classifier satisfactory -- both accuracy-wise and speed-wise --
we make a classifier with all of the DSL commands data. See the resource file 
["dsl-trie-classifier.json"](https://github.com/antononcube/Raku-DSL-Shared-Utilities-ComprehensiveTranslation/blob/main/resources/dsl-trie-classifier.json), 
of [AAp5].

```perl6
my $trie-to-export = [|%split2<training>, |%split2<testing>].map({ make-trie-basket($_, %knownWords) }).Array.&trie-create;
$trie-to-export.node-counts;
```
```
# {Internal => 6560, Leaves => 2175, Total => 8735}
```

```perl6
spurt 'dsl-trie-classifier.json', $trie-to-export.JSON;
```
```
# True
```

------

## References

### Articles

[AA1] Anton Antonov,
["A monad for classification workflows"](https://mathematicaforprediction.wordpress.com/2018/05/15/a-monad-for-classification-workflows/),
(2018),
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).

[AA2] Anton Antonov,
["Trie-based classifiers evaluation"](https://rakuforprediction.wordpress.com/2022/07/07/trie-based-classifiers-evaluation/),
(2022),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com/2022/07/07/trie-based-classifiers-evaluation/).

### Packages

[AAp1] Anton Antonov,
[Data::Generators Raku package](https://github.com/antononcube/Raku-Data-Generators),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[Data::ExampleDatasets Raku package](https://github.com/antononcube/Raku-Data-ExampleDatasets),
(2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[Data::Reshapers Raku package](https://github.com/antononcube/Raku-Data-Reshapers),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[Data::Summarizers Raku package](https://github.com/antononcube/Raku-Data-Summarizers),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov,
[DSL::Shared::Utilities::ComprehensiveTranslation Raku package](https://github.com/antononcube/Raku-DSL-Shared-Utilities-ComprehensiveTranslation),
(2020-2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov,
[Lingua::StopwordsISO Raku package](https://github.com/antononcube/Raku-Lingua-StopwordsISO),
(2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov,
[ML::AssociationRuleLearning Raku package](https://github.com/antononcube/Raku-ML-AssociationRuleLearning),
(2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp8] Anton Antonov,
[ML::ROCFunctions Raku package](https://github.com/antononcube/Raku-ML-ROCFunctions),
(2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp9] Anton Antonov,
[ML::TriesWithFrequencies Raku package](https://github.com/antononcube/Raku-ML-TriesWithFrequencies),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp10] Anton Antonov,
[Text::Plot Raku package](https://github.com/antononcube/Raku-Text-Plot),
(2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAp11] Anton Antonov,
[Functional parsers Mathematica package](https://github.com/antononcube/MathematicaForPrediction/blob/master/FunctionalParsers.m),
(2014),
[MathematicaForPrediction at GitHub](https://github.com/antononcube/MathematicaForPrediction).

### Repositories

[AAr1] Anton Antonov,
[ConversationalAgents project](https://github.com/antononcube/ConversationalAgents),
(2017-2022),
[GitHub/antononcube](https://github.com/antononcube).

[AAr2] Anton Antonov,
[NLP Template Engine](https://github.com/antononcube/NLP-Template-Engine),
(2021-2022),
[GitHub/antononcube](https://github.com/antononcube).

### Videos

[AAv1] Anton Antonov, 
["NLP Template Engine, Part 1"](https://youtu.be/a6PvmZnvF9I), 
(2021), 
[Simplified Machine Learning Workflows at YouTube](https://www.youtube.com/playlist?list=PLke9UbqjOSOi1fc0NkJTdK767cL9XHJF0).

[AAv2] Anton Antonov
["Raku for Prediction"](https://www.youtube.com/watch?v=frpCBjbQtnA),
(2021),
[TRC-2021](https://conf.raku.org/2021).


