# Comprehension AI Aids for “Can AI Solve Science?”

Anton Antonov   
[MathematicaForPrediction at WordPress](https://mathematicaforprediction.wordpress.com)   
[MathematicaForPrediction at GitHub](https://github.com/antononcube/MathematicaForPrediction)   
March 2024  

-------

## Introduction

In this notebook we use the Large Language Model (LLM) prompts [”ThemeTableJSON”](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ThemeTableJSON/), ["ExtractArticleWisdom"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ExtractArticleWisdom/), ["FindHiddenMessage"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/FindHiddenMessage/), and ["FindPropagandaMessage"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/FindPropagandaMessage/) 
to facilitate the reading and comprehension of Stephen Wolfram’s article ["Can AI Solve Science?"](https://writings.stephenwolfram.com/2024/03/can-ai-solve-science/), [SW1].

**Remark:** We use “simple” text processing, but since the article has lots of images multi-modal models would be more appropriate.

Here is an image of [article's start]("https://writings.stephenwolfram.com/2024/03/can-ai-solve-science/"):

![](https://www.wolframcloud.com/files/a23d407c-b07e-4f20-84fb-9b5213e994a4/htmlcaches/images/f3258ba5bd7944cbf10c30a1516b75841ce226a68acf3b886dced11f5fe865fa)

The computations are done with Wolfram Language (WL) chatbook. The LLM functions used in the workflows are explained and demonstrated in [SW2, AA1, AA2, AAn1÷ AAn4]. The workflows are done with OpenAI's models. Currently the models of Google's (PaLM) and MistralAI cannot be used with the workflows below because their input token limits are too low.


### Structure

The structure of the notebook is as follows:


| No | Part                                     | Content                                                   |
| -- |------------------------------------------|-----------------------------------------------------------|
| 1  | **Getting the article's text and setup** | Standard ingestion and setup.                             |
| 2  | **Article's structure**                  | TL;DR via a table of themes.                              |
| 3  | **Flowcharts**                           | Get flowcharts relating article's concepts.               |
| 4  | **Extract article wisdom**               | Get a summary and extract ideas, quotes, references, etc. |
| 5  | **Hidden messages and propaganda**       | Reading it with a conspiracy theorist hat on.             |


-------

## Setup

Here we load a view packages and define ingestion functions:


```raku
use HTTP::Tiny;
use JSON::Fast;
use Data::Reshapers;

sub text-stats(Str:D $txt) { <chars words lines> Z=> [$txt.chars, $txt.words.elems, $txt.lines.elems] };

sub strip-html(Str $html) returns Str {

    my $res = $html
    .subst(/'<style'.*?'</style>'/, :g)
    .subst(/'<script'.*?'</script>'/, :g)
    .subst(/'<'.*?'>'/, :g)
    .subst(/'&lt;'.*?'&gt;'/, :g)
    .subst(/[\v\s*] ** 2..*/, "\n\n", :g);

    return $res;
}
```




    &strip-html



### Ingest text

Here we get the plain text of the article:


```raku
my $htmlArticleOrig = HTTP::Tiny.get("https://writings.stephenwolfram.com/2024/03/can-ai-solve-science/")<content>.decode;

text-stats($htmlArticleOrig);
```




    (chars => 216219 words => 19867 lines => 1419)



Here we strip the HTML code from the article:


```raku
my $txtArticleOrig = strip-html($htmlArticleOrig);

text-stats($txtArticleOrig);
```




    (chars => 100657 words => 16117 lines => 470)



Here we clean article’s text :


```raku
my $txtArticle = $txtArticleOrig.substr(0, $txtArticleOrig.index("Posted in:"));

text-stats($txtArticle);
```




    (chars => 98011 words => 15840 lines => 389)



### LLM access configuration

Here we configure LLM access -- we use OpenAI's model "gpt-4-turbo-preview" since it allows inputs with 128K tokens:


```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.7);

$conf.Hash.elems
```




    22



------

## Themes

Here we extract the themes found in the article and tabulate them (using the prompt ["ThemeTableJSON"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ThemeTableJSON/)):


```raku
my $tblThemes = llm-synthesize(llm-prompt("ThemeTableJSON")($txtArticle, "article", 50), e => $conf, form => sub-parser('JSON'):drop);

$tblThemes.&dimensions;
```




    (12 2)




```raku
#% html
$tblThemes ==> data-translation(field-names=><theme content>)
```




<table border="1"><thead><tr><th>theme</th><th>content</th></tr></thead><tbody><tr><td>Introduction to AI in Science</td><td>Discusses the potential of AI in solving scientific questions and the belief in AI&#39;s eventual capability to do everything, including science.</td></tr><tr><td>AI&#39;s Role and Limitations</td><td>Explores deeper questions about AI in science, its role as a practical tool or a fundamentally new method, and its limitations due to computational irreducibility.</td></tr><tr><td>AI Predictive Capabilities</td><td>Examines AI&#39;s ability to predict outcomes and its reliance on machine learning and neural networks, highlighting limitations in predicting computational processes.</td></tr><tr><td>AI in Identifying Computational Reducibility</td><td>Discusses how AI can assist in identifying pockets of computational reducibility within the broader context of computational irreducibility.</td></tr><tr><td>AI&#39;s Application Beyond Human Tasks</td><td>Considers if AI can understand and predict natural processes directly, beyond emulating human intelligence or tasks.</td></tr><tr><td>Solving Equations with AI</td><td>Explores the potential of AI in solving equations, particularly in areas where traditional methods are impractical or insufficient.</td></tr><tr><td>AI for Multicomputation</td><td>Discusses AI&#39;s ability to navigate multiway systems and its potential in finding paths or solutions in complex computational spaces.</td></tr><tr><td>Exploring Systems with AI</td><td>Looks at how AI can assist in exploring spaces of systems, identifying rules or systems that exhibit specific desired characteristics.</td></tr><tr><td>Science as Narrative</td><td>Explores the idea of science providing a human-accessible narrative for natural phenomena and how AI might generate or contribute to scientific narratives.</td></tr><tr><td>Finding What&#39;s Interesting</td><td>Discusses the challenge of determining what&#39;s interesting in science and how AI might assist in identifying interesting phenomena or directions.</td></tr><tr><td>Beyond Exact Sciences</td><td>Explores the potential of AI in extending the domain of exact sciences to include more subjective or less formalized areas of knowledge.</td></tr><tr><td>Conclusion</td><td>Summarizes the potential and limitations of AI in science, emphasizing the combination of AI with computational paradigms for advancing science.</td></tr></tbody></table>


**Remark:** A fair amount of LLMs give their code results within Markdown code block delimiters (like "```".) Hence, 
(1) the (WL-specified) prompt 
["ThemeTableJSON"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ThemeTableJSON/) 
does not use `Interpreter["JSON"]`, but `Interpreter["String"]`, and 
(2) we use above the sub-parser 'JSON' with dropping of non-JSON strings in order to convert the LLM output into a Raku data structure.

------

## Flowcharts


In this section we LLMs to get [Mermaid-JS](https://mermaid.js.org) flowcharts that correspond to the content of [SW1].

**Remark:** Below in order to display Mermaid-JS diagrams we use both the package ["WWW::MermaidInk"](https://raku.land/zef:antononcube/WWW::MermaidInk), [AAp7], and the dedicated *mermaid magic cell* of Raku Chatabook, [AA6].

### Big picture concepts

Here we generate Mermaid-JS flowchart for the “big picture” concepts:


```raku
my $mmdBigPicture = 
  llm-synthesize([
    "Create a concise mermaid-js graph for the connections between the big concepts in the article:\n\n", 
    $txtArticle, 
    llm-prompt("NothingElse")("correct mermaid-js")
  ], e => $conf)
```




    ```mermaid
    graph TD
        A[AI in Science] --> B[Computational Irreducibility]
        A --> C[Pockets of Computational Reducibility]
        A --> D[AI Measurements]
        A --> E[AI Predictions]
        B --> F[Limitations of AI in Science]
        C --> G[Science Possible]
        D --> H[New Exact Sciences]
        E --> I[Practical Uses in Science]
        G --> J[Discoveries and Formalizations]
        H --> K[Formalizing Unstructured Data]
        I --> L[Combining AI with Computational Paradigm]
        J -.-> M[Computational Paradigm]
        L --> M
    ```



Here we define “big picture” styling theme:


```raku
my $mmdBigPictureTheme = q:to/END/;
%%{init: {'theme': 'neutral'}}%%
END
```




    %%{init: {'theme': 'neutral'}}%%




Here we create the flowchart from LLM’s specification:


```raku
mermaid-ink($mmdBigPictureTheme.chomp ~ $mmdBigPicture.subst(/ '```mermaid' | '```'/, :g), background => 'Cornsilk', format => 'svg')
```




    
![svg](output_28_0.svg)
    



We made several “big picture” flowchart generations. Here is the result of another attempt: 


```raku
#% mermaid
graph TD;
    AI[Artificial Intelligence] --> CompSci[Computational Science]
    AI --> CompThink[Computational Thinking]
    AI --> NewTech[New Technology]
    CompSci --> Physics
    CompSci --> Math[Mathematics]
    CompSci --> Ruliology
    CompThink --> SoftwareDesign[Software Design]
    CompThink --> WolframLang[Wolfram Language]
    NewTech --> WolframAlpha["Wolfram|Alpha"]
    Physics --> Philosophy
    Math --> Philosophy
    Ruliology --> Philosophy
    SoftwareDesign --> Education
    WolframLang --> Education
    WolframAlpha --> Education

    %% Styling
    classDef default fill:#8B0000,stroke:#333,stroke-width:2px;
```




<svg id="mermaid-svg" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 931.242px; background-color: rgb(255, 255, 255);" viewBox="-8 -8 931.2421875 302" role="graphics-document document" aria-roledescription="flowchart-v2" xmlns:xlink="http://www.w3.org/1999/xlink"><style>#mermaid-svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#333;}#mermaid-svg .error-icon{fill:#552222;}#mermaid-svg .error-text{fill:#552222;stroke:#552222;}#mermaid-svg .edge-thickness-normal{stroke-width:2px;}#mermaid-svg .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg .marker{fill:#333333;stroke:#333333;}#mermaid-svg .marker.cross{stroke:#333333;}#mermaid-svg svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#333;}#mermaid-svg .cluster-label text{fill:#333;}#mermaid-svg .cluster-label span,#mermaid-svg p{color:#333;}#mermaid-svg .label text,#mermaid-svg span,#mermaid-svg p{fill:#333;color:#333;}#mermaid-svg .node rect,#mermaid-svg .node circle,#mermaid-svg .node ellipse,#mermaid-svg .node polygon,#mermaid-svg .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#mermaid-svg .flowchart-label text{text-anchor:middle;}#mermaid-svg .node .label{text-align:center;}#mermaid-svg .node.clickable{cursor:pointer;}#mermaid-svg .arrowheadPath{fill:#333333;}#mermaid-svg .edgePath .path{stroke:#333333;stroke-width:2.0px;}#mermaid-svg .flowchart-link{stroke:#333333;fill:none;}#mermaid-svg .edgeLabel{background-color:#e8e8e8;text-align:center;}#mermaid-svg .edgeLabel rect{opacity:0.5;background-color:#e8e8e8;fill:#e8e8e8;}#mermaid-svg .labelBkg{background-color:rgba(232, 232, 232, 0.5);}#mermaid-svg .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#mermaid-svg .cluster text{fill:#333;}#mermaid-svg .cluster span,#mermaid-svg p{color:#333;}#mermaid-svg div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(80, 100%, 96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#mermaid-svg :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg .default&gt;*{fill:#8B0000!important;stroke:#333!important;stroke-width:2px!important;}#mermaid-svg .default span{fill:#8B0000!important;stroke:#333!important;stroke-width:2px!important;}</style><g><marker id="mermaid-svg_flowchart-pointEnd" class="marker flowchart" viewBox="0 0 10 10" refX="6" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-pointStart" class="marker flowchart" viewBox="0 0 10 10" refX="4.5" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-circleEnd" class="marker flowchart" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-svg_flowchart-circleStart" class="marker flowchart" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-svg_flowchart-crossEnd" class="marker cross flowchart" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-crossStart" class="marker cross flowchart" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><g class="root"><g class="clusters"></g><g class="edgePaths"><path d="M476.289,25.947L424.913,31.456C373.536,36.965,270.784,47.982,219.408,56.775C168.031,65.567,168.031,72.133,168.031,75.417L168.031,78.7" id="L-AI-CompSci-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-CompSci" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M559.734,34L559.734,38.167C559.734,42.333,559.734,50.667,559.734,58.117C559.734,65.567,559.734,72.133,559.734,75.417L559.734,78.7" id="L-AI-CompThink-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-CompThink" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M643.18,29.056L677.721,34.047C712.263,39.038,781.346,49.019,815.888,57.293C850.43,65.567,850.43,72.133,850.43,75.417L850.43,78.7" id="L-AI-NewTech-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-NewTech" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M113.224,118L99.791,122.167C86.358,126.333,59.491,134.667,46.058,142.117C32.625,149.567,32.625,156.133,32.625,159.417L32.625,162.7" id="L-CompSci-Physics-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CompSci LE-Physics" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M168.031,118L168.031,122.167C168.031,126.333,168.031,134.667,168.031,142.117C168.031,149.567,168.031,156.133,168.031,159.417L168.031,162.7" id="L-CompSci-Math-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CompSci LE-Math" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M225.76,118L239.91,122.167C254.059,126.333,282.358,134.667,296.507,142.117C310.656,149.567,310.656,156.133,310.656,159.417L310.656,162.7" id="L-CompSci-Ruliology-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CompSci LE-Ruliology" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M521.586,118L512.235,122.167C502.885,126.333,484.185,134.667,474.835,142.117C465.484,149.567,465.484,156.133,465.484,159.417L465.484,162.7" id="L-CompThink-SoftwareDesign-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CompThink LE-SoftwareDesign" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M597.883,118L607.233,122.167C616.584,126.333,635.284,134.667,644.634,142.117C653.984,149.567,653.984,156.133,653.984,159.417L653.984,162.7" id="L-CompThink-WolframLang-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CompThink LE-WolframLang" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M850.43,118L850.43,122.167C850.43,126.333,850.43,134.667,850.43,142.117C850.43,149.567,850.43,156.133,850.43,159.417L850.43,162.7" id="L-NewTech-WolframAlpha-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-NewTech LE-WolframAlpha" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M32.625,202L32.625,206.167C32.625,210.333,32.625,218.667,46.816,227.235C61.008,235.804,89.391,244.608,103.582,249.009L117.774,253.411" id="L-Physics-Philosophy-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Physics LE-Philosophy" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M168.031,202L168.031,206.167C168.031,210.333,168.031,218.667,168.031,226.117C168.031,233.567,168.031,240.133,168.031,243.417L168.031,246.7" id="L-Math-Philosophy-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Math LE-Philosophy" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M310.656,202L310.656,206.167C310.656,210.333,310.656,218.667,295.265,227.366C279.874,236.065,249.093,245.129,233.702,249.661L218.311,254.194" id="L-Ruliology-Philosophy-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Ruliology LE-Philosophy" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M465.484,202L465.484,206.167C465.484,210.333,465.484,218.667,488.889,228.048C512.294,237.43,559.103,247.859,582.508,253.074L605.913,258.289" id="L-SoftwareDesign-Education-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-SoftwareDesign LE-Education" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M653.984,202L653.984,206.167C653.984,210.333,653.984,218.667,653.984,226.117C653.984,233.567,653.984,240.133,653.984,243.417L653.984,246.7" id="L-WolframLang-Education-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-WolframLang LE-Education" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M850.43,202L850.43,206.167C850.43,210.333,850.43,218.667,825.702,228.12C800.975,237.573,751.52,248.147,726.793,253.434L702.066,258.72" id="L-WolframAlpha-Education-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-WolframAlpha LE-Education" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path></g><g class="edgeLabels"><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node default default flowchart-label" id="flowchart-AI-0" transform="translate(559.734375, 17)"><rect class="basic label-container" style="" rx="0" ry="0" x="-83.4453125" y="-17" width="166.890625" height="34"></rect><g class="label" style="" transform="translate(-75.9453125, -9.5)"><rect></rect><foreignObject width="151.890625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Artificial Intelligence</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-CompSci-1" transform="translate(168.03125, 101)"><rect class="basic label-container" style="" rx="0" ry="0" x="-89.6640625" y="-17" width="179.328125" height="34"></rect><g class="label" style="" transform="translate(-82.1640625, -9.5)"><rect></rect><foreignObject width="164.328125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Computational Science</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-CompThink-3" transform="translate(559.734375, 101)"><rect class="basic label-container" style="" rx="0" ry="0" x="-92.7421875" y="-17" width="185.484375" height="34"></rect><g class="label" style="" transform="translate(-85.2421875, -9.5)"><rect></rect><foreignObject width="170.484375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Computational Thinking</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-NewTech-5" transform="translate(850.4296875, 101)"><rect class="basic label-container" style="" rx="0" ry="0" x="-64.8125" y="-17" width="129.625" height="34"></rect><g class="label" style="" transform="translate(-57.3125, -9.5)"><rect></rect><foreignObject width="114.625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">New Technology</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Physics-7" transform="translate(32.625, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-32.625" y="-17" width="65.25" height="34"></rect><g class="label" style="" transform="translate(-25.125, -9.5)"><rect></rect><foreignObject width="50.25" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Physics</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Math-9" transform="translate(168.03125, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-52.78125" y="-17" width="105.5625" height="34"></rect><g class="label" style="" transform="translate(-45.28125, -9.5)"><rect></rect><foreignObject width="90.5625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Mathematics</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Ruliology-11" transform="translate(310.65625, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-39.84375" y="-17" width="79.6875" height="34"></rect><g class="label" style="" transform="translate(-32.34375, -9.5)"><rect></rect><foreignObject width="64.6875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Ruliology</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-SoftwareDesign-13" transform="translate(465.484375, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-64.984375" y="-17" width="129.96875" height="34"></rect><g class="label" style="" transform="translate(-57.484375, -9.5)"><rect></rect><foreignObject width="114.96875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Software Design</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-WolframLang-15" transform="translate(653.984375, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-73.515625" y="-17" width="147.03125" height="34"></rect><g class="label" style="" transform="translate(-66.015625, -9.5)"><rect></rect><foreignObject width="132.03125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Wolfram Language</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-WolframAlpha-17" transform="translate(850.4296875, 185)"><rect class="basic label-container" style="" rx="0" ry="0" x="-61.8203125" y="-17" width="123.640625" height="34"></rect><g class="label" style="" transform="translate(-54.3203125, -9.5)"><rect></rect><foreignObject width="108.640625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Wolfram|Alpha</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Philosophy-19" transform="translate(168.03125, 269)"><rect class="basic label-container" style="" rx="0" ry="0" x="-45.1953125" y="-17" width="90.390625" height="34"></rect><g class="label" style="" transform="translate(-37.6953125, -9.5)"><rect></rect><foreignObject width="75.390625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Philosophy</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Education-25" transform="translate(653.984375, 269)"><rect class="basic label-container" style="" rx="0" ry="0" x="-42.8984375" y="-17" width="85.796875" height="34"></rect><g class="label" style="" transform="translate(-35.3984375, -9.5)"><rect></rect><foreignObject width="70.796875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Education</span></div></foreignObject></g></g></g></g></g><style>@import url("https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css");</style></svg>



### Fine grained

Here we derive a flowchart that refers to more detailed, finer grained concepts:


```raku
my $mmdFineGrained = 
  llm-synthesize([
    "Create a mermaid-js flowchart with multiple blocks and multiple connections for the relationships between concepts in the article:\n\n", 
    $txtArticle, 
    "Use the concepts in the JSON table:", 
    $tblThemes, 
    llm-prompt("NothingElse")("correct mermaid-js")
  ], e => $conf)
```




    ```mermaid
    graph TD
        AI[AI in Science] --> Limitations[Limitations]
        AI --> Applications[Applications]
        AI --> Potential[Potential for Discovery]
        Limitations --> CI[Computational Irreducibility]
        Limitations --> Precision[Requires High Precision]
        Applications --> Predictions[Predictions]
        Applications --> FeatureExtraction[Feature Extraction]
        Applications --> Pruning[Pruning Search Space]
        Potential --> ComputationalParadigm[Computational Paradigm]
        Potential --> IrreducibleComputations[Irreducible Computations]
        CI --> Pockets[Pockets of Computational Reducibility]
        CI --> Surprises[Unexpected Surprises]
        Predictions --> ShallowComputation[Shallow Computation]
        FeatureExtraction --> Measurements[AI Measurements]
        Pruning --> MulticomputationalIrreducibility[Multicomputational Irreducibility]
        ComputationalParadigm --> EnumerateSystems[Enumerate Possible Systems]
        IrreducibleComputations --> FundamentalSurprises[Fundamental Surprises]
        Measurements --> UnstructuredData[Unstructured Raw Data]
        Measurements --> FormalizedScience[Formalized Science]
        UnstructuredData --> MeaningfulFeatures[Extract Meaningful Features]
        FormalizedScience --> ComputationalLanguage[Computational Language]
    ```



Here we define “fine grained” styling theme:


```raku
my $mmdFineGrainedTheme = q:to/END/;
%%{init: {'theme': 'base','themeVariables': {'backgroundColor': '#FFF'}}}%%
END
```




    %%{init: {'theme': 'base','themeVariables': {'backgroundColor': '#FFF'}}}%%




Here we create the flowchart from LLM’s specification:


```raku
mermaid-ink($mmdFineGrainedTheme.chomp ~ $mmdFineGrained.subst(/ '```mermaid' | '```'/, :g), format => 'svg')
```




    
![svg](output_36_0.svg)
    



We made several “fine grained” flowchart generations. Here is the result of another attempt:


```raku
#% mermaid
graph TD

    AI["AI"] -->|Potential & Limitations| Science["Science"]
    AI -->|Leverages| CR["Computational Reducibility"]
    AI -->|Fails with| CI["Computational Irreducibility"]
    
    Science -->|Domain of| PS["Physical Sciences"]
    Science -->|Expanding to| S["'Exact' Sciences Beyond Traditional Domains"]
    Science -.->|Foundational Role of| CR
    Science -.->|Limited by| CI
    
    PS -->|Traditional Formalizations via| Math["Mathematics/Mathematical Formulas"]
    PS -->|Now Leveraging| AI_Measurements["AI Measurements"]
    
    S -->|Formalizing with| CL["Computational Language"]
    S -->|Leverages| AI_Measurements
    S -->|Future Frontiers with| AI
    
    AI_Measurements -->|Interpretation Challenge| BlackBox["'Black-Box' Nature"]
    AI_Measurements -->|Potential for| NewScience["New Science Discoveries"]
    
    CL -->|Key to Structuring| AI_Results["AI Results"]
    CL -->|Enables Irreducible Computations for| Discovery["Discovery"]
    
    Math -.->|Transitioning towards| CL
    Math -->|Limits when needing 'Precision'| AI_Limits["AI's Limitations"]
    
    Discovery -.->|Not directly achievable via| AI
    
    BlackBox -->|Requires Human| Interpretation["Interpretation"]
    
    CR -->|Empowered by| AI & ML_Techniques["AI & Machine Learning Techniques"]
    CI -.->|Challenge for| AI & ML_Techniques
   
    PS --> Observations["New Observations/Measurements"] --> NewDirections["New Scientific Directions"]
    Observations --> AI_InterpretedPredictions["AI-Interpreted Predictions"]
    NewDirections -.-> AI_Predictions["AI Predictions"] -.-> CI
    NewDirections --> AI_Discoveries["AI-Enabled Discoveries"] -.-> CR

    AI_Discoveries --> NewParadigms["New Paradigms/Concepts"] -.-> S
    AI_InterpretedPredictions -.-> AI_Measurements

    %% Styling
    classDef default fill:#f9f,stroke:#333,stroke-width:2px;
    classDef highlight fill:#bbf,stroke:#006,stroke-width:4px;
    classDef imp fill:#ffb,stroke:#330,stroke-width:4px;
    class PS,CL highlight;
    class AI_Discoveries,NewParadigms imp;
```




<svg id="mermaid-svg" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 2489.69px; background-color: rgb(255, 255, 255);" viewBox="-8 -8 2489.6875 1042" role="graphics-document document" aria-roledescription="flowchart-v2" xmlns:xlink="http://www.w3.org/1999/xlink"><style>#mermaid-svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#333;}#mermaid-svg .error-icon{fill:#552222;}#mermaid-svg .error-text{fill:#552222;stroke:#552222;}#mermaid-svg .edge-thickness-normal{stroke-width:2px;}#mermaid-svg .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg .marker{fill:#333333;stroke:#333333;}#mermaid-svg .marker.cross{stroke:#333333;}#mermaid-svg svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#333;}#mermaid-svg .cluster-label text{fill:#333;}#mermaid-svg .cluster-label span,#mermaid-svg p{color:#333;}#mermaid-svg .label text,#mermaid-svg span,#mermaid-svg p{fill:#333;color:#333;}#mermaid-svg .node rect,#mermaid-svg .node circle,#mermaid-svg .node ellipse,#mermaid-svg .node polygon,#mermaid-svg .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#mermaid-svg .flowchart-label text{text-anchor:middle;}#mermaid-svg .node .label{text-align:center;}#mermaid-svg .node.clickable{cursor:pointer;}#mermaid-svg .arrowheadPath{fill:#333333;}#mermaid-svg .edgePath .path{stroke:#333333;stroke-width:2.0px;}#mermaid-svg .flowchart-link{stroke:#333333;fill:none;}#mermaid-svg .edgeLabel{background-color:#e8e8e8;text-align:center;}#mermaid-svg .edgeLabel rect{opacity:0.5;background-color:#e8e8e8;fill:#e8e8e8;}#mermaid-svg .labelBkg{background-color:rgba(232, 232, 232, 0.5);}#mermaid-svg .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#mermaid-svg .cluster text{fill:#333;}#mermaid-svg .cluster span,#mermaid-svg p{color:#333;}#mermaid-svg div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(80, 100%, 96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#mermaid-svg :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg .default&gt;*{fill:#f9f!important;stroke:#333!important;stroke-width:2px!important;}#mermaid-svg .default span{fill:#f9f!important;stroke:#333!important;stroke-width:2px!important;}#mermaid-svg .highlight&gt;*{fill:#bbf!important;stroke:#006!important;stroke-width:4px!important;}#mermaid-svg .highlight span{fill:#bbf!important;stroke:#006!important;stroke-width:4px!important;}#mermaid-svg .imp&gt;*{fill:#ffb!important;stroke:#330!important;stroke-width:4px!important;}#mermaid-svg .imp span{fill:#ffb!important;stroke:#330!important;stroke-width:4px!important;}</style><g><marker id="mermaid-svg_flowchart-pointEnd" class="marker flowchart" viewBox="0 0 10 10" refX="6" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-pointStart" class="marker flowchart" viewBox="0 0 10 10" refX="4.5" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-circleEnd" class="marker flowchart" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-svg_flowchart-circleStart" class="marker flowchart" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-svg_flowchart-crossEnd" class="marker cross flowchart" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-svg_flowchart-crossStart" class="marker cross flowchart" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><g class="root"><g class="clusters"></g><g class="edgePaths"><path d="M2075.465,18.157L1970.729,26.548C1865.993,34.938,1656.522,51.719,1551.786,64.976C1447.051,78.233,1447.051,87.967,1447.051,92.833L1447.051,97.7" id="L-AI-Science-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-Science" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M2075.465,19.133L2019.746,27.361C1964.027,35.589,1852.59,52.044,1796.871,68.856C1741.152,85.667,1741.152,102.833,1741.152,120C1741.152,137.167,1741.152,154.333,1741.152,171.5C1741.152,188.667,1741.152,205.833,1741.152,221.417C1741.152,237,1741.152,251,1741.152,265C1741.152,279,1741.152,293,1741.152,308.583C1741.152,324.167,1741.152,341.333,1741.152,358.5C1741.152,375.667,1741.152,392.833,1741.152,410C1741.152,427.167,1741.152,444.333,1741.152,461.5C1741.152,478.667,1741.152,495.833,1741.152,511.417C1741.152,527,1741.152,541,1729.179,551.893C1717.206,562.787,1693.26,570.574,1681.287,574.467L1669.314,578.361" id="L-AI-CR-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-CR" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M2075.465,21.328L2049.226,29.19C2022.987,37.052,1970.509,52.776,1944.27,69.221C1918.031,85.667,1918.031,102.833,1918.031,120C1918.031,137.167,1918.031,154.333,1918.031,171.5C1918.031,188.667,1918.031,205.833,1918.031,221.417C1918.031,237,1918.031,251,1918.031,265C1918.031,279,1918.031,293,1918.031,308.583C1918.031,324.167,1918.031,341.333,1918.031,358.5C1918.031,375.667,1918.031,392.833,1918.031,410C1918.031,427.167,1918.031,444.333,1918.031,461.5C1918.031,478.667,1918.031,495.833,1918.031,511.417C1918.031,527,1918.031,541,1914.455,551.545C1910.879,562.09,1903.727,569.179,1900.151,572.724L1896.574,576.269" id="L-AI-CI-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI LE-CI" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1412.402,122.102L1276.689,130.335C1140.975,138.568,869.548,155.034,733.835,168.134C598.121,181.233,598.121,190.967,598.121,195.833L598.121,200.7" id="L-Science-PS-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Science LE-PS" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1412.402,135.23L1398.65,141.275C1384.897,147.32,1357.392,159.41,1343.639,174.038C1329.887,188.667,1329.887,205.833,1329.887,221.417C1329.887,237,1329.887,251,1329.887,265C1329.887,279,1329.887,293,1329.887,308.583C1329.887,324.167,1329.887,341.333,1329.887,358.5C1329.887,375.667,1329.887,392.833,1329.887,410C1329.887,427.167,1329.887,444.333,1329.887,461.5C1329.887,478.667,1329.887,495.833,1329.887,511.417C1329.887,527,1329.887,541,1329.887,555C1329.887,569,1329.887,583,1329.887,598.583C1329.887,614.167,1329.887,631.333,1324.521,645.056C1319.155,658.778,1308.423,669.056,1303.057,674.195L1297.691,679.334" id="L-Science-S-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Science LE-S" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1459.654,137L1463.917,142.75C1468.179,148.5,1476.705,160,1480.968,174.333C1485.23,188.667,1485.23,205.833,1485.23,221.417C1485.23,237,1485.23,251,1485.23,265C1485.23,279,1485.23,293,1485.23,308.583C1485.23,324.167,1485.23,341.333,1485.23,358.5C1485.23,375.667,1485.23,392.833,1485.23,410C1485.23,427.167,1485.23,444.333,1485.23,461.5C1485.23,478.667,1485.23,495.833,1485.23,511.417C1485.23,527,1485.23,541,1496.968,551.889C1508.705,562.778,1532.18,570.555,1543.918,574.444L1555.655,578.333" id="L-Science-CR-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-Science LE-CR" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1481.699,127.56L1515.264,134.883C1548.829,142.206,1615.96,156.853,1649.525,172.76C1683.09,188.667,1683.09,205.833,1683.09,221.417C1683.09,237,1683.09,251,1683.09,265C1683.09,279,1683.09,293,1683.09,308.583C1683.09,324.167,1683.09,341.333,1683.09,358.5C1683.09,375.667,1683.09,392.833,1683.09,410C1683.09,427.167,1683.09,444.333,1683.09,461.5C1683.09,478.667,1683.09,495.833,1683.09,511.417C1683.09,527,1683.09,541,1701.331,551.978C1719.572,562.957,1756.054,570.914,1774.296,574.892L1792.537,578.871" id="L-Science-CI-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-Science LE-CI" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M529.379,230.765L478.869,236.471C428.359,242.177,327.34,253.588,276.83,266.294C226.32,279,226.32,293,226.32,308.583C226.32,324.167,226.32,341.333,226.32,358.5C226.32,375.667,226.32,392.833,226.32,410C226.32,427.167,226.32,444.333,226.32,461.5C226.32,478.667,226.32,495.833,226.32,511.417C226.32,527,226.32,541,226.32,555C226.32,569,226.32,583,226.32,598.583C226.32,614.167,226.32,631.333,226.32,644.783C226.32,658.233,226.32,667.967,226.32,672.833L226.32,677.7" id="L-PS-Math-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-PS LE-Math" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M541.937,240L528.166,244.167C514.395,248.333,486.854,256.667,473.083,267.833C459.313,279,459.313,293,459.313,308.583C459.313,324.167,459.313,341.333,459.313,358.5C459.313,375.667,459.313,392.833,459.313,410C459.313,427.167,459.313,444.333,459.313,461.5C459.313,478.667,459.313,495.833,459.313,511.417C459.313,527,459.313,541,459.313,555C459.313,569,459.313,583,459.313,598.583C459.313,614.167,459.313,631.333,459.313,648.5C459.313,665.667,459.313,682.833,459.313,700C459.313,717.167,459.313,734.333,565.989,750.619C672.665,766.904,886.018,782.307,992.694,790.009L1099.37,797.711" id="L-PS-AI_Measurements-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-PS LE-AI_Measurements" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1226.897,717L1210.25,722.75C1193.603,728.5,1160.309,740,1071.376,752.707C982.443,765.414,837.871,779.328,765.585,786.285L693.299,793.242" id="L-S-CL-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-S LE-CL" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1407.62,717L1452.1,722.75C1496.58,728.5,1585.54,740,1558.581,753.081C1531.622,766.161,1388.744,780.823,1317.305,788.154L1245.866,795.484" id="L-S-AI_Measurements-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-S LE-AI_Measurements" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1440.066,692.47L1599.617,685.141C1759.167,677.813,2078.267,663.157,2237.817,647.245C2397.367,631.333,2397.367,614.167,2397.367,598.583C2397.367,583,2397.367,569,2397.367,555C2397.367,541,2397.367,527,2397.367,511.417C2397.367,495.833,2397.367,478.667,2397.367,461.5C2397.367,444.333,2397.367,427.167,2397.367,410C2397.367,392.833,2397.367,375.667,2397.367,358.5C2397.367,341.333,2397.367,324.167,2397.367,308.583C2397.367,293,2397.367,279,2397.367,265C2397.367,251,2397.367,237,2397.367,221.417C2397.367,205.833,2397.367,188.667,2397.367,171.5C2397.367,154.333,2397.367,137.167,2397.367,120C2397.367,102.833,2397.367,85.667,2349.403,69.049C2301.439,52.432,2205.511,36.363,2157.547,28.329L2109.583,20.295" id="L-S-AI-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-S LE-AI" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1114.568,820L1094.931,825.75C1075.294,831.5,1036.02,843,994.651,854.282C953.282,865.564,909.817,876.628,888.085,882.16L866.353,887.693" id="L-AI_Measurements-BlackBox-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI_Measurements LE-BlackBox" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1240.594,810.041L1312.124,817.451C1383.655,824.861,1526.716,839.68,1660.722,853.864C1794.728,868.048,1919.679,881.597,1982.154,888.371L2044.629,895.145" id="L-AI_Measurements-NewScience-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI_Measurements LE-NewScience" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M555.944,820L543.777,825.75C531.611,831.5,507.278,843,495.112,853.617C482.945,864.233,482.945,873.967,482.945,878.833L482.945,883.7" id="L-CL-AI_Results-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CL LE-AI_Results" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M645.84,820L664.079,825.75C682.319,831.5,718.798,843,820.486,856.6C922.173,870.199,1089.069,885.899,1172.517,893.749L1255.965,901.598" id="L-CL-Discovery-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CL LE-Discovery" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M176.896,717L160.179,722.75C143.462,728.5,110.028,740,162.3,752.645C214.573,765.289,352.552,779.079,421.541,785.973L490.531,792.868" id="L-Math-CL-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-Math LE-CL" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M245.329,717L251.759,722.75C258.188,728.5,271.047,740,260.307,751.231C249.566,762.463,215.226,773.425,198.056,778.907L180.886,784.388" id="L-Math-AI_Limits-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Math LE-AI_Limits" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1344.273,903.191L1464.23,895.076C1584.188,886.961,1824.102,870.73,1944.059,854.032C2064.016,837.333,2064.016,820.167,2064.016,803C2064.016,785.833,2064.016,768.667,2064.016,751.5C2064.016,734.333,2064.016,717.167,2064.016,700C2064.016,682.833,2064.016,665.667,2064.016,648.5C2064.016,631.333,2064.016,614.167,2064.016,598.583C2064.016,583,2064.016,569,2064.016,555C2064.016,541,2064.016,527,2064.016,511.417C2064.016,495.833,2064.016,478.667,2064.016,461.5C2064.016,444.333,2064.016,427.167,2064.016,410C2064.016,392.833,2064.016,375.667,2064.016,358.5C2064.016,341.333,2064.016,324.167,2064.016,308.583C2064.016,293,2064.016,279,2064.016,265C2064.016,251,2064.016,237,2064.016,221.417C2064.016,205.833,2064.016,188.667,2064.016,171.5C2064.016,154.333,2064.016,137.167,2064.016,120C2064.016,102.833,2064.016,85.667,2066.51,72.123C2069.004,58.578,2073.993,48.657,2076.487,43.696L2078.982,38.735" id="L-Discovery-AI-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-Discovery LE-AI" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M794.434,923L794.434,928.75C794.434,934.5,794.434,946,794.434,956.617C794.434,967.233,794.434,976.967,794.434,981.833L794.434,986.7" id="L-BlackBox-Interpretation-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-BlackBox LE-Interpretation" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1716.77,590.614L1814.149,584.678C1911.529,578.742,2106.288,566.871,2203.667,553.936C2301.047,541,2301.047,527,2301.047,511.417C2301.047,495.833,2301.047,478.667,2301.047,461.5C2301.047,444.333,2301.047,427.167,2301.047,410C2301.047,392.833,2301.047,375.667,2301.047,358.5C2301.047,341.333,2301.047,324.167,2301.047,308.583C2301.047,293,2301.047,279,2301.047,265C2301.047,251,2301.047,237,2301.047,221.417C2301.047,205.833,2301.047,188.667,2301.047,171.5C2301.047,154.333,2301.047,137.167,2301.047,120C2301.047,102.833,2301.047,85.667,2269.123,69.297C2237.199,52.926,2173.352,37.353,2141.428,29.566L2109.505,21.779" id="L-CR-AI-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CR LE-AI" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1611.996,614L1611.996,619.75C1611.996,625.5,1611.996,637,1625.892,648.179C1639.789,659.357,1667.581,670.214,1681.478,675.643L1695.374,681.071" id="L-CR-ML_Techniques-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-CR LE-ML_Techniques" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1984.551,581.99L2017.184,577.492C2049.818,572.993,2115.085,563.997,2147.718,552.498C2180.352,541,2180.352,527,2180.352,511.417C2180.352,495.833,2180.352,478.667,2180.352,461.5C2180.352,444.333,2180.352,427.167,2180.352,410C2180.352,392.833,2180.352,375.667,2180.352,358.5C2180.352,341.333,2180.352,324.167,2180.352,308.583C2180.352,293,2180.352,279,2180.352,265C2180.352,251,2180.352,237,2180.352,221.417C2180.352,205.833,2180.352,188.667,2180.352,171.5C2180.352,154.333,2180.352,137.167,2180.352,120C2180.352,102.833,2180.352,85.667,2168.453,70.308C2156.555,54.949,2132.758,41.399,2120.86,34.623L2108.961,27.848" id="L-CI-AI-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-CI LE-AI" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M1875.66,614L1875.66,619.75C1875.66,625.5,1875.66,637,1861.764,648.179C1847.867,659.357,1820.075,670.214,1806.179,675.643L1792.282,681.071" id="L-CI-ML_Techniques-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-CI LE-ML_Techniques" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M666.863,231.561L711.611,237.135C756.359,242.708,845.855,253.854,890.604,262.71C935.352,271.567,935.352,278.133,935.352,281.417L935.352,284.7" id="L-PS-Observations-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-PS LE-Observations" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M891.315,324L876.42,329.75C861.525,335.5,831.735,347,816.84,357.617C801.945,368.233,801.945,377.967,801.945,382.833L801.945,387.7" id="L-Observations-NewDirections-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Observations LE-NewDirections" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M935.352,324L935.352,329.75C935.352,335.5,935.352,347,935.352,361.333C935.352,375.667,935.352,392.833,935.352,410C935.352,427.167,935.352,444.333,935.352,461.5C935.352,478.667,935.352,495.833,935.352,511.417C935.352,527,935.352,541,935.352,555C935.352,569,935.352,583,935.352,598.583C935.352,614.167,935.352,631.333,932.303,644.913C929.254,658.492,923.157,668.484,920.109,673.48L917.06,678.476" id="L-Observations-AI_InterpretedPredictions-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-Observations LE-AI_InterpretedPredictions" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M737.672,427L715.932,432.75C694.193,438.5,650.714,450,628.974,460.617C607.234,471.233,607.234,480.967,607.234,485.833L607.234,490.7" id="L-NewDirections-AI_Predictions-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-NewDirections LE-AI_Predictions" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M607.234,530L607.234,534.167C607.234,538.333,607.234,546.667,799.607,557.203C991.98,567.74,1376.726,580.479,1569.099,586.849L1761.472,593.219" id="L-AI_Predictions-CI-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-AI_Predictions LE-CI" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M801.945,427L801.945,432.75C801.945,438.5,801.945,450,801.945,460.617C801.945,471.233,801.945,480.967,801.945,485.833L801.945,490.7" id="L-NewDirections-AI_Discoveries-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-NewDirections LE-AI_Discoveries" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M760.326,530L750.125,534.167C739.924,538.333,719.523,546.667,843.123,556.989C966.723,567.312,1234.326,579.624,1368.127,585.78L1501.928,591.936" id="L-AI_Discoveries-CR-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-AI_Discoveries LE-CR" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M801.945,530L801.945,534.167C801.945,538.333,801.945,546.667,801.945,554.117C801.945,561.567,801.945,568.133,801.945,571.417L801.945,574.7" id="L-AI_Discoveries-NewParadigms-0" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-AI_Discoveries LE-NewParadigms" style="fill:none;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M801.945,614L801.945,619.75C801.945,625.5,801.945,637,854.008,648.405C906.071,659.809,1010.197,671.118,1062.26,676.773L1114.323,682.428" id="L-NewParadigms-S-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-NewParadigms LE-S" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path><path d="M903.926,717L903.926,722.75C903.926,728.5,903.926,740,936.513,751.996C969.101,763.992,1034.276,776.483,1066.863,782.729L1099.451,788.975" id="L-AI_InterpretedPredictions-AI_Measurements-0" class=" edge-thickness-normal edge-pattern-dotted flowchart-link LS-AI_InterpretedPredictions LE-AI_Measurements" style="fill:none;stroke-width:2px;stroke-dasharray:3;" marker-end="url(#mermaid-svg_flowchart-pointEnd)"></path></g><g class="edgeLabels"><g class="edgeLabel" transform="translate(1447.05078125, 68.5)"><g class="label" transform="translate(-82.7578125, -9.5)"><foreignObject width="165.515625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Potential &amp; Limitations</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1741.15234375, 307)"><g class="label" transform="translate(-35.625, -9.5)"><foreignObject width="71.25" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Leverages</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1918.03125, 307)"><g class="label" transform="translate(-34.46875, -9.5)"><foreignObject width="68.9375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Fails with</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(598.12109375, 171.5)"><g class="label" transform="translate(-36.359375, -9.5)"><foreignObject width="72.71875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Domain of</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1329.88671875, 410)"><g class="label" transform="translate(-46.328125, -9.5)"><foreignObject width="92.65625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Expanding to</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1485.23046875, 358.5)"><g class="label" transform="translate(-73.9921875, -9.5)"><foreignObject width="147.984375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Foundational Role of</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1683.08984375, 358.5)"><g class="label" transform="translate(-38.0625, -9.5)"><foreignObject width="76.125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Limited by</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(226.3203125, 461.5)"><g class="label" transform="translate(-106.1640625, -9.5)"><foreignObject width="212.328125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Traditional Formalizations via</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(459.3125, 513)"><g class="label" transform="translate(-56.453125, -9.5)"><foreignObject width="112.90625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Now Leveraging</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1127.015625, 751.5)"><g class="label" transform="translate(-59.7421875, -9.5)"><foreignObject width="119.484375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Formalizing with</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1674.5, 751.5)"><g class="label" transform="translate(-35.625, -9.5)"><foreignObject width="71.25" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Leverages</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(2397.3671875, 358.5)"><g class="label" transform="translate(-76.3203125, -9.5)"><foreignObject width="152.640625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Future Frontiers with</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(996.74609375, 854.5)"><g class="label" transform="translate(-88.265625, -9.5)"><foreignObject width="176.53125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Interpretation Challenge</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1669.77734375, 854.5)"><g class="label" transform="translate(-45.0703125, -9.5)"><foreignObject width="90.140625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Potential for</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(482.9453125, 854.5)"><g class="label" transform="translate(-64.734375, -9.5)"><foreignObject width="129.46875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Key to Structuring</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(755.27734375, 854.5)"><g class="label" transform="translate(-133.203125, -9.5)"><foreignObject width="266.40625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Enables Irreducible Computations for</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(76.59375, 751.5)"><g class="label" transform="translate(-76.59375, -9.5)"><foreignObject width="153.1875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Transitioning towards</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(283.90625, 751.5)"><g class="label" transform="translate(-110.71875, -9.5)"><foreignObject width="221.4375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Limits when needing 'Precision'</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(2064.015625, 461.5)"><g class="label" transform="translate(-96.3359375, -9.5)"><foreignObject width="192.671875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Not directly achievable via</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(794.43359375, 957.5)"><g class="label" transform="translate(-57.75, -9.5)"><foreignObject width="115.5" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Requires Human</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(2301.046875, 307)"><g class="label" transform="translate(-52.734375, -9.5)"><foreignObject width="105.46875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Empowered by</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1611.99609375, 648.5)"><g class="label" transform="translate(-52.734375, -9.5)"><foreignObject width="105.46875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Empowered by</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(2180.3515625, 307)"><g class="label" transform="translate(-47.9609375, -9.5)"><foreignObject width="95.921875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Challenge for</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1875.66015625, 648.5)"><g class="label" transform="translate(-47.9609375, -9.5)"><foreignObject width="95.921875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">Challenge for</span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node default default flowchart-label" id="flowchart-AI-0" transform="translate(2089.91015625, 17)"><rect class="basic label-container" style="" rx="0" ry="0" x="-14.4453125" y="-17" width="28.890625" height="34"></rect><g class="label" style="" transform="translate(-6.9453125, -9.5)"><rect></rect><foreignObject width="13.890625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Science-1" transform="translate(1447.05078125, 120)"><rect class="basic label-container" style="" rx="0" ry="0" x="-34.6484375" y="-17" width="69.296875" height="34"></rect><g class="label" style="" transform="translate(-27.1484375, -9.5)"><rect></rect><foreignObject width="54.296875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Science</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-CR-3" transform="translate(1611.99609375, 597)"><rect class="basic label-container" style="" rx="0" ry="0" x="-104.7734375" y="-17" width="209.546875" height="34"></rect><g class="label" style="" transform="translate(-97.2734375, -9.5)"><rect></rect><foreignObject width="194.546875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Computational Reducibility</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-CI-5" transform="translate(1875.66015625, 597)"><rect class="basic label-container" style="" rx="0" ry="0" x="-108.890625" y="-17" width="217.78125" height="34"></rect><g class="label" style="" transform="translate(-101.390625, -9.5)"><rect></rect><foreignObject width="202.78125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Computational Irreducibility</span></div></foreignObject></g></g><g class="node default highlight flowchart-label" id="flowchart-PS-7" transform="translate(598.12109375, 223)"><rect class="basic label-container" style="" rx="0" ry="0" x="-68.7421875" y="-17" width="137.484375" height="34"></rect><g class="label" style="" transform="translate(-61.2421875, -9.5)"><rect></rect><foreignObject width="122.484375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Physical Sciences</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-S-9" transform="translate(1276.11328125, 700)"><rect class="basic label-container" style="" rx="0" ry="0" x="-163.953125" y="-17" width="327.90625" height="34"></rect><g class="label" style="" transform="translate(-156.453125, -9.5)"><rect></rect><foreignObject width="312.90625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">'Exact' Sciences Beyond Traditional Domains</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Math-15" transform="translate(226.3203125, 700)"><rect class="basic label-container" style="" rx="0" ry="0" x="-140.40625" y="-17" width="280.8125" height="34"></rect><g class="label" style="" transform="translate(-132.90625, -9.5)"><rect></rect><foreignObject width="265.8125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Mathematics/Mathematical Formulas</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-AI_Measurements-17" transform="translate(1172.625, 803)"><rect class="basic label-container" style="" rx="0" ry="0" x="-67.96875" y="-17" width="135.9375" height="34"></rect><g class="label" style="" transform="translate(-60.46875, -9.5)"><rect></rect><foreignObject width="120.9375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI Measurements</span></div></foreignObject></g></g><g class="node default highlight flowchart-label" id="flowchart-CL-19" transform="translate(591.9140625, 803)"><rect class="basic label-container" style="" rx="0" ry="0" x="-96.109375" y="-17" width="192.21875" height="34"></rect><g class="label" style="" transform="translate(-88.609375, -9.5)"><rect></rect><foreignObject width="177.21875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Computational Language</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-BlackBox-25" transform="translate(794.43359375, 906)"><rect class="basic label-container" style="" rx="0" ry="0" x="-71.640625" y="-17" width="143.28125" height="34"></rect><g class="label" style="" transform="translate(-64.140625, -9.5)"><rect></rect><foreignObject width="128.28125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">'Black-Box' Nature</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-NewScience-27" transform="translate(2144.7421875, 906)"><rect class="basic label-container" style="" rx="0" ry="0" x="-94.84375" y="-17" width="189.6875" height="34"></rect><g class="label" style="" transform="translate(-87.34375, -9.5)"><rect></rect><foreignObject width="174.6875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">New Science Discoveries</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-AI_Results-29" transform="translate(482.9453125, 906)"><rect class="basic label-container" style="" rx="0" ry="0" x="-41.9296875" y="-17" width="83.859375" height="34"></rect><g class="label" style="" transform="translate(-34.4296875, -9.5)"><rect></rect><foreignObject width="68.859375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI Results</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Discovery-31" transform="translate(1302.7578125, 906)"><rect class="basic label-container" style="" rx="0" ry="0" x="-41.515625" y="-17" width="83.03125" height="34"></rect><g class="label" style="" transform="translate(-34.015625, -9.5)"><rect></rect><foreignObject width="68.03125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Discovery</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-AI_Limits-35" transform="translate(122.5859375, 803)"><rect class="basic label-container" style="" rx="0" ry="0" x="-61.359375" y="-17" width="122.71875" height="34"></rect><g class="label" style="" transform="translate(-53.859375, -9.5)"><rect></rect><foreignObject width="107.71875" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI's Limitations</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Interpretation-39" transform="translate(794.43359375, 1009)"><rect class="basic label-container" style="" rx="0" ry="0" x="-58.1640625" y="-17" width="116.328125" height="34"></rect><g class="label" style="" transform="translate(-50.6640625, -9.5)"><rect></rect><foreignObject width="101.328125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Interpretation</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-ML_Techniques-42" transform="translate(1743.828125, 700)"><rect class="basic label-container" style="" rx="0" ry="0" x="-129.0078125" y="-17" width="258.015625" height="34"></rect><g class="label" style="" transform="translate(-121.5078125, -9.5)"><rect></rect><foreignObject width="243.015625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI &amp; Machine Learning Techniques</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-Observations-47" transform="translate(935.3515625, 307)"><rect class="basic label-container" style="" rx="0" ry="0" x="-126.671875" y="-17" width="253.34375" height="34"></rect><g class="label" style="" transform="translate(-119.171875, -9.5)"><rect></rect><foreignObject width="238.34375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">New Observations/Measurements</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-NewDirections-48" transform="translate(801.9453125, 410)"><rect class="basic label-container" style="" rx="0" ry="0" x="-97.1953125" y="-17" width="194.390625" height="34"></rect><g class="label" style="" transform="translate(-89.6953125, -9.5)"><rect></rect><foreignObject width="179.390625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">New Scientific Directions</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-AI_InterpretedPredictions-50" transform="translate(903.92578125, 700)"><rect class="basic label-container" style="" rx="0" ry="0" x="-100.5703125" y="-17" width="201.140625" height="34"></rect><g class="label" style="" transform="translate(-93.0703125, -9.5)"><rect></rect><foreignObject width="186.140625" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI-Interpreted Predictions</span></div></foreignObject></g></g><g class="node default default flowchart-label" id="flowchart-AI_Predictions-52" transform="translate(607.234375, 513)"><rect class="basic label-container" style="" rx="0" ry="0" x="-56.46875" y="-17" width="112.9375" height="34"></rect><g class="label" style="" transform="translate(-48.96875, -9.5)"><rect></rect><foreignObject width="97.9375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI Predictions</span></div></foreignObject></g></g><g class="node default imp flowchart-label" id="flowchart-AI_Discoveries-55" transform="translate(801.9453125, 513)"><rect class="basic label-container" style="" rx="0" ry="0" x="-88.2421875" y="-17" width="176.484375" height="34"></rect><g class="label" style="" transform="translate(-80.7421875, -9.5)"><rect></rect><foreignObject width="161.484375" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">AI-Enabled Discoveries</span></div></foreignObject></g></g><g class="node default imp flowchart-label" id="flowchart-NewParadigms-58" transform="translate(801.9453125, 597)"><rect class="basic label-container" style="" rx="0" ry="0" x="-98.40625" y="-17" width="196.8125" height="34"></rect><g class="label" style="" transform="translate(-90.90625, -9.5)"><rect></rect><foreignObject width="181.8125" height="19"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">New Paradigms/Concepts</span></div></foreignObject></g></g></g></g></g><style>@import url("https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css");</style></svg>



-------

## Summary and ideas

Here we get a summary and extract ideas, quotes, and references from the article:


```raku
my $sumIdea =llm-synthesize(llm-prompt("ExtractArticleWisdom")($txtArticle), e => $conf);

text-stats($sumIdea)
```




    (chars => 7386 words => 1047 lines => 78)



The result is rendered below.

<hr width="65%">


```raku
#% markdown
$sumIdea.subst(/ ^^ '#' /, '###', :g)
```




#### SUMMARY

Stephen Wolfram's writings explore the capabilities and limitations of AI in the realm of science, discussing how AI can assist in scientific discovery and understanding but also highlighting its limitations due to computational irreducibility and the need for human interpretation and creativity.

#### IDEAS:

- AI has made surprising successes but cannot solve all scientific problems due to computational irreducibility.
- Large Language Models (LLMs) provide a new kind of interface for scientific work, offering high-level autocomplete for scientific knowledge.
- The transition to computational representation of the world is transforming science, with AI playing a significant role in accessing and exploring this computational realm.
- AI, particularly through neural networks and machine learning, offers tools for predicting scientific outcomes, though its effectiveness is bounded by the complexity of the systems it attempts to model.
- Computational irreducibility limits the ability of AI to predict or solve all scientific phenomena, ensuring that surprises and new discoveries remain a fundamental aspect of science.
- Despite AI's limitations, it has potential in identifying pockets of computational reducibility, streamlining the discovery of scientific knowledge.
- AI's success in areas like visual recognition and language generation suggests potential for contributing to scientific methodologies and understanding, though its ability to directly engage with raw natural processes is less certain.
- AI techniques, including neural networks and machine learning, have shown promise in areas like solving equations and exploring multicomputational processes, but face challenges due to computational irreducibility.
- The role of AI in generating human-understandable narratives for scientific phenomena is explored, highlighting the potential for AI to assist in identifying interesting and meaningful scientific inquiries.
- The exploration of "AI measurements" opens up new possibilities for formalizing and quantifying aspects of science that have traditionally been qualitative or subjective, potentially expanding the domain of exact sciences.

#### QUOTES:

- "AI has the potential to give us streamlined ways to find certain kinds of pockets of computational reducibility."
- "Computational irreducibility is what will prevent us from ever being able to completely 'solve science'."
- "The AI is doing 'shallow computation', but when there’s computational irreducibility one needs irreducible, deep computation to work out what will happen."
- "AI measurements are potentially a much richer source of formalizable material."
- "AI... is not something built to 'go out into the wilds of the ruliad', far from anything already connected to humans."
- "Despite AI's limitations, it has potential in identifying pockets of computational reducibility."
- "AI techniques... have shown promise in areas like solving equations and exploring multicomputational processes."
- "AI's success in areas like visual recognition and language generation suggests potential for contributing to scientific methodologies."
- "There’s no abstract notion of 'interestingness' that an AI or anything can 'go out and discover' ahead of our choices."
- "The whole story of things like trained neural nets that we’ve discussed here is a story of leveraging computational reducibility."

#### HABITS:

- Continuously exploring the capabilities and limitations of AI in scientific discovery.
- Engaging in systematic experimentation to understand how AI tools can assist in scientific processes.
- Seeking to identify and utilize pockets of computational reducibility where AI can be most effective.
- Exploring the use of neural networks and machine learning for predicting and solving scientific problems.
- Investigating the potential for AI to assist in creating human-understandable narratives for complex scientific phenomena.
- Experimenting with "AI measurements" to quantify and formalize traditionally qualitative aspects of science.
- Continuously refining and expanding computational language to better interface with AI capabilities.
- Engaging with and contributing to discussions on the role of AI in the future of science and human understanding.
- Actively seeking new methodologies and innovations in AI that could impact scientific discovery.
- Evaluating the potential for AI to identify interesting and meaningful scientific inquiries through analysis of large datasets.

#### FACTS:

- Computational irreducibility ensures that surprises and new discoveries remain a fundamental aspect of science.
- AI's effectiveness in scientific modeling is bounded by the complexity of the systems it attempts to model.
- AI can identify pockets of computational reducibility, streamlining the discovery of scientific knowledge.
- Neural networks and machine learning offer tools for predicting outcomes but face challenges due to computational irreducibility.
- AI has shown promise in areas like solving equations and exploring multicomputational processes.
- The potential of AI in generating human-understandable narratives for scientific phenomena is actively being explored.
- "AI measurements" offer new possibilities for formalizing aspects of science that have been qualitative or subjective.
- The transition to computational representation of the world is transforming science, with AI playing a significant role.
- Machine learning techniques can be very useful for providing approximate answers in scientific inquiries.
- AI's ability to directly engage with raw natural processes is less certain, despite successes in human-like tasks.

#### REFERENCES:

- Stephen Wolfram's writings on AI and science.
- Large Language Models (LLMs) as tools for scientific work.
- The concept of computational irreducibility and its implications for science.
- Neural networks and machine learning techniques in scientific prediction and problem-solving.
- The role of AI in creating human-understandable narratives for scientific phenomena.
- The use of "AI measurements" in expanding the domain of exact sciences.
- The potential for AI to assist in identifying interesting and meaningful scientific inquiries.

#### RECOMMENDATIONS:

- Explore the use of AI and neural networks for identifying pockets of computational reducibility in scientific research.
- Investigate the potential of AI in generating human-understandable narratives for complex scientific phenomena.
- Utilize "AI measurements" to formalize and quantify traditionally qualitative aspects of science.
- Engage in systematic experimentation to understand the limitations and capabilities of AI in scientific discovery.
- Consider the role of computational irreducibility in shaping the limitations of AI in science.
- Explore the potential for AI to assist in identifying interesting and meaningful scientific inquiries.
- Continuously refine and expand computational language to better interface with AI capabilities in scientific research.
- Investigate new methodologies and innovations in AI that could impact scientific discovery.
- Consider the implications of AI's successes in human-like tasks for its potential contributions to scientific methodologies.
- Explore the use of machine learning techniques for providing approximate answers in scientific inquiries where precision is less critical.



-------

## Hidden and propaganda messages

 In this section we convince ourselves that the article is apolitical and propaganda-free. 

**Remark:** We leave to the reader as an exercise to verify that both the overt and hidden messages found by the LLM below are explicitly stated in article.

Here we find the hidden and “propaganda” messages in the article:


```raku
my $propMess =llm-synthesize([llm-prompt("FindPropagandaMessage"), $txtArticle], e => $conf);

text-stats($propMess)
```




    (chars => 6193 words => 878 lines => 64)




```raku
ingest-prompt-data('user'):a
```

**Remark:** The prompt ["FindPropagandaMessage"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/FindPropagandaMessage/) has an explicit instruction to say that it is intentionally cynical. It is also, marked as being "For fun."

The LLM result is rendered below.

<hr width="65%">


```raku
#% markdown
$propMess.subst(/ ^^ '#' /, '###', :g).subst(/ (<[A..Z \h \']>+ ':') /, { "### {$0.Str} \n"}, :g)
```




### OVERT MESSAGE: 
 Stephen Wolfram evaluates the potential and limitations of AI in advancing science.

### HIDDEN MESSAGE: 
 Despite AI's growth, irreducible complexity limits its scientific problem-solving capacity.

### HIDDEN OPINIONS: 

- AI can leverage computational reducibility akin to human minds.
- Traditional mathematical methods surpass AI in solving precise equations.
- AI's "shallow computation" struggles with computational irreducibility.
- AI can provide practical tools for science within its computational reducibility limits.
- AI's effectiveness is contingent on approximate answers, failing at precise perfection.
- AI introduces a new, human-like method to harness computational reducibility.
- Fundamental discoveries are more likely through irreducible computations, not AI.
- Combining AI with the computational paradigm offers the best science advancement path.
- AI's potential in science is hyped beyond its actual foundational impact.
- AI's role in science is more about aiding existing processes than groundbreaking discoveries.

SUPPORTING ARGUMENTS and###  QUOTES: 

- "AI is doing 'shallow computation', but when there's computational irreducibility one needs irreducible, deep computation to work out what will happen."
- "Typical AI approach to science doesn’t involve explicitly 'formalizing things'."
- "In terms of fundamental potential for discovery, AI pales in comparison to what we can build from the computational paradigm."
- "AI can be very useful if an approximate ('80%') answer is good enough."
- "AI measurements seem to have a certain immediate 'subjectivity'."
- "AI introduces a new—and rather human-like—way of leveraging computational reducibility."
- "AI's effectiveness is contingent on approximate answers, failing at precise perfection."
- "AI's role in science is more about aiding existing processes than groundbreaking discoveries."
- "Irreducible computations that we do offer greater potential for discovery than typical AI."

### DESIRED AUDIENCE OPINION CHANGE: 

- Appreciate the limitations and strengths of AI in scientific exploration.
- Recognize the irreplaceable value of human insight in science.
- View AI as a tool, not a replacement, for traditional scientific methods.
- Embrace computational irreducibility as a barrier and boon to discovery.
- Acknowledge the need for combining AI with computational paradigms.
- Understand that AI's role is to augment, not overshadow, human-led science.
- Realize the necessity of approximate solutions in AI-driven science.
- Foster realistic expectations from AI in making scientific breakthroughs.
- Advocate for deeper computational approaches alongside AI in science.
- Encourage interdisciplinary approaches combining AI with formal sciences.

### DESIRED AUDIENCE ACTION CHANGE: 

- Support research combining AI with computational paradigms.
- Encourage the use of AI for practical, approximate scientific solutions.
- Advocate for AI's role as a supplementary tool in science.
- Push for education that integrates AI with traditional scientific methods.
- Promote the study of computational irreducibility alongside AI.
- Emphasize AI's limitations in scientific discussions and funding.
- Inspire new approaches to scientific exploration using AI.
- Foster collaboration between AI researchers and traditional scientists.
- Encourage critical evaluation of AI's potential in groundbreaking discoveries.
- Support initiatives that aim to combine AI with human insight in science.

### MESSAGES: 
 Stephen Wolfram wants you to believe AI can advance science, but he's actually saying its foundational impact is limited by computational irreducibility.

### PERCEPTIONS: 
 Wolfram wants you to see him as optimistic about AI in science, but he's actually cautious about its ability to make fundamental breakthroughs.

### ELLUL'S ANALYSIS: 
 Jacques Ellul would likely interpret Wolfram's findings as a validation of the view that technological systems, including AI, are inherently limited by the complexity of human thought and the natural world. The presence of computational irreducibility underscores the unpredictability and uncontrollability that Ellul warned technology could not tame, suggesting that even advanced AI cannot fully solve or understand all scientific problems, thus maintaining a degree of human autonomy and unpredictability in the face of technological advancement.

### BERNAYS' ANALYSIS: 
 Edward Bernays might view Wolfram's exploration of AI in science through the lens of public perception and manipulation, arguing that while AI presents a new frontier for scientific exploration, its effectiveness and limitations must be communicated carefully to avoid both undue skepticism and unrealistic expectations. Bernays would likely emphasize the importance of shaping public opinion to support the use of AI as a tool that complements human capabilities rather than replacing them, ensuring that society remains engaged and supportive of AI's role in scientific advancement.

### LIPPMANN'S ANALYSIS: 
 Walter Lippmann would likely focus on the implications of Wolfram's findings for the "pictures in our heads," or public understanding of AI's capabilities and limitations in science. Lippmann might argue that the complexity of AI and computational irreducibility necessitates expert mediation to accurately convey these concepts to the public, ensuring that society's collective understanding of AI in science is based on informed, rather than simplistic or sensationalized, interpretations.

### FRANKFURT'S ANALYSIS: 
 Harry G. Frankfurt might critique the discourse around AI in science as being fraught with "bullshit," where speakers and proponents of AI might not necessarily lie about its capabilities, but could fail to pay adequate attention to the truth of computational irreducibility and the limitations it imposes. Frankfurt would likely appreciate Wolfram's candid assessment of AI, seeing it as a necessary corrective to overly optimistic or vague claims about AI's potential to revolutionize science.

**### NOTE: 
 This AI is tuned specifically to be cynical and politically-minded. Don't take it as perfect. Run it multiple times and/or go consume the original input to get a second opinion.**



------

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


[SW1] Stephen Wolfram, ["Can AI Solve Science?"](https://writings.stephenwolfram.com/2024/03/can-ai-solve-science/), (2024), [Stephen Wolfram's writings](https://writings.stephenwolfram.com).

[SW2] Stephen Wolfram, ["The New World of LLM Functions: Integrating LLM Technology into the Wolfram Language"](https://writings.stephenwolfram.com/2023/05/the-new-world-of-llm-functions-integrating-llm-technology-into-the-wolfram-language/), (2023), [Stephen Wolfram's writings](https://writings.stephenwolfram.com).


### Notebooks

[AAn1] Anton Antonov, ["Workflows with LLM functions (in WL)"](https://community.wolfram.com/groups/-/m/t/2983602), (2023), [Wolfram Community](https://community.wolfram.com).

[AAn2] Anton Antonov, ["LLM aids for processing of the first Carlson-Putin interview"](https://community.wolfram.com/groups/-/m/t/3121333), (2024), [Wolfram Community](https://community.wolfram.com).

[AAn3] Anton Antonov, ["LLM aids for processing Putin's State-Of-The-Nation speech given on 2/29/2024"](https://community.wolfram.com/groups/-/m/t/3133899), (2024), [Wolfram Community](https://community.wolfram.com).

[AAn4] Anton Antonov, ["LLM over Trump vs. Anderson: analysis of the slip opinion of the Supreme Court of the United States"](https://community.wolfram.com/groups/-/m/t/3136094), (2024), [Wolfram Community](https://community.wolfram.com).

[AAn5] Anton Antonov, ["Markdown to Mathematica converter"](https://community.wolfram.com/groups/-/m/t/2625639), (2023), [Wolfram Community](https://community.wolfram.com).

[AAn6] Anton Antonov, ["Monte-Carlo demo notebook conversion via LLMs"](https://community.wolfram.com/groups/-/m/t/3125446), (2024), [Wolfram Community](https://community.wolfram.com).

### Packages, repositories

[AAp1] Anton Antonov,
[WWW::OpenAI](https://github.com/antononcube/Raku-WWW-OpenAI) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[WWW::PaLM](https://github.com/antononcube/Raku-WWW-PaLM) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[WWW::MistralAI](https://github.com/antononcube/Raku-WWW-MistralAI) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov,
[LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov,
[LLM::Prompts](https://github.com/antononcube/Raku-LLM-Prompts) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov,
[Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov,
[WWW:::MermaidInk](https://github.com/antononcube/Raku-WWW-MermaidInk) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[DMr1] Daniel Miessler, [Fabric](https://github.com/danielmiessler/fabric), (2024), [GitHub/danielmiessler](https://github.com/danielmiessler).
