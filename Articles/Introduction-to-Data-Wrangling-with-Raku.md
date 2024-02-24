
# Introduction to data wrangling with Raku

**Version 1.0**

Anton Antonov   
[RakuForPrediction at GitHub](https://github.com/antononcube/RakuForPrediction-book)   
December 2021   

## Introduction

One of my current life missions is the speeding up of the next [AI winter](https://en.wikipedia.org/wiki/AI_winter) coming. 
I have decided to use Raku to accomplish that mission. (Well, at least in the beginning...) 

Also, I find the term Artificial Intelligence (AI) to be a brilliant marketing phrase for extracting money of the 
US military complex and all kinds of financial investors. When people who claim to be, say, “data science professionals” 
use it in professional settings, I become highly suspicious of their “professionalism.” 
(Meaning, they are most likely some clueless wannabes.)

Back to next AI winter coming speed-up -- here is my plan:

1. Teach many people [how to be data scientist impostors](https://github.com/antononcube/HowToBeADataScientistImpostor-book)

2. Produce [tools](https://conf.raku.org/talk/157) that facilitate AI-code baristas

3. Wait for enough AI-people to hit enough AI walls.

4. Next AI winter

5. Profit 

See the presentation ["Raku for Prediction"](https://conf.raku.org/talk/157), [AAv2], that describes my efforts over Point 2 using Raku.

A long version of Point 3 is “wait for AI investors and managers to hit sufficiently many walls using a large, 
talentless mass of AI practitioners.”

The profit in Point 5 comes from "the field" being "clear", hence, less competition for getting funding and investments.
(Even after this clarification some might still think my list above is too similar to the one concocted by the gnomes in 
[South Park's "Gnomes"](https://en.wikipedia.org/wiki/Gnomes_(South_Park));
and that is just fine with me.)

Most data scientists spend most of their time doing data acquisition and data wrangling. 
Not data science, or AI, or whatever “really learned” work... 
So, if I am serious about influencing the evolution curves of AI, then I must get serious about influencing 
data acquisition and data wrangling incantations derivations in different programming languages; [AAv2]. 
Since I firmly believe that it is good to occasionally eat your own dog food, 
I programmed (in the last few weeks) data wrangling packages in Raku. 
Who knows, that might be a way to Rakunize AI, and -- 
[to rephrase Larry Wall](https://en.wikipedia.org/wiki/Raku_(programming_language)#History) 
-- some users might get their fix. (See also [FB1].)

As for data acquisition -- I have a 
[Data Acquisition Engine project](https://github.com/antononcube/Data-Acquisition-Engine-project), 
[AAr1, AAv6],
which has a conversational agent that uses code generation through a Raku package, [AAp7].
In order to discuss and exemplify data wrangling we have to utilize certain data acquisition functionalities.
Because of that, below are given explanations and examples of using Raku packages for retrieval of popular, 
well known datasets and for generation of random datasets.  

This document is fairly technical -- readers can just read or skim the next section and the section 
"Doing it like a Cro" and be done. Some might want to look and skim over the super-technical version 
"Data wrangling with Raku", [AA1].

**Remark:** Occasionally the code below might have the Raku expression `==>encode-to-wl()`. 
This is for serializing Raku objects into 
[Wolfram Language (WL)](https://en.wikipedia.org/wiki/Wolfram_Language) 
expressions. 
(This document was written as a [Mathematica](https://en.wikipedia.org/wiki/Wolfram_Mathematica) notebook.)

**Remark:** The target audience for this document mostly consists of people exposed to Perl’s and Raku’s cultures and cults. 
But most of this document should be accessible and of interest to the un-perled programmers or data scientists.

**Remark:** As it can be seen in the (long) presentation recording
["Multi language Data Acquisition Conversational Agent (extended version)"](https://www.youtube.com/watch?v=KlEl2b8oxb8),
[AAv6], a Raku data acquisition conversational agent can greatly leverage and utilize Raku data wrangling capabilities.  

**Remark:** Sentences with "universal", verifiable, or reproducible statements, assertions, and code use the "we form." 
Author's personal opinions and decisions statements use the "I form." 
Alternatively, I just used whatever form felt easier or more natural to write with.

------

## 楽-for ÷ with-楽

... aka ***“Raku-for vs with-Raku”*** *(also, maybe, "enjoyment for" vs "with enjoyment.")*

First, let us make the following distinction:

- Raku ***for*** data wrangling means using Raku to facilitate data wrangling in other programming languages and systems.

- Data wrangling ***with*** Raku means using Raku’s data structures and programming language for data wrangling.

Here is an example of Raku ***for*** data wrangling -- Python code is generated:

```perl6
ToDSLCode("
dsl target Python-pandas; 
load dataset iris; 
group by Species;
show counts"):code

# obj = example_dataset('iris')
# obj = obj.groupby([\"Species\"])
# print(obj.size())
```

Here is an example of data wrangling ***with*** Raku:

```perl6
my $obj = example-dataset(/ iris $ /);
$obj = group-by($obj, "Species");
say $obj>>.elems

# {setosa => 50, versicolor => 50, virginica => 50}
```

The following diagram:

- Summarizes my data wrangling activities
- Indicates future plans with dashed lines
- Represents “Raku-for” efforts with the hexagon
- Represents “with-Raku” efforts bottom-right

```mathematica
ImageCrop@Import["https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/DSLs-Interpreter-for-Data-Wrangling-Dec-2021-state.png"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/DSLs-Interpreter-for-Data-Wrangling-Dec-2021-state.png)

In order to illustrate the multilinguality of the approach here is an example of translating 
English data wrangling specs into Bulgarian, Korean, and Spanish data wrangling specs:

```perl6
for <Bulgarian Korean Spanish> -> $l {
 say '-' x 30;
 say ToDSLCode("dsl target " ~ $l ~"; 
 load dataset iris; 
 group by Species;
 show counts"):code
}

# ------------------------------
# зареди таблицата: "iris"
# групирай с колоните: Species
# покажи броя
# ------------------------------
# 테이블로드: "iris"
# 열로 그룹화: Species
# 쇼 카운트
# ------------------------------
# cargar la tabla: "iris"
# agrupar con columnas: \"Species\"
# mostrar recuentos
```

As the diagram above indicates, I intend to use that framework to narrate data wrangling Raku code with natural languages. 
Also, of course, translate into other programing languages.

**Remark:** For a long time I used the principle "the clothes have no emperor", [AAv1, CARH1], which, 
of course belongs to the "Raku-for-prediction" approach. By endowing Raku with (i) data wrangling capabilities, and 
(ii) the ability to generate data wrangling Raku code, I would say that those clothes might get properly manned. 
And vice versa -- the shoemaker's children will hop around properly shod.

------

## Datum fundamentum

... *aka* ***“Data structures and methodology”*** *(Also, means “given foundation” in Latin.)*

Let us define or outline the basic concepts of our Raku data wrangling endeavor.

### Dataset vs Data frame

Here are certain intuitive definitions of datasets and data frames:

- A ***dataset*** is a table that as a data structure is most naturally interpreted as an array of hashes, each hash representing a *row* in the table.

- A **data frame** is a table that as a data structure is most naturally interpreted as an array of hashes, each hash representing a *column* in the table. 

Mathematica uses datasets. S, R, and Python’s [pandas](https://pandas.pydata.org) use data frames.

The Raku system presented in this document uses datasets. Here is an example of a dataset with 3 rows and 2 columns:

```perl6
srand(128);
my $tbl=random-tabular-dataset(3,2).deepmap({ $_ ~~ Str ?? $_ !! round($_, 0.01) });
.say for |$tbl

# {controlling => -4.84, unlace => means}
# {controlling => 7.83, unlace => thyrotropin}
# {controlling => 11.92, unlace => parfait}
```

Here is how the corresponding data frame would have been structured:

```perl6
transpose($tbl)

# {controlling => [-4.84 7.83 11.92], unlace => [means thyrotropin parfait}]}
```

### Minimalist perspective

I do not want to make a special type (class) for datasets or data frames -- I want to use the standard Raku data structures. 
(At least at this point of my Raku data wrangling efforts.)

My reasons are:

1. The data can be picked up and transformed with typical, built-in commands.

    - Meaning, without the adherence to a certain data transformation methodology or dedicated packages.

2. Using standard, built-in structures is a type of “user interface” decision. 

3. The "user experience" can be achieved with- or provided by other transformation paradigms and packages.

Points 2 and 3 are, of course, consequences of point 1.

### Data structures in Raku

The data structures we focus on are datasets, and concretely in Raku we have the following dataset representations:

1. Array of hashes

1. Hash of hashes

1. Array of arrays

1. Hash of arrays

The order of the representations indicates their importance during the implementation of Raku data wrangling 
functionalities presented here: 

- Functionalities for the first two are primary and have unit tests

- Additionally, we accommodate the use of the last two.

When the framework and constellation of data wrangling functionalities matures all four data structures will have 
correct and consistent treatment.

### Target users

The target users are data scientists (full time, part time, and complete newbies) that want to:

- Do data wrangling of typical data science datasets with Raku

- Know that their Raku data wrangling efforts are relatively easily reproduced in other programming languages or systems

Alternatively, we can say that the target users are:

1. Data scientist impostors

1. Code baristas

1. Experienced data scientists who want to speed up their work

1. Data scientists who want to learn Raku

1. Raku programmers 

### Workflows considered

The following flow-chart encompasses the data transformations workflows we consider:

```mathematica
plWorkflows = ImageCrop@Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/ConceptualDiagrams/Tabular-data-transformation-workflows.png"]
```

![](https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/ConceptualDiagrams/Tabular-data-transformation-workflows.jpg)

**Remark:** We are going to refer to the methodology represented by the flow chart above as the 
***Data Transformation Workflows Model*** (DTWM). 
In this document we consider that methodology and the flow chart as synonyms.

Here are some properties of the methodology / flow chart:

- The flow chart is for tabular datasets, or for lists (arrays) or dictionaries (hashes) of tabular datasets

- In the flow chart only the data loading and summary analysis are not optional

- All other steps are optional

- Transformations like inner-joins are represented by the block “Combine groups”

- It is assumed that in real applications several iterations (loops) have to be run over the flow chart

In the world of the programming language R the orange blocks represent the so called Split-Transform-Combine pattern; 
see the article 
["The Split-Apply-Combine Strategy for Data Analysis"](https://www.jstatsoft.org/article/view/v040i01) 
by Hadley Wickham, [[HW1](https://www.jstatsoft.org/article/view/v040i01)].

**Remark:** R was (and probably still is) a fairly arcane programming language, so the explicit introduction of the 
Split-Transform-Combine pattern was of great help to R programmers. On the other hand, that pattern is fairly old 
and well known: it is inherent to SQL and it is met in parallel programming. 
(For example, see the WL function 
[`ParallelCombine`](http://reference.wolfram.com/mathematica/ref/ParallelCombine.html), 
[WRI2].)

Here is a simple use case scenario walk-through:

1. Obtain a tabular dataset from a warehouse.

2. Summarize and examine the dataset and decide that it does not have the desired form and content.

    - I.e. the data have to be wrangled.

3. Select only the columns that have the data of interest.

4. Filter the rows according to a certain operational criteria.

5. Split the rows -- i.e. group by the values in one of the columns.

6. Transform each group by combining the values of each column in some way.

    - For example, finding means or standard deviations of numerical columns.

7. Combine the transformed groups (into one “flat” tabular dataset.)

8. Reshape the data into long format and export it. 

The above list of steps is just one possible workflow. For more detailed examples and explanations see [AA1, AAv2, AAv3, AAv4].

### Fundamental operations

These operations are just basic:

- Column selection, renaming, and deletion

- Row selection and deletion

- Transposing

- Inner, left, and right joins

- Grouping by criteria

(Transposing of tabular or full-array data is also a basic functional programming operation.)

In data wrangling and data analysis the following three operations are non-basic, but still fundamental:

- [Cross tabulation](https://en.wikipedia.org/wiki/Contingency_table)

- [Long format conversion](https://en.wikipedia.org/wiki/Wide_and_narrow_data#Narrow)

- [Wide format conversion](https://en.wikipedia.org/wiki/Wide_and_narrow_data#Wide)

See [AA1, Wk1, Wk2, AAv1-AAv4] for more details.

**Remark:** The package 
["Data::Reshapers"](https://github.com/antononcube/Raku-Data-Reshapers), 
[AAp2], provides all functions mentioned in this sub-section.

------

## The size of the magic data does not determine how magic it is

... *aka* ***“Data acquisition of well known datasets into Raku”***.

We have to have access to some typical datasets used in Statistics teaching classes or in books and packages that 
exemplify Statistics concepts or explain related software designs and know-how. 
Also, of course, having those datasets would greatly benefit the data scientist impostors and the code baristas in 
their interaction with others.

The Raku package 
["Data::ExamplesDatasets"](https://github.com/antononcube/Raku-Data-ExampleDatasets),
[AAp3], provides functions for obtaining (relatively well known) example datasets. 
That package itself contains only datasets metadata -- the datasets are downloaded from the repository 
["Rdatasets"](https://github.com/vincentarelbundock/Rdatasets/), 
[VAB1]. 

Here we get a famous example dataset using a regex:

```perl6
my $iris=example-dataset(/iris $/);
$iris==>encode-to-wl
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1dhat25a5thym.png)

Here are the dimensions of the dataset we just imported:

```perl6
dimensions($iris)

# (150 6)
```

We can get the documentation URL for that dataset by using the function `item-to-doc-url`:

```perl6
use Data::ExampleDatasets::AccessData;
item-to-doc-url()<datasets::iris3>

# "https://vincentarelbundock.github.io/Rdatasets/doc/datasets/iris3.html"
```

Here we use WL to display dataset’s documentation:

```mathematica
WebImage[StringTrim@Out[145]]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1coskwrjgeumu.png)

### Summarizing the datasets collection

Here is a summary of the metadata dataset without the titles column and the columns with data and documentation URLs :

```perl6
records-summary(delete-columns(get-datasets-metadata(),<Title CSV Doc>), max-tallies=>12)

# +---------------------+------------------+--------------------+------------------+--------------------+--------------------+-----------------------+--------------------+---------------------+
# | Cols                | Package          | n_factor           | Item             | n_logical          | n_binary           | Rows                  | n_character        | n_numeric           |
# +---------------------+------------------+--------------------+------------------+--------------------+--------------------+-----------------------+--------------------+---------------------+
# | Min    => 1         | Stat2Data => 211 | Min    => 0        | salinity => 3    | Min    => 0        | Min    => 0        | Min    => 2           | Min    => 0        | Min    => 0         |
# | 1st-Qu => 3         | openintro => 206 | 1st-Qu => 0        | Grunfeld => 3    | 1st-Qu => 0        | 1st-Qu => 0        | 1st-Qu => 35          | 1st-Qu => 0        | 1st-Qu => 2         |
# | Mean   => 13.021203 | Ecdat     => 134 | Mean   => 1.290544 | housing  => 3    | Mean   => 0.030372 | Mean   => 1.940401 | Mean   => 3860.679656 | Mean   => 0.311175 | Mean   => 11.338109 |
# | Median => 5         | DAAG      => 121 | Median => 0        | smoking  => 3    | Median => 0        | Median => 0        | Median => 108         | Median => 0        | Median => 3         |
# | 3rd-Qu => 9         | AER       => 107 | 3rd-Qu => 2        | Mroz     => 3    | 3rd-Qu => 0        | 3rd-Qu => 2        | 3rd-Qu => 601.5       | 3rd-Qu => 0        | 3rd-Qu => 7         |
# | Max    => 6831      | MASS      => 87  | Max    => 64       | bmt      => 2    | Max    => 11       | Max    => 624      | Max    => 1414593     | Max    => 17       | Max    => 6830      |
# |                     | datasets  => 84  |                    | titanic  => 2    |                    |                    |                       |                    |                     |
# |                     | stevedata => 71  |                    | aids     => 2    |                    |                    |                       |                    |                     |
# |                     | carData   => 63  |                    | npk      => 2    |                    |                    |                       |                    |                     |
# |                     | boot      => 49  |                    | Hitters  => 2    |                    |                    |                       |                    |                     |
# |                     | HistData  => 46  |                    | Hedonic  => 2    |                    |                    |                       |                    |                     |
# |                     | HSAUR     => 41  |                    | goog     => 2    |                    |                    |                       |                    |                     |
# |                     | (Other)   => 525 |                    | (Other)  => 1716 |                    |                    |                       |                    |                     |
# +---------------------+------------------+--------------------+------------------+--------------------+--------------------+-----------------------+--------------------+---------------------+
```

Here is a histogram of the distribution of the number of rows across the examples datasets 
(getting the data in Raku, plotting the histogram with WL):

```perl6
select-columns(get-datasets-metadata(),"Rows")
==>transpose()
==>encode-to-wl()
```

```mathematica
Histogram[Log10@Normal[%["Rows"]], PlotRange -> All, PlotTheme -> "Detailed", FrameLabel -> {"lg of number of rows", "count"}, PlotLabel -> "Distribution of the number of rows of the example datasets"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1cx8e3kb02ggb.png)

The values of the plot above are logarithms with base 10. We can see that the majority of the datasets have between 10 and 1000 rows, which is also “confirmed” with the summary table above.

### Dataset identifiers

A dataset identifier is composed with a package name and an item name, separated by `"::"`. 
As it can be seen in the summary table above, a package can have multiple items, and the same item name might be found 
in multiple packages. Hence, with certain dataset specifications the function `example-dataset` gives a warning 
of multiple packages without retrieving any data:

```perl6
example-dataset(/ .* smoking .* /)

#ERROR: Found more than one dataset with the given spec: 
#ERROR: openintro::smoking	https://vincentarelbundock.github.io/Rdatasets/csv/openintro/smoking.csv
#ERROR: HSAUR::smoking	https://vincentarelbundock.github.io/Rdatasets/csv/HSAUR/smoking.csv
#ERROR: COUNT::smoking	https://vincentarelbundock.github.io/Rdatasets/csv/COUNT/smoking.csv(Any)
```

Here we retrieve a specific dataset using an identifier that consists of the package name and item name 
(separated with "::"):

```perl6
example-dataset('COUNT::smoking')
==>to-pretty-table()

# +---+--------+-----+------+-----+
# |   | smoker | age | male | sbp |
# +---+--------+-----+------+-----+
# | 1 |   1    |  34 |  1   | 131 |
# | 2 |   1    |  36 |  1   | 132 |
# | 3 |   0    |  30 |  1   | 122 |
# | 4 |   0    |  32 |  0   | 119 |
# | 5 |   1    |  26 |  0   | 123 |
# | 6 |   0    |  23 |  0   | 115 |
# +---+--------+-----+------+-----+
```

### Memoization

The main package function, `example-dataset`, has the adverb `keep`. 
If that adverb is given then `example-dataset` stores the web-retrieved data in the directory `XDG_DATA_HOME` 
and subsequently retrieves it from there. See 
["Freedesktop.org Specifications"](https://specifications.freedesktop.org) 
and [JS1] for more details on what is the concrete value of the environmental variable `XDG_DATA_HOME`.

------

## GYOD

... *aka* ***“Generate Your Own Datasets”***. *(Not “Get Your Own Dog”.)*

Instead of example datasets and dealing with potential problems, like, retrieving them, or just finding one, or two, 
or five that fit what we want to experiment with, why not simply generate random tabular datasets?! 

The function `random-tabular-dataset` of the package 
["Data::Generators"](https://modules.raku.org/dist/Data::Generators:cpan:ANTONOV), 
[AAp4], generates random tabular datasets using as arguments shape- and generators specs. 

### Completely random

Here is an example of a "completely random" dataset:

```perl6
srand(5);
random-tabular-dataset()
==>encode-to-wl
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0pa4gx1mt42s9.png)

### Specified column names and column value generators

We can also generate random datasets by specifying column names and column generators:

```perl6
srand(32);
random-tabular-dataset(10, <Task Story Epic>, generators=>{Task=>&random-pet-name, Epic=>&random-word})
==>encode-to-wl
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1od87l5ns165l.png)

**Remark:** The column “Story” does not have a user specified generator, hence a generator was (randomly) chosen for it.

### Using generating sets instead generating functions

Instead of using functions for the column generators we can use lists of objects: `random-tabular-dataset` generates 
automatically the corresponding sampling functions. Here we generate a random tabular dataset with $10$ rows, 
the columns “Eva”, “Jerry”, and “Project”, each column is assigned values from a small set of values:

```perl6
srand(1);
my $tblWork=random-tabular-dataset(10, 
                                   <Eva Jerry Project>, 
                                   generators=>{
                                         Eva=><Task Story Epic>, 
                                         Jerry=><Task Story Epic>, 
                                         Project=>(haikunate(tokenLength=>4) xx 4).List});
$tblWork==>encode-to-wl
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/16092phxsxor8.png)


------

## Data wrangling for dummies (a reference for the rest of us)

... *aka* ***“Espresso machine for code baristas”*** or ***“Data wrangling code generation”***.

Instead of expecting people to know how to use certain Raku packages and commands for data wrangling why not 
"just" generate the Raku code for them using natural language specifications? 
Good code baristas, then, can modify that code to client’s requirements.

Here we load the comprehensive translation package, [AAp8]:

```perl6
use DSL::Shared::Utilities::ComprehensiveTranslation;

(*"(Any)"*)
```

Here we define a command-string that specifies a data wrangling workflow:

```perl6
my $command='
load dataset starwars;
replace missing with "NA";
group by homeworld;
show counts'

# "load dataset starwars;replace missing with \"NA\";group by homeworld;show counts"
```

Here we translate that command to Raku:

```perl6
ToDSLCode('dsl target Raku::Reshapers;'~$command):code

# my $obj = example-dataset('starwars') ;
# $obj = $obj.deepmap({ ( ($_ eqv Any) or $_.isa(Nil) or $_.isa(Whatever) ) ?? \"NA\" !! $_ }) ;
# $obj = group-by( $obj, \"homeworld\") ;
# say \"counts: \", $obj>>.elems"*)
```

In case you are curious here is what the code above produces:

```perl6
my $obj = example-dataset('starwars') ;
$obj = $obj.deepmap({ ( ($_ eqv Any) or $_. isa(Nil) or $_. isa(Whatever) ) ?? "NA" !! $_ }) ;
$obj = group-by( $obj, "homeworld") ;
say "counts: ", $obj>>.elems

# counts: {Alderaan => 3, Aleen Minor => 1, Bespin => 1, Bestine IV => 1, Cato Neimoidia => 1, Cerea => 1, 
# Champala => 1, Chandrila => 1, Concord Dawn => 1, Corellia => 2, Coruscant => 3, Dathomir => 1, Dorin => 1,
# Endor => 1, Eriadu => 1, Geonosis => 1, Glee Anselm => 1, Haruun Kal => 1, Iktotch => 1, Iridonia => 1, Kalee => 1, 
# Kamino => 3, Kashyyyk => 2, Malastare => 1, Mirial => 2, Mon Cala => 1, Muunilinst => 1, NA => 10, Naboo => 11, 
# Nal Hutta => 1, Ojom => 1, Quermia => 1, Rodia => 1, Ryloth => 2, Serenno => 1, Shili => 1, Skako => 1, Socorro => 1,
# Stewjon => 1, Sullust => 1, Tatooine => 10, Toydaria => 1, Trandosha => 1, Troiken => 1, Tund => 1, Umbara => 1, 
# Utapau => 1, Vulpter => 1, Zolan => 1}
```

**Remark:**  For the same natural language command we can generate data wrangling code for other languages: 
Julia, Python, R, Wolfram Language.

The data wrangling natural language commands translator is based on DTWM described in the section 
"Datum fundamentum" above. For more extensive examples of its use see the presentation
["Multi-language Data-Wrangling Conversational Agent"](https://www.youtube.com/watch?v=pQk5jwoMSxs).

------

## Doing it like a Cro

... *aka* ***“Using a Cro-made web service for data wrangling code generation”***.

Thinking further about the professional lives of data scientist impostors and code baristas we can provide a Web service 
that translates natural language DSL into executable code. I implemented such Web service via the constellation of 
Raku libraries Cro; below we refer to it as the Cro Web Service (CWS). See the video [AAv5] for a demonstration.

### Getting code through the Web API

#### Data wrangling Raku code

Here is an example of using CWS through Mathematica’s web interaction function 
[`URLRead`](https://reference.wolfram.com/language/ref/URLRead.html), [WRI3]:

```mathematica
command = "dsl target Raku; 
include setup code; 
load the dataset iris; 
group by Species; show counts";

res = Import@URLRead[<|"Scheme" -> "http", 
  "Domain" -> "accendodata.net", "Port" -> "5040", "Path" -> "translate", 
  "Query" -> <|"command" -> command|>|>]

(*"{\"DSLTARGET\": \"Raku\",\"USERID\": \"\",\"CODE\": \"use Data::Reshapers;\\nuse Data::Summarizers;\\nuse Data::ExampleDatasets;\\n\\nmy $obj = example-dataset('iris') ;\\n$obj = group-by( $obj, \\\"Species\\\") ;\\nsay \\\"counts: \\\", $obj>>.elems\",\"SETUPCODE\": \"use Data::Reshapers;\\nuse Data::Summarizers;\\nuse Data::ExampleDatasets;\\n\",\"STDERR\": \"\",\"DSL\": \"DSL::English::DataQueryWorkflows\",\"DSLFUNCTION\": \"proto sub ToDataQueryWorkflowCode (Str $command, Str $target = \\\"tidyverse\\\", |) {*}\",\"COMMAND\": \"dsl target Raku; include setup code; load the dataset iris; group by Species; show counts\"}"*)
```

Here we convert the JSON output from CWS and display it in a tabular form:

```mathematica
ResourceFunction["GridTableForm"][List @@@ ImportString[res, "JSON"], TableHeadings -> {"Key", "Value"}]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0yjdl2h650tsp.png)

#### Latent semantic analysis R code

CWS provides code for other programming languages and types of workflows. 
Here is an example with Latent Semantic Analysis (LSA) workflow code in R:

```mathematica
command = "USER ID BaristaNo12;
dsl target R::LSAMon; 
include setup code;use aAbstracts; 
make document term matrix;apply LSI functions IDF, None, Cosine; 
extract 40 topics using method SVD;echo topics table" // StringTrim;

res = Import@URLRead[<|"Scheme" -> "http", 
  "Domain" -> "accendodata.net", "Port" -> "5040", "Path" -> "translate", 
  "Query" -> <|"command" -> command|>|>];

ResourceFunction["GridTableForm"][List @@@ ImportString[res, "JSON"], TableHeadings -> {"Key", "Value"}]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1x6v7pe98sefw.png)

**Remark:** As it can be seen above, CWS can be given user identifiers, which allows for additional personalization 
of the parsing and interpretation results.

### Getting code “on the spot“

Here is a diagram that shows the components of the system utilized through 
[Apple's Shortcuts](https://support.apple.com/guide/shortcuts/welcome/ios):

```mathematica
ImageCrop[Import["https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/DSL-Web-Service-via-Cro-with-WE-QAS-Shortcuts.png"]]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0ak5mpudq4tcm.png)

In that diagram we can trace the following Shortcuts execution steps:

1. In Mathematica notebook (or VS Code file) invoke Shortcuts

2. Using Siri’s speech-to-text functionality enter text

3. Shortcuts invokes CWS

4. The result is returned in JSON form to Shortcuts

5. Shortcuts examines the CWS result 

6. If the parsing is successful

    1. Shortcuts issues a notification

    2. Makes the corresponding code is available in the clipboard

7. If the parsing is not successful

    1. Shortcuts issues a notification

    2. Shows the full JSON output from CWS

(The video recording 
["Doing it like a Cro (Raku data wrangling Shortcuts demo)"](https://www.youtube.com/watch?v=wS1lqMDdeIY), 
[AAv5], demonstrates the steps above.)

------

## The one way to do it

... *aka* ***“Leveraging the universality of natural language and the simplicity of the data wrangling model”***.

The principle “there is more than one way to do it” is often found to be too constraining or too blocking. 
In my experience, code baristas and IT technology managers prefer one way of doing things. 
Also, not to be too exposed to the Paradox of choice too much. 
If anything, voluntary simplicity and predictable mediocrity are preferred. 
Which is fine, since we have a solution that serves well the simple minded when they are single minded. 

Here are the elements of the proposed solution:

- English-based Domain Specific Language (DSL) 

- DSL translators for most popular data science languages

- Simple to learn and keep in mind data wrangling methodology

- The generated programming codes are expected to be "good starting points"

    - I.e. additionally tweaked by users according to desired outcomes.

We can rephrase and summarize the above as:

- Rapid specification of data wrangling workflows is achieved by using an abstract grammar representation
  of data wrangling natural language commands, which allows different
  data wrangling implementations to be made for each programming language of interest.

Let us also point out that the proposed DSL data wrangling solution does not cover *all possible* 
data wrangling undertakings, but I claim that for tabular data collections we can streamline any complicated 
data wrangling to a large degree. 

To rephrase, I assume that 60÷80% of your data wrangling workflows can be handled with the solution. 
(Yeah, YMMV and Pareto principle combined.)

### [Appeal to authority](https://en.wikipedia.org/wiki/Argument_from_authority)

Of course, it is better to justify the approach by pointing out how it agrees with statements of certain authoritative 
figures. Say, Larry Wall and Lao Tze.

### Larry Wall

In Larry Wall's interview in the book "Masterminds of programming", [FB1], Larry describes Perl 6 (Raku) as:

> [...]  It will come with knobs to adjust its many different dimensions, 
> including the ability to hide all those dimensions that you aren’t currently interested in thinking about, 
> depending on which paradigm appeals to you to solve the problem at hand.

The DSL data wrangling solution adheres to Larry's statement:

- Its many dimensions are the operators and syntactical elements of the different programming languages 
  and related libraries 

- The hiding of those dimensions is achieved by using natural language DSL specifications that generate executable code
  for those dimensions 

- The grammar-actions design inherent to Raku provides the dimension hiding ability

### Lao Tze

The approach can be additionally justified by referring to Lao Tze’s 
["Tao Te Ching"](https://en.wikipedia.org/wiki/Tao_Te_Ching),
[Book 1, Chapter 11](https://www.egreenway.com/taoism/ttclz11.htm).

Here is a translation of that chapter:

> The thirty spokes unite in the one nave;   
but it is on the empty space for the axle, that the use of the wheel depends.  
Clay is fashioned into vessels;    
but it is on their empty hollowness, that their use depends.  
The door and windows are cut out from the walls to form an apartment;    
but it is on the empty space within, that its use depends.  
Therefore, what has a positive existence serves for profitable adaptation,   
and what has not that for actual usefulness.  
> ~ Translated by James Legge, 1891, Chapter 11

Here are some points that clarify how the DSL data wrangling solution can be seen as a manifestation of the
outlined principle:

- We use a slang not just for the coolness of pronouncing its words, but because of the things 
  we do not have to explain by using the slang.

  - Meaning, the "cool words" shape the slang, but slang's usefulness comes from what is not in its sentences.

- The natural language DSL for data wrangling allows for fast specification of data wrangling workflows
  in different programming languages because:
 
  - There is an abstract, largely universal model of the data wrangling workflows. (DTWM explained above.)
  
  - Natural language hides a lot of programming language semantic and syntactical details. 
  
- The "negative existence" ("emptiness") is provided by the natural language utilization: 
  no programming languages syntax, arcaneness, and unnaturalness.

- The "thirty spokes of wheel" are represented by the multiple programming languages and libraries 
  the DSL is translated to.

**Remark:** I used similar justification for translating Mathematica expressions for High Performance Fortran (HPF).   
See [AA3, AAn3]. Basically, the "negative existence" of Mathematica functional programming expressions and 
HPF are kind of similar and that allowed to write a translator from Mathematica to HPF.

-------

## Heavy-brained instead of lighthearted 

... *aka* ***“The data wrangling user guide is quite boring”***.

I had the intent to publish this document in ["Raku Advent Calendar"](https://raku-advent.blog).
(Did not happen.)

One of the 
["Raku Advent Calendar"](https://raku-advent.blog)
organizers after seeing one of my very initial drafts asked me to do something more "lighthearted." 

(I am not going to say names, but I will say that he uses the initials "JJ" and wrote a Raku cook-book.)

Well, this document is my lighthearted version of what I wanted to say about my efforts to endow the Raku ecosystem 
with data wrangling capabilities that resemble approaches in other, well known systems. 

The original, "heavy-brained" version is [AA1]. The heavy-brained version compiles all explanations
and usage examples given in the README files of the Raku packages [AAp1 ÷ AAp4].

**Remark:** There is a related
["Advent of Raku 2021"](https://github.com/codesections/advent-of-raku-2021) 
project.
(Related to the more general
["Advent of Code"](https://adventofcode.com/2021).)

-------

## A tale about the Wolf, the Ram, and the Raccoon

... *aka* ***“Connecting Raku to Mathematica”***.

The short version of the tale is the following:

> The Wolf, the Ram, and the Raccoon talked through a socket. The socket was provided by [ZeroMQ](https://zeromq.org). (The end.)

The long version is given in 
["Connecting Raku to Mathematica"](https://github.com/antononcube/RakuForPrediction-book/blob/main/Articles/Connecting-Mathematica-and-Raku.md), 
[AA2]. 
Using [ZeroMQ](https://zeromq.org) and a 
[Raku-to-WL serializer](https://github.com/antononcube/Raku-Mathematica-Serializer), 
[AAp6], we can execute Raku commands in Mathematica notebooks. 

Why is this useful? Mathematica is the most powerful mathematical software system and has one of the oldest,
most mature notebook solutions. Hence, connecting Raku to Mathematica allows facilitating and leveraging
some interesting synergies between the two.

Here are some Raku-centric implications of the last statement:

- Using notebooks facilitates interactive development or research (with Raku)

- The ability to visualize, and plot results (derived with Raku)

- Combining evaluations with other programming languages
 
  - That can be run in Mathematica notebooks: Python, R, Julia, etc.

- Literate programming

- Comparative testing of results correctness

    - Verifying that Raku new implementations do "the right thing"

    - Comparison with other languages "doing the same thing"

Here is an example of using the serializer command `encode-to-wl` to convert a tabular dataset generated in Raku 
with the command `random-tabular-dataset` and displaying it in Mathematica as a "native" 
[`Dataset`](https://reference.wolfram.com/language/ref/Dataset.html) 
object, [WRI1]:

```perl6
say to-pretty-table(random-tabular-dataset(3,5))

# +---------------------+-----------+-----------+------------+-------------+
# |       appalled      |  slammer  |    aura   | anglophile | accompanied |
# +---------------------+-----------+-----------+------------+-------------+
# |     salaciously     | 14.966161 | 91.331654 | 16.961175  |  15.061875  |
# | unconscientiousness | -1.617324 |  7.872224 | -4.440601  |   7.161489  |
# |         fey         |  8.157817 | 65.334798 |  2.785762  |   7.945464  |
# +---------------------+-----------+-----------+------------+-------------+
```

```perl6
random-tabular-dataset(3,5)==>encode-to-wl()
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/06rfvy2fpqecy.png)

**Remark:** In this document I use Mathematica and Wolfram Language (WL) as synonyms. 
If we have to be precise, we should say something like “Mathematica is the software system (product) and 
WL is the backend programming language in the software system.” Or something similar.

------

## Too green to be Red

*... aka* ***“Didn’t implement data wrangling for Red and couldn't install it”***.

I am using standard Raku data structures in this document. It would have been nice to show examples of 
data wrangling using the Raku package 
[Red](https://github.com/FCO/Red), 
[[FCO1](https://modules.raku.org/dist/Red:cpan:FCO)]. 
My reasons for not doing it can be summarized as "too much immaturity everywhere." More precisely:

- I did not have time to implement Red actions to the module 
["DSL::English::DataQueryWorkflows"](https://github.com/antononcube/Raku-DSL-English-DataQueryWorkflows), [AAp6]

    - I also have not sufficiently “understood” `Red`.

- I tried to install `Red` a few times and failed . 

    - I assume it is me, but it might be also Raku, or `Red`, or my quick jump onto 
      [macOS Monterey](https://www.apple.com/macos/monterey/)...

------

## Making the future more evenly distributed

... *aka* ***“Immediate and long term future plans”***.

Of course, by making and facilitating data scientist impostors and code baristas we make
more evenly distributed the future that has already arrived. 

Here are some of my future plans on data acquisition and data wrangling using Raku 
that will distribute the arrived future even more evenly:

- Data acquisition functionalities implementations.

- Data acquisition conversational agent implementation.

    - I implemented such an agent in WL, [AAv5].

    - It would be both nice and interesting to make a Raku implementation.

- More extensive unit tests.

    - So far the unit tests are at bare minimum.

- Performance investigations and improvements.

    - At this point I have not considered how fast or slow are the Raku data wrangling functions.

    - Of course, that has to be investigated in sufficient detail.

- Natural language commands translations for Object Relational Mapping system(s) like `Red`, [FCO1].

- Convert Mathematica data wrangling unit tests into Raku tests.

   - That applies to both the code generation from natural language commands tests and data wrangling operation tests. 
   
**Remark:** The quote paraphrased above and 
[attributed to William Gibson](https://quoteinvestigator.com/2012/01/24/future-has-arrived/)
is:
> The future has arrived — it's just not evenly distributed yet.

--------

## Setup

*This section has Mathematica and Raku code for running the document examples in a Mathematica notebook.*

### Load Mathematica packages

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuMode.m"]
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuEncoder.m"]
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuDecoder.m"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/19g68tfvhmqnr.png)

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/DSLMode.m"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0yzyhr88sy21g.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1wmxsuqlgnswh.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0ofsa237g19ec.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0dfjtj85b54nu.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1tq8z9o4nkf07.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1ot545n8z1fd0.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1l6gk8lswm9h6.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1loegav1evtwf.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/0owv72jay6frn.png)

### Start Raku process

```mathematica
KillRakuProcess[]
```

```mathematica
SetOptions[RakuInputExecute, Epilog -> (FromRakuCode[#, DisplayFunction -> (Dataset[#, MaxItems -> {Automatic, All}] &)] &)];
StartRakuProcess["Raku" -> "/Applications/Rakudo/bin/raku"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Introduction-to-Data-Wrangling-with-Raku/1rrcnu0nl91a6.png)

```mathematica
RakuMode[]
```

### Load Raku packages

```perl6
use Data::Generators;
use Data::Reshapers;
use Data::Summarizers;
use Data::ExampleDatasets;
use Haikunator;
use Mathematica::Serializer;

(*"(Any)"*)
```

```perl6
use DSL::Shared::Utilities::ComprehensiveTranslation;

(*"(Any)"*)
```

## References

### Articles, books

[AA1] Anton Antonov, "Data wrangling in Raku", (2021), RakuForPrediction-book at GitHub.

[AA2] Anton Antonov, 
["Connecting Raku to Mathematica"](https://github.com/antononcube/RakuForPrediction-book/blob/main/Articles/Connecting-Mathematica-and-Raku.md), 
(2021), 
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book).

[AA3] Anton Antonov,
["Translating Mathematica expressions to High Performance Fortran"](https://library.wolfram.com/infocenter/MathSource/5143/HPF.pdf),
(1999),
HiPer'99, Tromsoe, Norway.

[AAn3] Anton Antonov, 
["Translating Mathematica expression to High Performance Fortran"](https://www.wolframcloud.com/translating-mathematica-expression-to-high-performance-fortran--2018-10-10qgpsl/), 
from the Notebook Archive (2004), 
https://notebookarchive.org/2018-10-10qgpsl .

[CARH1] Charles Antony Richard Hoare,
(1980),
["The emperor's old clothes"](https://dl.acm.org/doi/10.1145/1283920.1283936)
ACM Turing award lectures January 2007 Year Awarded: 1980.
https://doi.org/10.1145/1283920.1283936.

[HW1] Hadley Wickham, 
["The Split-Apply-Combine Strategy for Data Analysis"](https://www.jstatsoft.org/article/view/v040i01), 
(2011), 
[Journal of Statistical Software](https://www.jstatsoft.org).

[FB1] Federico Biancuzzi and Shane Warden, 
(2009),
[Masterminds of Programming: Conversations with the Creators of Major Programming Languages](https://www.oreilly.com/library/view/masterminds-of-programming/9780596801670/). 
ISBN 978-0596515171. See page. 385.

### Functions, packages, repositories

[AAp1] Anton Antonov, 
[Data::Reshapers](https://modules.raku.org/dist/Data::Reshapers:cpan:ANTONOV), 
(2021), 
[Raku Modules](https://modules.raku.org).

[AAp2] Anton Antonov, 
[Data::Summarizers](https://github.com/antononcube/Raku-Data-Summarizers), 
(2021), 
[Raku Modules](https://modules.raku.org).

[AAp3] Anton Antonov, 
[Data::ExampleDatasets](https://github.com/antononcube/Raku-Data-ExampleDatasets), 
(2021), 
[Raku Modules](https://modules.raku.org).

[AAp4] Anton Antonov, 
[Data::Generators](https://modules.raku.org/dist/Data::Generators:cpan:ANTONOV), 
(2021), 
[Raku Modules](https://modules.raku.org).

[AAp5] Anton Antonov, 
[Mathematica::Serializer Raku package](https://github.com/antononcube/Raku-Mathematica-Serializer), 
(2021), 
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov, 
[DSL::English::DataQueryWorkflows Raku package](https://github.com/antononcube/Raku-DSL-English-DataQueryWorkflows), 
(2020-2021), 
[GitHub/antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov, 
[DSL::English::DataAcquisitionWorkflows Raku package](https://github.com/antononcube/Raku-DSL-English-DataAcquisitionWorkflows), 
(2021), [GitHub/antononcube](https://github.com/antononcube).

[AAp8] Anton Antonov, 
[DSL::Utilities::ComprehensiveTranslation](https://github.com/antononcube/Raku-DSL-Shared-Utilities-ComprehensiveTranslation), 
(2020-2021), 
[GitHub/antononcube](https://github.com/antononcube).

[AAr1] Anton Antonov,
[Data Acquisition Engine project](https://github.com/antononcube/Data-Acquisition-Engine-project),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[FCOp1] Fernando Correa de Oliveira, 
[Red](https://modules.raku.org/dist/Red:cpan:FCO), 
(last updated 2021-11-22), 
[Raku Modules](https://modules.raku.org).

[JSp1] Jonathan Stowe, 
[XDG::BaseDirectory](https://modules.raku.org/dist/XDG::BaseDirectory:cpan:JSTOWE), 
(last updated 2021-03-31), 
[Raku Modules](https://modules.raku.org).

[WRI1] Wolfram Research, (2014), 
[Dataset](https://reference.wolfram.com/language/ref/Dataset.html), 
Wolfram Language function, https://reference.wolfram.com/language/ref/Dataset.html (updated 2021).

[WRI2] Wolfram Research, (2008), 
[ParallelCombine](https://reference.wolfram.com/language/ref/ParallelCombine.html), 
Wolfram Language function, https://reference.wolfram.com/language/ref/ParallelCombine.html (updated 2010).

[WRI3] Wolfram Research, (2016), 
[URLRead](https://reference.wolfram.com/language/ref/URLRead.html), 
Wolfram Language function, https://reference.wolfram.com/language/ref/URLRead.html.

### Presentation video recordings

[AAv1] Anton Antonov, 
["Raku for Prediction"](https://conf.raku.org/talk/157), 
(2021), 
[The Raku Conference 2021](https://conf.raku.org).

[AAv2] Anton Antonov, 
["Multi-language Data-Wrangling Conversational Agent"](https://www.youtube.com/watch?v=pQk5jwoMSxs), 
(2020), 
Wolfram Technology Conference 2020, 
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv3] Anton Antonov, 
["Data Transformation Workflows with Anton Antonov, Session #1"](https://www.youtube.com/watch?v=iXrXMQdXOsM), 
(2020), 
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv4] Anton Antonov, 
["Data Transformation Workflows with Anton Antonov, Session #2"](https://www.youtube.com/watch?v=DWGgFsaEOsU), 
(2020), 
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv5] Anton Antonov, 
["Doing it like a Cro (Raku data wrangling Shortcuts demo)"](https://youtu.be/wS1lqMDdeIY), 
(2021), 
[YouTube/Anton.Antonov.Antonov](https://www.youtube.com/channel/UC5qMPIsJeztfARXWdIw3Xzw).

[AAv6] Anton Antonov, 
["Multi language Data Acquisition Conversational Agent (extended version)"](https://www.youtube.com/watch?v=KlEl2b8oxb8), 
(2021),
[YouTube/Anton.Antonov.Antonov](https://www.youtube.com/channel/UC5qMPIsJeztfARXWdIw3Xzw).
