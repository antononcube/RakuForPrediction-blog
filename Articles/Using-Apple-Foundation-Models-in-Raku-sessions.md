# Using Apple's Foundation Models in Raku Sessions

Anton Antonov  
November 2026


## Introduction

This document ([notebook](https://github.com/antononcube/Raku-WWW-LLaMA/blob/main/docs/Apple-Foundation-Models-usage-examples.ipynb)) demonstrates how to use Apple's [Foundation Models](https://developer.apple.com/documentation/foundationmodels) (FM) in Raku sessions.

- Direct invocation
  - Via `qx` and shell cells
- Via the package ["WWW::LLaMA"](https://github.com/antononcube/Raku-WWW-LLaMA)
   - Both FM and "WWW::LLaMA" are compatible with OpenAI's API
   - Hence, the latter can be used for LLM-generation with the former

---

## Direct invocation

Raku's [Computational Markdown documents](https://github.com/antononcube/Raku-Text-CodeProcessing) and 
[Jupyter notebooks](https://github.com/antononcube/Raku-Jupyter-Chatbook) support evaluation of OS shell cells. 
Hence, the easiest way to ise FM is to run in Jupyter cells like:

```shell
fm respond 'How many total countries in the world?'
```
```
# There are 195 countries in the world.
```

Or in a Raku session:

```raku
qx{fm respond 'Which are most used writting systems?'}
```
```
# Some of the most commonly used writing systems include:
# 
# 1. **Latin script** – Used widely in Europe, North America, and parts of Africa and Asia.
# 2. **Arabic script** – Used in the Middle East, North Africa, and parts of South Asia.
# 3. **Chinese characters** – Used in China, Taiwan, and Singapore.
# 4. **Japanese writing systems** – Including Kanji, Hiragana, and Katakana.
# 5. **Indian scripts** – Such as Devanagari and Tamil.
# 
# Usage can vary by region, culture, and context, so the "most used" can depend on specific criteria like population, literacy rates, or technological integration.
```

**Remark:** To use FM in this way does not require any setup. (Like the section below.)

Here is a more complicated example in which an image:

```raku
my $url = 'https://i.imgur.com/CStKxkw.jpeg';
shell "curl -O $url";
my $proc = shell "fm respond --image {$url.split('/').tail} 'Describe the image.'", :out;
$proc.out.slurp(:close)
```
```
# The image features a woman wearing a colorful dress with blue, yellow, and red geometric patterns. The dress has a black belt and puffed sleeves. She stands in front of a scenic landscape with rolling hills, trees, and mountains under a blue sky with clouds.
```

**Remark:** `fm` does not work with image URLs -- (local) file paths must be used.  

---

## Setup

Load packages:

```raku
use WWW::LLaMA;
use WWW::LLaMA::Models;

use LLM::Functions;
use LLM::Prompts;
use Text::SubParsers;

use Data::Reshapers;
use Data::Translators;
use Data::TypeSystem;

use Math::SparseMatrix;
```
```
# (Any)
```

In macOS 27+ run Apple's built-in Foundation Model (FM) with the command:

```sh
fm serve
```

Which will give the output:

```text
Foundation Models Serve
  url    http://127.0.0.1:1976
  access loopback-only

  · POST /v1/chat/completions
  · GET  /v1/models
  · GET  /health
```

Alternatively, start with a custom port / host / Unix socket:

```sh
fm serve --port 1976
fm serve --host 127.0.0.1 --port 1976
fm serve --socket /tmp/fm.sock
```

### Adding new LLaMA models

The models names "system" and "pcc" are recognized by "WWW::LLaMA:ver<1.0.5+>". In general, though, recognized model names can be added in the following way:

```raku, eval=FALSE
llama-known-models()<system> = True;
llama-known-models()<pcc> = True;
say llama-known-models();

llama-end-point-to-models()<chat/completions> = <gpt-3.5-turbo llamafile llama llava system pcc>;
say llama-end-point-to-models();
```

---

## Generation via subs of "WWW::LLaMA"

Set up the end point:

```raku
llama-base-url('http://127.0.0.1:1976')
```
```
# http://127.0.0.1:1976
```

Generate a response to question with `llama-playground`:

```raku
'What is the speed of a rocket leaving the Moon?'
==> llama-playground(path => 'chat/completion', model => 'system', format => 'values');
```
```
# The speed of a rocket leaving the Moon can vary significantly depending on the mission, rocket type, and specific trajectory. However, a general estimate for a typical lunar ascent might range from about 1,000 km/h (620 mph) to 3,000 km/h (1,860 mph). For precise calculations, you would need details about the rocket and mission parameters.
```

**Remark:** Alternatively, the option `base-url => 'http://127.0.0.1:1976'` can be given to `llama-playground`.

---

## Generations with the LLM-functions framework

### LLM configuration

Here is a corresponding LLM-configuration (see ["LLM::Functions"](https://github.com/antononcube/Raku-LLM-Functions)):

```raku
my $conf-fm = llm-configuration('LLaMA', model => 'system', base-url => 'http://127.0.0.1:1976', :4096max-tokens, temperature => 0.4);
```
```
# LLM::Configuration(:name("llama"), :model("system"), :module("WWW::LLaMA"), :max-tokens(4096))
```

### Simple LLM generation requests

Generate response with FM using `llm-synthesize`:

```raku
llm-synthesize([
    'How mane people live in Peru?',
    ],
    e => $conf-fm
)
```
```
# Approximately 33 million people live in Peru.
```


Raku code generation is not that good:

```raku
llm-synthesize([
    llm-prompt('CodeWriterX')('Raku'),
    'Iterate over a list of random reals.',
    ],
    e => $conf-fm
    )
```
```
# ```rakudo
# @rand(0..10, 100).each { print $_ }
# ```
```


### Complex queries

Currently, FM produces good results for simple queries, but for more complicated one the results are "approximately" good. For example:

```raku
my &ctFunc = 
llm-function( -> $what {[
    "Crows tabulate the $what and their time zones.",
    llm-prompt('NothingElse')('Markdown')
    ]},
    e => $conf-fm
)
```
```
# LLM::Function(-> **@args, *%args { #`(Block|6052521206272) ... }, 'llama')
```

```raku, results=asis
my $res = &ctFunc('top 10 largest by area countries')
```
| Country          | Area (sq km) | Time Zone       |
|------------------|--------------|-----------------|
| Russia           | 17,098,242   | UTC+2           |
| Canada           | 9,984,670    | UTC-5           |
| China            | 9,596,961    | UTC+8           |
| United States    | 9,524,275    | UTC-6           |
| Brazil           | 8,515,767    | UTC-3           |
| Australia        | 7,692,024    | UTC+8           |
| India            | 3,287,263    | UTC+5:30        |
| Argentina        | 2,780,400    | UTC-3           |
| Kazakhstan       | 2,724,900    | UTC+6           |
| Algeria          | 2,381,741    | UTC+1           |


That, of course, can be remedied by using more complicated prompts and additional transformations:

```raku
my &ctFunc2 = 
llm-function( -> $what {[
    "Cross-tabulate $what and their time zones.",
    'Give a JSON array of dictionaries, all having the same keys and at least one being "time-zone".',
    'Please, use short names for the time zones.',
    'Numeric results should be computable,', 
    'Do it quickly -- do not overthink it.',
    llm-prompt('NothingElse')('JSON')
    ]},
    e => $conf-fm,
    form => sub-parser('JSON'):drop
)
```
```
# LLM::Function(-> **@args, *%args { #`(Block|6052521274984) ... }, 'llama')
```

```raku
my @data = |&ctFunc2('Top 10 largest by area countries');
deduce-type(@data)

```
```
# Vector(Struct([area, country, time-zone], [Int, Str, Str]), 10)
```

Here is the obtained result:

```raku, results=asis
my @field-names = <country area time-zone>;
@data
==> to-html(:@field-names, align => 'left')
```
<table border="1"><thead><tr><th>country</th><th>area</th><th>time-zone</th></tr></thead><tbody><tr><td align=left>Russia</td><td align=left>17098242</td><td align=left>UTC+2</td></tr><tr><td align=left>Canada</td><td align=left>9984670</td><td align=left>UTC-5</td></tr><tr><td align=left>China</td><td align=left>9389300</td><td align=left>UTC+8</td></tr><tr><td align=left>United States</td><td align=left>9372610</td><td align=left>UTC-6</td></tr><tr><td align=left>Brazil</td><td align=left>8515767</td><td align=left>UTC-3</td></tr><tr><td align=left>Australia</td><td align=left>7692024</td><td align=left>UTC+10</td></tr><tr><td align=left>India</td><td align=left>3287263</td><td align=left>UTC+5:30</td></tr><tr><td align=left>Argentina</td><td align=left>2780400</td><td align=left>UTC-3</td></tr><tr><td align=left>Algeria</td><td align=left>2381741</td><td align=left>UTC+1</td></tr><tr><td align=left>Kazakhstan</td><td align=left>2272427</td><td align=left>UTC+6</td></tr></tbody></table>



Again, that result is table, not a contingency table, i.e. cross-tabulated result.
Here is the cross tabulation using `cross-tabulate` of ["Data::Reshapers"](https://github.com/antononcube/Raku-Data-Reshapers) given as a sparse matrix (see ["Math::SparseMatrix"](https://github.com/antononcube/Raku-Math-SparseMatrix)):

```raku
my $mat = cross-tabulate(@data, <country>, <time-zone>, :sparse);
$mat.print
```
```
# ––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
#                 UTC+1    UTC+10   UTC+2    UTC+5:30 UTC+6    UTC+8    UTC-3    UTC-5    UTC-6   
# ––––––––––––––┼–––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
# Algeria       │ 1        .        .        .        .        .        .        .        .       
# Argentina     │ .        .        .        .        .        .        1        .        .       
# Australia     │ .        1        .        .        .        .        .        .        .       
# Brazil        │ .        .        .        .        .        .        1        .        .       
# Canada        │ .        .        .        .        .        .        .        1        .       
# China         │ .        .        .        .        .        1        .        .        .       
# India         │ .        .        .        1        .        .        .        .        .       
# Kazakhstan    │ .        .        .        .        1        .        .        .        .       
# Russia        │ .        .        1        .        .        .        .        .        .       
# United States │ .        .        .        .        .        .        .        .        1
```