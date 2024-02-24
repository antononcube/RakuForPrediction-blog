# Using Tries for Markov chain text generation

Anton Antonov   
[MathematicaForPrediction at GitHub](https://github.com/antononcube/MathematicaForPrediction)   
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com)   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
January 2023

## Introduction

In this document we discuss the derivation and utilization of 
[Markov chains](https://en.wikipedia.org/wiki/Markov_chain) 
for random text generation. The Markov chains are computed, represented, and utilized with the data structure 
[Tries with frequencies](https://mathematicaforprediction.wordpress.com/?s=tries+with+frequencies), [AA2, AAp1, AAv1]. 
([Tries](https://en.wikipedia.org/wiki/Trie) are also known as "prefix trees.")

We can say that for a given text we use Tries with frequencies to derive language models, and we generate ***new,
plausible text*** using those language models.

In a previous article, [AA1], I discussed how text generation with Markov chains can be implemented with 
Wolfram Language (WL) sparse arrays.

**Remark:** Tries with frequencies can be also implemented with WL's 
[`Tree`](https://reference.wolfram.com/language/ref/Tree.html) structure. 
The implementation in [AAp5] -- which corresonds to [AAp1] -- relies only on 
[`Association`](https://reference.wolfram.com/language/guide/Associations.html), though. 
Further, during the Wolfram Technology Conference 2022 I think I successfully convinced 
[Ian Ford](https://community.wolfram.com/web/ianf) to implement Tries with frequencies using WL's `Tree`.
(Ideally, at some point his implementation would be more scalable and faster than my WL implementation.)

**Remark:** We can say that this document provides examples of making language models that are (much) simpler 
than Chat GPT's models, as mentioned by Stephen Wolfram in [SWv1]. 
With these kind of models we can easily generate sentences, like,
["The house ate the lettuce." or "The chair was happy."](https://www.youtube.com/watch?v=zLnhg9kir3Q&t=650s), [SWv1].

**Remark:** The package "Text::Markov", [PP1], by Paweł Pabian also generates text via Markov chains by
using a specialized, "targeted" implementation. This document presents a way of doing Markov chains 
simulations using the general data structure Tries with frequencies.

**Remark:** Language models similar to the ones built below can be used to do phrase completion and contextual spell checking.
The package 
["ML::Spellchecker"](https://github.com/antononcube/Raku-ML-Spellchecker), [AAp4], 
uses that approach.

**Remark:** This (computational) Markdown document corresponds to the Mathematica notebook
["Using Prefix trees for Markov chain text generation"](https://community.wolfram.com/groups/-/m/t/2819012).

-----

## Load packages

Here we load the packages
["ML::TriesWithFrequencies"](["Text::Plot"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies), [AAp1],
["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot), [AAp2], and
["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers), [AAp3]:

```perl6
use ML::TriesWithFrequencies;
use Text::Plot;
use Data::Reshapers;
```

------

## Ingest text

Download
["OriginOfSpecies.txt.zip"](https://github.com/antononcube/SimplifiedMachineLearningWorkflows-book/blob/master/Data/OriginOfSpecies.txt.zip)
file and uzip it.

Get the text from that file:

```perl6
my $text = slurp($*HOME ~ '/Downloads/OriginOfSpecies.txt');
$text.chars
```

Turn the text into separate words and punctuation characters::

```perl6
my $tstart = now;

my @words = $text.match(:g, / \w+ | <punct> /).map({ ~$_ })>>.lc;

my $tend = now;
say "Time to split into words: { $tend - $tstart }.";
```

Some statistics:

```perl6
say "Number of words: { @words.elems }.";
say "Number of unique words: { @words.Bag.elems }.";
```

Here is a sample:

```perl6
@words.head(32).raku;
```

Here is the plot that shows Pareto principle adherence to the word counts:

```perl6
text-pareto-principle-plot( @words.Bag.values.List, title => 'Word counts');
```

**Remark:** We see typical Pareto principle manifestation: 
≈10% of the unique words correspond to ≈80% of all words in the text.

------

## Markov chain construction and utilization

Derive 2-grams:

```perl6
my $n = 2;
my @nGrams = @words.rotor($n => -1);
@nGrams.elems
```

Here we create a trie using 2-grams:

```perl6
$tstart = now;

my $trWords = trie-create(@nGrams);

$tend = now;
say "Time to create the n-gram trie: { $tend - $tstart }.";
```

Example of random n-gram generation:

```perl6
.say for $trWords.random-choice(6):drop-root;
```

Markov chain generation:

```perl6
my @generated = ['every', 'author'];

for ^30 -> $i {
    @generated = [|@generated.head(*- 1), |$trWords.retrieve([@generated.tail,]).random-choice:drop-root]
}

say @generated;
```

Here we modify the code line above into a sub that:

1. Generates and splices random n-grams until `$maxSentences` number sentences are obtained
2. Make the generated text more "presentable"

```perl6
sub text-generation($trWords, @startPhrase, UInt $maxSentences = 10) {
    my @generated = @startPhrase>>.lc;
    my $nSentences = 0;
    loop {
        my @ph = $trWords.retrieve([@generated.tail,]).random-choice:drop-root;
        if @ph.tail ∈ <. ? !> { $nSentences++ };
        @generated = [|@generated.head(*- 1), |@ph];
        if $nSentences == $maxSentences { last; }
    }

    @generated[0] = @generated[0].tc;

    my $res = @generated.join(' ').subst(/ \s (<punct>) /, { ~$0 }):g;
    my $res2 = $res.subst(/ ('.' | '!' | '?') ' ' (\w) /, {  $0 ~ ' ' ~ $1.uc }):g;
    return $res2;
}
```

**Remark:** We consider a complete sentence to finish with period, question mark, or exclamatory mark (`<. ! ?>`).
Hence, the stopping criteria "counts" how many times those punctuation marks have been reached with the generated
n-grams.

Generate sentences:

```perl6
text-generation($trWords, <Consider the>, 3)
```

------

## Using larger n-grams

Let us repeat the text generation above with larger n-grams (e.g. 4-grams).

Here generate the n-gram trie:

```perl6
my $n = 4;
my @nGrams = @words.rotor($n => -1);
my $trWords = trie-create(@nGrams);
$trWords.node-counts
```

Generate text and show it:

```perl6
my $genText = text-generation($trWords, <Every author>, 6);
.trim.say for $genText.split(/ < . ! ? > /, :v).rotor(2)>>.join;
```

**Remark:** Using longer n-grams generally produces longer sentences since the probability to "reach" the period
punctuation character with longer n-grams becomes smaller.

-------

## Language model adherence verification

Let us convince ourselves that the method`random-choice` produces words with distribution that correspond to the
trie it is invoked upon.

Here we make a trie for a certain set of words:

```perl6
my @words = ['bar' xx 6].append('bark' xx 3).append('bare' xx 2).append('cam' xx 3).append('came').append('camelia' xx 4);
my $ptrOriginal = trie-create-by-split(@words).node-probabilities;
$ptrOriginal.node-counts
```

Here we generate random words with using the trie above and make a new trie with them:

```perl6
my $ptrGenerated = trie-create($ptrOriginal.random-choice(120):drop-root).node-probabilities;
$ptrGenerated.node-counts
```

Here is a comparison table between the two tries:

```perl6
my %tries = Original => $ptrOriginal, Generated => $ptrGenerated;
say to-pretty-table([%tries>>.form,], field-names => <Original Generated>, align => 'l');
```

We see that the probabilities at the nodes are very similar. It is expected that with large number of generation words
nearly the same probabilities to be obtained.

------

## Possible extensions

Possible extensions include the following:

- Finding Part Of Speech (POS) label for each word and making "generalized" sequences of POS labels.

  - Those kind of POS-based language models can be combined the with the "regular", word-based ones in variety of ways.
    
  - One such way is to use a POS-based model as a censurer of a word-based model. 
  
  - Another is to use a POS-based model to generate POS sequences, and then "fill-in" those sequences with actual words.
  
- N-gram-based predictions can be used to do phrase completions in (specialized) search engines. 

  - That can be especially useful if the phrases belong to a certain Domain Specific Language (DSL). 
    (And there is large enough collection of search queries with that DSL.)

- Instead of words any sequential data can be used. 

  - See [AAv1] for an application to predicting driving trips destinations.
  
  - Certain business simulation models can be done with Trie-based sequential models.
  
------

## References

### Articles

[AA1] Anton Antonov, 
["Markov chains n-gram model implementation](https://mathematicaforprediction.wordpress.com/2014/01/25/markov-chains-n-gram-model-implementation/), 
(2014), 
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).

[AA2] Anton Antonov, 
["Tries with frequencies for data mining"](https://mathematicaforprediction.wordpress.com/2013/12/06/tries-with-frequencies-for-data-mining/), 
(2013), 
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).

### Packages

[AAp1] Anton Antonov,
[ML::TriesWithFrequencies Raku package](https://raku.land/zef:antononcube/ML::TriesWithFrequencies),
(2021-2023),
[raku.land/zef:antononcube](https://raku.land/zef:antononcube).

[AAp2] Anton Antonov,
[Text::Plot Raku package](https://raku.land/zef:antononcube/Text::Plot),
(2022),
[raku.land/zef:antononcube](https://raku.land/zef:antononcube).

[AAp3] Anton Antonov,
[Data::Reshaperes Raku package](https://raku.land/zef:antononcube/Data::Reshapers),
(2021-2023),
[raku.land/zef:antononcube](https://raku.land/zef:antononcube).

[AAp4] Anton Antonov,
["ML::Spellchecker"](https://github.com/antononcube/Raku-ML-Spellchecker),
(2023),
[GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov, 
[TriesWithFrequencies Mathematica package,](https://github.com/antononcube/MathematicaForPrediction/blob/master/TriesWithFrequencies.m),
(2018), 
[MathematicaForPrediction at GitHub](https://github.com/antononcube/MathematicaForPrediction).


[PP1] Paweł Pabian,
[Text::Markov Raku package](https://raku.land/zef:bbkr/Text::Markov),
(2016-2023),
[raku.land/zef:bbkr](https://raku.land/zef:bbkr).

### Videos

[AAv1] Anton Antonov, 
["Prefix Trees with Frequencies for Data Analysis and Machine Learning"](https://www.youtube.com/watch?v=MdVp7t8xQbQ), 
(2017), 
Wolfram Technology Conference 2017.

[SWv1] Stephen Wolfram, 
["Stephen Wolfram Answers Live Questions About ChatGPT"](https://www.youtube.com/watch?v=zLnhg9kir3Q), 
(2023), 
[YouTube/Wolfram](https://www.youtube.com/@WolframResearch).