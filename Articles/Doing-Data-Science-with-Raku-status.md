- Data ingestion
    - Comment: This is fundamental, and all programming systems have such functionalities to varying degrees.
    - Pre-2021: Robust ingestion of JSON, CSV, and CBOR files; XML and other formats can be ingested, but not in a robust manner.
    - 2025: Various improved packages for working with JSON, CSV, markup, images, PDF, etc., with an umbrella ingestion function for them.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Data::Importers"](https://raku.land/zef:antononcube/Data::Importers), ["Data::Translators"](https://raku.land/zef:antononcube/Data::Translators), ["Omni-slurping with LLMing"](https://rakuforprediction.wordpress.com/2024/03/26/omni-slurping-with-llming/) [post]

- Data wrangling facilitation
    - Comment: Slicing, splitting, combining, aggregating, and summarizing data can be difficult and time-consuming.
    - Pre-2021: No serious efforts, especially in terms of streamlining data-wrangling workflows.
    - 2025: Two major efforts for streamlining data-wrangling workflows: one using “pure” Raku (good for exploration) and another interfacing with “outside” systems.
    - Essential: ✓
    - Current status: ★★★ (3.25)
    - References: ["Data::Reshapers"](https://raku.land/zef:antononcube/Data::Reshapers), ["Dan"](https://raku.land/zef:librasteve/Dan), ["Introduction to data wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku/) [post]

- Statistics for data exploration
    - Comment: This includes descriptive statistics (mean, median, 5-point summary), summarization, outlier identification, and statistical distribution functions.
    - Pre-2021: Various attempts; some are basic and “plain Raku” (e.g. ["Stats"](https://raku.land/github:MattOates/Stats)), some connect to [GSL](https://www.gnu.org/software/gsl/) (and do not work on macOS).
    - 2025: A couple of major efforts exist: one all-in-one package, the other spread out across various packages.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Statistics"](https://raku.land/zef:sumankhanal/Statistics), ["Statistics::Distributions"](https://raku.land/zef:antononcube/Statistics::Distributions), ["Statistics::OutlierIdentifiers"](https://raku.land/cpan:ANTONOV/Statistics::OutlierIdentifiers), ["Data::Summarizers"](https://raku.land/zef:antononcube/Data::Summarizers), ["Data::TypeSystem"](https://raku.land/zef:antononcube/Data::TypeSystem)

- Machine Learning (ML) algorithms (both unsupervised and supervised)
    - Comment: Unsupervised ML is often used for Exploratory Data Analysis (EDA); supervised ML is used to leverage data patterns in some way, and also for certain types of EDA.
    - Pre-2021: A few packages for unsupervised Machine Learning (ML), such as ["Text::Markov"](https://raku.land/zef:bbkr/Text::Markov).
    - 2025: At least one supervised ML package connecting (binding) to external systems, and a set of unsupervised ML packages for clustering, association-rule learning, fitting, tries with frequencies, and Recommendation Systems (RS). The RS and tries with frequencies can be used as classifiers.
    - Essential: ✓
    - Current status: ★★★ (2.5)
    - References: ["Algorithm::XGBoost"](https://github.com/titsuki/raku-Algorithm-XGBoost), [ML::* packages](https://raku.land/?q=ML%3A%3A), ["Fast and compact classifier of DSL commands"](https://rakuforprediction.wordpress.com/2022/07/31/fast-and-compact-classifier-of-dsl-commands/) [post], ["Chebyshev Polynomials and Fitting Workflows"](https://raku-advent.blog/2024/12/17/day-17-chebyshev-polynomials-and-fitting-workflows/) [post]

- Data visualization facilitation
    - Comment: Insightful plots over data are used in Data Science most of the time.
    - Pre-2021: A few small packages for plotting, at least one connecting to external systems (like Gnuplot), none of them particularly useful for Data Science.
    - 2025: There are two “solid” packages for Data Science visualizations: ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3) and ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts). There is also an ASCII-plots package, ["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot), which is useful when basic, coarse plots are sufficient.
    - Essential: ✓
    - Current status: ★★★★
    - References: ["JavaScript::D3"](https://raku.land/zef:antononcube/JavaScript::D3), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts), ["Text::Plot"](https://raku.land/zef:antononcube/Text::Plot), ["The Raku-ju hijack hack for D3.js"](https://www.youtube.com/watch?v=YIhx3FBWayo) [video], ["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k) [video]

- Interactive computing environment(s)
    - Comment: Any data exploration is done in an interactive manner, with multiple changes of the data and analysis or pattern-finding workflows.
    - Pre-2021: The (basic) Raku REPL, a related Emacs major mode, and the notebook environment ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel).
    - 2025: In addition to pre-2021 work, there is ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/) for [Wolfram Notebooks](https://www.wolfram.com/notebooks/), and ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook) for seamless integration with LLMs.
    - Essential: ✓
    - Current status: ★★★★
    - References: ["Connecting Mathematica and Raku"](https://rakuforprediction.wordpress.com/2021/12/30/connecting-mathematica-and-raku/) [post], ["Exploratory Data Analysis with Raku"](https://www.youtube.com/watch?v=YCnjMVSfT8w) [video]

- [Literate programming (LT)](https://en.wikipedia.org/wiki/Literate_programming)
    - Comment: LT is very important for Data Science (DS) because of DS needs for [Reproducible Research](https://en.wikipedia.org/wiki/Reproducibility#Reproducible_research).
    - Pre-2021: None, except ["Jupyter::Kernel"](https://raku.land/zef:bduggan/Jupyter::Kernel), but that is not useful because of the lack of good graphics.
    - 2025: LT is fully supported due to having multiple LT solutions, strong graphics capabilities, LLM integration, and computational document converters.
    - Essential: ✓
    - Current status: ★★★★★ (4.5)
    - References: ["Notebook transformations"](https://rakuforprediction.wordpress.com/2024/02/17/notebook-transformations/) [post], ["Raku Literate Programming via command line pipelines"](https://www.youtube.com/watch?v=2UjAdQaKof8) [video], ["Conversion and evaluation of Raku files"](https://www.youtube.com/watch?v=GJO7YqjGn6o) [video]

- Data generation and retrieval
    - Comment: For didactical and development purposes, random data generation and retrieval of well-known datasets are needed.
    - Pre-2021: Nothing more than the built-in Raku random generators (`pick`, `roll`).
    - 2025: Generators of random strings, words, pet names, date-times, distribution variates, and tabular datasets. Popular datasets from the R ecosystem can be downloaded and cached.
    - Essential: 𖧋
    - Current status: ★★★★ (3.5)
    - References: ["Data::Generators"](https://raku.land/zef:antononcube/Data::Generators), ["Data::ExampleDatasets"](https://raku.land/zef:antononcube/Data::ExampleDatasets), ["Data::Geographics"](https://raku.land/zef:antononcube/Data::Geographics), ["Geographics data in Raku demo"](https://www.youtube.com/watch?v=Rkk_MeqLj_k) [video]

- External Data Science (DS) and Machine Learning (ML) orchestration
    - Comment: An effective way to do DS and ML and easily move the developed computations to other systems. This allows reuse and provides confidence that the utilized DS or ML algorithms are properly implemented and fast.
    - Pre-2021: Various projects connecting to database systems (e.g. MySQL).
    - 2025: The project ["Dan"](https://raku.land/zef:librasteve/Dan) provides bindings to the data-wrangling library [Polars](https://docs.pola.rs). The project ["H2O::Client"](https://github.com/antononcube/Raku-H2O-Client) aims to provide both data-wrangling and ML orchestration to [H2O.ai](https://h2o.ai).
    - Essential: 𖧋
    - Current status: ★★★ (2.5)
    - References: ["Dan"](https://raku.land/zef:librasteve/Dan), ["Proc::ZMQed"](https://raku.land/zef:antononcube/Proc::ZMQed), ["WWW::WolframAlpha"](https://raku.land/zef:antononcube/WWW::WolframAlpha), ["H2O::Client"](https://github.com/antononcube/Raku-H2O-Client)

- Interactive interfaces to parameterized workflows (dashboards)
    - Comment: Very useful for getting data insights by dynamically changing different statistics based on parameters.
    - Pre-2021: None.
    - 2025: An effort, ["Air::Examples"](https://raku.land/zef:librasteve/Air::Examples), that brings interactivity via [HTMX](https://htmx.org), using the ["Cro"](https://cro.raku.org) package set and templates. Since [Google Charts provides interactivity](https://developers.google.com/chart/interactive/docs/gallery/controls), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts) can be extended to have those kinds of controls and dashboards.
    - Essential: 𖧋
    - Current status: ★
    - References: ["Air::Examples"](https://raku.land/zef:librasteve/Air::Examples), ["JavaScript::Google::Charts"](https://raku.land/zef:antononcube/JavaScript::Google::Charts)