# Data::Generators

## Introduction

This blog post proclaims and describes the Raku package
[Data::Generators](https://raku.land/zef:antononcube/Data::Generators), [AAp0],
that has functions for generating random strings, words, pet names, vectors, arrays, and
(tabular) datasets. 

### Motivation

The primary motivation for this package is to have simple, intuitively named functions
for generating random vectors (lists) and datasets of different objects.

Although, Raku has a fairly good support of random vector generation, it is assumed that commands
like the following are easier to use:

```{raku, eval = FALSE}
say random-string(6, chars => 4, ranges => [ <y n Y N>, "0".."9" ] ).raku;
```

The function `random-tabular-dataset` of this package -- and the package 
["Data::ExampleDatasets"](https://raku.land/zef:antononcube/Data::ExampleDatasets), [AAp2] --
made easier and more complete the development and testing of 
["Data::Resphapers"](https://raku.land/zef:antononcube/Data::Reshapers), [AAp1, AA1].



---------

## Neat example

Here is an example that showcases all functions in this package:

```perl6
use Data::Generators;
use Data::Reshapers;

random-tabular-dataset(12, <String Real Word PetName JobTitle>, 
                       generators => {String => &random-string, 
                       Real => {random-real(12,$_)}, 
                       Word => &random-word, 
                       PetName => &random-pet-name, 
                       JobTitle => &random-pretentious-job-title})
==> to-pretty-table                       
```
```
# +----------------------------------+----------------------+-----------+----------------------+---------------+
# |             JobTitle             |       PetName        |    Real   |        String        |      Word     |
# +----------------------------------+----------------------+-----------+----------------------+---------------+
# |    Principal Brand Synergist     |        Heidi         |  5.708458 |   UyPcaAuHSP88Thc    |   air-to-air  |
# |   Chief Optimization Executive   |       Estelle        |  8.785965 |  vG0dpN05aBi9IUh2a   |    stomper    |
# |       Chief Brand Designer       |   Gluten The Great   |  4.740449 |  0w3T9fNfLvpG9csGU   |    Acalypha   |
# |    Human Implementation Agent    |        Olive         | 11.218193 |        16Dwo         |   abjection   |
# | Regional Communications Producer |   Gluten The Great   |  5.285270 |     jI4uQhOlfkZQ     |    riveting   |
# |     Legacy Web Administrator     | Griffonpoint Rhaegar | 10.588294 |        CFRj9f        |    bromate    |
# |   Investor Resonance Assistant   |        Millie        |  5.407836 |      rFUI8tfsqj      |  left-handed  |
# |   Chief Applications Associate   |         Zort         |  5.728717 |  esEPl5psP7eqMe8dl   |  infomercial  |
# |   Senior Factors Orchestrator    |       Guinness       |  4.466792 |        RcSNo         |     Enesco    |
# |   Product Mobility Technician    |      Prima III       | 10.480143 |       LSsibSJ        |   polymerize  |
# |   Lead Markets Representative    |       MR Tuxx        |  3.783014 | 52esMn28ZHdBznusEsGm |  flexibleness |
# |   District Tactics Technician    |        Pepers        |  7.724836 |  X2hoIr5FcinFEQhvY   | nonhereditary |
# +----------------------------------+----------------------+-----------+----------------------+---------------+
```

------

## Random strings

The function `random-string` generates random strings.

Here is a random string:

```perl6
use Data::Generators;
random-string
```
```
# uSIrX93
```

Here we generate a vector of random strings with length 4 and characters that belong to specified ranges:

```raku
say random-string(6, chars => 4, ranges => [ <y n Y N>, "0".."9" ] ).raku;
```
```
# ("3Y54", "Nn25", "2643", "09yn", "9909", "Y730")
```

------

## Random words

The function `random-word` generates random words.

Here is a random word:

```perl6
random-word
```
```
# stucco
```

Here we generate a list with 12 random words:

```raku
random-word(12)
```
```
# (splashing seafront pilaster lungi eagle catacorner unimpressed funnel-shaped drawnwork doctorfish adjacent frenzied)
```

Here we generate a table of random words of different types:

```raku
use Data::Reshapers;
my @dfWords = do for <Any Common Known Stop> -> $wt { $wt => random-word(6, type => $wt) };
say to-pretty-table(@dfWords);
```
```
# +--------+------------+--------------+---------------+--------------+--------------+----------+
# |        |     0      |      2       |       5       |      4       |      3       |    1     |
# +--------+------------+--------------+---------------+--------------+--------------+----------+
# | Any    |  wiretap   | concreteness |  undisclosed  |   elegant    |   Piemonte   |  vanity  |
# | Common | timorously | incapability |     bemuse    | lithographer |    vouch     | illusory |
# | Known  | adactylia  |      RN      | labyrinthitis |   Coreidae   | maltreatment |  squab   |
# | Stop   |     K      |     you      |       F       |      O       |    under     |   also   |
# +--------+------------+--------------+---------------+--------------+--------------+----------+
```

**Remark:** `Whatever` can be used instead of `'Any'`.

**Remark:** The function `to-pretty-table` is from the package 
[Data::Reshapers](https://modules.raku.org/dist/Data::Reshapers:cpan:ANTONOV).

------

## Random pet names

The function `random-pet-name` generates random pet names.

The pet names are taken from publicly available data of pet license registrations in
the years 2015–2020 in Seattle, WA, USA. See [DG1].

Here is a random pet name:

```perl6
random-pet-name
```
```
# Atticus
```

The following command generates a list of six random pet names:

```raku
srand(32);
random-pet-name(6).raku
```
```
# ("Millie", "Schmidt", "Professor Nibblesworth", "Millie", "Benzo", "Darcy")
```

The named argument `species` can be used to specify specie of the random pet names. 
(According to the specie-name relationships in [DG1].)

Here we generate a table of random pet names of different species:

```raku
my @dfPetNames = do for <Any Cat Dog Goat Pig> -> $wt { $wt => random-pet-name(6, species => $wt) };
say to-pretty-table(@dfPetNames);
```
```
# +------+--------+-----------+----------------------+------------+-------------+----------------+
# |      |   5    |     3     |          1           |     4      |      2      |       0        |
# +------+--------+-----------+----------------------+------------+-------------+----------------+
# | Any  | Millie |   Rubia   |       Hermann        |  Guinness  |    Nai'a    |    Hermann     |
# | Cat  |  Sea   | Peter pan |        Fanny         |   Danny    |   Greyman   |    Gulliver    |
# | Dog  | Konbu  |  Clarence | Ruth Bader Greenberg | Peppercorn | Bella Noche | My Girl Friday |
# | Goat | Aggie  |    Nick   |        Grace         |   Teddy    |     Max     |     Sassy      |
# | Pig  | Millie |   Millie  |       Guinness       |  Guinness  |    Millie   |    Atticus     |
# +------+--------+-----------+----------------------+------------+-------------+----------------+
```

**Remark:** `Whatever` can be used instead of `'Any'`.

The named argument (adverb) `weighted` can be used to specify random pet name choice 
based on known real-life number of occurrences:

```raku
srand(32);
say random-pet-name(6, :weighted).raku
```
```
# ("Brussels Sprout", "Atticus", "Starsky", "Schmidt", "Cornflower", "Milo")
```

The weights used correspond to the counts from [DG1].

**Remark:** The implementation of `random-pet-name` is based on the Mathematica implementation
[`RandomPetName`](https://resources.wolframcloud.com/FunctionRepository/resources/RandomPetName),
[AAf1].

------

## Random pretentious job titles

The function `random-pretentious-job-title` generates random pretentious job titles.

Here is a random pretentious job title:

```perl6
random-pretentious-job-title
```
```
# Relational Optimization Analyst
```

The following command generates a list of six random pretentious job titles:

```raku
random-pretentious-job-title(6).raku
```
```
# ("National Factors Engineer", "Chief Quality Designer", "Dynamic Tactics Architect", "Regional Identity Engineer", "Legacy Mobility Analyst", "Lead Group Facilitator")
```

The named argument `number-of-words` can be used to control the number of words in the generated job titles.

The named argument `language` can be used to control in which language the generated job titles are in.
At this point, only Bulgarian and English are supported.

Here we generate pretentious job titles using different languages and number of words per title:

```raku
my $res = random-pretentious-job-title(12, number-of-words => Whatever, language => Whatever);
say ‌‌to-pretty-table($res.rotor(3));
```
```
# +--------------------------------------+---------------------------+-----------------------------------+
# |                  0                   |             1             |                 2                 |
# +--------------------------------------+---------------------------+-----------------------------------+
# |          Мениджър на Пазари          | Фасилитатор на Интеграция |     Посредник на Конфигурации     |
# |  Клиентов Консултант по Изследвания  |      Research Planner     |            Orchestrator           |
# |               Директор               |     Плановик по Мрежи     |             Associate             |
# | Динамичен Администратор по Директиви |     Branding Synergist    | Директен Разработчик по Парадигми |
# +--------------------------------------+---------------------------+-----------------------------------+
```

**Remark:** `Whatever` can be used as values for the named arguments `number-of-words` and `language`.

**Remark:** The implementation uses the job title phrases in https://www.bullshitjob.com . 
It is, more-or-less, based on the Mathematica implementation 
[`RandomPretentiousJobTitle`](https://resources.wolframcloud.com/FunctionRepository/resources/RandomPretentiousJobTitle),
[AAf2].

------

## Random reals

This module provides the function `random-real` that can be used to generate lists of real numbers
using the uniform distribution.

Here is a random real:

```raku
say random-real(); 
```
```
# 0.5884674149505779
```

Here is a random real between 0 and 20:

```raku
say random-real(20); 
```
```
# 13.263658596119852
```

Here are six random reals between -2 and 12:

```raku
say random-real([-2,12], 6);
```
```
# (8.870681165460763 1.8612006267010126 6.575438206723781 7.519498832129216 11.977501199028579 11.40407320117866)
```

Here is a 4-by-3 array of random reals between -3 and 3:

```raku
say random-real([-3,3], [4,3]);
```
```
# [[-0.605344400722295 2.2357479203127255 0.278595195645174]
#  [2.758777563339059 -2.9181888559528977 -0.4885047451940778]
#  [-1.5054832770870736 2.8380077665645045 1.5546501176218097]
#  [-2.5686794378128166 -1.6379792732766252 0.6872287112668176]]
```


**Remark:** The signature design follows Mathematica's function
[`RandomReal`](https://reference.wolfram.com/language/ref/RandomVariate.html).


------

## Random variates

This module provides the function `random-variate` that can be used to generate lists of real numbers
using distribution specifications.

Here are examples:

```raku
say random-variate(NormalDistribution.new(:mean(10), :sd(20)), 5); 
```
```
# (-45.05443644020868 31.227470535033653 18.94517845607982 -13.718986366325858 55.19749082632336)
```

```raku
say random-variate(NormalDistribution.new( µ => 10, σ => 20), 5); 
```
```
# (20.090877012476692 37.38051554664387 -9.310714294155787 -5.996047502062092 16.417517557389573)
```

```raku
say random-variate(UniformDistribution.new(:min(2), :max(60)), 5);
```
```
# (23.15167681651898 56.6173691500257 27.80856840150481 12.017787851638628 23.436153972237744)
```

**Remark:** Only Normal distribution and Uniform distribution are implemented at this point.

**Remark:** The signature design follows Mathematica's function
[`RandomVariate`](https://reference.wolfram.com/language/ref/RandomVariate.html).

Here is an example of 2D array generation:

```raku
say random-variate(NormalDistribution.new, [3,4]);
```
```
# [[-1.2410155103090272 -0.0427247351772782 0.38921934353288223 -0.3668680366625281]
#  [2.344512773090577 0.7230743262953035 -1.3585656156206458 0.6356811957055507]
#  [1.9720976091224587 -1.680793520684229 1.0049458293849742 0.3606851853146195]]
```

------

## Random tabular datasets

The function `random-tabular-dataset` can be used generate tabular *datasets*.

**Remark:** In this module a *dataset* is (usually) an array of arrays of pairs.
The dataset data structure resembles closely Mathematica's data structure 
[`Dataset`](https://reference.wolfram.com/language/ref/Dataset.html), [WRI2]. 

**Remark:** The programming languages R and S have a data structure called "data frame" that
corresponds to dataset. (In the Python world the package `pandas` provides data frames.)
Data frames, though, are column-centric, not row-centric as datasets.
For example, data frames do not allow a column to have elements of heterogeneous types.

Here are basic calls:

```{perl6, eval=FALSE}
random-tabular-dataset();
random-tabular-dataset(Whatever):row-names;
random-tabular-dataset(Whatever, Whatever);
random-tabular-dataset(12, 4);
random-tabular-dataset(Whatever, 4);
random-tabular-dataset(Whatever, <Col1 Col2 Col3>):!row-names;
```

Here is example of a generated tabular dataset that column names that are cat pet names:

```raku
my @dfRand = random-tabular-dataset(5, 3, column-names-generator => { random-pet-name($_, species => 'Cat') });
say to-pretty-table(@dfRand);
```
```
# +------------+-------------+--------------+
# |   Tont2    |    Barnum   |     Seua     |
# +------------+-------------+--------------+
# |  bilimbi   |    chuck    | Pleuronectes |
# | churchyard |  Hemigalus  |   satirise   |
# | Polycirrus |   Brescia   |  metalworks  |
# |  azurite   |   racemose  |    chilli    |
# | syllabize  | threatening |   wagoner    |
# +------------+-------------+--------------+
```

The display function `to-pretty-table` is from
[`Data::Reshapers`](https://modules.raku.org/dist/Data::Reshapers:cpan:ANTONOV).

**Remark:** At this point only
[*wide format*](https://en.wikipedia.org/wiki/Wide_and_narrow_data)
datasets are generated. (The long format implementation is high in my TOOD list.)

**Remark:** The signature design and implementation are based on the Mathematica implementation
[`RandomTabularDataset`](https://resources.wolframcloud.com/FunctionRepository/resources/RandomTabularDataset),
[AAf3].

---------


## References

### Articles

[AA1] Anton Antonov,
["Introduction to Data Wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku),
(2021),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["Pets licensing data analysis"](https://mathematicaforprediction.wordpress.com/2020/01/20/pets-licensing-data-analysis/), 
(2020), 
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com).

### Data repositories

[DG1] Data.Gov,
[Seattle Pet Licenses](https://catalog.data.gov/dataset/seattle-pet-licenses),
[catalog.data.gov](https://catalog.data.gov).

### Functions

[AAf1] Anton Antonov,
[RandomPetName](https://resources.wolframcloud.com/FunctionRepository/resources/RandomPetName),
(2021),
[Wolfram Function Repository](https://resources.wolframcloud.com/FunctionRepository).

[AAf2] Anton Antonov,
[RandomPretentiousJobTitle](https://resources.wolframcloud.com/FunctionRepository/resources/RandomPretentiousJobTitle),
(2021),
[Wolfram Function Repository](https://resources.wolframcloud.com/FunctionRepository).

[AAf3] Anton Antonov,
[RandomTabularDataset](https://resources.wolframcloud.com/FunctionRepository/resources/RandomTabularDataset),
(2021),
[Wolfram Function Repository](https://resources.wolframcloud.com/FunctionRepository).

[SHf1] Sander Huisman,
[RandomString](https://resources.wolframcloud.com/FunctionRepository/resources/RandomString),
(2021),
[Wolfram Function Repository](https://resources.wolframcloud.com/FunctionRepository).

[WRI1] Wolfram Research (2010), 
[RandomVariate](https://reference.wolfram.com/language/ref/RandomVariate.html), 
Wolfram Language function.

[WRI2] Wolfram Research (2014),
[Dataset](https://reference.wolfram.com/language/ref/Dataset.html),
Wolfram Language function.

### Packages

[AAp0] Anton Antonov,
[Data::Generators Raku package](https://github.com/antononcube/Raku-Data-Generators),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp1] Anton Antonov,
[Data::Reshapers Raku package](https://github.com/antononcube/Raku-Data-Reshapers),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[Data::ExampleDatasets Raku package](https://github.com/antononcube/Raku-Data-ExampleDatasets),
(2021),
[GitHub/antononcube](https://github.com/antononcube).