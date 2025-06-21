# Parameterized Literate Programming

Anton Antonov    
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
June 2025

-----

## Introduction

[Literate Programming (LT)](https://en.wikipedia.org/wiki/Literate_programming), [Wk1], blends code and documentation into a narrative, prioritizing human readability. Code and explanations are _interwoven_, with tools extracting code for compilation and documentation for presentation, enhancing clarity and maintainability.

LT is commonly employed in scientific computing and data science for reproducible research and open access initiatives. Today, millions of programmers use literate programming tools.

Raku has several LT solutions:

- Raku's built-in Pod6 markup language
- [Wolfram notebooks](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/), [AAp3]
- [Jupyter notebooks](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp4, BDp1]
- [Executable documents](https://raku.land/zef:antononcube/Text::CodeProcessing) in Markdown, Pod6, and Org-mode formats, [AAp1, AAv1]

This document ([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Notebooks/Jupyter/Parameterized-Literate-Programming.ipynb)) discusses executable documents parameterization -- or parameterized reports -- provided by ["Text::CodeProcessing"](https://raku.land/zef:antononcube/Text::CodeProcessing), [AAp1].

**Remark:** Providing report parameterization has been in my TODO list since the beginning of programming "Text::CodeProcessing". I finally did it in order to facilitate parameterized Large Language Model (LLM) workflows. See the LLM template ["LLM-comprehensive-summary-Raku.md"](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Templates/LLM-comprehensive-summary-Raku.md).

The document has three main sections:

- Using YAML document header to specify parameters
    - Description and examples
- LLM templates with parameters
- Operating System (OS) shell execution with specified parameters

**Remark:** The programmatically rendered Markdown is put within three-dots separators.

-----

## Setup

Load packages:


```raku
use Text::CodeProcessing;
use Lingua::NumericWordForms;
```

-----

## YAML front-matter with parameters

For a given text or file we can _execute_ that text or file and produce its _woven_ version using:

- The sub `StringCodeChunksEvaluation` in a Raku session
- The Command Line Interface (CLI) script `file-code-chunks-eval` in an OS shell


Consider the following Markdown text (of a certain [file](https://github.com/antononcube/Raku-Text-CodeProcessing/blob/main/resources/Template.md)):


````raku
sink my $txt = q:to/END/;
---
title: Numeric word forms generation (template)
author: Anton Antonov
date: 2025-06-19
params:
    sample-size: 5
    min: 100
    max: 10E3
    to-lang: "Russian"
---

Generate a list of random numbers:

```raku
use Data::Generators;

my @ns = random-real([%params<min>, %params<max>], %params<sample-size>)».floor
```

Convert to numeric word forms:

```raku
use Lingua::NumericWordForms;

.say for @ns.map({ $_ => to-numeric-word-form($_, %params<to-lang>) })
```
END
````

The parameters of that executable document are given in [YAML](https://en.wikipedia.org/wiki/YAML) format -- similar to ["parameterized reports"](https://rmarkdown.rstudio.com/lesson-6.html) of R Markdown documents. (Introduced and provided by Posit, formerly RStudio.)

- **Declaring parameters:**
    - Parameters are declared using the `params` field within the YAML header of the document. 
    - For example, the text above creates the parameter "sample-size" and assigns it the default value `5`.

- **Using parameters in code:**
    - Parameters are made available within the Raku environment as a read-only hashmap named `%params`. 
    - To access a parameter in code, call `%params<parameter-name>`.

- **Setting parameter values:**
    - To create a report that uses a new set of parameter values add:
       - `%params` argument to `StringCodeChunksEvaluation`  
       - `--params` argument to the CLI script `file-code-chunks-eval`


Here is the woven (or executed) version of the text:


```raku
#% markdown
StringCodeChunksEvaluation($txt, 'markdown')
==> { .subst(/^ '---' .*? '---'/) }()
```

<div id="dot" style="text-align: center;">. . .</div>

Generate a list of random numbers:

```raku
use Data::Generators;

my @ns = random-real([100, 10000], 5)».floor
```
```
# [3925 6533 3215 2983 1395]
```

Convert to numeric word forms:

```raku
use Lingua::NumericWordForms;

.say for @ns.map({ $_ => to-numeric-word-form($_, 'Russian') })
```
```
# 3925 => три тысячи девятьсот двадцать пять
# 6533 => шесть тысяч пятьсот тридцать три
# 3215 => три тысячи двести пятнадцать
# 2983 => две тысячи девятьсот восемьдесят три
# 1395 => одна тысяча триста девяносто пять
```

<div id="dot" style="text-align: center;">. . .</div>


**Remark:** In order to be easier to read the results, the YAML header ware removed (with `subst`.)

Here we change parameters -- different sample size and language for the generated word forms:


```raku
#% markdown
StringCodeChunksEvaluation($txt, 'markdown', params => {:7sample-size, to-lang => 'Japanese'})
==> { .subst(/^ '---' .*? '---'/) }()
```

<div id="dot" style="text-align: center;">. . .</div>

Generate a list of random numbers:

```raku
use Data::Generators;

my @ns = random-real([100, 10000], 7)».floor
```
```
# [8684 5057 7732 2091 7098 7941 6846]
```

Convert to numeric word forms:

```raku
use Lingua::NumericWordForms;

.say for @ns.map({ $_ => to-numeric-word-form($_, 'Japanese') })
```
```
# 8684 => 八千六百八十四
# 5057 => 五千五十七
# 7732 => 七千七百三十二
# 2091 => 二千九十一
# 7098 => 七千九十八
# 7941 => 七千九百四十一
# 6846 => 六千八百四十六
```

<div id="dot" style="text-align: center;">. . .</div>


----

## LLM application

From LLM-workflows perspective parameterized reports can be seen as:
- An alternative using LLM functions and prompts, [AAp5, AAp6]
- Higher-level utilization of LLM functions workflows 

To illustrate the former consider this short LLM template:


````raku
sink my $llmTemplate = q:to/END/;
---
params:
    question: 'How many sea species?'
    model: 'gpt-4o-mini'
    persona: SouthernBelleSpeak
---

For the question:

> %params<question>

The answer is:

```raku, results=asis, echo=FALSE, eval=TRUE
use LLM::Functions;
use LLM::Prompts;

my $conf = llm-configuration('ChatGPT', model => %params<model>);

llm-synthesize([llm-prompt(%params<persona>), %params<question>], e => $conf)
```
END
````

Here we execute that LLM template providing different question and LLM persona:


```raku
#% markdown
StringCodeChunksEvaluation(
    $llmTemplate, 
    'markdown', 
    params => {question => 'How big is Texas?', persona => 'SurferDudeSpeak'}
).subst(/^ '---' .* '---'/)
```

<div id="dot" style="text-align: center;">. . .</div>


For the question:

> 'How big is Texas?'

The answer is:


Whoa, bro! Texas is like, totally massive, man! It's like the second biggest state in the whole USA, after that gnarly Alaska, you know? We're talking about around 268,000 square miles of pure, wild vibes, bro! That's like a whole lot of room for the open road and some epic waves if you ever decide to cruise on over, dude! Just remember to keep it chill and ride the wave of life, bro!


<div id="dot" style="text-align: center;">. . .</div>


-----

## CLI parameters

In order to demonstrate CLI usage of parameters below we:

- Export the Markdown string into a file
- Invoke the CLI `file-code-chunks-eval`
    - In a Raku-Jupyter notebook this can be done with the magic `#% bash`
    - Alternatively, `run` and `shell` can be used
- Import the woven file and render its content

### Export to Markdown file


```raku
spurt($*CWD ~ '/LLM-template.md', $llmTemplate)
```




    True



### CLI invocation

Specifying the template parameters using the CLI is done with the named argument `--params` with a value that is a valid hashmap Raku code:


```raku
#% bash
file-code-chunks-eval LLM-template.md --params='{question=>"Where is Iran?", persona=>"DrillSergeant"}'
```


**Remark:** If the output file is not specified then the output file name is the CLI input file argument with the string '_woven' placed before the extension.

### Import and render 

Import the woven file and render it (again, remove the YAML header for easier reading):


```raku
#% markdown
slurp($*CWD ~ '/LLM-template_woven.md')
==> {.subst(/ '---' .*? '---' /)}()
```

<div id="dot" style="text-align: center;">. . .</div>

For the question:

> 'Where is Iran?'

The answer is:


YOU LISTEN UP, MAGGOT! IRAN IS LOCATED IN THE MIDDLE EAST, BOUNDED BY THE CASPIAN SEA TO THE NORTH AND THE PERSIAN GULF TO THE SOUTH! NOW GET YOUR HEAD OUT OF THE CLOUDS AND PAY ATTENTION! I DON'T HAVE TIME FOR YOUR LAZY QUESTIONS! IF I SEE YOU SLACKING OFF, YOU'LL BE DOING PUSH-UPS UNTIL YOUR ARMS FALL OFF! DO YOU UNDERSTAND ME? SIR!


<div id="dot" style="text-align: center;">. . .</div>


-----

## References

### Packages

[AAp1] Anton Antonov, [Text::CodeProcssing Raku package](https://github.com/antononcube/Raku-Text-CodeProcessing), (2021-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, [Lingua::NumericWordForms Raku package](https://github.com/antononcube/Raku-Lingua-NumericWordForms), (2021-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov, [RakuMode Wolfram Language paclet](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/RakuMode/), (2023), [Wolfram Language Paclet Repository](https://resources.wolframcloud.com/PacletRepository/).

[AAp4] Anton Antonov, [Jupyter::Chatbook Raku package](https://github.com/antononcube/Raku-Jupyter-Chatbook), (2023-2024), [GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov, [LLM::Functions Raku package](https://github.com/antononcube/Raku-LLM-Functions), (2023-2025), [GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov, [LLM::Prompts Raku package](https://github.com/antononcube/Raku-LLM-Prompts), (2023-2025), [GitHub/antononcube](https://github.com/antononcube).

[BDp1] Brian Duggan, [Jupyter::Kernel Raku package](https://github.com/bduggan/raku-jupyter-kernel), (2017-2024), [GitHub/bduggan](https://github.com/bduggan).

### Videos

[AAv1] Anton Antonov, ["Raku Literate Programming via command line pipelines"](https://www.youtube.com/watch?v=2UjAdQaKof8), (2023), [YouTube/@AAA4prediction](https://www.youtube.com/@AAA4prediction).

