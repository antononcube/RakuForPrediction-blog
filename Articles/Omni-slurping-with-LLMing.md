# Omni-slurping with LLMing

Anton Antonov   
RakuForPrediction at WordPress   
March 2024

------

## Introduction

In this notebook we demonstrate the use of the Raku package ["Data::Importers"](https://raku.land/zef:antononcube/Data::Importers), that offers a convenient solution for importing data from URLs and files. This package supports a variety of data types such as CSV, HTML, PDF, text, and images, making it a versatile tool for data manipulation.

One particularly interesting application of "Data::Importers" is its inclusion into workflows based on Large Language Models (LLMs). Generally speaking, having an easy way to ingest diverse range of data formats -- like what "Data::Importers" aims to do -- makes a wide range of workflows for data processing and analysis easier to create.

In this blog post, we will demonstrate how "Data::Importers" can work together with LLMs, providing real-world examples of their combined usage in various situations. Essentially, we will illustrate the power of merging omni-slurping with LLM-ing to improve data-related activities.


The main function of "Data::Importers" is `data-import`. Its functionalities are incorporated into suitable overloads of the built-in `slurp` subroutine.

Notebook structure:

1. Setup
2. HTML summary and cross tabulation
3. PDF summary
4. CSV filtering
5. Image vision
6. Image vision with re-imaging

-----

## Setup

Here a lot of packages used below:


```raku
use Data::Importers;
use Data::Reshapers;
use Data::Summarizers;
use JSON::Fast;
use JavaScript::D3;
```

Here we configure the Jupyter notebook to display JavaScript graphics, [AAp7, AAv1]:


```raku
#% javascript

require.config({
     paths: {
     d3: 'https://d3js.org/d3.v7.min'
}});

require(['d3'], function(d3) {
     console.log(d3);
});
```






------

## HTML summary and cross tabulation

A key motivation behind creating the "Data::Importers" package was to efficiently retrieve HTML pages, extract plain text, and import it into a Jupyter notebook for subsequent LLM transformations and content processing.

Here is a pipeline that gets an LLM summary of a certain recent [Raku blog post](https://rakudoweekly.blog/2024/03/25/2024-13-veyoring-again):


```raku
my $htmlURL = 'https://rakudoweekly.blog/2024/03/25/2024-13-veyoring-again/';

$htmlURL
==> slurp(format => 'plaintext')
==> { say text-stats($_); $_ }()
==> llm-prompt('Summarize')()
==> llm-synthesize()
```

    (chars => 2814 words => 399 lines => 125)





    Paul Cochrane returns to the Raku community with a guide on enabling Continuous Integration on Raku projects using AppVeyor. Core developments include improvements by Elizabeth Mattijsen on Metamodel classes for faster compilation and performance. New and updated Raku modules are also featured in this week's news.



Here is another LLM pipeline that ingests the HTML page and produces an HTML table derived from the page's content:


```raku
#% html

$htmlURL
==> slurp(format => 'plaintext')
==> { say "Contributors table:"; $_ }() 
==> {["Cross tabulate into a HTML table the contributors", 
        "and type of content with the content itself", 
        "for the following text:\n\n", 
        $_, 
        llm-prompt('NothingElse')('HTML')]}()
==> llm-synthesize(e => llm-configuration('Gemini', max-tokens => 4096, temperature => 0.65))
```

    Contributors table:





<table>
  <thead>
    <tr>
      <th>Contributor</th>
      <th>Content Type</th>
      <th>Content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Paul Cochrane</td>
      <td>Tutorial</td>
      <td>Building and testing Raku in AppVeyor</td>
    </tr>
    <tr>
      <td>Dr. Raku</td>
      <td>Tutorial</td>
      <td>How To Delete Directories</td>
    </tr>
    <tr>
      <td>Dr. Raku</td>
      <td>Tutorial</td>
      <td>Fun File Beginners Project</td>
    </tr>
    <tr>
      <td>Dr. Raku</td>
      <td>Tutorial</td>
      <td>Hash Examples</td>
    </tr>
    <tr>
      <td>Elizabeth Mattijsen</td>
      <td>Development</td>
      <td>Metamodel classes for faster compilation and performance and better stability</td>
    </tr>
    <tr>
      <td>Stefan Seifert</td>
      <td>Development</td>
      <td>Fixed several BEGIN time lookup issues</td>
    </tr>
    <tr>
      <td>Elizabeth Mattijsen</td>
      <td>Development</td>
      <td>Fixed an issue with =finish if there was no code</td>
    </tr>
    <tr>
      <td>Samuel Chase</td>
      <td>Shoutout</td>
      <td>Nice shoutout!</td>
    </tr>
    <tr>
      <td>Fernando Santagata</td>
      <td>Self-awareness test</td>
      <td>Self-awareness test</td>
    </tr>
    <tr>
      <td>Paul Cochrane</td>
      <td>Deep rabbit hole</td>
      <td>A deep rabbit hole</td>
    </tr>
    <tr>
      <td>anavarro</td>
      <td>Question</td>
      <td>How to obtain the Raku language documentation ( Reference) offline</td>
    </tr>
    <tr>
      <td>Moritz Lenz</td>
      <td>Comment</td>
      <td>On ^ and $</td>
    </tr>
    <tr>
      <td>LanX</td>
      <td>Comment</td>
      <td>The latest name</td>
    </tr>
    <tr>
      <td>ilyash</td>
      <td>Comment</td>
      <td>Automatic parsing of args</td>
    </tr>
    <tr>
      <td>emporas</td>
      <td>Comment</td>
      <td>Certainly looks nice</td>
    </tr>
    <tr>
      <td>faiface</td>
      <td>Comment</td>
      <td>Went quite bad</td>
    </tr>
    <tr>
      <td>Ralph Mellor</td>
      <td>Comment</td>
      <td>On Raku's design decisions regarding operators</td>
    </tr>
    <tr>
      <td>option</td>
      <td>Example</td>
      <td>An example Packer file</td>
    </tr>
    <tr>
      <td>Anton Antonov</td>
      <td>Module</td>
      <td>Data::Importers</td>
    </tr>
    <tr>
      <td>Ingy døt Net</td>
      <td>Module</td>
      <td>YAMLScript</td>
    </tr>
    <tr>
      <td>Alexey Melezhik</td>
      <td>Module</td>
      <td>Sparrow6, Sparky</td>
    </tr>
    <tr>
      <td>Patrick Böker</td>
      <td>Module</td>
      <td>Devel::ExecRunnerGenerator</td>
    </tr>
    <tr>
      <td>Steve Roe</td>
      <td>Module</td>
      <td>PDF::Extract</td>
    </tr>
  </tbody>
</table>



------

## PDF summary

Another frequent utilization of LLMs is the processing of PDF files found (intentionally or not) while browsing the Web. (Like, arXiv.org articles, UN resolutions, or court slip opinions.)

Here is a pipeline that gets an LLM summary of an oral argument brought up recently (2024-03-18) to The US Supreme Court, ([22-842 "NRA v. Vullo"](https://www.supremecourt.gov/oral_arguments/argument_transcripts/2023/22-842_c1o2.pdf)):


```raku
'https://www.supremecourt.gov/oral_arguments/argument_transcripts/2023/22-842_c1o2.pdf'
==> slurp(format=>'text')
==> llm-prompt('Summarize')()
==> llm-synthesize(e=>llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview'))
```


    The Supreme Court of the United States dealt with a case involving the National Rifle Association (NRA) and Maria T. Vullo, challenging actions taken by New York officials against the NRA's insurance programs. The NRA argued that their First Amendment rights were violated when New York officials, under the guidance of Maria Vullo and Governor Andrew Cuomo, used coercive tactics to persuade insurance companies and banks to sever ties with the NRA, citing the promotion of guns as the reason. These actions included a direct threat of regulatory repercussions to insurance underwriter Lloyd's and the issuance of guidance letters to financial institutions, suggesting reputational risks associated with doing business with the NRA. The court discussed the plausibility of coercion and the First Amendment claim, reflecting on precedents like Bantam Books, and the extent to which government officials can use their regulatory power to influence the actions of third parties against an organization due to its advocacy work.


---- 

## CSV filtering

Here we ingest from GitHub a CSV file that has datasets metadata: 


```raku
my $csvURL = 'https://raw.githubusercontent.com/antononcube/Raku-Data-ExampleDatasets/main/resources/dfRdatasets.csv';
my $dsDatasets = data-import($csvURL, headers => 'auto');

say "Dimensions   : {$dsDatasets.&dimensions}";
say "Column names : {$dsDatasets.head.keys}";
say "Type         : {deduce-type($dsDatasets)}";
```

    Dimensions   : 1745 12
    Column names : n_logical n_character n_numeric Doc Rows Cols Package Title Item CSV n_binary n_factor
    Type         : Vector(Assoc(Atom((Str)), Atom((Str)), 12), 1745)


Here is a table with a row sample:


```raku
#% html
my $field-names = <Package Item Title Rows Cols>;
my $dsDatasets2 = $dsDatasets>>.Hash.Array;
$dsDatasets2 = select-columns($dsDatasets2, $field-names);
$dsDatasets2.pick(12) ==> data-translation(:$field-names)
```


<table border="1"><thead><tr><th>Package</th><th>Item</th><th>Title</th><th>Rows</th><th>Cols</th></tr></thead><tbody><tr><td>robustbase</td><td>wagnerGrowth</td><td>Wagner&#39;s Hannover Employment Growth Data</td><td>63</td><td>7</td></tr><tr><td>openintro</td><td>age_at_mar</td><td>Age at first marriage of 5,534 US women.</td><td>5534</td><td>1</td></tr><tr><td>AER</td><td>MurderRates</td><td>Determinants of Murder Rates in the United States</td><td>44</td><td>8</td></tr><tr><td>Stat2Data</td><td>RadioactiveTwins</td><td>Comparing Twins Ability to Clear Radioactive Particles</td><td>30</td><td>3</td></tr><tr><td>rpart</td><td>kyphosis</td><td>Data on Children who have had Corrective Spinal Surgery</td><td>81</td><td>4</td></tr><tr><td>boot</td><td>gravity</td><td>Acceleration Due to Gravity</td><td>81</td><td>2</td></tr><tr><td>survival</td><td>diabetic</td><td>Ddiabetic retinopathy</td><td>394</td><td>8</td></tr><tr><td>gap</td><td>mfblong</td><td>Internal functions for gap</td><td>3000</td><td>10</td></tr><tr><td>Ecdat</td><td>Mofa</td><td>International Expansion of U.S. MOFAs (majority-owned Foreign Affiliates in Fire (finance, Insurance and Real Estate)</td><td>50</td><td>5</td></tr><tr><td>drc</td><td>chickweed</td><td>Germination of common chickweed (_Stellaria media_)</td><td>35</td><td>3</td></tr><tr><td>MASS</td><td>Pima.tr</td><td>Diabetes in Pima Indian Women</td><td>200</td><td>8</td></tr><tr><td>MASS</td><td>shrimp</td><td>Percentage of Shrimp in Shrimp Cocktail</td><td>18</td><td>1</td></tr></tbody></table>


Here we use an LLM to pick rows that related to certain subject:


```raku
my $res = llm-synthesize([
    'From the following JSON table pick the rows that are related to air pollution.', 
    to-json($dsDatasets2), 
    llm-prompt('NothingElse')('JSON')
], 
e => llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.65),
form => sub-parser('JSON'):drop)
```


    [{Cols => 6, Item => airquality, Package => datasets, Rows => 153, Title => Air Quality Data} {Cols => 5, Item => iris, Package => datasets, Rows => 150, Title => Edgar Anderson's Iris Data} {Cols => 11, Item => mtcars, Package => datasets, Rows => 32, Title => Motor Trend Car Road Tests} {Cols => 5, Item => USPersonalExpenditure, Package => datasets, Rows => 5, Title => US Personal Expenditure Data (1940-1950)} {Cols => 4, Item => USArrests, Package => datasets, Rows => 50, Title => US Arrests for Assault (1960)}]



```raku
use WWW::Gemini;
gemini-count-tokens(to-json($dsDatasets2))
```


    {totalTokens => 99530}


Here is the tabulated result:


```raku
#% html
$res ==> data-translation(:$field-names)
```


<table border="1"><thead><tr><th>Package</th><th>Item</th><th>Title</th><th>Rows</th><th>Cols</th></tr></thead><tbody><tr><td>AER</td><td>CigarettesB</td><td>Cigarette Consumption Data</td><td>46</td><td>3</td></tr><tr><td>AER</td><td>CigarettesSW</td><td>Cigarette Consumption Panel Data</td><td>96</td><td>9</td></tr><tr><td>plm</td><td>Cigar</td><td>Cigarette Consumption</td><td>1380</td><td>9</td></tr></tbody></table>


----

## Image vision

One of the cooler recent LLM-services enhancements is the access to AI-vision models. For example, AI-vision models are currently available through interfaces of OpenAI, Gemini, or LLaMA.

Here we use `data-import` instead of (the overloaded) `slurp`:


```raku
#% markdown
my $imgURL2 = 'https://www.wolframcloud.com/files/04e7c6f6-d230-454d-ac18-898ee9ea603d/htmlcaches/images/2f8c8b9ee8fa646349e00c23a61f99b8748559ed04da61716e0c4cacf6e80979';
my $img2 = data-import($imgURL2, format => 'md-image');
```

Here is AI-vision invocation:


```raku
llm-vision-synthesize('Describe the image', $img2, e => 'Gemini')
```




     The image shows a blue-white sphere with bright spots on its surface. The sphere is the Sun, and the bright spots are solar flares. Solar flares are bursts of energy that are released from the Sun's surface. They are caused by the sudden release of magnetic energy that has built up in the Sun's atmosphere. Solar flares can range in size from small, localized events to large, global eruptions. The largest solar flares can release as much energy as a billion hydrogen bombs. Solar flares can have a number of effects on Earth, including disrupting radio communications, causing power outages, and triggering geomagnetic storms.



**Remark:** The image is taken from the Wolfram Community post ["Sympathetic solar flare and geoeffective coronal mass ejection"](https://community.wolfram.com/groups/-/m/t/3146725), [JB1].

**Remark:** The AI vision above is done Google's "gemini-pro-vision'. Alternatively, OpenAI's "gpt-4-vision-preview" can be used.

------

## Image vision with re-imaging

In this section we show how to import a certain statistical image, get data from the image, and make another similar statistical graph. Similar workflows are discussed ["Heatmap plots over LLM scraped data"](https://rakuforprediction.wordpress.com/2024/01/27/heatmap-plots-over-llm-scraped-data/), [AA1]. The plots are made with "JavaScript::D3", [AAp3].

Here we ingest an image with statistics of fuel exports:


```raku
#% markdown
my $imgURL = 'https://pbs.twimg.com/media/GG44adyX0AAPqVa?format=png&name=medium';
my $img = data-import($imgURL, format => 'md-image')
```

Here is a fairly non-trivial request for data extraction from the image:


```raku
my $resFuel = llm-vision-synthesize([
    'Give JSON dictionary of the Date-Country-Values data in the image', 
    llm-prompt('NothingElse')('JSON')
    ], 
    $img, form => sub-parser('JSON'):drop)
```




    [Date-Country-Values => {Jan-22 => {Bulgaria => 56, China => 704, Croatia => 22, Denmark => 118, Finland => 94, France => 140, Germany => 94, Greece => 47, India => 24, Italy => 186, Lithuania => 142, Netherlands => 525, Poland => 165, Romania => 122, South Korea => 327, Spain => 47, Sweden => 47, Total => 3192, Turkey => 170, UK => 68, USA => 24}, Jan-24 => {Brunei => 31, China => 1151, Egypt => 70, Ghana => 35, Greece => 103, India => 1419, Korea => 116, Myanmar => 65, Netherlands => 33, Oman => 23, Total => 3381, Turkey => 305, Unknown => 30}}]



Here is we modify the prompt above in order to get a dataset (an array of hashes):


```raku
my $resFuel2 = llm-vision-synthesize([
    'For data in the image give the corresponding JSON table that is an array of dictionaries each with the keys "Date", "Country", "Value".', 
    llm-prompt('NothingElse')('JSON')
    ],
    $img, 
    max-tokens => 4096,
    form => sub-parser('JSON'):drop)
```




    [{Country => USA, Date => Jan-22, Value => 24} {Country => Turkey, Date => Jan-22, Value => 170} {Country => Croatia, Date => Jan-22, Value => 22} {Country => Sweden, Date => Jan-22, Value => 47} {Country => Spain, Date => Jan-22, Value => 47} {Country => Greece, Date => Jan-22, Value => 47} {Country => Bulgaria, Date => Jan-22, Value => 56} {Country => UK, Date => Jan-22, Value => 68} {Country => Germany, Date => Jan-22, Value => 94} {Country => Finland, Date => Jan-22, Value => 94} {Country => Denmark, Date => Jan-22, Value => 118} {Country => Romania, Date => Jan-22, Value => 122} {Country => France, Date => Jan-22, Value => 140} {Country => Lithuania, Date => Jan-22, Value => 142} {Country => Poland, Date => Jan-22, Value => 165} {Country => Italy, Date => Jan-22, Value => 186} {Country => Netherlands, Date => Jan-22, Value => 525} {Country => India, Date => Jan-22, Value => 24} {Country => Japan, Date => Jan-22, Value => 70} {Country => South Korea, Date => Jan-22, Value => 327} {Country => China, Date => Jan-22, Value => 704} {Country => Unknown, Date => Jan-24, Value => 30} {Country => Ghana, Date => Jan-24, Value => 35} {Country => Egypt, Date => Jan-24, Value => 70} {Country => Oman, Date => Jan-24, Value => 23} {Country => Turkey, Date => Jan-24, Value => 305} {Country => Netherlands, Date => Jan-24, Value => 33} {Country => Greece, Date => Jan-24, Value => 103} {Country => Brunei, Date => Jan-24, Value => 31} {Country => Myanmar, Date => Jan-24, Value => 65} {Country => Korea, Date => Jan-24, Value => 116} {Country => China, Date => Jan-24, Value => 1151} {Country => India, Date => Jan-24, Value => 1419}]



Here is how the obtained dataset looks like:


```raku
#% html
$resFuel2>>.Hash ==> data-translation()
```




<table border="1"><thead><tr><th>Value</th><th>Date</th><th>Country</th></tr></thead><tbody><tr><td>24</td><td>Jan-22</td><td>USA</td></tr><tr><td>170</td><td>Jan-22</td><td>Turkey</td></tr><tr><td>22</td><td>Jan-22</td><td>Croatia</td></tr><tr><td>47</td><td>Jan-22</td><td>Sweden</td></tr><tr><td>47</td><td>Jan-22</td><td>Spain</td></tr><tr><td>47</td><td>Jan-22</td><td>Greece</td></tr><tr><td>56</td><td>Jan-22</td><td>Bulgaria</td></tr><tr><td>68</td><td>Jan-22</td><td>UK</td></tr><tr><td>94</td><td>Jan-22</td><td>Germany</td></tr><tr><td>94</td><td>Jan-22</td><td>Finland</td></tr><tr><td>118</td><td>Jan-22</td><td>Denmark</td></tr><tr><td>122</td><td>Jan-22</td><td>Romania</td></tr><tr><td>140</td><td>Jan-22</td><td>France</td></tr><tr><td>142</td><td>Jan-22</td><td>Lithuania</td></tr><tr><td>165</td><td>Jan-22</td><td>Poland</td></tr><tr><td>186</td><td>Jan-22</td><td>Italy</td></tr><tr><td>525</td><td>Jan-22</td><td>Netherlands</td></tr><tr><td>24</td><td>Jan-22</td><td>India</td></tr><tr><td>70</td><td>Jan-22</td><td>Japan</td></tr><tr><td>327</td><td>Jan-22</td><td>South Korea</td></tr><tr><td>704</td><td>Jan-22</td><td>China</td></tr><tr><td>30</td><td>Jan-24</td><td>Unknown</td></tr><tr><td>35</td><td>Jan-24</td><td>Ghana</td></tr><tr><td>70</td><td>Jan-24</td><td>Egypt</td></tr><tr><td>23</td><td>Jan-24</td><td>Oman</td></tr><tr><td>305</td><td>Jan-24</td><td>Turkey</td></tr><tr><td>33</td><td>Jan-24</td><td>Netherlands</td></tr><tr><td>103</td><td>Jan-24</td><td>Greece</td></tr><tr><td>31</td><td>Jan-24</td><td>Brunei</td></tr><tr><td>65</td><td>Jan-24</td><td>Myanmar</td></tr><tr><td>116</td><td>Jan-24</td><td>Korea</td></tr><tr><td>1151</td><td>Jan-24</td><td>China</td></tr><tr><td>1419</td><td>Jan-24</td><td>India</td></tr></tbody></table>



Here we rename or otherwise transform the columns of the dataset above in order to prepare it for creating a heatmap plot (we also show the deduced type):


```raku
my $k = 1;
my @fuelDataset = $resFuel2.map({ 
    my %h = $_.clone; 
    %h<z> = log10(%h<Value>); 
    %h<y> = %h<Country>; 
    %h<x> = %h<Date>; 
    %h<label> = %h<Value>;
    %h.grep({ $_.key ∈ <x y z label> }).Hash }).Array;

deduce-type(@fuelDataset);
```




    Vector(Struct([label, x, y, z], [Int, Str, Str, Num]), 33)



Here is the heatmap plot:


```raku
#%js
js-d3-heatmap-plot(@fuelDataset,
                width => 700,
                height => 500,
                color-palette => 'Reds',
                plot-labels-color => 'White',
                plot-labels-font-size => 18,
                tick-labels-color => 'steelblue',
                tick-labels-font-size => 12,
                low-value => 0,
                high-value => 3.5,
                margins => {left => 100, right => 0},
                mesh => 0.01,
                title => 'Russia redirecting seaborne crude amid sanctions, 1000 b/d')
```






Here are the corresponding totals:


```raku
group-by($resFuel2, 'Date').map({ $_.key => $_.value.map(*<Value>).sum })
```




    (Jan-24 => 3381 Jan-22 => 3192)



------

## References

### Articles

[AA1] Anton Antonov,
["Heatmap plots over LLM scraped data"](https://rakuforprediction.wordpress.com/2024/01/27/heatmap-plots-over-llm-scraped-data/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).


[JB1] Jeffrey Bryant​,
["Sympathetic solar flare and geoeffective coronal mass ejection​"](​https://community.wolfram.com/groups/-/m/t/3146725),
(2024),
[Wolfram Community](​https://community.wolfram.com).

### Packages

[AAp1] Anton Antonov,
[Data::Importers Raku package](https://github.com/antononcube/Raku-Data-Importers),
(2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp2] Anton Antonov,
[LLM::Functions Raku package](https://github.com/antononcube/Raku-LLM-Functions),
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp3] Anton Antonov,
[JavaScript::D3 Raku package](https://github.com/antononcube/Raku-JavaScript-D3),
(2022-2024),
[GitHub/antononcube](https://github.com/antononcube).

### Videos

[AAv1] Anton Antonov,
["Random mandalas generation (with D3.js via Raku)"](https://youtu.be/THNnofZEAn4),
(2022),
[Anton Antonov's YouTube channel](https://www.youtube.com/@AAA4prediction).
