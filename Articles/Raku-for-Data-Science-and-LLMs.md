# Raku for Data Science and Large Language Models

## Introduction 

In this document has (overlapping) lists of Raku packages for doing different types of workflows 
in Data Science (DS) and Large Language Models (LLM) utilization.

------

## Code generation

We can say that the most frequently used Machine Learning (ML) workflows are in:
- Classification
- Latent Semantic Analysis (LSA),
- Regression
- Recommendations

In the broader field of Data Science (DS) we also add Data Wrangling.

Each of these ML or DS sub-fields has it own Domain Specific Language (DSL).

There is a set of Raku packages for facilitate the creation of Data Science workflows in _other_ programming languages.
(Julia, Python, R, Wolfram Language.)

The grammar-based ones have the "DSL::" prefix -- see, for example, ["DSL::English::*"](https://raku.land/?q=DSL%3A%3AEnglish) at [raku.land](https://raku.land/).

The LLM based one is ["ML::NLPTemplateEngine"](https://raku.land/zef:antononcube/ML::NLPTemplateEngine).

------

## Data wrangling 

Data wrangling, summarization, and generation:

- ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers)
- ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers)
- ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators)

Generation of data wrangling workflows code:

- ["DSL::English::DataQueryWorkflows"](https://raku.land/zef:antononcube/DSL::English::DataQueryWorkflows)

------

## Exploratory Data Analysis

At this point Raku is fully equipped to Exploratory Data Analysis (EDA) over small to moderate size datasets.
(E.g. less than 100,000 rows.)

Here are the elements for EDA:

- Easy data ingestion 
  - Of files of different types and different kinds of locations
  - See ["Data::Importers"](https://raku.land/zef:antononcube/Data::Importers)
- Data wrangling
  - See the previous section.
- Visualization
  - ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3)
  - ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts)
- Extensive support data.
  - Ready to do computations with
  - See ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geograohics)
- Interactive development environment(s)
  - These are "notebook solutions" 
  - ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel)
  - ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook)
  - ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/)

------

## Machine Learning & Statistics

- Unsupervised learning
  - ["ML::Clustering"](https://raku.land/zef:antononcube/ML::Clustering)
  - ["ML::TriesWithFrequencies"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies)
  - ["ML::AssociationRuleLearning"](https://raku.land/zef:antononcube/ML::AssociationRuleLearning)
  - ["Math::Nearest"](https://raku.land/zef:antononcube/Math::Nearest)
- Supervised learning
  - ["ML::ROCFunctions"](https://raku.land/zef:antononcube/ML::ROCFunctions)
- Fitting / regression
  - ["Math::Fitting"](https://raku.land/zef:antononcube/Math::Fitting)
- Distributions 
  - ["Statistics::Distributions"](https://raku.land/zef:antononcube/Statistics::Distributions)
- Outliers
  - ["Statistics::OutlierIdentifiers](https://raku.land/cpan:ANTONOV/Statistics::OutlierIdentifiers)

-----

## LLM support

### LLM services access

Main or well known LLM-services providers (OpenAI, Google) have dedicated LLM packages:

- ["WWW::OpenAI"](https://raku.land/zef:antononcube/WWW::OpeanAI)
- ["WWW::PaLM"](https://raku.land/zef:antononcube/WWW::PaLM)
- ["WWW::Gemini"](https://raku.land/zef:antononcube/WWW::Gemini)
- ["WWW::MistralAI"](https://raku.land/zef:antononcube/WWW::MistralAI)
- ["WWW::LLaMA"](https://raku.land/zef:antononcube/WWW::LLaMA)

All these LLM packages are loaded, ready to use in Raku chatbooks, (notebooks of ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook).)

### LLM workflows

There is a set of packages that facilitates the creation of workflows that are "provider independent":

- ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions)
- ["LLM::Prompts"](https://raku.land/zef:antononcube/LLM::Prompts)
- ["LLM::RetrievalAugmentedGeneration"](https://raku.land/zef:antononcube/LLM::RetrievalAugmentedGeneration)

"LLM::Functions" and "LLM::Prompts" are loaded, ready to use in Raku chatbooks.

----

## Literate programming

The Raku package 
["Text::CodeProcessing"](https://raku.land/zef:antononcube/Text::CodeProcessing) 
can be used to "execute" computational documents in 
formats of Markdown, Org-mode, Pod6.

The Jupyter Raku-kernel packages:
["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel) and
["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook),
provide cells for rendering the output of LaTeX, HTML, Markdown, Mermaid-JS code or specifications.

The package 
["Markdown::Grammar"](https://raku.land/zef:antononcube/Markdown::Grammar)
can be used in notebook conversion workflows; see [AA1].

------

## References

### Articles, blog posts

[AA1] Anton Antonov,
["Notebook transformations"](https://rakuforprediction.wordpress.com/2024/02/17/notebook-transformations/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["Omni-slurping with LLMing"](https://rakuforprediction.wordpress.com/2024/03/26/omni-slurping-with-llming/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov,
["Age at creation for programming languages stats"](https://rakuforprediction.wordpress.com/2024/05/25/age-at-creation-for-programming-languages-stats/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).


### Videos

[AAv1] Anton Antonov,
["Exploratory Data Analysis with Raku"](https://www.youtube.com/watch?v=YCnjMVSfT8w),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov,
["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).


[AAv3] Anton Antonov,
["Raku RAG demo"](https://www.youtube.com/watch?v=JHO2Wk1b-Og),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction). 

[AAv4] Anton Antonov,
["Robust LLM pipelines (Mathematica, Python, Raku)"](https://www.youtube.com/watch?v=QOsVTCQZq_s),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).  