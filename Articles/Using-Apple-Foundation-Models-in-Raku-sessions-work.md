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

Or in a Raku session:

```raku
qx{fm respond 'Which are most used writting systems?'}
```

**Remark:** To use FM in this way does not require any setup. (Like the section below.)

Here is a more complicated example in which an image is described:

```raku
my $url = 'https://i.imgur.com/CStKxkw.jpeg';
shell "curl -O $url";
my $proc = shell "fm respond --image {$url.split('/').tail} 'Describe the image.'", :out;
$proc.out.slurp(:close)
```

**Remark:** `fm` does not work with image URLs -- (local) file paths must be used.

The Raku package ["LLM::Prompts"](https://raku.land/zef:antononcube/LLM::Prompts) provides a Command Line Interface (CLI) script for finding and retrieval of its prompts. That CLI script can be used together with `fm` via the option `--instructions` (`-i`):

```shell
fm respond 'How many people live in Brazil?' -i "$(llm-prompt NumericOnly)"
```

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

Generate a response to question with `llama-playground`:

```raku
'What is the speed of a rocket leaving the Moon?'
==> llama-playground(path => 'chat/completion', model => 'system', format => 'values');
```

**Remark:** Alternatively, the option `base-url => 'http://127.0.0.1:1976'` can be given to `llama-playground`.

---

## Generations with the LLM-functions framework

### LLM configuration

Here is a corresponding LLM-configuration (see ["LLM::Functions"](https://github.com/antononcube/Raku-LLM-Functions)):

```raku
my $conf-fm = llm-configuration('LLaMA', model => 'system', base-url => 'http://127.0.0.1:1976', :4096max-tokens, temperature => 0.4);
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


Raku code generation is not that good:

```raku
llm-synthesize([
    llm-prompt('CodeWriterX')('Raku'),
    'Iterate over a list of random reals.',
    ],
    e => $conf-fm
    )
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

```raku, results=asis
my $res = &ctFunc('top 10 largest by area countries')
```

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

```raku
my @data = |&ctFunc2('Top 10 largest by area countries');
deduce-type(@data)

```

Here is the obtained result:

```raku, results=asis
my @field-names = <country area time-zone>;
@data
==> to-html(:@field-names, align => 'left')
```


Again, that result is table, not a contingency table, i.e. cross-tabulated result.
Here is the cross tabulation using `cross-tabulate` of ["Data::Reshapers"](https://github.com/antononcube/Raku-Data-Reshapers) given as a sparse matrix (see ["Math::SparseMatrix"](https://github.com/antononcube/Raku-Math-SparseMatrix)):

```raku
my $mat = cross-tabulate(@data, <country>, <time-zone>, :sparse);
$mat.print
```