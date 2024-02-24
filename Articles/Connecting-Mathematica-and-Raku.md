# Connecting Mathematica and Raku

**Version 0.8**

Anton Antonov    
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)  
December 2021  


## Introduction

Connecting [Mathematica](https://www.wolfram.com/mathematica/) and [Raku](https://raku.org) allows facilitating and leveraging some interesting synergies between the two systems.

In this document we describe several ways of connecting Mathematica and Raku:

1. Using external, operating system runs

1. Using in-process communication sockets

1. Using a Web service

Additional, related topics are:

1. Encoders and decoders (for both Mathematica and Raku expressions)

1. The making of notebook Raku-style and Raku-cells

1. The making and utilization of Domain Specific Language (DSL) cells

**Remark:** In this document I use Mathematica and Wolfram Language (WL) as synonyms. If we have to be precise, we could say something like “Mathematica is the software system and WL is the backend programming language in the software system.”

### Preliminary examples

Here is an example of a Raku cell:

```perl6
say (1+1_000)**2

(*"1002001"*)
```

Here is an example of a Domain Specific Language (DSL) cell that does parsing and interpretation only 
(generates code, does not evaluate it):

```dsl
DSL MODULE DataQuery;
use dfTitanic;
group by passengerSex;
show counts
```

```mathematica
obj = dfTitanic;
obj = GroupBy[ obj, #["passengerSex"]& ];
Echo[Map[ Length, obj], "counts:"]
```

(Below we provide more detailed examples.)

### Why is this useful?

#### Mathematica-centric answers

Here are some Mathematica-centric reasons about the usefulness of connecting Mathematica and Raku:

- Raku has built-in UTF-8 symbols treatment and bignums, hence it is interesting to compare its computational model with that of Mathematica or other external evaluators that Mathematica supports.

    - In my view Raku is the only “true” *potential* competitor of Mathematica that is not a LISP descendant.

        - I plan to discuss this in another document, not in this one.

- Raku has a great built-in system of grammars and interpreter actions, which can be used to complete, replace, or amplify WL’s built-in functionalities.

- The utilization of a constellation of DSL packages for code generation from the [”Raku for Prediction” project](https://github.com/antononcube/RakuForPrediction-book).

    - I admit, this is a very biased and personal reason.

#### Raku-centric answers

Here are some Raku-centric reasons about the usefulness of connecting Mathematica and Raku:

- Mathematica is the most powerful mathematical software system and has one of the oldest, most mature notebook solutions.

- Using notebooks facilitates interactive development or research (in general and with Raku.)

- The ability to visualize, and plot results derived with Raku.

- Combining evaluations with other programming languages.

    - Other programming languages that can be run in Mathematica notebooks: Python, R, Julia, etc.

    - It is a great way to demonstrate the ideas and abilities of the 
      [“Raku for Prediction” project](https://github.com/antononcube/RakuForPrediction-book).

- [Literate programming](https://en.wikipedia.org/wiki/Literate_programming).

- Comparative testing of results correctness:

    - Verifying that new Raku implementations do "the right thing"

    - Comparison with other languages "doing the same thing"

### Orientation mind-map

The following mind-map shows the topics covered in this document from a “package perspective” (the 
[linked PDF version](https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/Connecting-Mathematica-and-Raku-mind-map.pdf) 
has “life” hyperlinks):

[![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/Connecting-Mathematica-and-Raku-mind-map.png)](https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/Connecting-Mathematica-and-Raku-mind-map.pdf)

### Reading orientation

A list of short descriptions of the sections below and their importance follows. 
(The **important** sections are written with bold font weight; the ***not important*** ones in bold font weight and italic slant.)

- ***“The journey”*** outlines my development efforts to make Raku available into Mathematica notebooks and other types of documents.

- **“RakuMode”** describes the use of Raku cells in notebooks.

- **“DSLMode”** describes the use of DSL cells that utilize Raku evaluations.

- **“Web service”** describes the [web service](https://antononcube.shinyapps.io/DSL-evaluations/) 
   programmed in Raku that leverages the use of the [Wolfram Engine](https://www.wolfram.com/engine/) 
   for generating code through a [NLP Template Engine](https://github.com/antononcube/NLP-Template-Engine).

- ***“Encoders and decoders”*** discusses the programming and application of Raku and WL encoders.

- **“Example: Numeric word forms”** shows how Raku package commands can be used to parse integer names generated with WL built-in commands. (A “synergy” demo.)

- ***“Example: Stoichiometry”*** shows how Raku package commands can be used to retrieve chemical elements data and balance chemical equations, and how that compares to WL’s built-in functionalities. (A “comparison” demo.)

- ***“Making of the DSL cells”*** discusses how the DSL cells (style data) were programmed.

- ***“Making of the Raku cell”*** discusses how the Raku cell (style data) and “prefix” icon was programmed.

- ***“Future plans”*** outlines future plans for related development efforts.

### Execution

This notebook can be executed, but Raku have to be installed for that. 
The function `StartRakuProcess` takes the option setting `Raku->"some/path/to/raku"` that allows to specify 
where the Raku executable is. (To get Raku see https://raku.org or https://rakudo.org .)

------

## The journey

This diagram summarizes my “Raku connectivity” journey:

```mathematica
plJourney = ImageCrop@Import["https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/Raku-hook-up-to-notebooks-journey.jpg"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0azdgmmhtn602.png)

Here is some narration:

1. I developed a dozen of Raku DSL packages that can translate natural language specifications into executable programming code in different languages.

2. Initially I used simple Operating System (OS) redirection calls to get the code generated by the DSL packages from specialized DSL-mode cells.

    1. For example, the WTC-2020 presentation [“Multi-language Data-Wrangling Conversational Agent”](https://www.youtube.com/watch?v=pQk5jwoMSxs),  [AAv1], used that mechanism.

    2. At that point I developed the Python, R, and WL packages with names ["ExternalParsersHookup"](https://github.com/antononcube/ConversationalAgents/tree/master/Packages/).

    3. I also implemented related Raku-mode and DSL-mode notebook styles for Mathematica.

3. The initial approach had two significant problems:

    1. The Raku and DSL cells did not “keep state” between each other -- each cell was executing code on its own.

    2. The evaluation was slow, since every time Raku had to be started and the corresponding packages loaded.

4. The want to “do it right” raised a fair amount of questions; the main ones are:

    1. How to start a resident process from within, say, Mathematica? 

    2. How to establish a connection to that process?

    3. What are the applicable (and “standard”) software components or solutions for that kind of architectures?

5. Turned out Mathematica was fairly well equipped to do these kind of inter-process connections. 

    1. See the guides:

        - ["Direct Control of External Processes"](https://reference.wolfram.com/language/guide/DirectControlOfExternalProcesses.html)

        - ["External Language Interfaces"](https://reference.wolfram.com/language/guide/ExternalLanguageInterfaces.html)

    2. I was aware of many of the external process WL functionalities, but [SocketConnect](https://reference.wolfram.com/language/ref/SocketConnect.html) had the relatively recent addition of [ZeroMQ](https://zeromq.org) (in 2017) which I was not aware of.

6. Research and reading for possible solutions.

    1. Considered JVM architectures and org-mode related solutions.

    2. The [Babel org-mode](https://orgmode.org/worg/org-contrib/babel/) solution I found for evaluating Raku source blocks was very nice and insightful to read, but essentially the same as my first Mathematica-to-Raku connection solution.

7. After some research and reading I decided to use [ZeroMQ](https://zeromq.org).

    1. ZeroMQ is used by other external evaluators in Mathematica (explained well in the documentation.)

    2. The [ZeroMQ documentation](https://zeromq.org/get-started/) is fun to read and has examples in multiple programming languages.

8. It seemed better to generalize the problem and develop a Raku module for sandboxed Raku execution.

    1. The evaluation cells are now the code cells in notebooks, or Markdown, org-mode, or Raku Pod6 files.

    2. The sandboxed Raku can have a “persistent” context that is accessed by the evaluation cells.

    3. Implemented the Raku package ["Text::CodeProcessing"](https://modules.raku.org/dist/Text::CodeProcessing:cpan:ANTONOV), [AAp9].

    4. Studied the work on [connecting Raku to Jupyter by Brian Duggan](https://github.com/bduggan/p6-jupyter-kernel), [BD1].

9. Implemented the corresponding WL packages that utilize the Raku package [“Text::CodeProcessing”](https://modules.raku.org/dist/Text::CodeProcessing:cpan:ANTONOV).

    1. Made several versions of those implementations connecting Raku to R, Python, and Wolfram Engine.

    2. The Raku-to-Wolfram-Engine connection was used in dedicated Web services.

------

## RakuMode

Here we load the ["RakuMode.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuMode.m) package:

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuMode.m"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/019tm4hg43dpy.png)

**Remark:** The package ["RakuMode.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuMode.m) is very lightweight code-wise. The only “large part” is the Camelia icon for the Raku evaluation cells.

We convert notebooks into Raku-mode with this command:

```mathematica
RakuMode[]
```

In Raku-mode we have Raku cells that allow evaluation of Raku code (within Mathematica notebooks.)

Raku-mode cells execute Raku code via either:

- RunProces

- The socket connection functions BinaryWrite,  SocketReadMessage, and ByteArrayToString

### No ZeroMQ connections

Without ZeroMQ sockets 
["RakuMode.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuMode.m) 
uses the (very lightweight) package 
["RakuCommand.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuCommand.m). 
Here is an example:

```perl6
say(1+1_000)

# 1001
```

Let us make an intentional omission in order to illustrate that RunProcess is used:

```perl6
1+1_000
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1qx8lyemquwgg.png)


We get the message above because we essentially executed the shell command:

```shell
raku -e "1_1_000"
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/06aldtvfu2gf0.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0axdqw853zczv.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0m2y8cd79n6v7.png)

### Via ZeroMQ

Using Raku cells that do Raku evaluations over [ZeroMQ](https://zeromq.org) sockets is a primary use case of the packages described in this document. ZeroMQ is used in WL for other external evaluators ([Python](https://reference.wolfram.com/language/workflow/ConfigurePythonForExternalEvaluate.html.en), [Julia](https://reference.wolfram.com/language/workflow/ConfigureJuliaForExternalEvaluate.html.en), etc.)

First we start a Raku process:

```mathematica
StartRakuProcess[]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0brgm1yborxld.png)

Here we create a Raku cell -- using the shortcut “Shift-|” -- and specify the loading of the package [”Lingua::NumericWordForms”](https://modules.raku.org/dist/Lingua::NumericWordForms:cpan:ANTONOV):

```perl6
use Lingua::NumericWordForms

# "(Any)"
```

Here we define a variable and assign to it an array of numeric word forms (in Bulgarian, English, and Spanish):

```perl6
my @nforms = [‘двеста осемдесет и седем’, ‘two hundred and five’, ‘ochocientos setenta y dos’];

# [двеста осемдесет и седем two hundred and five ochocientos setenta y dos]
```

Here we parse-and-interpret several numeric word forms into numbers (and show the corresponding languages):

```perl6
from-numeric-word-form(@nforms):p

# (bulgarian => 287 english => 205 spanish => 872)
```

The last Raku cell uses a function provided by the package in the first cell and a variable defined in the second cell. In other words, there is a common Raku context that is accessed by those cells.

### Flow chart walkthrough

Let us provide a schematic description of the example in the previous sub-section. The following flow chart summarizes the creation and evaluation of Raku cells:

```mathematica
ImageCrop@Import["https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/Raku-execution-in-Mathematica-notebook.jpg"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0ufb0tefazu9g.png)

Here is a narrative for the flow chart above:

1. The user converts the Mathematica notebook into RakuMode and starts a Raku process with StartRakuProcess

    1. StartRakuProcess uses StartProcess to start Raku 

    2. Socket connection is established with the Raku process though ZeroMQ

2. The Raku process loads the package [”Text::CodeProcessing”](https://modules.raku.org/dist/Text::CodeProcessing:cpan:ANTONOV)

    1. That package is used to start a sandboxed Raku environment 

    2. The sandboxed Raku environment can be seen as REPL that has its own context

3. The user makes a Raku cell and enters Raku code

4. The user triggers the evaluation of the cell

5. The cell content evaluation is done with the function RakuInputExecute

    1. Raku code is converted to a binary array and sent through a ZeroMQ socket to Raku REPL

    2. Raku REPL evaluates the code 

    3. The result is send back to WL through the ZeroMQ socket

6. The result of the Raku cell evaluation is placed in the notebook as an output cell

**Remark:** In the flow chart there is an optional application of the Mathematica and Raku encoders and decoders. The examples below provide more details.

------

## DSLMode

Here we load the DSLMode package (which triggers the loading of other packages for different computational workflows):

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/DSLMode.m"];
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1k6gc0yam33d7.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1fr7yisd7duix.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/07thl9qnsfb7y.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/09prrph90tspg.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0a7g9xk2k3t6x.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0cilz41olsms0.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/17pj1nyjw6yuk.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/09e7rsgw6829m.png)

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0r49s4dj7y85u.png)

Here we convert the notebook into DSL-mode:

```mathematica
DSLMode[]
```

Here we start a Raku process if we have not started one already:

```mathematica
(*StartRakuProcess[]*)
```

Here we use natural language commands to specify a data wrangling workflow in a ***DSL parsing cell***:

DSL TARGET Python::pandas;
include setup code;
load the dataset iris;
group by the column Species;
show counts

```python
import pandas
from ExampleDatasets import *

obj = example_dataset('iris')
obj = obj.groupby(["Species"])
print(obj.size())

# Species
# setosa        50
# versicolor    50
# virginica     50
# dtype: int64
```

**Remark:** Evaluating the DSL cell above produces a Python cell, which was then manually evaluated.

In the next example we use a DSL execution cell, but in order to see the DSL parser-interpreter result we are going to change the method option of the function DSLInputExecute to print the generated code before execution:

```mathematica
SetOptions[DSLInputExecute, Method -> "PrintAndEvaluate"]

(*{Method -> "PrintAndEvaluate"}*)
```

Here is a ***DSL evaluation cell*** with the same data wrangling workflow specification as above except the target language is Raku:

DSL TARGET Raku::Reshapers;
include setup code;
load the dataset mtcars ;
group by the column cyl;
show counts

```perl6
use Data::Reshapers;
use Data::Summarizers;
use Data::ExampleDatasets;

my $obj = example-dataset('mtcars') ;
$obj = group-by( $obj, "cyl") ;
say "counts: ", $obj>>.elems

# counts: {4 => 11, 6 => 7, 8 => 14}
```

**Remark:** This time the generated code was automatically evaluated when the DSL cell was evaluated.

**Remark:** The DSL parsing cell has light blue background, the DSL evaluation cell has light yellow background.

Here we can see the output from Raku before it is post-processed in ExternalParsersHookup`ToDSLCode:

```mathematica
res = 
   ToDSLCode["DSL TARGET Python::pandas;load the dataset mtcars ;group by the column cyl;show counts", Method -> Identity];
ResourceFunction["GridTableForm"][List @@@ Normal[KeySort@res], TableHeadings -> {"Key", "Value"}]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0u5k3a35gzoof.png)

The underlying Raku function is ToDSLCode from the package [”DSL::Shared::Utilities::ComprehensiveTranslation”](https://github.com/antononcube/Raku-DSL-Shared-Utilities-ComprehensiveTranslation):

```perl6
ToDSLCode('DSL TARGET Python::pandas;
load the dataset mtcars ;
group by the column cyl;
show counts')

{CODE => obj = example_dataset('mtcars')obj = obj.groupby([\"cyl\"])print(obj.size()),
 COMMAND => DSL TARGET Python::pandas;load the dataset mtcars ;group by the column cyl;show counts, 
 DSL => DSL::English::DataQueryWorkflows, 
 DSLFUNCTION => proto sub ToDataQueryWorkflowCode (Str $command, Str $target = \"tidyverse\", |) {*}, 
 DSLTARGET => Python::pandas, 
 USERID => }
```

**Remark:** By  default Raku’s ToDSLCode returns a hash. WL’s ToDSLCode returns an association with Method->Identity.

------

## Web service

We can provide a Web service via the constellation of Raku libraries [Cro](https://cro.services) 
that translates natural language DSL specifications into executable code. See the video [AAv4] for a demonstration of 
such a system. Below we refer to it as the Cro Web Service (CWS). 

### Getting template code

Here is an example of using CWS through Mathematica’s web interaction function 
[`URLRead`](https://reference.wolfram.com/language/ref/URLRead.html), [WRI3], 
in order to get the R code of Latent Semantic Analysis (LSA) workflow:

```mathematica
command = "use aAbstracts; make document term matrix;apply LSI functions IDF, None, Cosine; extract 40 topics using method SVD;echo topics table" // StringTrim;
res = Import@URLRead[<|"Scheme" -> "http", "Domain" -> "accendodata.net", "Port" -> "5040", "Path" -> {"translate", "qas"}, "Query" -> <|"command" -> command, "lang" -> "R"|>|>];
```

```mathematica
ResourceFunction["GridTableForm"][List @@@ ImportString[res, "JSON"], TableHeadings -> {"Key", "Value"}]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1em852uqkxidb.png)

The code was obtained by using a LSA template, the slots of which were filled-in by utilizing a Question Answering System (QAS). 
See the project 
["NLP Template Engine"](https://github.com/antononcube/NLP-Template-Engine) 
and the movie 
["NLP Template Engine, Part 1"](https://youtu.be/a6PvmZnvF9I). 

**Remark:** The QAS utilized in this implementation is based on WL’s function 
[FindTextualAnswer](https://reference.wolfram.com/language/ref/FindTextualAnswer.html.en).

### Schematic overview

Here is a components diagram of the process utilized above:

```mathematica
ImageCrop@Import["https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/DSL-Web-Service-via-Cro-with-WE-QAS.jpg"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0p6duuveas2jd.png)

The components are (left to right):

- Any notebook or Integrated Development Environment (IDE) editable file.

- Web service API for CWS

    - We can also have an [interactive interface](https://antononcube.shinyapps.io/DSL-evaluations/), say, hosted at [shinyapps.io](https://www.shinyapps.io).

- Web service run on a certain server

    - The diagram indicates that I use [DigitalOcean](https://www.digitalocean.com).

- CWS that is up and running

- Up and running Wolfram Engine to which CWS connects via ZeroMQ

Here is some narrative for getting DSL translation code by the NLP Template Engine:

1. In a notebook invoke a call to CWS

2. CWS uses the “resident” process implemented in Raku (using the family of libraries Cro)

3. CWS connects to Wolfram Engine through ZeroMQ

4. Wolfram Engine uses the packages of NLP Template Engine to fill-in the slots of relevant code templates

   - The relevant templates a guessed by a Machine Learning classifier.

5. The result is given back to the notebook

------

## Encoders and decoders

### Setup

Here we load a WL package with a decoder of Raku expressions:

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuDecoder.m"]
```

Here we load a WL package with an encoder to Raku expressions:

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/RakuEncoder.m"]
```

Here we load a Raku package that has functions to encode Raku objects into WL expressions:

```perl6
use Mathematica::Serializer;

(*"(Any)"*)
```

### Encoding to Raku

Here we create a small random dataset:

```mathematica
SeedRandom[12];
dsRand = ResourceFunction["RandomTabularDataset"][{4, 3}]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0e28qoh61g8ud.png)

Here is how the dataset looks encoded in Raku (an array of hashes):

```mathematica
ToRakuCode[dsRand]

(*"[%('agree'=>-7.08447,'prankster'=>'while','rapture'=>-34),%('agree'=>-8.28714,'prankster'=>'extreme','rapture'=>86),%('agree'=>1.69727,'prankster'=>'esprit','rapture'=>-2),%('agree'=>9.38245,'prankster'=>'wintergreen','rapture'=>52)]"*)
```

Here we assign the encoded dataset to a Raku variable:

```mathematica
RakuInputExecute["my @dsAWs = " <> ToRakuCode[dsRand]];
```

Here is the dataset tabulated in Raku:

```perl6
use Data::Reshapers;
say to-pretty-table(@dsAWs)

# +---------+-------------+-----------+
# | rapture |  prankster  |   agree   |
# +---------+-------------+-----------+
# |   -34   |    while    | -7.084470 |
# |    86   |   extreme   | -8.287140 |
# |    -2   |    esprit   |  1.697270 |
# |    52   | wintergreen |  9.382450 |
# +---------+-------------+-----------+
```

### Decoding from Raku

In the following Raku cell we encode the result as a WL expression:

```perl6
@dsAWs==>encode-to-wl()

(*"WLEncoded[List[Association[Rule[\"rapture\",-34],Rule[\"prankster\",\"while\"],Rule[\"agree\",Rational[-708447,100000]]],Association[Rule[\"rapture\",86],Rule[\"agree\",Rational[-414357,50000]],Rule[\"prankster\",\"extreme\"]],Association[Rule[\"agree\",Rational[169727,100000]],Rule[\"rapture\",-2],Rule[\"prankster\",\"esprit\"]],Association[Rule[\"rapture\",52],Rule[\"agree\",Rational[187649,20000]],Rule[\"prankster\",\"wintergreen\"]]]]"*)
```

Here the string above is converted to a WL expression:

```mathematica
ToExpression[%] // First
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/08okmkt54fg6k.png)



Here we set Raku cells to use the decoding function FromRakuCode if the head of the expression is WLEncoded:

```mathematica
SetOptions[RakuInputExecute, Epilog -> FromRakuCode]

(*{"ModuleDirectory" -> "", "ModuleName" -> "", "Process" -> Automatic, Epilog -> FromRakuCode}*)
```

Let us evaluate the previous Raku cell again (we get a WL Dataset object right away):

```perl6
@dsAWs==>encode-to-wl()
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0d4rpdkdiyhea.png)

------

## Example: Numeric word forms

We showed some examples of parsing numeric word forms in the section “RakuMode”. In this section we extend those examples in order to demonstrate how Raku can be used in Mathematica to extend or replace existing functionalities or provide missing ones.

Here we make an association of numbers and their numeric word forms in English, then tabulate that association:

```mathematica
SeedRandom[26];
aNumericWordForms = KeySort@Association[# -> IntegerName[#, {"English", "Words"}] & /@ Join[RandomInteger[10^4, 3], RandomInteger[10^7, 3]]];
ResourceFunction["GridTableForm"][aNumericWordForms]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1cjnkb7z3ojvw.png)

Here make another number-to-word-form association using multiple languages:

```mathematica
SeedRandom[26];
aNumericWordForms2 = KeySort@Association[# -> IntegerName[#, {RandomChoice[{"Bulgarian", "Japanese", "Spanish"}], "Words"}] & /@ Join[RandomInteger[10^4, 3], RandomInteger[10^7, 3]]];
ResourceFunction["GridTableForm"][aNumericWordForms2]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1gjtq62uz503c.png)

Mathematica (Version 13) can parse English numeric word forms

```mathematica
SemanticInterpretation /@ aNumericWordForms

(*<|16 -> 16, 4286 -> 4286, 5481 -> 5481, 29695 -> 29695, 8224333 -> 8224333, 9537119 -> 9537119|>*)
```

But, at this point Mathematica (Version 13) cannot parse numeric word forms in other languages:

```mathematica
SemanticInterpretation[Values[aNumericWordForms2]]

(*{$Failed, $Failed, $Failed, $Failed, $Failed, $Failed}*)
```

The Raku package “Lingua::NumericWordForms” recognizes (automatically) a dozen of languages. Here is an example:

```perl6
from-numeric-word-form(['two hundred and five', 'двеста осемдесет и седем']):p

# (english => 205 bulgarian => 287)
```

Here is example of Raku-parsing of the values of the association created above:

```mathematica
RakuInputExecute["from-numeric-word-form(" <> ToRakuCode[Values[aNumericWordForms2]] <>"):p"]

(*"(japanese => 16 bulgarian => 4286 spanish => 5481 japanese => 29695 spanish => 8224333 japanese => 9537119)"*)
```

### Performance

Raku converts faster than WL numeric word forms into numbers. Here is the WL timing:

```mathematica
AbsoluteTiming[
  SemanticInterpretation[Values[aNumericWordForms]] 
 ]

(*{0.786108, {1131, 7504, 9970, 1016128, 5341656, 6865271}}*)
```

Here is the Raku timing:

```mathematica
AbsoluteTiming[
  RakuInputExecute["from-numeric-word-form(" <> ToRakuCode[Values[aNumericWordForms]] <>")"] 
 ]

(*{0.226441, "(1131 7504 9970 1016128 5341656 6865271)"}*)
```

------

## Example: Stoichiometry

In this section we make a brief comparison of Mathematica and Raku over chemical elements data retrieval, 
molecular mass calculations, and chemical equation balancing.

In 2007, while working on [WolframAlpha](https://www.wolframalpha.com) (W|A) , I wrote the first versions of W|A’s 
chemical molecules parser and functions for molecular mass calculations and chemical equation balancing. 
(See the raw chapters in [AAr3].) In the beginning of 2021 I wrote similar functions for Raku, see the package 
[”Chemistry::Stoichiometry”](https://modules.raku.org/dist/Chemistry::Stoichiometry:cpan:ANTONOV), 
[AAp13].

Mathematica Version 6.0 (released in 2007) introduced the function 
[`ElementData`](https://reference.wolfram.com/language/ref/ElementData.html). 
Mathematica Version 13.0 (released December 2021) has the function 
[`ReactionBalance`](https://reference.wolfram.com/language/ref/ReactionBalance.html) 
that balances chemical equations.

Here we load the Raku package [AAp13]:

```perl6
use Chemistry::Stoichiometry;
```

### Chemical element data

Here we get element data for Chlorine:

```perl6
chemical-element-data(‘Cl’);

# {Abbreviation => Cl, AtomicNumber => 17, AtomicWeight => 35.45, Block => p, Group => 17, Name => chlorine, 
#  Period => 3, Series => Halogen, StandardName => Chlorine}
```

Mathematica has a much larger list element properties:

```mathematica
ElementData["Cl", "Properties"] // Length

(*86*)
```

But let us get the properties that Raku has:

```mathematica
lsProps = {"Abbreviation", "AtomicNumber", "AtomicWeight", "Block", "Group", "Name", "Period", "Series", "StandardName"};
Map[# -> ElementData["Cl", #] &, lsProps]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0j1ehb4atls73.png)

Both Mathematica and Raku know the full names of the chemical elements, but Raku has multi-language support.
Here we use Raku to retrieve the names of Chlorine in different languages using the abbreviation "Cl":

```perl6
<Bulgarian English German Japanese Persian Russian Spanish>.map({ $_ => chemical-element(‘Cl’, $_ ) })

# (Bulgarian => Хлор English => Chlorine German => Chlor Japanese => 塩素 Russian => Хлор Spanish => Cloro)"*)
```

Here we get different types of data using Japanese, English, and Russian element names:

```perl6
[atomic-weight(‘ガリウム’), chemical-element-data(‘oxygen’):weight, chemical-element-data(‘кислород’):abbr]

# [69.723 15.999 O]
```

The Japanese name “ガリウム” above is for Gallium:

```perl6
chemical-element-data(‘ガリウム’)

# {Abbreviation => Ga, AtomicNumber => 31, AtomicWeight => 69.723, Block => p, Group => 13, Name => gallium, 
#  Period => 4, Series => PoorMetal, StandardName => Gallium}
```

### Molecular mass

Here we assign a molecule formula (of “diphenyliodonium bromide”) to a variable in Raku:

```perl6
my $molecule=‘(C6H5)2IBr’

# (C6H5)2IBr
```

Here using Mathematica we find the molecular mass:

```mathematica
ChemicalFormula[StringTrim[%]]["MolecularMass"]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/1afakvynpdbqu.png)

Here is the molecular mass computed with Raku:

```perl6
molecular-mass($molecule)

# 361.02047000000005
```

In both cases the molecule formula is parsed into pairs of element names and multipliers and for each pair the mass is computed using the corresponding element atomic mass; then the masses corresponding to all pairs are totaled.

### Chemical equation balancing

Chemical equation balancing can be done representing the molecules in the equation as points of a vector space, and then solving a corresponding system of linear equations.

Here we assign to a Raku variable a chemical equation string:

```perl6
my $chemEq=‘C2H5OH + O2 = H2O + CO2’;

# "C2H5OH + O2 = H2O + CO2"
```

Here we balance the equation with Mathematica:

```mathematica
ReactionBalance[%]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0gilts1bdhhl4.png)

Here we balance the equation with Raku:

```perl6
balance-chemical-equation($chemEq)

# [1*C2H5OH + 3*O2 -> 2*CO2 + 3*H2O]
```

(We can see that results are the same.)

------

## Making of the DSLMode cells

Initially I borrowed ideas from the 
[Wolfram Function Repository (WFR)](https://resources.wolframcloud.com/FunctionRepository) function 
["DarkMode"](https://resources.wolframcloud.com/FunctionRepository/resources/DarkMode/). 
After that I was pretty much directed by [
Kuba Podkalicki](https://community.wolfram.com/web/kubapod) 
how to design and implement the functionalities behind the DSL cells.

The package 
["DSLMode.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/DSLMode.m) 
is small, just “a front” for the pretty big package 
["ExternalParsersHookup.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/ExternalParsersHookup.m), 
which has all DSLs from the [RakuForPrediction project](https://conf.raku.org/talk/157) being represented in it.

DSL cell has a hard copy of the Raku cell style data from 
["RakuMode.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuMode.m) 
in order to have only one package dependency (that of 
[“ExternalParsersHookup.m”](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/ExternalParsersHookup.m)
.)

------

## Making of the RakuMode cell

I tried to make the Raku cells to resemble external evaluation cells to a large degree, but still look decisively different. 

- As background cell color I used background color in the input/output cells in the [Raku site](https://raku.org) documentation. 

    - E.g. here: https://docs.raku.org/language/objects .

- Finding and making the "prefix" cell icon took a fairly long time. 

    - I used the logo images from this GitHub repository: https://github.com/MadcapJake/metamorphosis .

    - After some experimentation the “hex camelia” logos seem to fit the best.

- The package “RakuMode.m” had several revisions.

    - The latest is to refactor the hex-camelia graphics in a different package, ["HexCameliaIcons.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/HexCameliaIcons.m).

- Except the Raku cell itself, probably the most interesting or peculiar part of the package “RakuMode.m” is that it has the ZeroMQ Raku code it uses as a WL string template.

### Cell icon

Here are some [Camelia logos](https://github.com/MadcapJake/metamorphosis):

```mathematica
ResourceFunction["GridTableForm"][{{
    Import["https://raw.githubusercontent.com/MadcapJake/metamorphosis/master/images/Orig-Camelia.png"], 
    Import["https://raw.githubusercontent.com/uzluisf/metamorphosis/master/hex_camelia/perl6-color-logo1.png"]}}, 
  TableHeadings -> {"Standard", "Hex"}, Background -> None]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0oj2r4aoag3et.png)

Here is the graphics object used to make the image of the Camelia icon used for Raku cells:

```mathematica
Import["https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/Packages/WL/HexCameliaIcons.m"];
GetHexCameliaGraphics[]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0r4qg16bo0ugj.png)

**Remark:** A selected GitHub repository image was croped and vectorized with [ImageGraphics](https://reference.wolfram.com/language/ref/ImageGraphics.html). For more details see the descriptions in the package ["HexCameliaIcons.m"](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/HexCameliaIcons.m), [AAp3].

### ZeroMQ template code

Here is the Raku code string template for the ZeroMQ connection:

```mathematica
Magnify[RakuMode`Private`zmqScript, 0.8]
```

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Articles/Diagrams/Connecting-Mathematica-and-Raku/0fq8e2ahieogt.png)

### Comparison with other external evaluation cells

Here is how the Raku cell looks compared to other external evaluator cells:

```perl6
1+1_000
```

```julia
4^3
```

```python
1+1_000
```

```r
{seq(1,10,2)}
```

(The order of the cells above is Raku, Julia, Python, R.)

------

## Future plans

Here are future plans that (mostly) directly related to the Mathematica-and-Raku connectivity functionalities discussed above:

- More robust WL to Raku encoder.

- More robust ZeroMQ connection maintenance and support.

- Raku slang for Wolfram Language.

- Raku slang for DSL workflows.

- Making similar connections from Raku to other languages (Python, Julia).

    - Already done for R.

------

## References

### Functions

[WRI1][ Wolfram Research, (2014), [RunProcess](https://reference.wolfram.com/language/ref/RunProcess.html.en), Wolfram Language function, https://reference.wolfram.com/language/ref/RunProcess.html.

[WRI2] Wolfram Research, (2014), [StartProcess](https://reference.wolfram.com/language/ref/StartProcess.html), Wolfram Language function, https://reference.wolfram.com/language/ref/StartProcess.html.

[WRI3] Wolfram Research, (2016), [URLRead](https://reference.wolfram.com/language/ref/URLRead.html.en), Wolfram Language function, https://reference.wolfram.com/language/ref/URLRead.html.

### Mathematica packages

[AAp1] Anton Antonov, 
[DSLMode Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/DSLMode.m),
(2020-2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp2] Anton Antonov, 
[ExternalParsersHookup Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/ExternalParsersHookup.m),
(2020-2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp3] Anton Antonov, 
[HexCameliaIcons Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/HexCameliaIcons.m),
(2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp4] Anton Antonov, 
[RakuCommand Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuCommand.m),
(2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp5] Anton Antonov, 
[RakuDecoder Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuDecoder.m),
(2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp6] Anton Antonov, 
[RakuEncoder Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuEncoder.m),
(2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

[AAp7] Anton Antonov, 
[RakuMode Mathematica package](https://github.com/antononcube/ConversationalAgents/blob/master/Packages/WL/RakuMode.m),
(2020-2021),
[ConversationalAgents at GitHub/antononcube](https://github.com/antononcube/ConversationalAgents).

### Raku packages

[AAp9] Anton Antonov, [Text::CodeProcessing](https://modules.raku.org/dist/Text::CodeProcessing:cpan:ANTONOV), (2021), [Raku Modules](https://modules.raku.org).

[AAp10] Anton Antonov, [Lingua::NumericWordForms](https://modules.raku.org/dist/Lingua::NumericWordForms:cpan:ANTONOV), (2021), [Raku Modules](https://modules.raku.org).

[AAp11] Anton Antonov, [Data::Reshapers](https://modules.raku.org/dist/Data::Reshapers:cpan:ANTONOV), (2021), [Raku Modules](https://modules.raku.org).

[AAp12] Anton Antonov, [DSL::English::DataQueryWorkflows Raku package](https://github.com/antononcube/Raku-DSL-English-DataQueryWorkflows), (2020), [GitHub/antononcube](https://github.com/antononcube).

[AAp13] Anton Antonov, [Chemistry::Stoichiometry](https://modules.raku.org/dist/Chemistry::Stoichiometry:cpan:ANTONOV), (2021), [Raku Modules](https://modules.raku.org).

### Repositories

[AAr1] Anton Antonov, 
["Raku for Prediction" book](https://github.com/antononcube/RakuForPrediction-book), 
(2021), 
[GitHub/antononcube](https://github.com/antononcube).

[AAr2] Anton Antonov, 
[NLP Template Engine](https://github.com/antononcube/NLP-Template-Engine), 
(2021), 
[GitHub/antononcube](https://github.com/antononcube).

[AAr3] Anton Antonov, 
["Mathematica for Chemists and Chemical Engineers" book project](https://github.com/antononcube/MathematicaForChemistsAndChemicalEngineers-book), 
(2020), 
[GitHub/antononcube](https://github.com/antononcube).

[BD1] Brian Duggan et al., 
[p6-jupyter-kernel](https://github.com/bduggan/p6-jupyter-kernel), 
(2017-2020), 
[GitHub/bduggan](https://github.com/bduggan).

### Videos

[AAv1] Anton Antonov, [“Multi-language Data-Wrangling Conversational Agent”](https://www.youtube.com/watch?v=pQk5jwoMSxs), WTC-2020 presentation, (2020),[ Wolfram at YouTube](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv2] Anton Antonov, ["Raku for Prediction"](https://conf.raku.org/talk/157), (2021), The Raku Conference.

[AAv3] Anton Antonov, ["NLP Template Engine, Part 1"](https://youtu.be/a6PvmZnvF9I), (2021), [Simplified Machine Learning Workflows at YouTube.](https://www.youtube.com/playlist?list=PLke9UbqjOSOi1fc0NkJTdK767cL9XHJF0)

[AAv4] Anton Antonov, ["Doing it like a Cro (Raku data wrangling Shortcuts demo)"](https://www.youtube.com/watch?v=wS1lqMDdeIY), (2021), [Simplified Machine Learning Workflows at YouTube.](https://www.youtube.com/playlist?list=PLke9UbqjOSOi1fc0NkJTdK767cL9XHJF0)
