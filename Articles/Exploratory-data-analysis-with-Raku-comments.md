

------

## Reddit post comment

Related WordPress blog posts:

- ["Age at creation for programming languages stats"](https://rakuforprediction.wordpress.com/2024/05/25/age-at-creation-for-programming-languages-stats/) (English)

- ["Статистики върху възрастта на създателите на езици за програмиране"](https://rakuforprediction.wordpress.com/2024/05/24/статистики-върху-възрастта-на-създат/)  (Bulgarian)

Notebooks:

- ["Computational exploration for the ages of programming language creators dataset"](https://community.wolfram.com/groups/-/m/t/3180327) (Mathematica)

- ["Age at creation for programming languages stats"](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Age-at-creation-for-programming-languages-stats.ipynb) (Jupyter)


-------

## [EDN comment by *raiph*](https://www.reddit.com/r/rakulang/comments/1d079vg/comment/l5nh36j)


As I think you know, what you've been doing caught my attention a few years ago and has remained compelling since then.

However, I've never found the "raku for prediction" or "computational conversational agents" labels resonated for me. I ignored that because it seemed more interesting to focus on what you were actually doing rather than being distracted by the labels you were applying to the broader field you are working within.

Today I decided it would be helpful to come up with my own label for what I see, a label that is deliberately as narrow as makes sense to me to encompass the last couple years worth of your blog posts. I googled for a few minutes and something quickly crystalized: EDNs, Exploratory Data Notebooks.

Are you aware of the EDN term/market? If you are -- or even if you're not but are willing to spend a few minutes getting up to speed on the field by browsing a suitable google search -- I'd love to hear your thoughts about some questions:

- Would you agree that AI features/data are a natural drop-in addition for EDNs, so don't alter the appropriateness of EDNs?

- Do you feel EDNs would be a good label for most if not all of the Raku blog posts you've published in the last couple years? If you think EDNs mostly captures what you've been publishing, yet misses key elements, what are those elements? If EDNs so seriously misses the mark you think it would be a poor label, why do you feel that way?

- Are you aware of ExplainED and can you imagine it being another useful piece of the puzzle?

- Can you imagine EDNs using the same techniques you've been using that explore the EDN field (the tech) and/or market?

-------

## Answer to *raiph*

### Sure, it is just EDN

Exploratory Data Notebooks (EDN) is a fairly valid -- and narrow -- way of characterizing what I am doing in Raku's Ecosystem (RE).

About the validity:

- My [latest video](https://youtu.be/YCnjMVSfT8w) showcases my data wrangling, data summarization, graphics work RE exactly as an EDN effort.

- The script ["raku-data-science-install.sh"](https://github.com/antononcube/RakuForPrediction-book/blob/main/scripts/raku-data-science-install.sh)
shows that:
  - 60%+ of the packages I have published in Raku's Ecosystem are about Data Science
  - ≈ 30% are about Data Wrangling (hence EDN)
 
Why do I think EDN is too narrow:

- My notebook-centric work is not needed for EDN.
  - It is first and foremost about using Large Language Models (LLMs) with ease.
  - Brian Duggan's "Jupyter::Kernel" is fairly sufficient for EDN.
- With Raku, I more interested in facilitating the specification of computational workflows.
  - Not performing/executing those workflows with Raku.
  - (Next section elaborates on that)

### Meta language

I started Raku *for* Prediction (R4P) in order to simplify Machine Learning (ML) teachings and explanations.
I am used and using Raku to facilitate or speed-up the workflows for making predictions -- 
predictions *with* Raku can be made for a fairly small set of problems in Machine Learning or Scientific Computing. 

To be more precise R4P solves the following problem:

> Specifications of computational workflows using Natural Domain Specific Languages (DSLs) are translated
> into concrete executable code of different programming languages, libraries, or systems. 
> (Like, Wolfram Language, R, Python, Julia.)
 
#### Example 1

For example, the following Raku code produces a Wolfram Language (WL) workflow/pipeline for Epidemiological Modeling simulations:

```raku
create with the model susceptible exposed infected two hospitalized recovered;
assign 100000 to the susceptible population;
set infected normally symptomatic population to be 0;
set infected severely symptomatic population to be 1;
assign 0.56 to contact rate of infected normally symptomatic population;
assign 0.58 to contact rate of infected severely symptomatic population;
assign 0.1 to contact rate of the hospitalized population;
simulate for 240 days;
plot populations results;
```

```mathematica
ECMMonUnit[SEI2HRModel[t]] ⟹
ECMMonAssignInitialConditions[<|SP[0] -> 100000|>] ⟹
ECMMonAssignInitialConditions[<|INSP[0] -> 0|>] ⟹
ECMMonAssignInitialConditions[<|ISSP[0] -> 1|>] ⟹
ECMMonAssignRateRules[<|β[INSP] -> 0.56|>] ⟹
ECMMonAssignRateRules[<|β[ISSP] -> 0.58|>] ⟹
ECMMonAssignRateRules[<|β[HP] -> 0.1|>] ⟹
ECMMonSimulate["MaxTime" -> 240] ⟹
ECMMonPlotSolutions[ "Stocks" -> __ ~~ "Population"]
```

#### Example 2

Here is another example for creating a Latent Semantic Analysis (LSA) pipeline in R:

```raku
DSL MODULE LSAMon;
create from textHamlet;
make document term matrix with stemming FALSE and automatic stop words;
apply LSI functions global weight function IDF, local term weight function TermFrequency, normalizer function Cosine;
extract 12 topics using method NNMF and max steps 12 and 20 min number of documents per term;
show topics table with 12 terms;
show thesaurus table for king, castle, denmark;
```


```r
LSAMonUnit(textHamlet) %>%
LSAMonMakeDocumentTermMatrix( stemWordsQ = FALSE, stopWords = NULL) %>%
LSAMonApplyTermWeightFunctions(globalWeightFunction = "IDF", localWeightFunction = "None", normalizerFunction = "Cosine") %>%
LSAMonExtractTopics( numberOfTopics = 12, method = "NNMF",  maxSteps = 12, minNumberOfDocumentsPerTerm = 20) %>%
LSAMonEchoTopicsTable(numberOfTerms = 12) %>%
LSAMonEchoStatisticalThesaurus(words = c("king", "castle", "denmark"))
```

#### Not with Raku

Note that Raku is not equipped to Epidemiological Modeling simulations or LSA.
Yes, with Raku we can outsource the computations to Python/R/WL or LLMs, but we currently cannot make those computations with Raku. 

#### Using Data Wrangling workflows

I thought the teaching of ML is going to be simple with those natural DSLs. Turned out that audience / students are
too concerned and focuses on "why is this working" / "what is the concrete code I need to type in" etc.
Basically, interested into the concrete know-how not in the general mathematical principles. 

So, at that point I decided to program and use the construction of Data Wrangling pipelines in R4P project. 

### The narrowing instinct

I understand the "narrowing instinct" to compact, communicate, or understand efforts like mine into 
a narrower scope, like, exploratory data analysis, or some other single paradigm point of view.

This happens consistently for Mathematica and, more generally, for the efforts of Wolfram Research, Inc. (WRI).
Stephen Wolfram and WRI are often asked questions like "Why don't you make Mathematica more Data Science friendly?"
and similar.

Simply (and cynically) put, Mathematica is made for brilliant physicists, not for people who try to sell their skills in
USA's IT job market. Physicists tend to have a very wide and diverse mathematical education, and for them mathematics 
and computer science are "just tools." Most IT professionals want to feel or sell themselves as experts -- 
that is not a scientific pre-disposition.

So, similarly, with R4P (damn I hate that acronym now!) I try to address a wide scope of computational workflows 
that include not just Data Science and Machine Learning, but also Scientific Computing and Mathematical Modeling.

At this point Raku can be only (barely) used to *do* Data Science and Machine Learning. Most Raku users / developers / practitioners
are not interested in doing Science and Mathematical Modeling with Raku. Hence, the inclination to see efforts like R4P
as data-centric in one way or the other -- data manipulation and wrangling is universally understood. 





