# Using Gemini's Function Calling

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
June 2025


-----

## Introduction


This notebook shows how to do [Function Calling](https://ai.google.dev/gemini-api/docs/function-calling) workflows with Large Language Models (LLMs) of Gemini. 

The Raku package ["WWW::Gemini"](https://github.com/antononcube/Raku-WWW-Gemini), [AAp2], is used.


### Examples and big picture


The rest of the notebook gives concrete code how to do function calling with Gemini's LLMs using Raku.

There are [similar workflows](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/), [AA1], with other LLM providers. (Like, OpenAI.) They follow the same structure, although there are some small differences. (Say, in the actual specifications of tools.)

It would be nice to have:
- Universal programming interface for those function calling interfaces.
- Facilitation of tool descriptions derivations.
    - Via Raku's introspection or using suitable LLM prompts.
        - ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions), [AAp3], can be used for both approaches.

This notebook belongs to a collection of notebooks describing how to do LLM function calling with Raku.


-----

## Setup


Load packages:

```raku
use WWW::Gemini;
use JSON::Fast;
```

Choose a model:

```raku
my $model = "gemini-2.0-flash";
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

### First communication with Gemini


Initialize messages and tools:

```raku
# User prompt
my $prompt = 'What is the weather like in Boston, MA, USA?';

# Prepare the API request payload
my @messages = [{role => 'user',parts => [ %( text => $prompt ) ]}, ];

my @tools = [%weather-function, ];
```

Send the first chat completion request:

```raku
my $response = gemini-generate-content(
    @messages,
    :$model,
    :@tools
);
```

The response is already parsed from JSON to Raku. Here is its JSON form:

```raku
to-json($response)
```

### Refine the response with functional calls


The following copy of the messages is not required, but it makes repeated experiments easier:

```raku
my @messages2 = @messages;
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

**Remark:** Note that if `get-current-weather` is applied then the loop above immediately finishes.


-----

## References


### Articles, blog posts

[AA1] Anton Antonov,
["LLM function calling workflows (Part 1, OpenAI)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/),
(2025),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["LLM function calling workflows (Part 2, Google's Gemini)"](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-2-google-gemini/),
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
