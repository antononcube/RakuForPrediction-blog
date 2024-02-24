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
```
# (Any)
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
```
# 893121
```

Turn the text into separate words and punctuation characters::

```perl6
my $tstart = now;

my @words = $text.match(:g, / \w+ | <punct> /).map({ ~$_ })>>.lc;

my $tend = now;
say "Time to split into words: { $tend - $tstart }.";
```
```
# Time to split into words: 1.044725336.
```

Some statistics:

```perl6
say "Number of words: { @words.elems }.";
say "Number of unique words: { @words.Bag.elems }.";
```
```
# Number of words: 170843.
# Number of unique words: 6927.
```

Here is a sample:

```perl6
@words.head(32).raku;
```
```
# ("introduction", ".", "when", "on", "board", "h", ".", "m", ".", "s", ".", "'", "beagle", ",", "'", "as", "naturalist", ",", "i", "was", "much", "struck", "with", "certain", "facts", "in", "the", "distribution", "of", "the", "inhabitants", "of").Seq
```

Here is the plot that shows Pareto principle adherence to the word counts:

```perl6
text-pareto-principle-plot( @words.Bag.values.List, title => 'Word counts');
```
```
# Word counts                         
#     0.00           0.29          0.58           0.87        
# +---+--------------+-------------+--------------+----------+      
# |                                                          |      
# +               ****************************************   +  1.00
# |        ********                                          |      
# +     ****                                                 +  0.80
# |    **                                                    |      
# +   **                                                     +  0.60
# |   *                                                      |      
# |   *                                                      |      
# +   *                                                      +  0.40
# |   *                                                      |      
# +   *                                                      +  0.20
# |   *                                                      |      
# +                                                          +  0.00
# +---+--------------+-------------+--------------+----------+      
#     0.00           2000.00       4000.00        6000.00
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
```
# 170842
```

Here we create a trie using 2-grams:

```perl6
$tstart = now;

my $trWords = trie-create(@nGrams);

$tend = now;
say "Time to create the n-gram trie: { $tend - $tstart }.";
```
```
# Time to create the n-gram trie: 10.348492716.
```

Example of random n-gram generation:

```perl6
.say for $trWords.random-choice(6):drop-root;
```
```
# (means of)
# (see britain)
# (why certain)
# (more indispensable)
# (animal with)
# (myrmica .)
```

Markov chain generation:

```perl6
my @generated = ['every', 'author'];

for ^30 -> $i {
    @generated = [|@generated.head(*- 1), |$trWords.retrieve([@generated.tail,]).random-choice:drop-root]
}

say @generated;
```
```
# [every author ' s teazle , has originated in so be the case is always with such local , of cats , or habits and during some length and exterminate the principle]
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
```
# &text-generation
```

**Remark:** We consider a complete sentence to finish with period, question mark, or exclamatory mark (`<. ! ?>`).
Hence, the stopping criteria "counts" how many times those punctuation marks have been reached with the generated
n-grams.

Generate sentences:

```perl6
text-generation($trWords, <Consider the>, 3)
```
```
# Consider the secondary causes of the ascertained facts as domestication; and therefore equals all that great region: secondary formation, between them personally unknown laws of the fresh- ranges, and coast. Nor will always succeed in lesser degree by one man who is on my theory no progeny of the organs, i should this was at drones, a geometrical ratio of organs, that the number. The least is hardly any part of forms of honey.
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
```
# {Internal => 76219, Leaves => 53251, Total => 129470}
```

Generate text and show it:

```perl6
my $genText = text-generation($trWords, <Every author>, 6);
.trim.say for $genText.split(/ < . ! ? > /, :v).rotor(2)>>.join;
```
```
# Every author has remarked with those accumulated by tearing out its young, in some detail.
# .
# .
# .
# On the other constitutional and structural difference between the tongue( not any of the working ants differing widely from both cases the laws of growth, they did nothing can be easier than to admit that a multitude of forms, as we shall have been upraised, denuded, that peculiarities appearing in the males to struggle with distinct species), will have plenty of cases the sterility is alone affected.
# Nevertheless, looking to all the intermediate zones, that in our attention to any secondary formation, yet clearly related to each other hand, we are tending in the middle; hence the thoulouse and the common and much diffused, having produced, and where continents now exist in lesser numbers in a small isolated area, have often bred together and have been largely preserved.
# When we not find them a long, and without being around us may be supposed to a woodpecker, lesser changes in their classifications; and, under confinement, with great: if every form which are descended from north to south to north as far as lies in the record; and on insects on the view which i could easily show even a small genera.
# Both in our domestic productions, and insects in preserving them from danger.
# Grouse, and so on the other hand, during subsidence, but only in a slight differences accumulated during the alternate periods, it would never account for instance, not at the corresponding ages, can clearly understand on the young birds are generally quite unknown: in cryptocerus, the uppermost line would be only a state of nature is prodigal in the world performs an action for one special purpose, and that gradations in the eyes are of no service to it with indomitable perseverance, he pleases.
# " so with plants, have been the main agency in the production of the most naturalists, the nest of a degree of variability is favourable, including under this subject.
# From the adult, and subsequently have of course their mother to fly away.
# But the existence of discovering in a gradual increase in comparison with the creation of each species first existed more individuals,-- is included, from some one or gravel, will not admire a state of nature may be said to have taken with the court.
# " the benefit of the simpler case of the world on my theory have no just right to conclude that species are not see a single domestic animal can rarely be proved to be much more closely related to the latter does not build its own nest, presenting gradations of some kind, or obstacles to free migration at some former chapter.
# When at last produced by the same relative positions?
# If species are some apparent exceptions, hardly ever tell.
# On any continent, out of the conditions of life, in the hybrids, moreover other forms, are marine, and that there is, as mr.
# Waterhouse has remarked, and do not know, the chain of ordinary generation has once reject my theory no doubt were five feet four toes webbed, before those of any one or where the manufactory still in action of the conditions of life in which the several marks do not always absolutely perfect fertility of varieties which have undergone much modification in animals which unite for each birth; but in this case be grafted with no less than 100 plants of a moment on the fertile parents which alone, as yet imperfectly known, are really far greater, or madeira, we shall see no real difficulty in proving its loss through disuse or natural selection, except in the scale of life have been suddenly and temporarily increased in any country, and all feeding on the sudden appearance of whole groups subordinate to groups of species, yet are not necessarily the best of my judgment in some cases.
# The wonder, then, it is the reason why certain seeds, however, would be a tendency to show the blood, has come to inhabit distant land.
# Some of the complex instinct can possibly have been converted by subsidence into which it is more fertile, and extremely complex eye could be hereafter briefly mentioned.
# I will always have a naturalist, reflecting on all sorts of things may depart from its utmost to increase, its numbers than the varieties is immense; and this not breed, though rising higher on the view of various breeds, follows from the bird' s agency.
# I think there is clearly shown in their new homes.
# As i think there is not that the latter having read in the supplement to lyell' orbigny and others to their proper to that zone, they occasionally blind, and its enormously developed: this seems to have occurred to me that in the first appear and disappear; though here only state that if even the air.
# We should bear in such trifling points are; and obscured; we may infer that they all have varied under the first commencement having inhabited a protected from violent movement as in our imagination to give milk.
# In this same respect.
# Closely connected with the mainland, would not know, the same rules apply, because there can be no other class of all known animals, in the external conditions of the crossing of life has been formed by natural history.
# And which can increase of various animals having been but which i am certain of the conditions of the same with turf which has long period; for it can rarely has very distinct but consecutive stages, by which consequently have not known of any in the new specific forms which i have collected on this curious to ascertain whether any rock, out of the eocene inhabitants of its embryonic career is active, imperfect though it necessarily here is at present, that we might have been expected, systematic affinity between the several causes of error to argue that some at least perfect.
# It is now known cause of the flowering plants of the order on the fertile parents have lost some few still living descendants, was thought to be ranked as species having transmitted descendants.
# If the nature of the case at present, almost arbitrary.
# Several of each variety will be nearly related to the diversity in the breeds of fowls which each species has not been time in the one short period from each other far more easily than others; or were unknown, occur on the leaves; how unfavourable exposure to tree.
# We can understand how is it possible arrangement, would be a great error to suppose that it then of bees, acclimatisation must be saved without detriment to the individual or variety increases inordinately in numbers; and in some few cases throw light on the same area the more they will have taken, and it is with most of them could hardly have kept; thus, when we see, that although there is a large and extremely few of the baltic, or those which distinguish species from species of wild cattle were displaced by some botanists as huber has stated that the most complex relations of these great mutations in the forms marked a14 to that of all organic forms.
# Hence when in habits or structure of the parent.
# The common to whole groups of beings greatly struck those admirable memoir on the union of two species when first selected a pigeon, which had heads four instead of hair, and of spreading widely.
# Throughout the world; and here a powerful agent always lost in many species having similar slight deviation of life; and in some whole of this period of life active powers of swimming through the water, they could be formed of the pacific ocean.
# We can unite two forms of life changing drama.
# We here have perfect and as useful to man.
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
```
# {Internal => 10, Leaves => 3, Total => 13}
```

Here we generate random words with using the trie above and make a new trie with them:

```perl6
my $ptrGenerated = trie-create($ptrOriginal.random-choice(120):drop-root).node-probabilities;
$ptrGenerated.node-counts
```
```
# {Internal => 10, Leaves => 3, Total => 13}
```

Here is a comparison table between the two tries:

```perl6
my %tries = Original => $ptrOriginal, Generated => $ptrGenerated;
say to-pretty-table([%tries>>.form,], field-names => <Original Generated>, align => 'l');
```
```
# +----------------------------------+---------------------------------+
# | Original                         | Generated                       |
# +----------------------------------+---------------------------------+
# | TRIEROOT => 1                    | TRIEROOT => 1                   |
# | ├─b => 0.5789473684210527        | ├─b => 0.5333333333333333       |
# | │ └─a => 1                       | │ └─a => 1                      |
# | │   └─r => 1                     | │   └─r => 1                    |
# | │     ├─e => 0.18181818181818182 | │     ├─e => 0.21875            |
# | │     └─k => 0.2727272727272727  | │     └─k => 0.203125           |
# | └─c => 0.42105263157894735       | └─c => 0.4666666666666667       |
# |   └─a => 1                       |   └─a => 1                      |
# |     └─m => 1                     |     └─m => 1                    |
# |       └─e => 0.625               |       └─e => 0.6428571428571429 |
# |         └─l => 0.8               |         └─l => 1                |
# |           └─i => 1               |           └─i => 1              |
# |             └─a => 1             |             └─a => 1            |
# +----------------------------------+---------------------------------+
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