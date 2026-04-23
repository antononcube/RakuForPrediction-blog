# Chatnik: LLM Host in the Shell -- Part 1: First Examples & Design Principles

## Introduction

["Chatnik"](https://raku.land/zef:antononcube/Chatnik) is Raku package that provides Command Line Interface (CLI) 
scripts for conversing with multiple, persistent Large Language Model (LLM) personas. 
Files of the host Operating System (OS) are used to maintain persistency.

Chatnik features:

- UNIX Shell pipelining for LLM interaction
- Has a database of LLM-chat objects
- Can conenct to different models from different LLM providers
- Has access to a large repository of prompts
- Provides convenient access to interaction history messages
- Has management tools for the LLM chat objects database
- Preprocesses prompts based on a simple Domain Specific Language (DSL)
- Supports the loading of user LLM personas specified in JSON file

**Remark:** "Chatnik" closely follows the LLM-chat objects interaction system of the Raku package ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp3].
(Using OS Shell instead of Jupyter notebooks.)

### Leveraged LLM packages

The access to LLMs is provided by the packages 
["WWWW::OpenAI"](https://github.com/antononcube/Raku-WWW-OpenAI), 
["WWWW::Gemini"](https://github.com/antononcube/Raku-WWW-Gemini), 
["WWW::MistralAI"](https://github.com/antononcube/Raku-WWW-MistralAI),
["WWW::LLaMA"](https://github.com/antononcube/Raku-WWW-LLaMA),
["WWW::Ollama"](https://github.com/antononcube/Raku-WWW-Ollama).

The creation and interaction LLM-chat object functionalities are provided by ["LLM::Functions"](), [AAp1].

Prompt collection, prompt spec DSL and related prompt expansion by ["LLM::Prompts"], [AAp2]:

### Some questions to answer

- Why do it?
- Why it was relatively easy to do?
- Why it is useful?

Before answering those questions, let us first look at a few introductory (toy) examples.

----

## Introductory examples


### Chat with Yoda

Here we create an LLM persona -- by naming it and "priming it" with a prompt -- and start interacting with it:

```shell
llm-chat --chat-id=yoda --prompt=@Yoda 'Hi! Who are you?'
```

### MOTD pipeline

Here we specify a pipeline for 
1. Getting a fortune
2. Echoing it
3. Using the fortune to make a limerick


```
fortune | tee /dev/tty | llm-chat --prompt="make a limeric from the given text"
```

```
Good judgement comes from experience.  Experience comes from bad judgement.
		-- Jim Horning
		
There once was a man named Jim,
Whose wisdom was never too slim.
He said with a grin,
“Good judgment comes in,
From mistakes that look rather grim.”
```


----

## Architectural design

----

## Basic (serious) examples

----

## Chat objects management (preview)

----


----

## References

### Packages

[AAp1] Anton Antonov
[LLM::Functions, Raku package](https://github.com/antononcube/Raku-LLM-Functions),
(2023-2026),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov
[LLM::Prompts, Raku package](https://github.com/antononcube/Raku-LLM-Prompts),
(2023-2025),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov
[Jupyter::Chatbook, Raku package](https://github.com/antononcube/Raku-Jupyter-Chatbook),
(2023-2026),
[GitHub/antononcube](https://github.com/antononcube).

[JSp1] Jonathan Stowe,
[XDG::BaseDirectory, Raku package](https://github.com/jonathanstowe/XDG-BaseDirectory),
(2016-2026),
[GitHub/jonathanstowe](https://github.com/jonathanstowe).

### Videos

[AAv1] Anton Antonov,
["Integrating Large Language Models with Raku"](https://youtu.be/-OxKqRrQvh0?si=5LEj8-Dtcxjn-0QR&t=548),
(2023),
[The Raku Conference 2023 at YouTube](https://www.youtube.com/@therakuconference6823).

