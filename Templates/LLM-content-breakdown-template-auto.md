# [LLM] over "Large Language Models and The End of Programming - CS50 Tech Talk with Dr. Matt Welsh"

### *Tabular, visual, and textual breakdowns and summaries*

[Anton Antonov](https://rakuforprediction.wordpress.com/about/)
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
use WWW::Gemini;
use Hash::Merge;

my %knownRecords = data-import($*CWD ~ '/../Records/YouTubeChannels.json');

sub pretty-name(Str $nm) is export { $nm.subst(/\W/,'-',:g).subst(/ '-' + /, '-', :g).subst(/'-' $/) };

my $conf4 = llm-configuration('ChatGPT', model => 'gpt-4o', max-tokens => 4096, temperature => 0.5);
my $conf = $conf4;
#my $conf = llm-configuration('Gemini', model => 'gemini-1.5-pro-latest', max-tokens => 8192, base-url => 'https://generativelanguage.googleapis.com/v1beta/models', temperature => 0.5);
my $pauseTime = $conf.Hash<name>.lc eq 'gemini' ?? 60 !! 0;
```


```perl6, results=hide, echo=FALSE
my %record = 
   author => 'Matt Welsh',
   post-link => 'https://rakuforprediction.wordpress.com/2024/05/25/age-at-creation-for-programming-languages-stats/',
   site => 'https://rakuforprediction.wordpress.com',
   title => "Large Language Models and The End of Programming - CS50 Tech Talk with Dr. Matt Welsh",
   channel => 'MISSING CHANNEL',
   channel-name => 'MISSING CHANNEL NAME',
   transcript-file-name => $*CWD ~ '/../Data/' ~ '',
   type => 'video',
   video-id => 'JhCl-GeT4jw',
   video-link => 'https://www.youtube.com/watch?v=JhCl-GeT4jw';

%record<gen-file-name> = pretty-name(%record<title>);
        
%record = merge-hash(%record, %knownRecords<CS50> // %());
 
my Bool $transcriptFileExists = False;
      
if %record<type> eq 'video' {
   if ! %record<video-link> {  
      %record<video-link> = 'https://www.youtube.com/watch?v=' ~ %record<video-id>;
   }

   $transcriptFileExists = try %record<transcript-file-name>.IO.f;
   say '$transcriptFileExists :', $transcriptFileExists.raku; 
   if $! || !$transcriptFileExists {
      %record<transcript-file-name> = $*CWD ~ '/../Data/' ~ %record<gen-file-name> ~ '-YouTube.txt';
   }
} else {
   %record<text-file-name> = $*CWD ~ '/../Data/' ~ %record<gen-file-name> ~ '.txt' ;
} 

.say for %record.pairs.sort(*.key);
```

***Text statistics:***

```perl6, echo=FALSE
my $source = do if %record<type> eq 'video' {
   if %record<video-id> && !$transcriptFileExists {
      shell "youtube_transcript_api {%record<video-id>} --format text > {%record<transcript-file-name>.subst(/\s/, '\ ')}";
   }
   %record<transcript-file-name>;
} else {
   %record<post-link>
}

my $txtEN = data-import($source, 'plaintext');
say [|text-stats($txtEN), |gemini-count-tokens($txtEN).pairs];
say %record<gen-file-name>;
```

-----

## Introduction

```perl6, results=asis, echo=FALSE
if %record<type>.lc eq 'video' {
   say
   "This post applies various Large Language Model (LLM) summarization prompts to the transcript of the program\n",
   "[«{%record<title>}»]({%record<video-link>})\n",
   "by the YouTube channel ",
   "[{%record<channel-name>}]({%record<channel>}).";
} else {
   say
   "This post applies various Large Language Model (LLM) summarization prompts to the post\n",
   "[«{%record<title>}»]({%record<post-link>})\n",
   "by ",
   "[{%record<author>}]({%record<site>}).";
}
```

Here is a table of themes discussed in the text:

```perl6, results=asis, echo=FALSE, eval=TRUE
sleep($pauseTime);
my $tblThemes = llm-synthesize(llm-prompt("ThemeTableJSON")($txtEN, "article", 30), e => $conf, form => sub-parser('JSON'):drop);
$tblThemes ==> data-translation(field-names=><theme content>)
```

```perl6, results=asis, echo=FALSE
if %record<type> eq 'video' {
say '
**Remark:** The LLM results below were obtained from the "raw" transcript, which did not have punctuation.

**Remark:** The transcription software had problems parsing the names of mentioned people and locations. Some of the names were manually corrected.'
}
```

Post’s structure:

1. **Most important or provocative statements**    
   Extending the summary.
2. **Mind-map**   
   For orientation.
3. **Summary, ideas, and recommendations**     
   The main course.
4. **Sophisticated feedback**        
   While wearing hats of different colors.

-----

## Most important or provocative statements

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


------

## Mind-map

Here is a mind-map summarizing the text:

```perl6, results=asis, echo=FALSE, eval=TRUE
#sleep($pauseTime);
my $mmdBigPicture = llm-synthesize([
    "Create a concise mermaid-js mind-map -- not a flowchart -- for the text:\n\n",
    $txtEN,
    llm-prompt("NothingElse")("correct mermaid-js")
], e=>$conf4);
```

-------

## Summary, ideas, and recommendations

```perl6, results=asis, echo=FALSE, eval=TRUE
#Here we get a summary and extract ideas, quotes, and recommendations from the text:

sleep($pauseTime);
my $sumIdea = llm-synthesize(llm-prompt("ExtractArticleWisdom")($txtEN), e => $conf);

$sumIdea.subst(/ ^^ '#' /, '###', :g)
```

-------

## Sophisticated feedback 

In this section we try to give feedback and ideas while wearing different hats.
Like "black hat", "white hat", etc.

The LLM result is rendered below.

<hr width="65%">

```perl6, results=asis, echo=FALSE, eval=TRUE 
sleep($pauseTime);
my $sophFeed = llm-synthesize(llm-prompt("SophisticatedFeedback")($txtEN, 'HTML', :!ideas), e => $conf);

$sophFeed.subst('```html').subst('```')
```
