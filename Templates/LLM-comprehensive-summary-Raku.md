# LLM comprehensive summary

Anton Antonov 
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com)
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)
April-May 2025

```perl6, results=asis, echo=FALSE
"Generated on {Date.today.Str}"
```

-----

## Introduction

In this computational Markdown file we apply different LLM prompts in order to comprehensively (and effectively) summarize large texts.

**Remark:** This Markdown file is intended to serve as a template for the initial versions of comprehensive text analyses.

**Remark:** This Markdown template has corresponding notebooks versions: 
(i) [Wolfram Language notebook](https://community.wolfram.com/groups/-/m/t/3448842), 
(ii) [Raku-Jupyter notebook](),
(iii) [Python-Jupyter notebook]().

**Remark:** All remarks in italics are supposed to be removed.

-----

## Setup

```raku, results=hide, echo=FALSE
use LLM::Functions;
use LLM::Prompts;
use Text::SubParsers;
use ML::FindTextualAnswer;
use Data::Importers;
use Data::Translators;
use WWW::MermaidInk;
use WWW::YouTube;
use Hash::Merge;
use Graph;

my $conf4o-mini = llm-configuration('ChatGPT', model => 'gpt-4o-mini', max-tokens => 8192, temperature => 0.5);
my $conf41-mini = llm-configuration('ChatGPT', model => 'gpt-4.1-mini', max-tokens => 8192, temperature => 0.5);
my $conf-gemini-flash = llm-configuration('Gemini', model => 'gemini-2.0-flash', max-tokens => 8192, temperature => 0.5);

# Choose an LLM access configuration or specify your own
my $conf = $conf4o-mini;
```

------

## Ingestion

**Remark:** Chose whether to analyze a text from a file or to analyze the transcript of a YouTube video.


Ingest text from a file:

```raku, eval=TRUE
#my $txtFocus = slurp('');
#text-stats($txtFocus)
```

Ingest the transcript of a YouTube video:

```raku
my $txtFocus = youtube-transcript("eIR_OjWWjtE", format => 'text');
text-stats($txtFocus)
```

**Remark:** The text ingested above is the transcript of the video ["Live CEOing Ep 886: Design Review of LLMGraph"](https://www.youtube.com/watch?v=ewU83vHwN8Y).

**Remark:** The transcript of a YouTube video can be obtained in several ways:
- Use the Raku package ["WWW::YouTube"](https://raku.land/zef:antononcube/WWW::YouTube)
- On macOS, download the audio track and use the program [hear](https://sveinbjorn.org/hear) 
- Use the Python package [“pytube”](https://pypi.org/project/pytube/) (or [“pytubefix”](https://pypi.org/project/pytubefix/)) 

---------

## Summary

Summarize the text:

```raku, results=asis, echo=FALSE, eval=TRUE
llm-synthesize([llm-prompt("Summarize"), $txtFocus], e => $conf)
```

----------

## Tabulate topics

Extract and tabulate text topics:

```perl6, results=asis, echo=FALSE, eval=TRUE
my $tblThemes = llm-synthesize(llm-prompt("ThemeTableJSON")($txtFocus, "article", 30), e => $conf, form => sub-parser('JSON'):drop);
$tblThemes ==> data-translation(field-names=><theme content>)
```

---------

## Mind-map

Generate Mermaid-JS code of a corresponding mind-map:

```perl6, results=asis, echo=FALSE, eval=TRUE
my $mmdBigPicture = llm-synthesize([
    "Create a concise mermaid-js mind-map -- not a flowchart -- for the text:\n\n",
    $txtFocus,
    llm-prompt("NothingElse")("correct mermaid-js")
], e=>$conf);
```

-------

## Sophisticated feedback

Give sophisticated feedback using different “idea hats”:

```perl6, results=asis, echo=FALSE, eval=TRUE 
my $sophFeed = llm-synthesize(llm-prompt("SophisticatedFeedback")($txtFocus, 'HTML', :!ideas), e => $conf);

$sophFeed.subst('```html').subst('```')
```

-----

## Specific questions

Get answers to specific questions (if any.)

```raku, echo=FALSE
my $questions = q:to/END/;
What technology? What it is used for?"
END
```

```raku, results=asis, echo=FALSE, eval=TRUE 
my $ans = llm-synthesize([$questions, $txtFocus], e => $conf);
```

#### Structured

**Remark:** Instead of using the simple workflow above, more structured question-answer response can be obtained 
using the function `find-textual-answer` of the package ["ML::FindTextualAnswer"](https://raku.land/zef:antononcube/ML::FindTextualAnswer).

```raku, results=asis, echo=FALSE, eval=TRUE 
find-textual-answer(
        $txtFocus, 
        ["Who is talking?", "Which technology is discussed?", "What product(s) are discussed?", "Which versions?"], 
        finder => llm-evaluator($conf))
```

-------

## Extracted wisdom or cynical insights

**Remark:** Choose one of the prompts 
[“ExtractArticleWisdom”](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ExtractArticleWisdom/) or 
[“FindPropagandaMessage”](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/FindPropagandaMessage/).
(The latter tends to be more fun.)

```raku
my $prompt = True ?? llm-prompt("ExtractArticleWisdom")() !! llm-prompt("FindPropagandaMessage");
text-stats($prompt)
```

```raku, results=asis, echo=FALSE, eval=TRUE
my $sumIdea = 
    llm-synthesize([
        $prompt,
        'TEXT START',
        $txtFocus,
        'TEXT END'
     ], e => $conf);

$sumIdea.subst(/ ^^ '#' /, '###', :g)
```
