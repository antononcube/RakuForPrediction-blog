# [LLM] Extracting wisdom and hidden messages

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog)      
```perl6, results=asis, echo=FALSE
Date.today.Str
```

```perl6, results=hide, echo=FALSE
use LLM::Functions;
use LLM::Prompts;
use Text::SubParsers;
use Data::Importers;
use Data::Translators;
use WWW::MermaidInk;

my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.5);
```

***Text statistics:***

```perl6, echo=FALSE
my $ytID = '';
my $source;
if $ytID {
   shell "youtube_transcript_api $ytID --format text > {$ytID}.txt";
   $source = "{$ytID}.txt";
} else {
   $source = $*CWD ~ '/../Data/Integrating-Large-Language-Models-with-Raku-YouTube.txt';
}
my $txtEN = data-import($source, 'plaintext');
text-stats($txtEN)
```

-----

## Introduction

This post applies various Large Language Model (LLM) summarization prompts to the transcript of the program
["Integrating Large Language Models with Raku"](https://www.youtube.com/watch?v=-OxKqRrQvh0) 
by the YouTube channel [The Raku Conference](https://www.youtube.com/@therakuconference6823).


Here is a table of the most important or provocative statements in the text:

```perl6, results=asis, echo=FALSE, eval=TRUE
my $imp = llm-synthesize([
    "Give the most important or provocative statements in the following speech.\n\n", 
    $txtEN,
    "Give the results as a JSON array with subject-statement pairs.",
    llm-prompt('NothingElse')('JSON')
    ], e => $conf, form => sub-parser('JSON'):drop);
    
$imp ==> data-translation(field-names => <subject statement>)
```

```perl6, results=asis, echo=FALSE
if $source.ends-with('YouTube.txt') || $ytID {
say '
**Remark:** The LLM results below were obtained from the "raw" transcript, which did not have punctuation.

**Remark:** The transcription software had problems parsing the names of the participants. Some of the names were manually corrected.'
}
```

Post’s structure:

1. **Themes**    
   Instead of a summary.
2. **Mind-map**   
   An even better summary replacement!
3. **Diagram**   
   Connecting the concepts.
4. **Summary, ideas, and recommendations**     
   The main course.
5. **Hidden and propaganda messages**     
   Didactic POV.

-----

## Themes

Instead of a summary consider this table of themes:

```perl6, results=asis, echo=FALSE, eval=TRUE
my $tblThemes = llm-synthesize(llm-prompt("ThemeTableJSON")($txtEN, "article", 30), e => $conf, form => sub-parser('JSON'):drop);
$tblThemes ==> data-translation(field-names=><theme content>)
```

------

## Mind-map

Here is a mind-map summarizing the text:

```perl6, results=asis, echo=FALSE, eval=TRUE
my $mmdBigPicture = llm-synthesize([
    "Create a concise mermaid-js mind-map for the text:\n\n",
    $txtEN,
    llm-prompt("NothingElse")("correct mermaid-js")
], e=>$conf);
```

```perl6, output.prompt=NONE, output.lang=mermaid, echo=FALSE, eval=TRUE
#my $mmdBigPictureTheme = '%%{ init: {\'theme\': \'forest\' } }%%';
#mermaid-ink($mmdBigPictureTheme ~ "\n" ~ $mmdBigPicture.trim.subst(/ ^ '```mermaid' | '```' $ /, :g), format => 'md-image', background => 'BlanchedAlmond')
```

------

## Diagram

Here is a flowchart summarizing the post:

```perl6, results=asis, echo=FALSE, eval=FALSE
my $mmdBigPictureFlow = llm-synthesize([
    "Create a mermaid-js flowchart for the text:\n\n",
    $txtEN,
    llm-prompt("NothingElse")("correct mermaid-js")
], e=>$conf);
```

```perl6, output.prompt=NONE, output.lang=mermaid, echo=FALSE, eval=FALSE
#mermaid-ink($mmdBigPictureTheme ~ "\n" ~ $mmdBigPictureFlow.trim.subst(/ ^ '```mermaid' | '```' $ /, :g), format => 'md-image', background => 'BlanchedAlmond')
```


-------

## Summary, ideas, and recommendations

```perl6, results=asis, echo=FALSE, eval=TRUE
#Here we get a summary and extract ideas, quotes, and recommendations from the text:

my $sumIdea = llm-synthesize(llm-prompt("ExtractArticleWisdom")($txtEN), e => $conf);

$sumIdea.subst(/ ^^ '#' /, '###', :g)
```


-------

## Hidden and propaganda messages

In this section we try to find is the text apolitical and propaganda-free.

**Remark:** We leave to the reader as an exercise to verify that both the overt and hidden messages found by the LLM below are explicitly stated in the text.

**Remark:** The LLM prompt "FindPropagandaMessage" has an explicit instruction to say that it is intentionally cynical. 
It is also, marked as being "For fun."

The LLM result is rendered below.

<hr width="65%">

```perl6, results=asis, echo=FALSE, eval=TRUE
my $propMess = llm-synthesize([llm-prompt("FindPropagandaMessage"), $txtEN], e => $conf);

$propMess.subst(/ ^^ '#' /, '###', :g).subst(/ ^^ (<[A..Z \h \']>+ ':') /, { "### {$0.Str} \n"}, :g)
```
