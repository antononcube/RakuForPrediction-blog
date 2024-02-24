# Using DALL-E models in Raku

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)    
December 2023    

## Introduction

In this document we proclaim recent creation and updates of several Raku packages that facilitate the utilization of the OpenAI's [DALL-E 3](https://openai.com/dall-e-3) model. See [AAp1, AAp3, AAp4, AAp5].

The exposition of this document was designed and made using the [Jupyter framework](https://jupyter.org) -- we use a Raku Jupyter chat-enabled notebook, [AAp2].

**Remark:** Chat-enabled notebooks are called **chatbooks**.

We discuss workflows and related User eXperience (UX) challenges, then we demonstrate image-generation workflows.

The demonstrations are within a Raku chatbook, see ["Jupyter::Chatbook"](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp4]. That allows interactive utilization of Large Language Models (LLMs) and related Artificial Intelligence (AI) services.

In the past, we made presentations and movies using DALL-E 2; see [AAv1, AAv2]. Therefore, in this document we use DALL-E 3 based examples. Nevertheless, we provide a breakdown table that summarizes and compares the parameters of DALL-E 2 and DALL-E 3.

Since this document is written at the end of the year 2023, we use the generation of Christmas and winter themed images as examples. Like the following one:


```raku
#% dalle, model=dall-e-3, size=landscape
Generate snowfall with a plain black backgroud. Make some of the snowflakes look like butterflies.
```

![](./Diagrams/Using-DALL-E-models-in-Raku/butterfly-snowflakes-small.png)

## Document structure

Here is the structure of the document:

- Image generation related workflows 
- Implementation and UX challenges
- Image generation examples
    - Showing how the challenges are addressed.
- Programmatic example
    - Via AI vision.
- Breakdown table of DALL-E 2 vs DALL-E 3 parameters

-------

## Workflows

### Just generating images

The simplest workflow is:

1. Pick a DALL-E model.
2. Pick image generation parameters.
    - Choose the result format: 
        - URL -- the image is hosted remotely
        - Base64 string -- the image is displayed in the notebook, [AAp5], and stored locally
3. Generate the image.
4. If not satisfied or tired goto 1.

### Review and export images 

It would be nice, of course, to be able to review the images generated within a given Jupyter session and export a selection of them. (Or all of them.)

Here is a "single image export" workflow:

1. Review generated (and stored) images in the current session
2. Select an image
3. Export the image by specifying:
   - Index (integer or `Whatever`)
   - File name (string or `Whatever`))

Here is an "all images export" workflow:

1. Review generated (and stored) images in the current session
2. Export all images in the current session

The "automatic" file names in both workflows are constructed with a file name prefix specification (using the magic cell option "prefix".)

**Remark:** In order to publish this article to GitHub and WordPress we used the "all image export" workflow.

### Programmatic utilization

Programmatic workflows tie up image generation via "DALL-E direct access" with further programmatic manipulation. One example, is given in ["AI vision via Raku"](https://rakuforprediction.wordpress.com/2023/11/25/ai-vision-via-raku/), [AA3]:

1. LLM-generate a few images that characterize parts of a story.
    - Do several generations until the images correspond to "story's envisioned look."
2. Narrate the images using the LLM “vision” functionality.
3. Use an LLM to generate a story over the narrations.


------

## Challenges and implementations that address them

We can say that a Jupyter notebook provides a global "management session" that orchestrates (a few) LLM "direct access" sessions, and a Raku REPL session.

Here is a corresponding Mermaid-JS diagram:


```raku
#% mermaid
graph LR
    JuSession{{Jupyter session}} <--> Raku{{"Raku<br>REPL"}}
    JuSession --> DALLE{{"DALL-E<br>proxy"}}
    DALLE <--> Files[(File system)]
    DALLE <--> IMGs[(Images)]
    Raku <--> Files
    DALLE --> Cb([Clipboard])
    Raku <--> Cb
    DALLE -.- |How to exchange data?|Raku
    subgraph Chatbook
        JuSession
        Raku
        DALLE
        IMGs
    end
    subgraph OS
        Files
        Cb
    end
```

![](./Diagrams/Using-DALL-E-models-in-Raku/Jupyter-sesssion-DALLE-proxy-and-Raku-REPL.png)

**Remark:** Raku chatbooks have Mermaid-JS magic cell that use ["WWW::MermaidInk"](https://raku.land/zef:antononcube/WWW::MermaidInk), [AAp6].

### Challenges

Here is a list of challenges to be addressed in order to facilitate the workflows outlined above:

- How the Raku REPL session communicates with the DALL-E access "session"?
- How an image is displayed in:
    - Jupyter notebook
    - Markdown document
- What representation of images to use?
    - Base64 strings, or URLs, or both?
- How images are manipulated in Raku?
    - What kind of manipulations are:
        - Desireable
        - Possible (i.e. relatively easy to implement)
- How an image generated in a "direct access" Jupyter cell is made available in the running along Raku session?
    - More generally, how the different magic cells "communicate" their results to the computational cells?
- If a Jupyter notebook session stores the generated images:
    - How to review images?
    - How to delete images?
    - How to select an image and export it?
    - How to export all images?
    - How to find the file names of the exported images?

### Solution outline 

Addressing the implementation and UX challenges is done with three principle components:

- A dedicated Raku package for handling images in markup environments; see ["Image::Markup::Utilities"](https://raku.land/zef:antononcube/Image::Markup::Utilities), [AAp5]. Allows:
    - Streamlined display of images in Markdown and Jupyter documents
    - Image import from file path or URL
    - Image export to file system
- Storing of DALL-E generated images within a Jupyter session.
    - In order to provide the Raku session access to DALL-E session artifacts.
- Chatbook magic cell type for *meta operations* over stored DALL-E images.
    - In order review, delete, or export images.

Additional solution elements are:

- Export paths are placed in the clipboard.
- DALL-E generation cells have parameter "response-format" that allows getting both URLs and Base64 strings.
- If the result is a URL then it is placed in the OS clipboard.
- Instead of introducing an image Raku class we consider an image to be "just" a Base64 string or an URL. See [AAp5].


**Remark:** OpenAI's vision works with both Base64 and URLs. URLs are preferred for [longer running conversations](https://platform.openai.com/docs/guides/vision/managing-images). 

-------

## Basic image generation workflows

Here we ask *the default* LLM about Chinese horoscope signs of this and next year: 


```raku
#% chat, temperature=0.3
Which Chinese horoscope signs are the years 2023 and 2024?
```




    The Chinese zodiac follows a 12-year cycle, with each year associated with a specific animal sign. In 2023, the Chinese zodiac sign will be the Rabbit, and in 2024, it will be the Dragon.



### Image generation followed by "local" display

Here we ask OpenAI's DALL-E system (and service) to generate an image of a dragon chasing a rabbit:


```raku
#% dalle, model=dall-e-3, size=landscape
A Chinese ink wash painting of a zodiac dragon chasing a zodiac rabbit. Fractal snow and clouds. Use correct animal anatomy!
```

![](./Diagrams/Using-DALL-E-models-in-Raku/chinese-ink-wash-small.png)

The DALL-E magic cell argument "model" can be `Whatever` of one of "dall-e-2" or "dall-e-3".

### Using remote URL of the generated image

Here we use a different prompt to generate a certain image that is both Raku- and winter related -- we request the image of DALL-E's response to be given as a URL and the result of the chatbook cell to be given in JSON format:


```raku
#% dalle, model=dall-e-3, size=landscape, style=vivid, response-format=url, format=json
A digital painting of raccoons having a snowball fight around a Christmas tree.
```




{
  "data": [
    {
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-KbuLSsqssXAPQFZORGWZzuN0/user-Ss9QQAmz9L5UJDcmKnhxnRoT/img-ResmF1iVDSxEGb6ORTkjy200.png?st=2023-12-18T00%3A46%3A22Z&se=2023-12-18T02%3A46%3A22Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-12-17T19%3A56%3A04Z&ske=2023-12-18T19%3A56%3A04Z&sks=b&skv=2021-08-06&sig=YWFpfGlDry1L5C8aU1hMUhum02uTvyqwAyOgi3A9ZaU%3D",
      "revised_prompt": "Create a digital painting showing a merry scene of raccoons engaging in a playful snowball fight. The backdrop should consist of a snow-filled winter landscape illuminated by the soft glow of a magnificently decorated Christmas tree. The raccoons should be exuding joy, raising their tiny paws to throw snowballs, their eyes twinkling with mischief. The Christmas tree should be grand and towering, adorned with vibrant ornaments, strands of sparkly tinsel, and a star at the top, reflecting both tradition and cheer."
    }
  ],
  "created": 1702863982
}



**Remark:** DALL-E 3 model revises the given prompts. We can paste the result above into a Raku computational cell using the clipboard shortcut, (Cmd-V).

Here we display the image directly as a Markdown image link (using the command `paste` of ["Clipboard"](https://raku.land/zef:antononcube/Clipboard)):


```raku
#% markdown
my $url = from-json(paste)<data>.head<url>;
"![]($url)"
```

![](./Diagrams/Using-DALL-E-models-in-Raku/raccoons-with-snowballs-small.png)


Alternatively, we can import the image from the given URL and display it using the magic `%% markdown` (intentionally not re-displayed again):


```raku
# %% markdown
my $img = image-import($url);
$img.substr(0,40)
```




    ![](data:image/jpeg;base64,iVBORw0KGgoAA



Here we export into the local file system the imported image:


```raku
image-export($*CWD ~ '/raccoons-with-snowballs.png', $img).IO.parts.tail
```




    basename => raccoons-with-snowballs.png



**Remark:** Generally speaking, importing and displaying image Base64 strings is a few times slower than using image URLs.

------

## Meta cells for image review and export

Here we use a DALL-E meta cell to see how many images were generated in this session:


```raku
#% dalle meta
elems
```




    2



Here we export the second image (dragon and rabbit) into a file named "chinese-ink-wash.png":


```raku
#% dalle export, index=1
chinese-ink-wash.png
```




    chinese-ink-wash.png



Here we show all generated images:


```raku
#% dalle meta
show
```

Here we export all images (into file names with the prefix "advent2023"):


```raku
#% dalle export, index=all, prefix=advent2023
```

-----

## Programmatic workflow with AI vision

Let us demonstrate the [OpenAI's Vision](https://platform.openai.com/docs/guides/vision) using the "raccoons and snowballs" image generated above:


```raku
llm-vision-synthesize("Write a limerick based on the image.", $img)
```




    In a forest so snowy and bright,
    Raccoons played in the soft moonlight.
    By a tree grand and tall,
    They frolicked with a ball,
    And their laughter filled the night.



**Exercise:** Verify that limerick fits the image.

**Remark:** The functions `llm-vision-synthesize` and `llm-vision-function` were added to ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions) after writing (and posting) ["AI vision via Raku"](https://rakuforprediction.wordpress.com/2023/11/25/ai-vision-via-raku/), [AA3]. We plan to make a more dedicated demonstration of those functions in the near future.

--------

## Breakdown of model parameters

As it was mentioned above, the DALL-E magic cell argument "model" can be `Whatever` of one of "dall-e-2" or "dall-e-3".

Not all parameters that are valid for one of the models are valid or respected by the other -- see the subsection ["Create image"](https://platform.openai.com/docs/api-reference/images/create) of [OpenAI's documentation](https://platform.openai.com/docs/api-reference).

Here is a table that shows a breakdown of the model-parameter relationships:

| Parameter | Type | Required/Optional | Default | dall-e-2 | dall-e-3 | Valid Values |
|-----------|------|------------------|---------|----------|----------|--------------|
| prompt | string | Required | N/A | ✔️ | ✔️ | Maximum length is 1000 characters for dall-e-2 and 4000 characters for dall-e-3 |
| model | string | Optional | dall-e-2 | ✔️ | ✔️ | N/A |
| n | integer or null | Optional | 1 | ✔️ | ✔️ (only n=1) | Must be between 1 and 10. For dall-e-3, only n=1 is supported |
| quality | string | Optional | standard | ❌ | ✔️ | N/A |
| response_format | string or null | Optional | url | ✔️ | ✔️ | Must be one of url or b64_json |
| size | string or null | Optional | 1024x1024 | ✔️ (256x256, 512x512, 1024x1024) | ✔️ (1024x1024, 1792x1024, 1024x1792) | Must be one of 256x256, 512x512, or 1024x1024 for dall-e-2. Must be one of 1024x1024, 1792x1024, or 1024x1792 for dall-e-3 models |
| style | string or null | Optional | vivid | ❌ | ✔️ | Must be one of vivid or natural |

-------

## References

### Articles

[AA1] Anton Antonov, ["Workflows with LLM functions"](https://rakuforprediction.wordpress.com/2023/08/01/workflows-with-llm-functions/), (2023), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov, ["Re-programming to Python of LLM- and Chatbook packages"](https://rakuforprediction.wordpress.com/2023/10/15/re-programming-to-python-of-llm-and-chatbook-packages/), (2023), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA3] Anton Antonov, ["AI vision via Raku"](https://rakuforprediction.wordpress.com/2023/11/25/ai-vision-via-raku/), (2023), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[OAIb1] OpenAI team, ["New models and developer products announced at DevDay"](https://openai.com/blog/new-models-and-developer-products-announced-at-devday), (2023), [OpenAI/blog](https://openai.com/blog).


### Packages

[AAp1] Anton Antonov, [WWW::OpenAI](https://github.com/antononcube/Raku-WWW-OpenAI) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov, [WWW::PaLM](https://github.com/antononcube/Raku-WWW-PaLM) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov, [LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).

[AAp4] Anton Antonov, [Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov, [Image::Markup::Utilities](https://github.com/antononcube/Raku-Image-Markup-Utilities) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov, [WWW::MermaidInk](https://github.com/antononcube/Raku-WWW-MermaidInk) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube).


### Videos

[AAv1] Anton Antonov, ["Jupyter Chatbook LLM cells demo (Raku)"](https://www.youtube.com/watch?v=cICgnzYmQZg), (2023), [YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov, ["Integrating Large Language Models with Raku"](https://www.youtube.com/watch?v=-OxKqRrQvh0), (2023), [YouTube/@therakuconference6823](https://www.youtube.com/@therakuconference6823).
