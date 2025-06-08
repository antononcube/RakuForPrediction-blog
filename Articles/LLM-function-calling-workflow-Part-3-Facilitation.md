
# LLM function calling workflow (Part 3, Facilitation)

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
June 2025


-----

## Introduction


This document ([notebook](https://github.com/antononcube/Raku-WWW-Gemini/blob/main/docs/LLM-parallel-function-calling-workflow.ipynb)) shows how to do parallel [Function Calling](https://ai.google.dev/gemini-api/docs/function-calling) workflows with Large Language Models (LLMs) of Gemini. 

The Raku package ["WWW::Gemini"](https://github.com/antononcube/Raku-WWW-Gemini), [AAp2], is used.


### Examples and big picture


The rest of the document gives concrete code how to do streamline multiple-tool function calling with Gemini's LLMs using Raku. 
Gemini's function calling example ["Parallel Function Calling"](https://ai.google.dev/gemini-api/docs/function-calling#parallel_function_calling), [Gem1], is followed.

This document belongs to a [collection of documents](https://rakuforprediction.wordpress.com/category/llm-function-calling/) describing how to do LLM function calling with Raku.

Compared to the previously described LLM workflows with OpenAI, [[AA1](https://rakuforprediction.wordpress.com/2025/06/01/llm-function-calling-workflows-part-1-openai/)], and Gemini, [[AA2](https://rakuforprediction.wordpress.com/2025/06/07/llm-function-calling-workflows-part-2-googles-gemini/)], the Gemini LLM workflow in this document demonstrates:

- Use of multiple tools (parallel function calling)
- Automatic generation of hashmap (or JSON) tool descriptors
- Streamlined computation of multiple tool results from multiple LLM requests

The streamlining is achieved by using the provided by ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions), [AAp3]:

- Classes `LLM::Tool`, `LLM::ToolRequest`, and `LLM::ToolResult`
- Subs `llm-tool-definition` and `generate-llm-tool-result`
    - The former sub leverages Raku's introspection features. 
    - The latter sub matches tools and requests in order to compute tool responses.


-----

## Setup


Load packages:

```raku
use JSON::Fast;
use Data::Reshapers;
use Data::TypeSystem;
use LLM::Tooling;
use WWW::Gemini;
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


Define a few subs -- _tools_ -- with sub- and argument descriptions (i.e. attached Pod values, or [declarator blocks](https://docs.raku.org/language/pod#Declarator_blocks)):

```raku
#| Powers the spinning disco ball.
sub power-disco-ball-impl(
    Int:D $power #= Whether to turn the disco ball on or off.
    ) returns Hash {
    return { status => "Disco ball powered " ~ ($power ?? 'on' !! 'off') };
}
#= A status dictionary indicating the current state.

#| Play some music matching the specified parameters.
sub start-music-impl(
    Int:D $energetic, #=  Whether the music is energetic or not.
    Int:D $loud       #= Whether the music is loud or not.
    ) returns Hash {
    my $music-type = $energetic ?? 'energetic' !! 'chill';
    my $volume = $loud ?? 'loud' !! 'quiet';
    return { music_type => $music-type, volume => $volume };
    #= A dictionary containing the music settings.
}

#| Dim the lights.
sub dim-lights-impl(
    Numeric:D $brightness #= The brightness of the lights, 0.0 is off, 1.0 is full.
    ) returns Hash {
    return { brightness => $brightness };
}
#= A dictionary containing the new brightness setting.
```
```
# &dim-lights-impl
```

**Remark:** See the corresponding Python definitions in the section ["Parallel Function Calling"](https://ai.google.dev/gemini-api/docs/function-calling#parallel_function_calling) of [Gem1].


The sub `llm-tool-definition` can be used to _automatically_ generate the Raku-hashmaps or JSON-strings of the tool descriptors in the (somewhat universal) format required by LLMs:

```raku
llm-tool-definition(&dim-lights-impl, format => 'json')
```
```
# {
#   "function": {
#     "type": "function",
#     "name": "dim-lights-impl",
#     "strict": true,
#     "description": "Dim the lights.",
#     "parameters": {
#       "required": [
#         "$brightness"
#       ],
#       "additionalProperties": false,
#       "type": "object",
#       "properties": {
#         "$brightness": {
#           "type": "number",
#           "description": "The brightness of the lights, 0.0 is off, 1.0 is full."
#         }
#       }
#     }
#   },
#   "type": "function"
# }
```

**Remark:** The sub `llm-tool-description` is invoked in `LLM::Tool.new`. Hence (ideally) `llm-tool-description` would not be user-invoked that often.


These are the tool descriptions to be communicated to Gemini:

```raku
my @tools =
{
    :name("power-disco-ball-impl"), 
    :description("Powers the spinning disco ball."), 
    :parameters(
        {
            :type("object")
            :properties( {"\$power" => {:description("Whether to turn the disco ball on or off."), :type("integer")}}), 
            :required(["\$power"]), 
        }), 
},
{
    :name("start-music-impl"), 
    :description("Play some music matching the specified parameters."), 
    :parameters(
        {
            :type("object")
            :properties({
                "\$energetic" => {:description("Whether the music is energetic or not."), :type("integer")}, 
                "\$loud" => {:description("Whether the music is loud or not."), :type("integer")}
            }), 
            :required(["\$energetic", "\$loud"]), 
        }),
},
{
    :name("dim-lights-impl"), 
    :description("Dim the lights."), 
    :parameters(
        {
            :type("object")
            :properties({"\$brightness" => {:description("The brightness of the lights, 0.0 is off, 1.0 is full."), :type("number")}}), 
            :required(["\$brightness"]), 
        }), 
};

deduce-type(@tools)
```
```
# Vector(Struct([description, name, parameters], [Str, Str, Hash]), 3)
```

Here are additional tool-mode configurations (see ["Function calling modes"](https://ai.google.dev/gemini-api/docs/function-calling?example=weather#function_calling_modes) of [Gem1]):

```raku
my %toolConfig =
  functionCallingConfig => {
    mode => "ANY",
    allowedFunctionNames => <power-disco-ball-impl start-music-impl dim-lights-impl>
  };
```
```
# {functionCallingConfig => {allowedFunctionNames => (power-disco-ball-impl start-music-impl dim-lights-impl), mode => ANY}}
```

### First communication with Gemini


Initialize messages:

```raku
# User prompt
my $prompt = 'Turn this place into a party!';

# Prepare the API request payload
my @messages = [{role => 'user',parts => [ %( text => $prompt ) ]}, ];
```
```
# [{parts => [text => Turn this place into a party!], role => user}]
```

Send the first chat completion request:

```raku
my $response = gemini-generate-content(
    @messages,
    :$model,
    :@tools,
    :%toolConfig
);

deduce-type($response)
```
```
# Struct([candidates, modelVersion, responseId, usageMetadata], [Hash, Str, Str, Hash])
```

```raku
deduce-type($response)
```
```
# Struct([candidates, modelVersion, responseId, usageMetadata], [Hash, Str, Str, Hash])
```

The response is already parsed from JSON to Raku. Here is its JSON form:

```raku
to-json($response)
```
```
# {
#   "candidates": [
#     {
#       "avgLogprobs": -0.0012976408004760742,
#       "content": {
#         "parts": [
#           {
#             "functionCall": {
#               "name": "start-music-impl",
#               "args": {
#                 "$energetic": 1,
#                 "$loud": 1
#               }
#             }
#           },
#           {
#             "functionCall": {
#               "name": "power-disco-ball-impl",
#               "args": {
#                 "$power": 1
#               }
#             }
#           },
#           {
#             "functionCall": {
#               "args": {
#                 "$brightness": 0.5
#               },
#               "name": "dim-lights-impl"
#             }
#           }
#         ],
#         "role": "model"
#       },
#       "safetyRatings": [
#         {
#           "probability": "NEGLIGIBLE",
#           "category": "HARM_CATEGORY_HATE_SPEECH"
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
#           "probability": "NEGLIGIBLE",
#           "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT"
#         }
#       ],
#       "finishReason": "STOP"
#     }
#   ],
#   "usageMetadata": {
#     "candidatesTokensDetails": [
#       {
#         "tokenCount": 30,
#         "modality": "TEXT"
#       }
#     ],
#     "promptTokensDetails": [
#       {
#         "tokenCount": 113,
#         "modality": "TEXT"
#       }
#     ],
#     "promptTokenCount": 113,
#     "candidatesTokenCount": 30,
#     "totalTokenCount": 143
#   },
#   "responseId": "sOxFaOrFF-SfnvgPgITLqQ8",
#   "modelVersion": "gemini-2.0-flash"
# }
```

### Refine the response with functional calls


The following copy of the messages is not required, but it makes repeated experiments easier:

```raku
my @messages2 = @messages;
```
```
# [{parts => [text => Turn this place into a party!], role => user}]
```

Let us define an `LLM::Tool` object for each tool:

```raku
my @toolObjects = [&power-disco-ball-impl, &start-music-impl, &dim-lights-impl].map({ LLM::Tool.new($_) });

.say for @toolObjects
```
```
# LLMTool(power-disco-ball-impl, Powers the spinning disco ball.)
# LLMTool(start-music-impl, Play some music matching the specified parameters.)
# LLMTool(dim-lights-impl, Dim the lights.)
```

Make an `LLM::Request` object for each request from the (first) LLM response:

```raku
my @requestObjects = $response<candidates>»<content>»<parts>.&flatten»<functionCall>.map({ LLM::ToolRequest.new( $_<name>, $_<args>) });

.say for @requestObjects
```
```
# LLMToolRequest(start-music-impl, :$loud(1), :$energetic(1), :id(Whatever))
# LLMToolRequest(power-disco-ball-impl, :$power(1), :id(Whatever))
# LLMToolRequest(dim-lights-impl, :$brightness(0.5), :id(Whatever))
```

Using the relevant tool for each request compute tool's response (which are `LLM::ToolResponse` objects):

```raku
.say for @requestObjects.map({ generate-llm-tool-response(@toolObjects, $_) })».output
```
```
# {music_type => energetic, volume => loud}
# {status => Disco ball powered on}
# {brightness => 0.5}
```

Alternatively, the `LLM::ToolResponse` objects can be converted into hashmaps structured according a particular LLM function calling style (Gemini in this case):

```raku
.say for @requestObjects.map({ generate-llm-tool-response(@toolObjects, $_) })».Hash('Gemini')
```
```
# {functionResponse => {name => start-music-impl, response => {content => {music_type => energetic, volume => loud}}}}
# {functionResponse => {name => power-disco-ball-impl, response => {content => {status => Disco ball powered on}}}}
# {functionResponse => {name => dim-lights-impl, response => {content => {brightness => 0.5}}}}
```

Process the response:
- Make a request object for each function call request
- Compute the tool results
- Form corresponding user message with those results
- Send the messages to the LLM

```raku
my $assistant-message = $response<candidates>[0]<content>;
if $assistant-message<parts> {

    # Find function call parts and make corresponding tool objects
    my @requestObjects;
    for |$assistant-message<parts> -> %part {
        if %part<functionCall> {
            @requestObjects.push: LLM::ToolRequest.new( %part<functionCall><name>, %part<functionCall><args> ) 
        }
    }    

    # Add assistance message
    @messages2.push($assistant-message);

    # Compute tool responses
    my @funcParts = @requestObjects.map({ generate-llm-tool-response(@toolObjects, $_) })».Hash('Gemini');

    # Make and add the user response
    my %function-response =
        role => 'user',
        parts => @funcParts;

    @messages2.push(%function-response);
                
    # Send the second request with function result
    my $final-response = gemini-generate-content(
        @messages2,
        :@tools,
        :$model,
        format => "raku"
    );
                
    say "Assistant: ", $final-response<candidates>[0]<content><parts>».<text>.join("\n");

} else {
    say "Assistant: $assistant-message<content>";
}
```
```
# Assistant: Alright! I've started some energetic and loud music, turned on the disco ball, and dimmed the lights to 50% brightness. Let's get this party started!
```

**Remark** Compared to the workflows in [AA1, AA2] the code above in simpler, more universal and robust, and handles all tool requests


-----

## Conclusion


We can observe and conclude that LLM function calling workflows are greatly simplified by:

- Leveraging Raku introspection
    - This requires documenting the subs and their parameters.
- Using dedicated classes that represent tool:
    - Definitions, (`LLM::Tool`)
    - Requests, (`LLM::ToolRequest`)
    - Responses, (`LLM::ToolResponse`)
- Having a sub (`generate-llm-tool-response`) that automatically matches request objects to tool objects and produces the corresponding response objects.
    - Note the Gemini's documentation does not show that matching in the corresponding function calling example ["Parallel Function Calling"](https://ai.google.dev/gemini-api/docs/function-calling#parallel_function_calling).

Raku's LLM tools automation is similar to Gemini's ["Automatic Function Calling (Python Only)"](https://ai.google.dev/gemini-api/docs/function-calling?example=weather#automatic_function_calling_python_only).

The Wolfram Language LLM tooling functionalities are reflected in Raku's "LLM::Tooling", [WRI1].


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

[AA3] Anton Antonov,
["LLM function calling workflows (Part 3, Facilitation)"](https://rakuforprediction.wordpress.com/2025/06/08/llm-function-calling-workflows-part-3-facilitation/),
(2025),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[Gem1] Google Gemini,
["Gemini Developer API"](https://ai.google.dev/gemini-api/docs).

[WRI1] Wolfram Research, Inc.
["LLM-Related Functionality" guide](https://reference.wolfram.com/language/guide/LLMFunctions.html).




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
