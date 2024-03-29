# LLM aids for processing Putin's State-Of-The-Nation speech 

### ***... given on February, 29, 2024***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog)   
March 2024

-------

## Introduction

![](http://static.kremlin.ru/media/events/photos/big2x/yOcwEC7CgATJomZ91n7fh3rCnMJhcLWU.jpg)

In this document
([notebook](https://github.com/antononcube/RakuForPrediction-blog/blob/main/Articles/LLM-aids-for-processing-of-Putin-State-Of-The-Nation-speech-on-2024-02-29.md)) 
we provide aids and computational workflows for the analysis of Vladimir Putin's 
[State Of The Nation speech](http://www.kremlin.ru/events/president/news/73585) given on February 29th, 2024.
We use Large Language Models (LLMs). We walk through various steps involved in examining and understanding the speech in a systematic and reproducible manner.

The speech transcript (in Russian) is taken from [kremlin.ru](http://www.kremlin.ru/events/president/news/73585).

The computations are done with a [Raku chatbook](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp6, AAv1÷AAv3]. The LLM functions used in the workflows are explained and demonstrated in [AA1, AAv3].
The workflows are done with OpenAI's models [AAp1]. Currently, the models of Google's (PaLM), [AAp2], and MistralAI, [AAp3], cannot be used with the workflows below because their input token limits are too low.

**Remark:** An important feature of the LLM workflows (and underlying models) is that although the speech transcript is in Russian, the LLM results are in English.

A similar set of workflows is described in ["LLM aids for processing of the first Carlson-Putin interview"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/), [AA2], and it has been reused to a large degree below.

The following table -- derived from Putin's speech -- should be of great interest to people living in Western countries (with governments that want to fight Russia):

<table border="1"><thead><tr><th>name</th><th>russian_name</th><th>type</th><th>status</th><th>description</th><th>damage</th></tr></thead><tbody><tr><td><span><a href="https://en.wikipedia.org/wiki/Kh-47M2_Kinzhal">Kinzhal</a></span></td><td>Кинжал</td><td>Hypersonic Airborne Missile System</td><td>Operational</td><td>A hypersonic missile capable of striking targets with high precision over long distances at speeds exceeding Mach 5.</td><td>High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/3M22_Zircon">Zircon</a></span></td><td>Циркон</td><td>Hypersonic Cruise Missile</td><td>Operational</td><td>A sea-based hypersonic cruise missile designed to attack naval and ground targets.</td><td>High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Avangard_(hypersonic_glide_vehicle)">Avangard</a></span></td><td>Авангард</td><td>Hypersonic Glide Vehicle</td><td>Operational</td><td>Mounted on an intercontinental ballistic missile, it can carry a nuclear payload and maneuver at high speeds to evade missile defense systems.</td><td>Very High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Peresvet_(laser_weapon)">Peresvet</a></span></td><td>Пересвет</td><td>Laser Weapon System</td><td>Operational</td><td>A laser system purportedly designed to counter aerial threats and possibly to disable satellites and other space assets.</td><td>Variable</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/9M730_Burevestnik">Burevestnik</a></span></td><td>Буревестник</td><td>Nuclear-Powered Cruise Missile</td><td>In Testing</td><td>Claimed to have virtually unlimited range thanks to its nuclear power source, designed for strategic bombing missions.</td><td>Potentially Very High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Status-6_Oceanic_Multipurpose_System">Poseidon</a></span></td><td>Посейдон</td><td>Nuclear-Powered Underwater Drone</td><td>In Development</td><td>An autonomous underwater vehicle intended to carry nuclear warheads to create radioactive tsunamis near enemy coastlines.</td><td>Catastrophic</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/RS-28_Sarmat">Sarmat</a></span></td><td>Сармат</td><td>Intercontinental Ballistic Missile</td><td>Operational</td><td>A heavy missile intended to replace the aging Soviet-era Voevoda, capable of carrying multiple nuclear warheads.</td><td>Catastrophic</td></tr></tbody></table>

### Structure

The structure of the notebook is as follows:

1. **Getting the speech text and setup**   
    Standard ingestion and setup.
2. **Summary**   
    The speech in brief.
3. **Themes**      
    TL;DR via a table of themes.
4. **Important parts**   
    What are the most important parts or most provocative statements?
5. **Talking to the West**    
    LLM pretends to be Putin and addresses the West.
6. **Weapons tabulation**   
    For people living in the West. 

------

## Getting the speech text and setup

Here we load packages and define a text statistics function:


```raku
use HTTP::Tiny;
use JSON::Fast;
use Data::Reshapers;

sub text-stats(Str:D $txt) { <chars words lines> Z=> [$txt.chars, $txt.words.elems, $txt.lines.elems] }
```


### Ingest text

Here we ingest the text of the speech:


```raku
my $url = 'https://raw.githubusercontent.com/antononcube/RakuForPrediction-blog/main/Data/Putin-State-of-the-Nation-Address-2024-02-29-Russian.txt';
my $txtRU = HTTP::Tiny.new.get($url)<content>.decode;

$txtRU .= subst(/ \v+ /, "\n", :g);
text-stats($txtRU)
```

```
(chars => 93212 words => 12797 lines => 290)
```


### LLM access configuration

Here we configure LLM access -- we use OpenAI's model "gpt-4-turbo-preview" since it allows inputs with 128K tokens:


```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.7);
$conf.Hash.elems

#  22
```

### LLM functions

Here we define an LLM translation function:


```raku
my &fTrans = llm-function({"Translate from $^a to $^b the following text:\n $^c"}, e => $conf)
```

Here we make a function of extracting *significant* parts from the interview:


```raku
my &fProv = llm-function({"Which are the top $^a most $^b in the following speech? Answer in English.\n\n" ~ $txtRU}, e => $conf)
```

------

## Summary

Here we summarize the speech via an LLM synthesis:


```raku
my $summary = llm-synthesize([
    "Summarize the following speech within 300 words.\n\n", 
    $txtRU,
    ], e => $conf);

text-stats($summary)
```

```
# (chars => 1625 words => 217 lines => 5)
```


Show the summary in Markdown format:


```raku
#% markdown
$summary
```

In his address, Vladimir Putin emphasized the vision for Russia's future, focusing on strategic tasks crucial for the country's long-term development. He highlighted the importance of direct engagement with citizens, including workers, educators, scientists, and military personnel, acknowledging their role in shaping government actions and initiatives. Putin expressed gratitude to various professionals and emphasized plans for large-scale investments in social services, demographics, the economy, science, technology, and infrastructure.

Putin stressed the need for a more equitable tax system, supporting families and businesses investing in development and innovation, while closing loopholes for tax evasion. He announced significant financial support for regional development, infrastructure modernization, and environmental protection. The address included plans to enhance Russia's transportation network, including highways, airports, and the Northern Sea Route, to boost economic and tourism potential.

The speech also highlighted the importance of supporting veterans and participants of the special military operation, proposing a new professional development program "The Time of Heroes" to prepare them for leadership roles in various sectors. Putin called for a collective effort from the state, society, and business to achieve national goals, emphasizing that the success of these plans heavily relies on the courage and determination of Russian soldiers currently in combat. He concluded by expressing confidence in Russia's future victories and successes, backed by national solidarity and resilience.



---- 

## Themes

Here we make an LLM request for finding and distilling the themes or the speech:


```raku
my $llmParts = llm-synthesize([
    'Split the following speech into thematic parts:', 
    $txtRU,
    "Return the parts as a JSON array.",
    llm-prompt('NothingElse')('JSON')
    ], e => $conf, form => sub-parser('JSON'):drop);

deduce-type($llmParts)
```

```
# Vector(Assoc(Atom((Str)), Atom((Str)), 2), 10)
```

Here we tabulate the found themes:


```raku
#%html
$llmParts ==> data-translation(field-names => <theme content>)
```

<table border="1"><thead><tr><th>theme</th><th>content</th></tr></thead><tbody><tr><td>Introduction</td><td>Addressing the Federal Assembly, focusing on the future, strategic tasks, and long-term development.</td></tr><tr><td>Economic Development and Strategic Goals</td><td>Action program formed through dialogs, addressing real people&#39;s needs, and focusing on strategic development tasks.</td></tr><tr><td>National Projects and Strategic Initiatives</td><td>Efforts in various sectors including regional development, technology, economy, and social programs.</td></tr><tr><td>Social Programs and Demographics</td><td>Initiatives aimed at supporting families, increasing birth rates, and improving living standards.</td></tr><tr><td>Education and Youth Development</td><td>Improving education systems, supporting youth, and creating opportunities for professional growth.</td></tr><tr><td>Technological Development and Innovation</td><td>Investing in new technologies, supporting startups, and enhancing Russia&#39;s competitiveness.</td></tr><tr><td>Infrastructure Development</td><td>Improvements in transportation, utilities, and urban development to enhance quality of life.</td></tr><tr><td>Environmental Protection and Sustainability</td><td>Programs for ecological conservation, waste management, and promoting green technologies.</td></tr><tr><td>Defense and Security</td><td>Acknowledging the role of military personnel and veterans in national security and development.</td></tr><tr><td>National Unity and Future Vision</td><td>Emphasizing solidarity, resilience, and the collective effort towards Russia&#39;s prosperity.</td></tr></tbody></table>


------

## Important parts

### Most important statements

Here we get the most important statements:


```raku
#% markdown
&fProv(3, "important statements")
```


Given the extensive nature of the speech, identifying the top 3 most important statements depends on the context of what one considers "important"—whether it be strategic goals, domestic policies, military actions, or socio-economic initiatives. However, based on the broad significance and impact, the following three statements can be highlighted as critically important:

1. **Strategic Development and Sovereignty**: "Самостоятельность, самодостаточность, суверенитет нужно доказывать, подтверждать каждый день. Речь идёт о нашей и только нашей ответственности за настоящее и за будущее России. Это наша родина, родина наших предков, и она нужна и дорога только нам и, конечно, потомкам, которым мы обязаны передать сильную и благополучную страну."
   - This statement underscores the importance Putin places on Russia's autonomy, self-sufficiency, and sovereignty, emphasizing the responsibility to maintain and strengthen these principles for the country's future.

2. **Special Military Operation and its Heroes**: "Такие, безусловно, не отступят, не подведут и не предадут. Они и должны выходить на ведущие позиции и в системе образования и воспитания молодёжи, и в общественных объединениях, в госкомпаниях, бизнесе, в государственном и муниципальном управлении, возглавлять регионы, предприятия в конечном итоге, самые крупные отечественные проекты."
   - This part highlights the role and valor of those participating in what Russia calls the "special military operation," suggesting they should be integrated into leading positions across all sectors of society, reflecting the operation's significance in Putin's vision for Russia.

3. **Long-term Development Plans and Investments**: "Несмотря на сложный период, несмотря на нынешние испытания и трудности, мы намечаем долгосрочные планы. Программа, которую обозначил сегодня в Послании, носит объективный и фундаментальный характер. Это программа сильной, суверенной страны, которая уверенно смотрит в будущее. Для достижения поставленных целей у нас есть и ресурсы, и колоссальные возможности."
   - This statement underlines Putin's commitment to Russia's long-term strategic development and investments despite current challenges, portraying an optimistic and determined vision for the country's future.

These statements encapsulate key themes of sovereignty, military valor, and long-term development, which are recurrent in Putin's address, highlighting their importance in the broader context of Russia's direction and policies under his leadership.



#### Most provocative statements for the West

Here we (try to) get the most provocative statements form Western's politician's point of view:


```raku
#% markdown
&fProv(3, "provocative statements from Western politician's point of view")
```


Given the extensive content of the speech, identifying the top 3 most provocative statements from a Western politician's point of view involves subjective interpretation, as what might be considered provocative can vary depending on specific sensitivities and current geopolitical contexts. However, based on the themes and assertions made in Vladimir Putin's speech, here are three statements or themes that could be seen as particularly provocative or significant from a Western perspective:

1. **Defense of the "Russian Spring" and actions in Crimea and Donbass**: Putin's celebration of the 10th anniversary of the "Russian Spring" and the pride in the actions of Crimea, Sevastopol, and the people of Donbass could be seen as provocative. This is because the annexation of Crimea by Russia in 2014 and the ongoing conflict in Eastern Ukraine are viewed by many Western countries as violations of international law and Ukraine's sovereignty.

2. **Criticism of the West and Claims of Western Aggression**: Putin's assertion that the West, with its "colonial habits" of inciting national conflicts globally, aims not just to contain Russia's development but to turn it into a dependent territory, reflects a strong criticism of Western policies. He accuses the West of wanting to bring discord and weaken Russia from within, similar to what he claims was done in Ukraine. This portrayal of the West seeking Russia's strategic defeat could be seen as particularly provocative amid current tensions.

3. **Strategic Nuclear Forces Readiness and Advanced Weapons Development**: Putin's statement about the readiness of Russia's strategic nuclear forces and the mention of advanced weapons systems, such as hypersonic missiles and other strategic capabilities, could be viewed as a provocative show of military strength. The emphasis on nuclear readiness and the development of weapons that can bypass missile defense systems underscore the ongoing arms race and can be seen as a direct challenge to NATO and Western military capabilities.

These points, among others in the speech, reflect Russia's stance on key geopolitical issues, its criticism of Western policies, and its emphasis on military and strategic strength. The provocative nature of these statements lies in their challenge to the current international order and the implications for security and stability in Europe and beyond.


-------

## Talking to the West

Taking into account the content of the LLM results above here we craft and execute a special prompt that makes the LLM "pretend" that it is V. Putin and addresses bellicose Western politicians:


```raku
my $westTellPOV = llm-synthesize([
    "You are the speaker of the following speech.",
    "Make a short statement addressing Western politicians based on the following full speech.",
    "The Western politicians you address are very bellicose, so, accentuate on Russia's ability:",
    "(i) to deploy and use dangerous weapons, and",
    "(ii) be unaffected by economic sanctions.",
    "Mention concrete weapons.",
    # llm-prompt('NothingElse')('Statements on weapons, sanctions, and wars in English'),
    "\n\n", 
    $txtRU,
    ], e => $conf);

text-stats($westTellPOV)
```

```
# (chars => 1651 words => 236 lines => 5)
```


Here we render the LLM result as in Markdown format:


```raku
#% markdown
$westTellPOV
```

Addressing Western politicians, it is imperative to recognize that Russia stands as a formidable power, possessing a formidable arsenal and an economy resilient to sanctions. Our strategic nuclear forces remain on high alert, including the deployment of avant-garde hypersonic systems like "Kinzhal" and the "Tsirkon" hypersonic missile, which have already been proven in combat efficiency. Additionally, the "Avangard" hypersonic glide vehicles and "Peresvet" laser systems bolster our defensive capabilities, alongside the ongoing tests of the "Burevestnik" nuclear-powered cruise missile and the "Poseidon" unmanned underwater vehicle. These advanced weapons systems underscore our technological prowess and military readiness.

Moreover, Russia's economy has demonstrated its resilience in the face of external pressures, including sanctions. Our commitment to the welfare of our citizens, the development of our country, and the protection of our sovereignty remains unwavering. The strength of our economy is further evidenced by our ability to undertake significant national projects and to invest in our social and economic infrastructure, ensuring the prosperity and security of our nation.

In light of these capabilities and our steadfast resolve, it is crucial for Western politicians to reassess their approach towards Russia. Engaging in dialogue and mutual respect is the pathway to peace and stability in the international arena. We urge Western leaders to consider the implications of their actions and to work towards constructive relations with Russia. The time is now to foster understanding and cooperation for the benefit of all.



-----

## Weapons tabulation

From the point of views of Western citizens and politicians of great interest (should be) the statements in the speech that discuss weapons for mass destruction and weapons that give decisive military advantage. Here we synthesize an LLM response that tabulates mentioned weapons' names, status, and descriptions:


```raku
my $weapons = llm-synthesize([
    "Briefly describe the weapons mentioned in the following speech.",
    "Give the result in English with JSON data structure that is table with the column names: name, type, status, description, damage.",
    "Make sure the descriptions provide some level of detail.",
    llm-prompt('NothingElse')('JSON'),
    "\n\n", 
    $txtRU,
    ], e => $conf, form => sub-parser('JSON'):drop);

deduce-type($weapons)
```

```
# Vector(Assoc(Atom((Str)), Atom((Str)), 5), 7)
```

Here we render the table:


```raku
#%html
$weapons ==> data-translation(field-names => <name type status description damage>)
```


<table border="1"><thead><tr><th>name</th><th>type</th><th>status</th><th>description</th><th>damage</th></tr></thead><tbody><tr><td>Кинжал</td><td>Hypersonic Airborne Missile System</td><td>Operational</td><td>A hypersonic missile capable of striking targets with high precision over long distances at speeds exceeding Mach 5.</td><td>High</td></tr><tr><td>Циркон</td><td>Hypersonic Cruise Missile</td><td>Operational</td><td>A sea-based hypersonic cruise missile designed to attack naval and ground targets.</td><td>High</td></tr><tr><td>Авангард</td><td>Hypersonic Glide Vehicle</td><td>Operational</td><td>Mounted on an intercontinental ballistic missile, it can carry a nuclear payload and maneuver at high speeds to evade missile defense systems.</td><td>Very High</td></tr><tr><td>Пересвет</td><td>Laser Weapon System</td><td>Operational</td><td>A laser system purportedly designed to counter aerial threats and possibly to disable satellites and other space assets.</td><td>Variable</td></tr><tr><td>Буревестник</td><td>Nuclear-Powered Cruise Missile</td><td>In Testing</td><td>Claimed to have virtually unlimited range thanks to its nuclear power source, designed for strategic bombing missions.</td><td>Potentially Very High</td></tr><tr><td>Посейдон</td><td>Nuclear-Powered Underwater Drone</td><td>In Development</td><td>An autonomous underwater vehicle intended to carry nuclear warheads to create radioactive tsunamis near enemy coastlines.</td><td>Catastrophic</td></tr><tr><td>Сармат</td><td>Intercontinental Ballistic Missile</td><td>Operational</td><td>A heavy missile intended to replace the aging Soviet-era Voevoda, capable of carrying multiple nuclear warheads.</td><td>Catastrophic</td></tr></tbody></table>



**Remark:** We could have specified in the prompt the following column names to be used: "name_english, name_russian, type, status, description, damage".
But it turns out that (with the LLM current models) the results are less reproducible. Hence, we use "name, type, status, description, damage" and adjust with corresponding translations below.

Since the results of the above LLM synthesis are often given in Russian or have interlaced Russian and English names or phrases here we translate the LLM result into English:


```raku
$weapons
==> to-json()
==> &fTrans("Russian", "English") 
==> my $weaponsEN;

deduce-type($weaponsEN);
```

```
# Vector(Assoc(Atom((Str)), Atom((Str)), 5), 7)
```


Here we derive a dictionary of weapon names and corresponding Wikipedia URLs:


```raku
#`[
my %urlTbl = llm-synthesize([
    "Provide a JSON dictionary of the Wikipedia hyperlinks for these weapons:",
    $weaponsEN.map(*<name>).join(', '),
    ], e => $conf, form => sub-parser('JSON'):drop);

deduce-type(%urlTbl);
]
```


Here is a direct assignment of one the results of the code above, for which we have verified the hyperlinks:


```raku
my %urlTbl = {:Avangard("https://en.wikipedia.org/wiki/Avangard_(hypersonic_glide_vehicle)"), :Burevestnik("https://en.wikipedia.org/wiki/9M730_Burevestnik"), :Kinzhal("https://en.wikipedia.org/wiki/Kh-47M2_Kinzhal"), :Peresvet("https://en.wikipedia.org/wiki/Peresvet_(laser_weapon)"), :Poseidon("https://en.wikipedia.org/wiki/Status-6_Oceanic_Multipurpose_System"), :Sarmat("https://en.wikipedia.org/wiki/RS-28_Sarmat"), :Zircon("https://en.wikipedia.org/wiki/3M22_Zircon")};
%urlTbl.elems;

# 7
```

Here we craft a prompt with which we merge the Russian names column of the weapons table derived first into the translated table:


```raku
my $weaponsEN2 = llm-synthesize([
    "Take the Russian names of the first JSON table and put them in a new column in the second JSON table:",
    "1st table:\n", to-json($weapons),
    "2nd table:\n", to-json($weaponsEN)
], e => $conf, form => sub-parser('JSON'):drop);

deduce-type($weaponsEN2)
```

```
# Vector(Assoc(Atom((Str)), Atom((Str)), 6), 7)
```

Here we make a corresponding HTML table and modify the (English) names column to have hyperlinks:


```raku
#% html
$weaponsEN2 ==> data-translation(field-names=><name russian_name type status description damage>) ==> my $weaponsEN3;
my &reg = / '<td>' (<{%urlTbl.keys.join(' | ')}>) '</td>' /;
$weaponsEN3.subst(&reg, {
    "<td><span><a href=\"{%urlTbl{$0.Str}}\">{$0.Str}</a></span></td>"
}, :g)
```


<table border="1"><thead><tr><th>name</th><th>russian_name</th><th>type</th><th>status</th><th>description</th><th>damage</th></tr></thead><tbody><tr><td><span><a href="https://en.wikipedia.org/wiki/Kh-47M2_Kinzhal">Kinzhal</a></span></td><td>Кинжал</td><td>Hypersonic Airborne Missile System</td><td>Operational</td><td>A hypersonic missile capable of striking targets with high precision over long distances at speeds exceeding Mach 5.</td><td>High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/3M22_Zircon">Zircon</a></span></td><td>Циркон</td><td>Hypersonic Cruise Missile</td><td>Operational</td><td>A sea-based hypersonic cruise missile designed to attack naval and ground targets.</td><td>High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Avangard_(hypersonic_glide_vehicle)">Avangard</a></span></td><td>Авангард</td><td>Hypersonic Glide Vehicle</td><td>Operational</td><td>Mounted on an intercontinental ballistic missile, it can carry a nuclear payload and maneuver at high speeds to evade missile defense systems.</td><td>Very High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Peresvet_(laser_weapon)">Peresvet</a></span></td><td>Пересвет</td><td>Laser Weapon System</td><td>Operational</td><td>A laser system purportedly designed to counter aerial threats and possibly to disable satellites and other space assets.</td><td>Variable</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/9M730_Burevestnik">Burevestnik</a></span></td><td>Буревестник</td><td>Nuclear-Powered Cruise Missile</td><td>In Testing</td><td>Claimed to have virtually unlimited range thanks to its nuclear power source, designed for strategic bombing missions.</td><td>Potentially Very High</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/Status-6_Oceanic_Multipurpose_System">Poseidon</a></span></td><td>Посейдон</td><td>Nuclear-Powered Underwater Drone</td><td>In Development</td><td>An autonomous underwater vehicle intended to carry nuclear warheads to create radioactive tsunamis near enemy coastlines.</td><td>Catastrophic</td></tr><tr><td><span><a href="https://en.wikipedia.org/wiki/RS-28_Sarmat">Sarmat</a></span></td><td>Сармат</td><td>Intercontinental Ballistic Missile</td><td>Operational</td><td>A heavy missile intended to replace the aging Soviet-era Voevoda, capable of carrying multiple nuclear warheads.</td><td>Catastrophic</td></tr></tbody></table>



--------

## References

### Articles

[AA1] Anton Antonov,
["Workflows with LLM functions"](https://rakuforprediction.wordpress.com/2023/08/01/workflows-with-llm-functions/),
(2023),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov,
["LLM aids for processing of the first Carlson-Putin interview"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/),
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[OAIb1] OpenAI team,
["New models and developer products announced at DevDay"](https://openai.com/blog/new-models-and-developer-products-announced-at-devday),
(2023),
[OpenAI/blog](https://openai.com/blog).

### Packages

[AAp1] Anton Antonov,
[WWW::OpenAI](https://github.com/antononcube/Raku-WWW-OpenAI) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).


[AAp2] Anton Antonov,
[WWW::PaLM](https://github.com/antononcube/Raku-WWW-PaLM) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).


[AAp3] Anton Antonov,
[WWW::MistralAI](https://github.com/antononcube/Raku-WWW-MistralAI) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).


[AAp4] Anton Antonov,
[WWW::LLaMA](https://github.com/antononcube/Raku-WWW-LLaMA) Raku package,
(2024),
[GitHub/antononcube](https://github.com/antononcube).


[AAp5] Anton Antonov,
[LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package,
(2023),
[GitHub/antononcube](https://github.com/antononcube).


[AAp6] Anton Antonov,
[Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package,
(2023),
[GitHub/antononcube](https://github.com/antononcube).


[AAp7] Anton Antonov,
[Image::Markup::Utilities](https://github.com/antononcube/Raku-Image-Markup-Utilities) Raku package,
(2023),
[GitHub/antononcube](https://github.com/antononcube).


### Videos

[AAv1] Anton Antonov,
["Jupyter Chatbook LLM cells demo (Raku)"](https://www.youtube.com/watch?v=cICgnzYmQZg),
(2023),
[YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv2] Anton Antonov,
["Jupyter Chatbook multi cell LLM chats teaser (Raku)"](https://www.youtube.com/watch?v=wNpIGUAwZB8),
(2023),
[YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction).

[AAv3] Anton Antonov
["Integrating Large Language Models with Raku"](https://www.youtube.com/watch?v=-OxKqRrQvh0),
(2023),
[YouTube/@therakuconference6823](https://www.youtube.com/@therakuconference6823).
