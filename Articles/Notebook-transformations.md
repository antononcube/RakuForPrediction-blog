# Notebook transformations

## Introduction

In this document we describe a series of different (computational) notebook transformations using 
different tools. We are using a series of recent articles and notebooks for processing the 
English and Russian texts of a [recent 2-hour long interview](http://en.kremlin.ru/events/president/news/73411).
The workflows given in the notebooks are in Raku and Wolfram Language (WL).

**Remark:** Wolfram Language (WL) and Mathematica are used as synonyms in this document.

**Remark:** Using notebooks with Large Language Model (LLM) workflows is convenient because the WL LLM functions
are also implemented in Python and Raku, [AA1, AAp1, AAp2]. 

We can say that this blog post attempts to advertise the Raku package 
["Markdown::Grammar"](https://github.com/antononcube/Raku-Markdown-Grammar), [AAp3], 
demonstrated in the videos:
- ["Markdown to Mathematica converter (CLI and StackExchange examples)"](https://www.youtube.com/watch?v=39ekokgnoqE), [AAv5, AA4]
- ["Markdown to Mathematica converter (Jupyter notebook example)"](https://www.youtube.com/watch?v=Htmiu3ZI05w), [AAv6]

**TL;DR:** Using Markdown as an intermediate format we can transform easily enough between Jupyter- and Mathematica notebooks.

------

## Transformation trip

Here is a recent application of the converter related to my ["LLM aids for processing of the first Carlson-Putin interview"](https://community.wolfram.com/groups/-/m/t/3121333) series of posts.

The transformation trip starts with the notebook of the article ["LLM aids for processing of the first Carlson-Putin interview"](https://community.wolfram.com/groups/-/m/t/3121333), [AA1].

1. Make the Raku Jupyter notebook
    - With the [LLM aids for the Carlson-Putin interview](https://github.com/antononcube/RakuForPrediction-book/blob/main/Notebooks/Jupyter/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview.ipynb), [AAn1]
2. Convert the Jupyter notebook into Markdown
    - Using Jupyter's built-in converter
3. Publish the [Markdown version to WordPress](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/), [AA2]
4. Convert the Markdown file into a Mathematica notebook
    - Using the Raku package, ["Markdown::Grammar"](https://raku.land/zef:antononcube/Markdown::Grammar), [AA4, AAp3, AAv5, AAv6]
    - The [obtained notebook](https://www.wolframcloud.com/obj/antononcube/Published/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Raku.nb) uses the WL paclet ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/), [AAp4]
5. Publish that to [Wolfram Community](https://community.wolfram.com)
    - That notebook was deleted by moderators, because it does not feature Wolfram Language (WL)
6. Make the corresponding Mathematica notebook using [WL LLM functions](https://reference.wolfram.com/language/guide/LLMFunctions.html)
7. [Publish to Wolfram Community](https://community.wolfram.com/groups/-/m/t/3121333)
8. Make [the Russian version with the Russian transcript](https://www.wolframcloud.com/obj/antononcube/Published/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-WL-Russian.nb)
9. Publish to Wolfram Community
    - That notebook was deleted by the moderators, because it is not in English
10. Convert the Mathematica notebook to Markdown
    - Using Kuba Podkalicki's [M2MD](https://github.com/kubaPod/M2MD), [KPp1]
11. [Publish to WordPress][1], [AA3]
12. Convert the Markdown file to Jupyter
    - Using [jupytext](https://jupytext.readthedocs.io/en/latest/)
13. Re-make [the (Russian described) workflows using Raku](https://github.com/antononcube/RakuForPrediction-book/blob/main/Notebooks/Jupyter/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Russian.ipynb), [AAn5]
14. Re-make workflows using Python, [AAn6], [AAn7]

-----

Here is the corresponding Mermaid-JS diagram (using the paclet [`MermaidJS`](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/MermaidJS/) or the function [`MermaidInk`](https://resources.wolframcloud.com/FunctionRepository/resources/MermaidInk/)):

```raku, results=asis
use WWW::MermaidInk;

my $diagram = q:to/END/;
graph TD
   A[Make the Raku Jupyter notebook] --> B[Convert the Jupyter notebook into Markdown]
   B --> C[Publish to WordPress]
   C --> D[Convert the Markdown file into a Mathematica notebook]
   D --> E[Publish that to Wolfram Community]
   E --> F[Make the corresponding Mathematica notebook using WL functions]
   F --> G[Publish to Wolfram Community]
   G --> H[Make the Russian version with the Russian transcript]
   H --> I[Publish to Wolfram Community]
   I --> J[Convert the Mathematica notebook to Markdown]
   J --> K[Publish to WordPress]
   K --> L[Convert the Markdown file to Jupyter]
   L --> M[Re-make the workflows using Raku]
   M --> N[Re-make the workflows using Python]
   C -.-> WordPress{{Word Press}}
   K -.-> WordPress
   E -.-> |Deleted:<br>features Raku| WolframCom{{Wolfram Community}}
   G -.-> WolframCom
   I -.-> |"Deleted:<br>not in English"|WolframCom
   D -.-> MG[[Markdown::Grammar]]
   B -.-> Ju{{Jupyter}}
   L -.-> jupytext[[jupytext]]
   J -.-> M2MD[[M2MD]]
   E -.-> RakuMode[[RakuMode]]
END

say mermaid-ink($diagram, format => 'md-image');
```

![enter image description here][2]


----- 

## Clarifications

### Russian versions

[The first Carlson-Putin interview](https://tuckercarlson.com/putin/) that is processed in the notebooks was held both in English and Russian.
I think just doing the English study is "half-baked." 
Hence, I did the workflows with the Russian text and translated to Russian the related explanations. 

**Remark:** The Russian versions are done in all three programming languages: Python, Raku, Wolfram Language.
See [AAn4, AAn5, AAn7].

### Using different programming languages

From my point of view, having Raku-enabled Mathematica / WL notebook is a strong statement about WL.
Fair amount of coding was required for the paclet ["RakuMode"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/), [AAp4].  

To have that functionality implemented is preconditioned on WL having 
[extensive external evaluation functionalities](https://reference.wolfram.com/language/guide/ExternalLanguageInterfaces.html).

When we compare WL, Python, and R over Machine Learning (ML) projects, WL always appears to be the best choice for ML. (Overall.)

I do use these sets of comparison posts at Wolfram Community to support my arguments in discussions regarding which programming language is better. (Or bigger.)

#### Example comparison: WL workflows

The following three Wolfram Community posts are more or less the same content -- "Workflows with LLM functions" -- but in different programming languages:

- [Python](https://community.wolfram.com/groups/-/m/t/3027072)
- [Raku](https://community.wolfram.com/groups/-/m/t/2982320)
- [WL](https://community.wolfram.com/groups/-/m/t/2983602)

#### Example comparison: LSA over mandala collections

The following Wolfram Community posts are more or less the same content -- 
["LSA methods comparison over random mandalas deconstruction"](https://www.youtube.com/watch?v=nKlcts5aGwY), [AAv1] -- 
but in different programming languages:

- [Python](https://community.wolfram.com/groups/-/m/t/2508233)
- [WL](https://community.wolfram.com/groups/-/m/t/2508248)

**Remark:** The movie, [AAv1], linked in those notebooks also shows a comparison with the LSA workflow in R.

### Using Raku with LLMs

I generally do not like using Jupyter notebooks, but using Raku with LLMs is very convenient [AAv2, AAv3, AAv4].
WL is clunkier when it comes to pre- or post-processing of LLM results.

Also, the Raku Chatbooks, [AAp5], provided better environment for display of the often Markdown formatted
results of LLMs. (Like the ones in notebooks discussed here.)

-----

## References

### Articles

[AA1] Anton Antonov,
["Workflows with LLM functions"](https://rakuforprediction.wordpress.com/2023/08/01/workflows-with-llm-functions/),
(2023),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov,
["LLM помогает в обработке первого интервью Карлсона-Путина"](https://mathematicaforprediction.wordpress.com/2024/02/13/llm-помогает-в-обработке-первого-интервь/),
(2024),
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com/).

[AA4] Anton Antonov,
["Markdown to Mathematica converter"](https://community.wolfram.com/groups/-/m/t/2625639),
(2022).
[Wolfram Community](https://community.wolfram.com).

### Notebooks

[AAn1] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://github.com/antononcube/RakuForPrediction-book/blob/main/Notebooks/Jupyter/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview.ipynb),
(**Raku/Jupyter**),
(2024),
[RakuForPrediction-book at GitHub/antononcube](https://github.com/antononcube/RakuForPrediction-book).

[AAn2] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://www.wolframcloud.com/obj/antononcube/Published/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Raku.nb),
(**Raku/Mathematica**),
(2024),
[WolframCloud/antononcube](https://www.wolframcloud.com/obj/antononcube/Published/).

[AAn3] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://www.wolframcloud.com/obj/antononcube/Published/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-WL.nb),
(**WL/Mathematica**),
(2024),
[WolframCloud/antononcube](https://www.wolframcloud.com/obj/antononcube/Published/).

[AAn4] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://www.wolframcloud.com/obj/antononcube/Published/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-WL-Russian.nb),
(**in Russian**),
(**WL/Mathematica**),
(2024),
[WolframCloud/antononcube](https://www.wolframcloud.com/obj/antononcube/Published/).

[AAn5] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://github.com/antononcube/RakuForPrediction-book/blob/main/Notebooks/Jupyter/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Russian.ipynb),
(**in Russian**),
(**Raku/Jupyter**),
(2024),
[RakuForPrediction-book at GitHub/antononcube](https://github.com/antononcube/RakuForPrediction-book).

[AAn6] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://github.com/antononcube/PythonForPrediction-blog/blob/main/Notebooks/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Python.ipynb),
(**Python/Jupyter**),
(2024),
[PythonForPrediction-blog at GitHub/antononcube](https://github.com/antononcube/PythonForPrediction-blog).

[AAn7] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://github.com/antononcube/PythonForPrediction-blog/blob/main/Notebooks/LLM-aids-for-processing-of-the-first-Carlson-Putin-interview-Python-Russian.ipynb),
(**in Russian**),
(**Python/Jupyter**),
(2024),
[PythonForPrediction-blog at GitHub/antononcube](https://github.com/antononcube/PythonForPrediction-blog).


### Packages, paclets

[AAp1] Anton Antonov,
[LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[LLM::Prompts](https://github.com/antononcube/Raku-LLM-Prompts) Raku package,
(2023),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[Markdown::Grammar Raku package](https://github.com/antononcube/Raku-Markdown-Grammar),
(2022-2023),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[RakuMode WL paclet](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/),
(2022-2023),
[Wolfram Language Paclet Repository](https://resources.wolframcloud.com/PacletRepository/).

[AAp5] Anton Antonov,
[Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[KPp1] Kuba Podkalicki's,
[M2MD WL paclet](https://github.com/kubaPod/M2MD),
(2018-2023),
[GitHub/kubaPod](https://github.com/kubaPod/).


### Videos

[AAv1] Anton Antonov
["Random Mandalas Deconstruction in R, Python, and Mathematica (Greater Boston useR Meetup, Feb 2022)"](https://www.youtube.com/watch?v=nKlcts5aGwY)
(2022),
[YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov,
["Jupyter Chatbook LLM cells demo (Raku)"](https://www.youtube.com/watch?v=cICgnzYmQZg)
(2023),
[YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv3] Anton Antonov,
["Jupyter Chatbook multi cell LLM chats teaser (Raku)"](https://www.youtube.com/watch?v=wNpIGUAwZB8),
(2023),
[YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv4] Anton Antonov
["Integrating Large Language Models with Raku"](https://www.youtube.com/watch?v=-OxKqRrQvh0),
(2023),
[YouTube/@therakuconference6823](https://www.youtube.com/@therakuconference6823).

[AAv5] Anton Antonov, 
["Markdown to Mathematica converter (CLI and StackExchange examples)"](https://www.youtube.com/watch?v=39ekokgnoqE), 
(2022), 
[Anton A. Antonov's channel at YouTube](https://www.youtube.com/@AAA4prediction).

[AAv6] Anton Antonov,
["Markdown to Mathematica converter (Jupyter notebook example)"](https://www.youtube.com/watch?v=Htmiu3ZI05w),
(2022),
[Anton A. Antonov's channel at YouTube](https://www.youtube.com/@AAA4prediction).

[1]: https://mathematicaforprediction.wordpress.com/2024/02/13/llm-%D0%BF%D0%BE%D0%BC%D0%BE%D0%B3%D0%B0%D0%B5%D1%82-%D0%B2-%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B5-%D0%BF%D0%B5%D1%80%D0%B2%D0%BE%D0%B3%D0%BE-%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%B2%D1%8C
[2]: https://community.wolfram.com//c/portal/getImageAttachment?filename=LLM-for-Carlson-Putin-interview-publications.png&userId=143837

