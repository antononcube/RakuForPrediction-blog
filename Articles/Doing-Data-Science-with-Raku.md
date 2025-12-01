# Doing Data Science with Raku

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)  
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)  
October 2024   
November 2025

## Introduction 

This document provides an overview of Raku packages, as well as related documents and presentations, 
for doing Data Science (DS) using Raku.

This simple mnemonic can be utilized for what Data Science (DS) is while this document is being read:

> Data Science = Programming + Statistics + *Curiosity*

**Remark:** By definition anytime we deal with data we do Statistics.

We are mostly interested in DS workflows -- the Raku facilitation of using 
Large Language Models (LLMs) is seen here as:

- An (excellent) supplement to standard, non-LLM DS workflows facilitation
- A device to use -- and solve -- Unsupervised Machine Learning (ML) tasks 

(And because of our strong curiosity drive, we are completely not shy using LLMs to do DS.)

### What is needed to do Data Science?

Here is a wordier and almost technical definition of DS:

> ***Data Science*** is the process of exploring and summarizing data, uncovering hidden patterns, building predictive models, and creating clear visualizations to reveal insights.
It is analytical work analysts, researchers, or scientists would do over raw data in order to understand it and utilize those insights. 

**Remark:** "Utilize insights" would mean "machine learning" to many.

This is the general workflow (or loop) for doing DS:

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Data-science-over-a-small-movie-dataset-Part-1/Data-analysis-cycles.jpg)

Assume you have a general purpose language which is very good at dealing with text and a package ecosystem with *well maintained* part dedicated to doing various Web development tasks and workflows. (I.e. trying to re-live Perl's glory days.) What new components the ecosystem of that programming language has to be endowed with in order to make it useful for doing Data Science?

The list below gives such components. They are ranked by importance (most important first), but all are important -- i.e. each is "un-skippable" or indispensable.

- Data import and export
- Data wrangling facilitation
- Statistics for data exploration
- Machine Learning algorithms (both unsupervised and supervised)
- Data visualization facilitation
- Interactive computing environment(s)
- Literate programming

Additional, really nice to have, but not indispensable components are:

- Data generation and retrieval
- Interfaces to other programming languages and ecosystems
- Interactive interfaces to parameterized workflows (i.e. dashboards)
- LLM utilization facilitation

### Just an overview of packages

This document contains overlapping lists of Raku packages that are used for
performing various types of workflows in Data Science (DS) and related utilization
of Large Language Models (LLMs).

**Remark:** [The original version](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Articles/Raku-for-Data-Science-and-LLMs.md) 
of this document written a year ago had mostly 
the purpose of proclaiming (and showing off) Raku's tooling for DS, ML, and LLMs.

At least half a dozen packages for performing ML or Data wrangling 
in Raku have not been included for three reasons:

1. Those packages cannot be installed.
   - Mostly, because of external (third party) dependencies.
2. When tried or experimented with, the packages do not provide faithful or complete results.
   - I.e. precision and recall are not good.
3. The functionalities in those packages are two awkward to use in computational workflows.
   - It is understandable to have ecosystem packages with incomplete or narrow development state.
   - But many of those packages are permanently in those states.
   - Additionally, the authors have not shown or documented how the functionalities are used in longer computational chains or real-world use cases.
   
The examples given below are only for illustration purposes, and by no mean exhaustive.
We refer to related blog posts, videos, and package READMEs for more details.

**Remark:** The packages mentioned in this document can be installed with the script 
["raku-data-science-install.sh"](https://github.com/antononcube/RakuForPrediction-book/blob/main/scripts/raku-data-science-install.sh).

### How to read it?

There are three ways to read this document:

- Just look (or maybe, download) the mind map in the next section.
- Just browse or read the summary list in the next section and skim over the rest of the sections.
- Read all sections and read or browse the linked articles and notebooks. 

Actually, it is assumed that many readers would read one random section, hence, 
most of the sections are mostly self-contained.

------

## Summary of Data Science components and status in Raku 

The list below summarizes how Raku covers the DS components listed above. Each component-item has sub-items for its "previous" state (pre-2021), current state (2025), essential or not mark, a 1-5 star rating of its current state, and references.
There are also corresponding [table](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Doing-Data-Science-with-Raku-status-table.md) and [mind-map](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Doing-Data-Science-with-Raku-status-mind-map-light.pdf).

- Data ingestion
    - Comment: That is fundamental and all programming systems have such functionalities to various degrees.
    - Pre-2021: Robust JSON, CSV, CBOR files ingestion; XML and other formats can be ingested, but not in a robust manner.
    - 2025: Various (improved) packages for working with JSON, CSC, markup images, PDF, etc. Umbrella ingestion function for them.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Data::Importers"](https://raku.land/zef:antononcube/Data::Importers), ["Data::Translators"](https://raku.land/zef:antononcube/Data::Translators), ["Omni-slurping with LLMing"](https://rakuforprediction.wordpress.com/2024/03/26/omni-slurping-with-llming/) [post]
- Data wrangling facilitation
    - Comment: Slicing, splitting, combining, aggregating, summarizing data can be difficult and time consuming.
    - Pre-2021: No serious efforts, especially, in terms of streamlining data wrangling workflows.
    - 2025: Two major efforts for streamlining data wrangling workflows one using "pure" Raku (good for exploration) and other interfaces "outside" systems.
    - Essential: ✓
    - Current status: ★★★ (3.25)
    - References: ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers), ["Dan"](https://raku.land/zef:librasteve/Dan), ["Introduction to data wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku/) [post]
- Statistics for data exploration
    - Comment: This includes descriptive statistics (mean, median, 5-point summary), summarization, outlier identification, and statistical distribution functions.
    - Pre-2021: Various attempts, some are basic and "plain-Raku" (e.g. ["Stats"](https://raku.land/github:MattOates/Stats)), some connect to [GSL](https://www.gnu.org/software/gsl/) (and do not work on macOS.)
    - 2025: A couple of major efforts exist, one is all-in-one package, the other has is spread out in various packages.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Statistics"](https://raku.land/zef:sumankhanal/Statistics), ["Statistics::Distributions"](https://raku.land/zef:antononcube/Statistics::Distributions), ["Statistics::OutlierIdentifiers"](https://raku.land/cpan:ANTONOV/Statistics::OutlierIdentifiers), ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers), ["Data::TypeSystem"](https://raku.land/zef:antononcube/Data::TypeSystem)
- Machine Learning (ML) algorithms (both unsupervised and supervised)
    - Comment: Unsupervised ML is often used for Exploratory Data Analysis (EDA); supervised ML is used to leverage data patterns in some way, but also for certain type of EDA.
    - Pre-2021: A few packages for doing unsupervised Machine Learning (ML) (like ["Text::Markov"](https://raku.land/zef:bbkr/Text::Markov).)
    - 2025: At least supervised ML package connecting (binding) to external systems, a set of unsupervised ML packages for clustering, associating rules learning, fitting, tries with frequencies, and Recommendation Systems (RS). The RS and tries with tries with frequencies can be used as classifiers.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Algorithm::XGBoost](https://github.com/titsuki/raku-Algorithm-XGBoost), [ML::* packages](https://raku.land/?q=ML%3A%3A)
- Data visualization facilitation
    - Comment: Insightful plots over data are used in Data Science most of the time.
    - Pre-2021: A few small packages for plotting, at least one connecting external systems (like GnuPlot), none of them that useful for Data Science.
    - 2025: There are two "solid" packages Data Science visualizations, ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts); there is also an ASCI-plots package ["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot) which is useful when basic, coarse plots are sufficient.
    - Essential: ✓
    - Current status: ★★★★
    - References: ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts), ["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot), ["The Raku-ju hijack hack for D3.js"](https://www.youtube.com/watch?v=YIhx3FBWayo) [video], ["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k) [video]
- Interactive computing environment(s)
    - Comment: Any data exploration is done in interactive manner with multiple changes of the data, and analysis or pattern finding workflows.
    - Pre-2021: The (basic) Raku REPL, related Emacs major-mode, and the notebook environment ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel).
    - 2025: In addition to pre-2021 work there are ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/) for [Wolfram Notebooks](https://www.wolfram.com/notebooks/), ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook) for seamless integration with LLMs.
    - Essential: ✓
    - Current status: ★★★★
    - References: ["Connecting Mathematica and Raku"](https://rakuforprediction.wordpress.com/2021/12/30/connecting-mathematica-and-raku/) [post], ["Exploratory Data Analysis with Raku](https://www.youtube.com/watch?v=YCnjMVSfT8w) [video]
- [Literate programming (LT)](https://en.wikipedia.org/wiki/Literate_programming)
    - Comment: LT is very important for Data Science (DS) because of the DS needs for [Reproducible Research](https://en.wikipedia.org/wiki/Reproducibility#Reproducible_research).
    - Pre-2021: None, except ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel), but that not useful because of the lack of good graphics.
    - 2025: LT is fully supported due to having multiple LT solutions, strong graphics capabilities, LLM integration, and computational documents converters.
    - Essential: ✓
    - Current status: ★★★★★ (4.5)
    - References: ["Notebook transformations"](https://rakuforprediction.wordpress.com/2024/02/17/notebook-transformations/) [post], ["Raku Literate Programming via command line pipelines"](https://www.youtube.com/watch?v=2UjAdQaKof8) [video], ["Conversion and evaluation of Raku files"](https://www.youtube.com/watch?v=GJO7YqjGn6o) [video]
- Data generation and retrieval
    - Comment: For didactical and development purposes random data generation and retrieval of well known dataset is needed.
    - Essential: 𖧋
    - Current status: ★★★★ (3.5)
    - References: ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators), ["Data::ExampleDatasets"](https://raku.land/zef:antononcube/Data::ExampleDatasets), ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics), ["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k) [video]
- External Data Science (DS) and Machine Learning (ML) orchestration
    - Comment: Effective way to do DS and ML _and_ easily move the developed computations to other systems. Allows reuse and having confidence that the utilized DS or ML algorithms are properly implemented and fast.
    - Pre-2021: Various projects connecting to database systems (e.g. MySQL.)
    - 2025:
    - Essential: 𖧋
    - Current status: ★★★ (2.5)
    - References: ["Dan"](https://raku.land/zef:librasteve/Dan), ["Proc::ZMQed"](https://raku.land/zef:antononcube/Proc::ZMQed), ["WWW::WolframAlpha"](https://raku.land/zef:antononcube/WWW::WolframAlpha), ["H2O::Client"](https://github.com/antononcube/Raku-H2O-Client)
- Interactive interfaces to parameterized workflows (dashboards)
    - Comment: Very useful for getting data insights by dynamically changing different statistics based on parameters.
    - Pre-2021: None.
    - 2025: An effort, ["Air::Examples"](https://raku.land/zef:librasteve/Air::Examples), that brings interactivity via [HTMX](https://htmx.org) is using the ["Cro"](https://cro.raku.org) package set and templates; since [Google Charts provides interactivity](https://developers.google.com/chart/interactive/docs/gallery/controls) ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts) can be extended to have those kind of controls and dashboards.
    - Essential: 𖧋
    - Current status: ★
    - References: ["Air::Examples"](https://raku.land/zef:librasteve/Air::Examples), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts)


[![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Doing-Data-Science-with-Raku-status-mind-map-light.png)](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Doing-Data-Science-with-Raku-status-mind-map-light.pdf)

------

## Code generation

For a few years I used Raku to "only" make parser-interpreters for Data Science (DS) and Machine Learning (ML) 
workflows specified with natural language commands. This is the "Raku for prediction" of "cloths have no emperor" approach; see [AA2].
At some point I decided that Raku has to have its own, _useful_ DS and ML packages. (This document proclaims the consequences of that decision.)

Consider the problem:

> Develop conversational agents for Machine Learning workflows that generate correct and executable code 
> using natural language specifications.

The problem is simplified with the observation that the most frequently used ML workflows are in the ML subdomains of:

- Classification
- Latent Semantic Analysis (LSA),
- Regression
- Recommendations

In the broader field of DS we also add Data Wrangling.

Each of these ML or DS sub-fields has it own Domain Specific Language (DSL).

There is a set of Raku packages that facilitate the creation of Data Science workflows in _other_ programming languages.
(Julia, Python, R, Wolfram Language.)

The grammar-based ones have the "DSL::" prefix -- see, for example, ["DSL::English::*"](https://raku.land/?q=DSL%3A%3AEnglish) at [raku.land](https://raku.land/).

The LLM based ones are ["ML::NLPTemplateEngine"](https://raku.land/zef:antononcube/ML::NLPTemplateEngine) and ["DSL::Examples"](https://raku.land/zef:antononcube/DSL::Examples).

### Examples 

Here is an example of using the Command Line Interface (CLI) script of "ML::NLPTemplateEngine":

```
concretize --l=Python make a quantile regression pipeline over dfTemperature using 24 knots an interpolation order 2
```

```
# qrObj = (Regressionizer(dfTemperature)
# .echo_data_summary()
# .quantile_regression(knots = 24, probs = [{0.25, 0.5, 0.75}], order = 2)
# .plot(date_plot = False)
# .errors_plot(relative_errors = False, date_plot = False))
```

------

## Data wrangling 

Most data scientists spend most of their time doing data acquisition and data wrangling. 
Not data science, or AI, or whatever “really learned” work. 
(For a more elaborated rant, see ["Introduction to data wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku/), [AA2].)

Data wrangling, summarization, and generation is done with the packages:

- ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers)
- ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers)
- ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators)

Example datasets retrieval is done with the package:

- ["Data::ExampleDatasets"](https://raku.land/zef:antononcube/Data::ExampleDatasets)

Generation of data wrangling workflows code is done with the package:

- ["DSL::English::DataQueryWorkflows"](https://raku.land/zef:antononcube/DSL::English::DataQueryWorkflows)

the functionalities of which are summarized in this diagram:

![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-book/main/Diagrams/DSLs-Interpreter-for-Data-Wrangling-November-2025-state-small.png)

### Examples

Data wrangling with "Data::Reshapers":

```raku
use Data::Reshapers;
my @dsTitanic = get-titanic-dataset();
cross-tabulate(@dsTitanic, <passengerSex>, <passengerSurvival>)
```

```
# {female => {died => 127, survived => 339}, male => {died => 682, survived => 161}}
```

Data wrangling code generation via CLI:

```shell
dsl-translation -l=Raku "use @dsTitanic; group by passengerSex; show the counts"
```

```
# $obj = @dsTitanic ;
# $obj = group-by($obj, "passengerSex") ;
# say "counts: ", $obj>>.elems
```

------

## Exploratory Data Analysis

At this point Raku is fully equipped to do Exploratory Data Analysis (EDA) over small to moderate size datasets.
(E.g. less than 100,000 rows.) See [AA4, AAv5].

[![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Exploratory-data-analysis-with-Raku-YouTube.png)](https://www.youtube.com/watch?v=YCnjMVSfT8w)

Here are EDA stages and related Raku packages:

- Easy data ingestion
  - ["Data::Importers"](https://raku.land/zef:antononcube/Data::Importers)
    allows for "seamless" import different kinds of data (files or URLs) via:
      - ["JSON::Fast"](https://raku.land/cpan:TIMOTIMO/JSON::Fast)
      - ["Image::Markup::Utilities"](https://raku.land/zef:antononcube/Image::Markup::Utilities)
      - ["PDF::Extract"](https://raku.land/zef:librasteve/PDF::Extract)
      - ["Text::CSV"](https://raku.land/zef:Tux/Text::CSV)
- Data wrangling
  - *See the previous section*
- Visualization
  - ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3), [AAv4]
  - ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts), [AAv5]
- Example- and factual data:
  - I.e. data ready to do computations with
  - ["Data::ExampleDatasets"](https://raku.land/zef:antononcube/Data::ExampleDatasets)
  - ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics), [AAv5]
  - The following entity packages provide concrete names of entities of different kinds:
    - ["DSL::Entity::*"](https://raku.land/?q=DSL%3A%3AEntity)
  - The entity packages have grammar roles for gazetteer Named Entity Recognition (NER)
- Interactive development environment(s)
  - These are often referred to as "notebook solutions" 
  - ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel)
  - ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook)
  - ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/)

------

## Machine Learning & Statistics

The topics of Machine Learning (ML) and Statistics are too big to be described with more than
an outline in this document. The curious or studious readers can check or read and re-run the notebooks [AAn2, AAn3, AAn4]. 

Here are Raku packages for doing ML and Statistics: 
- Unsupervised learning
  - ["ML::Clustering"](https://raku.land/zef:antononcube/ML::Clustering)
  - ["ML::TriesWithFrequencies"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies)
  - ["ML::AssociationRuleLearning"](https://raku.land/zef:antononcube/ML::AssociationRuleLearning)
  - ["Math::Nearest"](https://raku.land/zef:antononcube/Math::Nearest)
  - ["LLM::RetrievalAugmentedGeneration"](https://raku.land/zef:antononcube/LLM::RetrievalAugmentedGeneration)
- Supervised learning
  - ["ML::TriesWithFrequencies"](https://raku.land/zef:antononcube/ML::TriesWithFrequencies)
  - ["ML::ROCFunctions"](https://raku.land/zef:antononcube/ML::ROCFunctions)
  - ["ML::SparseMatrixRecommender"](https://raku.land/zef:antononcube/ML::SparseMatrixRecommender)
- Fitting / regression
  - ["Math::Fitting"](https://raku.land/zef:antononcube/Math::Fitting)
- Distributions 
  - ["Statistics"](https://raku.land/zef:sumankhanal/Statistics)
  - ["Statistics::Distributions"](https://raku.land/zef:antononcube/Statistics::Distributions)
- Outliers
  - ["Statistics::OutlierIdentifiers"](https://raku.land/cpan:ANTONOV/Statistics::OutlierIdentifiers)

----

## Literate programming

["Literate Programming (LP)"](https://en.wikipedia.org/wiki/Literate_programming)
tooling is very important for doing Data Science (DS). 
At this point Raku has four LP solutions (or "notebook solutions"):

- ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel)
- ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook)
- ["Text::CodeProcessing"](https://raku.land/zef:antononcube/Text::CodeProcessing)
- ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/)

The Jupyter Raku-kernel packages
["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel) and
["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook)
provide cells for rendering the output of LaTeX, HTML, Markdown, or Mermaid-JS code or specifications;
see [AAv2].

The package "Text::CodeProcessing" can be used to "weave" (or "execute") computational documents that are
Markdown-, Org-mode-, or Pod6 files; see [AAv2].

"RakuMode" is a Wolfram Language (WL) paclet for using Raku in WL notebooks. 
(See the next section for the "opposite way" -- using WL in Raku sessions.)

**Remark:** WL is also known as "Mathematica".

The package 
["Markdown::Grammar"](https://raku.land/zef:antononcube/Markdown::Grammar)
can be used in notebook conversion workflows; see [AA1, AAv1].

**Remark:** This document itself is a "computational document" -- it has executable Raku and Shell code cells.
The published version of this document was obtained by "executing it" with the command:

```
file-code-chunks-eval Raku-for-Data-Science-and-LLMs.md
```

[![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Conversion-and-evaluation-of-Raku-files-YouTube.png)](https://www.youtube.com/watch?v=GJO7YqjGn6o)

------

## Interconnections

A nice complement to the Raku's DS and LLM functionalities is the ability to easily connect
to other computational systems like Python, R, or Wolfram Language (WL).

The package ["Proc::ZMQed"](https://raku.land/zef:antononcube/Proc::ZMQed) allows the connection
to Python, R, and WL via [ZeroMQ](https://zeromq.org); see [AAv3].

The package ["WWW::WolframAlpha"](https://raku.land/zef:antononcube/WWW::WolframAlpha) 
can be used to get query answers from [WolframAlpha (W|A)](https://www.wolframalpha.com).
Raku chatbooks have also magic cells for accessing W|A; see [AA3].

[![](https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/refs/heads/main/Articles/Diagrams/Doing-Data-Science-with-Raku/Using-Wolfram-Engine-in-Raku-sessions-YouTube.png)](https://www.youtube.com/watch?v=nWeGkJU3wdM)

------

## Cross language workflows

The packages listed in this document, along with the related articles and videos, 
support and demonstrate computational workflows that work across different programming languages.

- Data wrangling workflows code generation is for Julia, Python, R, Raku, SQL, and Wolfram Language (WL).
- Raku's data wrangling functionalities adhere the DSLs and workflows of the popular [Python "pandas"](https://pandas.pydata.org) and [R "tidyverse"](https://www.tidyverse.org).
- More generally, ML workflows code generators as a rule target R, Python, and WL.
  - At this point Raku does not have specialized ML software monads.
- The Raku DSL for interacting with LLMs is also implemented in Python and WL; see [AAv8].
  - To be clear, WL's design of LLM functions was copied (or transferred) to Raku.

------

## References

### Articles, blog posts

[AA1] Anton Antonov,
["Connecting Mathematica and Raku"](https://rakuforprediction.wordpress.com/2021/12/30/connecting-mathematica-and-raku/),
(2021),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["Introduction to data wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku/),
(2021),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov,
["Notebook transformations"](https://rakuforprediction.wordpress.com/2024/02/17/notebook-transformations/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA4] Anton Antonov,
["Omni-slurping with LLMing"](https://rakuforprediction.wordpress.com/2024/03/26/omni-slurping-with-llming/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA5] Anton Antonov,
["Chatbook New Magic Cells"](https://rakuforprediction.wordpress.com/2024/05/18/chatbook-new-magic-cells/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA6] Anton Antonov,
["Age at creation for programming languages stats"](https://rakuforprediction.wordpress.com/2024/05/25/age-at-creation-for-programming-languages-stats/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).


### Notebooks

[AAn1] Anton Antonov,
["Connecting Raku with Wolfram Language and Mathematica"](https://community.wolfram.com/groups/-/m/t/2434981),
(2021),
[Wolfram Community](https://community.wolfram.com).

[AAn2] Anton Antonov, ["Data science over small movie dataset -- Part 1"](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Data-science-over-a-small-movie-dataset-Part-1.ipynb), (2025), [RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog/).

[AAn3] Anton Antonov, ["Data science over small movie dataset -- Part 1"](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Data-science-over-a-small-movie-dataset-Part-2.ipynb), (2025), [RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog/).

[AAn4] Anton Antonov, ["Data science over small movie dataset -- Part 3"](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Data-science-over-a-small-movie-dataset-Part-3.ipynb), (2025), [RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog/).


### Videos

[AAv1] Anton Antonov,
["Markdown to Mathematica converter (Jupyter notebook example)"](https://www.youtube.com/watch?v=Htmiu3ZI05w),
(2022),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov,
["Conversion and evaluation of Raku files"](https://www.youtube.com/watch?v=GJO7YqjGn6o),
(2022),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv3] Anton Antonov,
["Using Wolfram Engine in Raku sessions"](https://www.youtube.com/watch?v=nWeGkJU3wdM),
(2022),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv4] Anton Antonov,
["LLaMA models running guide (Raku)"](https://www.youtube.com/watch?v=zVX-SqRfFPA),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv5] Anton Antonov,
["Conversion and evaluation of Raku files"](https://www.youtube.com/watch?v=GJO7YqjGn6o),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv6] Anton Antonov,
["Raku Literate Programming via command line pipelines"](https://www.youtube.com/watch?v=2UjAdQaKof8),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv7] Anton Antonov,
["Exploratory Data Analysis with Raku"](https://www.youtube.com/watch?v=YCnjMVSfT8w),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv8] Anton Antonov,
["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).

[AAv9] Anton Antonov,
["Raku RAG demo"](https://www.youtube.com/watch?v=JHO2Wk1b-Og),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction). 

[AAv10] Anton Antonov,
["Robust LLM pipelines (Mathematica, Python, Raku)"](https://www.youtube.com/watch?v=QOsVTCQZq_s),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction).