# Using Gemini's Function Calling

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
June 2025


-----

## Introduction

This document
([notebook](https://github.com/antononcube/Raku-WWW-Gemini/blob/main/docs/LLM-function-calling-workflow.ipynb))
shows how to do [Function Calling](https://ai.google.dev/gemini-api/docs/function-calling) workflows with Large Language Models (LLMs) of
[Google's Gemini](https://ai.google.dev/gemini-api).

The Raku package ["WWW::Gemini"](https://github.com/antononcube/Raku-WWW-Gemini), [AAp2], is used.

### Examples and big picture

The rest of the document gives concrete code how to do function calling with Gemini's LLMs using Raku.

There are [similar workflows](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/), [AA1], with other LLM providers. (Like, OpenAI.) They follow the same structure, although there are some small differences. (Say, in the actual specifications of tools.)

This document belongs to a collection of notebooks describing how to do LLM function calling with Raku.

*The Gemini LLM workflow in this document is quite similar to the OpenIA workflow described in [AA1].
While there are variations in the tool configurations and how the elements of the LLM responses are obtained,
the overall procedure outline and diagrams in [AA1] also apply to the workflows presented here.*


-----

## Setup


Load packages:

```raku
use WWW::Gemini;
use JSON::Fast;
```
```
# (Any)
```

Choose a model:

```raku
my $model = "gemini-2.0-flash";
```
```
# gemini-2.0-flash
```

------

## Workflow


### Define a local function


This is the "tool" to be communicated to Gemini. (I.e. define the local function/sub.)

```raku
sub get-current-weather(Str:D $location, Str:D $unit = "fahrenheit") returns Str {
    return "It is currently sunny in $location with a temperature of 72 degrees $unit.";
}
```
```
# &get-current-weather
```

Define the function specification (as prescribed in [Gemini's function calling documentation](https://ai.google.dev/gemini-api/docs/function-calling?example=weather)):

```raku
my %weather-function = %(
    name => 'get-current-weather',
    description => 'Get the current weather in a given location',
    parameters => %(
        type => 'object',
        properties => %(
            location => %(
                type => 'string',
                description => 'The city and state, e.g., Boston, MA'
            )
        ),
        required => ['location']
    )
);
```
```
# {description => Get the current weather in a given location, name => get-current-weather, parameters => {properties => {location => {description => The city and state, e.g., Boston, MA, type => string}}, required => [location], type => object}}
```

### First communication with Gemini


Initialize messages and tools:

```raku
# User prompt
my $prompt = 'What is the weather like in Boston, MA, USA?';

# Prepare the API request payload
my @messages = [{role => 'user',parts => [ %( text => $prompt ) ]}, ];

my @tools = [%weather-function, ];
```
```
# [{description => Get the current weather in a given location, name => get-current-weather, parameters => {properties => {location => {description => The city and state, e.g., Boston, MA, type => string}}, required => [location], type => object}}]
```

Send the first chat completion request:

```raku
my $response = gemini-generate-content(
    @messages,
    :$model,
    :@tools
);
```
```
# {candidates => [{avgLogprobs => -3.7914659414026473e-06, content => {parts => [{functionCall => {args => {location => Boston, MA}, name => get-current-weather}}], role => model}, finishReason => STOP, safetyRatings => [{category => HARM_CATEGORY_HATE_SPEECH, probability => NEGLIGIBLE} {category => HARM_CATEGORY_DANGEROUS_CONTENT, probability => NEGLIGIBLE} {category => HARM_CATEGORY_HARASSMENT, probability => NEGLIGIBLE} {category => HARM_CATEGORY_SEXUALLY_EXPLICIT, probability => NEGLIGIBLE}]}], modelVersion => gemini-2.0-flash, responseId => zDpEaIClFpu97dcPpqOWiA8, usageMetadata => {candidatesTokenCount => 9, candidatesTokensDetails => [{modality => TEXT, tokenCount => 9}], promptTokenCount => 41, promptTokensDetails => [{modality => TEXT, tokenCount => 41}], totalTokenCount => 50}}
```

The response is already parsed from JSON to Raku. Here is its JSON form:

```raku
to-json($response)
```
```
# {
#   "usageMetadata": {
#     "totalTokenCount": 50,
#     "promptTokensDetails": [
#       {
#         "tokenCount": 41,
#         "modality": "TEXT"
#       }
#     ],
#     "candidatesTokenCount": 9,
#     "candidatesTokensDetails": [
#       {
#         "tokenCount": 9,
#         "modality": "TEXT"
#       }
#     ],
#     "promptTokenCount": 41
#   },
#   "modelVersion": "gemini-2.0-flash",
#   "candidates": [
#     {
#       "finishReason": "STOP",
#       "safetyRatings": [
#         {
#           "category": "HARM_CATEGORY_HATE_SPEECH",
#           "probability": "NEGLIGIBLE"
#         },
#         {
#           "probability": "NEGLIGIBLE",
#           "category": "HARM_CATEGORY_DANGEROUS_CONTENT"
#         },
#         {
#           "probability": "NEGLIGIBLE",
#           "category": "HARM_CATEGORY_HARASSMENT"
#         },
#         {
#           "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT",
#           "probability": "NEGLIGIBLE"
#         }
#       ],
#       "content": {
#         "parts": [
#           {
#             "functionCall": {
#               "args": {
#                 "location": "Boston, MA"
#               },
#               "name": "get-current-weather"
#             }
#           }
#         ],
#         "role": "model"
#       },
#       "avgLogprobs": -3.7914659414026473e-06
#     }
#   ],
#   "responseId": "zDpEaIClFpu97dcPpqOWiA8"
# }
```

### Refine the response with functional calls


The following copy of the messages is not required, but it makes repeated experiments easier:

```raku
my @messages2 = @messages;
```
```
# [{parts => [text => What is the weather like in Boston, MA, USA?], role => user}]
```

Process the response -- invoke the tool, give the tool result to the LLM, get the LLM answer:

```raku
my $assistant-message = $response<candidates>[0]<content>;
if $assistant-message<parts> {

    for |$assistant-message<parts> -> %part {
        if %part<functionCall> {
            
            @messages2.push($assistant-message);

            my $func-name = %part<functionCall><name>;
            my %args = %part<functionCall><args>;

            
            if $func-name eq 'get-current-weather' {
                my $location = %args<location>;
                my $weather = get-current-weather($location);

                my %function-response =
                            role => 'user',
                            parts => [{ 
                                functionResponse => {
                                    name => 'get-current-weather',
                                    response => %( content => $weather )
                                } 
                            }];

                @messages2.push(%function-response);
                
                # Send the second request with function result
                my $final-response = gemini-generate-content(
                    @messages2,
                    :@tools,
                    :$model,
                    format => "raku"
                );
                
                say "Assistant: ", $final-response<candidates>[0]<content><parts>».<text>.join("\n");

                last
            }
        }
    }
} else {
    say "Assistant: $assistant-message<content>";
}
```
```
# Assistant: The weather in Boston, MA is currently sunny with a temperature of 72 degrees Fahrenheit.
```

**Remark:** Note that if `get-current-weather` is applied then the loop above immediately finishes.


-----

## References


### Articles, blog posts

[AA1] Anton Antonov,
["LLM function calling workflows (Part 1, OpenAI)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/),
(2025),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["LLM function calling workflows (Part 2, Google's Gemini)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-2-googles-gemini/),
(2025),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).



### Packages 

[AAp1] Anton Antonov,
[WWW::OpenAI Raku package](https://github.com/antononcube/Raku-WWW-OpenAI),
(2023-2025),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[WWW::Gemini Raku package](https://github.com/antononcube/Raku-WWW-Gemini),
(2023-2025),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[LLM::Functions Raku package](https://github.com/antononcube/Raku-LLM-Functions),
(2023-2025),
[GitHub/antononcube](https://github.com/antononcube).
