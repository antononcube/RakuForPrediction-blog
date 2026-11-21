# Using Apple’s Foundation Models in Raku Sessions

*Anton Antonov*  
*November 2026*



## 📦 Quick Start

1. **Run `fm serve`** (default `http://127.0.0.1:1976`). If already running from a previous session, shell/`qx` calls work immediately; otherwise start it first.
2. **In your Raku file/notebook**, load the required modules and set the base URL:

```raku  
use WWW::LLaMA;  
'What is the speed of a rocket leaving the Moon?' ==>
    llama-playground(path => 'chat/completion', model => 'system', base-url => 'http://127.0.0.1:1976', format => 'values');  
```
```
# The speed of a rocket leaving the Moon depends on the rocket's design and mission specifics. Generally, the escape velocity from the Moon is about 2.38 kilometers per second (approximately 8,600 kilometers per hour). However, rockets may achieve higher speeds depending on the propulsion system and mission requirements. For precise information, you would need details about the specific rocket and mission.
```  
3. **Try the simplest FM call** (in a shell cell):
```shell  
fm respond 'How many total countries in the world?'  
# There are 195 countries in the world.`
```
```
# There are 195 countries in the world.
```  
4. **Consult the companion Jupyter notebook** for live, executable examples:  
   ["Apple-Foundation-Models-usage-examples.ipynb"](https://github.com/antononcube/Raku-WWW-LLaMA/blob/main/docs/Apple-Foundation-Models-usage-examples.ipynb)


---

## Introduction

This document demonstrates how to use [Apple’s Foundation Models](https://developer.apple.com/documentation/foundationmodels) (FM) in Raku sessions. It is designed for developers who want to integrate FM capabilities—such as text generation, image description, and structured data extraction—into Raku workflows, Jupyter notebooks, or computational Markdown documents.

A companion [Jupyter notebook](https://github.com/antononcube/Raku-WWW-LLaMA/blob/main/docs/Apple-Foundation-Models-usage-examples.ipynb) provides live, executable examples alongside this guide.

The guide is organized into the following major sections:

1. **Direct invocation** – calling `fm` from shell cells or Raku `qx`
2. **Setup** – required packages and starting the FM server
3. **Generation via `WWW::LLaMA` subs** – configuring and using the high‑level Raku interface
4. **Generations with the LLM‑functions framework** – custom `llm-function` definitions and synthesis

---

## 1. Direct Invocation

Raku’s computational Markdown and Jupyter notebooks support evaluation of OS shell cells, enabling the simplest FM call without any prior setup.

### Shell / `qx` examples

```shell
fm respond 'How many total countries in the world?'
```
```
# There are 195 countries in the world.
```

```raku
qx{fm respond 'Which are most used writing systems?'}
```
```
# The most widely used writing systems are:
# 
# 1. **Latin script** – Used in English, Spanish, French, German, and many other languages.
# 2. **Arabic script** – Used in Arabic, Persian, Urdu, and other languages.
# 3. **Chinese characters** – Used in Mandarin, Cantonese, and other Chinese languages.
# 4. **Devanagari** – Used in Hindi and Nepali.
# 5. **Bengali** – Used in Bengali and Assamese.
```  


**Remark:** For `llama-*` and `llm-*` subroutines the FM server must be running (`fm serve`). The bare `fm respond` command can work in isolated shell/Jupyter cells, but full integration requires the server.

### Image description

> **⚠️ Important:** `fm respond --image` requires a **local file path**, not a remote URL. Download the image first (e.g., via `curl -O`), then pass the local path.

```raku
my $url = 'https://i.imgur.com/CStKxkw.jpeg';
shell "curl -O $url";
my $proc = shell "fm respond --image {$url.split('/').tail} 'Describe the image.'", :out;
$proc.out.slurp(:close)
```
```
# The image shows a woman wearing a colorful dress with blue, yellow, and red geometric patterns. The dress has a high neckline and a black belt. She has red lips and her hair is styled in an updo. The background features a landscape with rolling hills, trees, and a blue sky with clouds.
```  

---

## 2. Setup

### Load required packages

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

*Usage note:* 
- `WWW::LLaMA` and `WWW::LLaMA::Models` are used in Setup and Generation via `WWW::LLaMA` subs. 
- `LLM::Functions`, `LLM::Prompts`, and `Text::SubParsers` drive the LLM‑functions framework.
- `Data::Translators` and `Data::TypeSystem` provide data display and summarization. 
- `Data::Reshapers` and `Math::SparseMatrix` support the data wrangling and analysis.

### Start Apple’s Foundation Model server

**On macOS 27+ start Apple’s built‑in Foundation Model server:**

```sh
fm serve
```  

**Custom port / host / socket:**

```sh
fm serve --port 1976
fm serve --host 127.0.0.1 --port 1976
fm serve --socket /tmp/fm.sock
```  

**Server output:**

```
Foundation Models Serve
  url    http://127.0.0.1:1976
  access loopback-only

  · POST /v1/chat/completions
  · GET  /v1/models
  · GET  /health
```  

### Model registration

The system‑recognized model names `system` and `pcc` are available in `WWW::LLaMA::ver<1.0.5+>`. To register additional models:

```raku, eval=FALSE
llama-known-models()<system> = True;
llama-known-models()<pcc> = True;
say llama-known-models();

llama-end-point-to-models()<chat/completions> = <gpt-3.5-turbo llamafile llama llava system pcc>;
say llama-end-point-to-models();
```  

> **Note:** Registration is only needed for non‑default models. The `system` and `pcc` models work out‑of‑the‑box.


---

## 3. Generation via `WWW::LLaMA` subs

Set the base URL for the FM server (once per session):

```raku
llama-base-url('http://127.0.0.1:1976')
```
```
# http://127.0.0.1:1976
```  


### Chat completion with `llama-playground`

```raku
'What is the speed of a rocket leaving the Moon?' ==> 
        llama-playground(path => 'chat/completion', model => 'system', format => 'values');
```
```
# The speed of a rocket leaving the Moon depends on several factors, including the rocket’s design, mission objectives, and the specifics of the lunar environment. 
# 
# For a typical ascent from the lunar surface, a rocket might achieve a velocity of approximately 2,000–3,000 meters per second (about 7,200–10,800 kilometers per hour). This is a rough estimate and can vary significantly based on the mission’s requirements and the propulsion system used.
```  

> **Remark:** The `base-url` option can also be passed directly to `llama-playground`: `llama-playground(base-url => 'http://127.0.0.1:1976', ...)`.

---

## 4. Generations with the LLM‑functions framework

### LLM configuration

```raku
my $conf-fm = llm-configuration('LLaMA', model => 'system', base-url => 'http://127.0.0.1:1976', :max-tokens(4096), temperature => 0.4);
```
```
# LLM::Configuration(:name("llama"), :model("system"), :module("WWW::LLaMA"), :max-tokens(4096))
```

### Simple LLM generation requests

**Query:** “How many people live in Peru?”

```raku
llm-synthesize([
    'How many people live in Peru?',
], e => $conf-fm);
```
```
# Approximately 33 million people live in Peru.
```  

**Raku code generation** (note: current FM output for code tasks is approximate):

```raku
llm-synthesize([
    llm-prompt('CodeWriterX')('Raku'),
    'Iterate over a list of random reals.',
], e => $conf-fm);
```
```
# ```rakudo
# @rand(0..1000, 100).each { say $_ }
# ```
```  


### Complex queries

FM produces reliable results for simple factual queries. For cross‑domain or structured tasks, enrich prompts and use post‑processing.

**Example custom `llm-function`:**

```raku
my &ctFunc =
llm-function( -> $what {[
    "Cross-tabulate $what and their time zones.",
    'Give a JSON array of dictionaries, all having the same keys and at least one being "time-zone".',
    'Please, use short names for the time zones.',
    'Numeric results should be computable,',
    'Do it quickly -- do not overthink it.',
    llm-prompt('NothingElse')('JSON')
  ]}, 
  e => $conf-fm, 
  form => sub-parser('JSON'):drop);
```
```
# LLM::Function(-> **@args, *%args { #`(Block|3207846872152) ... }, 'llama')
```  


```raku
my @data = |&ctFunc('Top 10 largest by area countries');
deduce-type(@data);
```
```
# Vector(Struct([area, country, time-zone], [Int, Str, Str]), 10)
```

> **Approximate results:** FM outputs for complex queries (tabulation, code generation, natural‑language tasks) are inherently approximate. Validate results against authoritative sources when precision is critical, and use post‑processing (JSON parsing, `cross-tabulate`, unit conversion) to improve reliability.

Here is the cross‑tabulation using `cross-tabulate` of ["Data::Reshapers"](https://github.com/antononcube/Raku-Data-Reshapers) given as a sparse matrix (see ["Math::SparseMatrix"](https://github.com/antononcube/Raku-Math-SparseMatrix)):

```raku
my $mat = cross-tabulate(@data, <country>, <time-zone>, :sparse);
$mat.print
```
```
# ––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
#                 AEST ART  BRT  CST  EST  IST  KST  MSK  WAT 
# ––––––––––––––┼–––––––––––––––––––––––––––––––––––––––––––––
# Algeria       │ .    .    .    .    .    .    .    .    1   
# Argentina     │ .    1    .    .    .    .    .    .    .   
# Australia     │ 1    .    .    .    .    .    .    .    .   
# Brazil        │ .    .    1    .    .    .    .    .    .   
# Canada        │ .    .    .    .    1    .    .    .    .   
# China         │ .    .    .    1    .    .    .    .    .   
# India         │ .    .    .    .    .    1    .    .    .   
# Kazakhstan    │ .    .    .    .    .    .    1    .    .   
# Russia        │ .    .    .    .    .    .    .    1    .   
# United States │ .    .    .    .    1    .    .    .    .
```