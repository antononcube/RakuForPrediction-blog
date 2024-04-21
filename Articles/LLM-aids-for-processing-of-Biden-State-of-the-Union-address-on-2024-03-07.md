# LLM aids for processing Biden's State of the Union address

### ***... given on March, 7, 2024***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog)   
March 2024

-------

## Introduction

![](https://www.whitehouse.gov/wp-content/uploads/2021/01/P20240307AS-1589.jpg?resize=1038,692)

In this notebook we provide aids and computational workflows for the analysis of Joe Biden's State of the Union address given on March 7th, 2024.
We use Large Language Models (LLMs). We walk through various steps involved in examining and understanding the speech in a systematic and reproducible manner.

The speech transcript is taken from [whitehouse.gov](https://www.whitehouse.gov/briefing-room/speeches-remarks/2024/03/07/remarks-of-president-joe-biden-state-of-the-union-address-as-prepared-for-delivery-2/).

The computations are done with a [Raku chatbook](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp6, AAv1÷AAv3]. The LLM functions used in the workflows are explained and demonstrated in [AA1, AAv3].
The workflows are done with OpenAI's models [AAp1]. Currently the models of Google's (PaLM), [AAp2], and MistralAI, [AAp3], cannot be used with the workflows below because their input token limits are too low.

Similar set of workflows and prompts are described in:

- ["LLM aids for processing of the first Carlson-Putin interview"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/), [AA2]
- ["LLM aids for processing Putin’s State-Of-The-Nation speech"](https://rakuforprediction.wordpress.com/2024/03/03/llm-aids-for-processing-putins-state-of-the-nation-speech/), [AA3]
- ["Comprehension AI Aids for “Can AI Solve Science?”](https://rakuforprediction.wordpress.com/2024/03/09/comprehension-ai-aids-for-can-ai-solve-science/), [AA4]

The prompts of the latter are used below.

The following table -- derived from Biden's address -- has the most important or provocative statements (found by an LLM):

<table border="1"><thead><tr><th>subject</th><th>statement</th></tr></thead><tbody><tr><td>Global politics and security</td><td>Overseas, Putin of Russia is on the march, invading Ukraine and sowing chaos throughout Europe and beyond.</td></tr><tr><td>Support for Ukraine</td><td>But Ukraine can stop Putin if we stand with Ukraine and provide the weapons it needs to defend itself.</td></tr><tr><td>Domestic politics</td><td>A former American President actually said that, bowing down to a Russian leader.</td></tr><tr><td>NATO</td><td>Today, we’ve made NATO stronger than ever.</td></tr><tr><td>Democracy and January 6th</td><td>January 6th and the lies about the 2020 election, and the plots to steal the election, posed the gravest threat to our democracy since the Civil War.</td></tr><tr><td>Reproductive rights</td><td>Guarantee the right to IVF nationwide!</td></tr><tr><td>Economic recovery and policies</td><td>15 million new jobs in just three years – that’s a record!</td></tr><tr><td>Healthcare and prescription drug costs</td><td>Instead of paying $400 a month for insulin seniors with diabetes only have to pay $35 a month!</td></tr><tr><td>Education and tuition</td><td>Let’s continue increasing Pell Grants for working- and middle-class families.</td></tr><tr><td>Tax reform</td><td>It’s time to raise the corporate minimum tax to at least 21% so every big corporation finally begins to pay their fair share.</td></tr><tr><td>Gun violence prevention</td><td>I’m demanding a ban on assault weapons and high-capacity magazines!</td></tr><tr><td>Immigration</td><td>Send me the border bill now!</td></tr><tr><td>Climate action</td><td>I am cutting our carbon emissions in half by 2030.</td></tr><tr><td>Israel and Gaza conflict</td><td>Israel has a right to go after Hamas.</td></tr><tr><td>Vision for America&#39;s future</td><td>I see a future where the middle class finally has a fair shot and the wealthy finally have to pay their fair share in taxes.</td></tr></tbody></table>

### Structure

The structure of the notebook is as follows:

1. **Getting the speech text and setup**   
    Standard ingestion and setup.
2. **Themes**      
    TL;DR via a table of themes.
3. **Most important or provocative statements**   
    What are the most important or provocative statements?
4. **Summary and recommendations**    
    Extracting speech wisdom.
5. **Hidden and propaganda messages**   
    For people living in USA. 

------

## Getting the speech text and setup

Here we load packages and define a text statistics function and HTML stripping function:


```raku
use HTTP::Tiny;
use JSON::Fast;
use Data::Reshapers;

sub text-stats(Str:D $txt) { <chars words lines> Z=> [$txt.chars, $txt.words.elems, $txt.lines.elems] }

sub strip-html(Str $html) returns Str {

    my $res = $html
    .subst(/'<style'.*?'</style>'/, :g)
    .subst(/'<script'.*?'</script>'/, :g)
    .subst(/'<'.*?'>'/, :g)
    .subst(/'&lt;'.*?'&gt;'/, :g)
    .subst(/'&nbsp;'/, ' ', :g)
    .subst(/[\v\s*] ** 2..*/, "\n\n", :g);

    return $res;
}
```




    &strip-html



### Ingest text

Here we ingest the text of the speech:


```raku
my $url = 'https://www.whitehouse.gov/briefing-room/speeches-remarks/2024/03/07/remarks-of-president-joe-biden-state-of-the-union-address-as-prepared-for-delivery-2/';
my $htmlEN = HTTP::Tiny.new.get($url)<content>.decode;

$htmlEN .= subst(/ \v+ /, "\n", :g);

my $txtEN = strip-html($htmlEN);

$txtEN .= substr($txtEN.index('March 07, 2024') .. ($txtEN.index("Next Post:") - 1));

text-stats($txtEN)
```




    (chars => 37702 words => 6456 lines => 453)



### LLM access configuration

Here we configure LLM access -- we use OpenAI's model "gpt-4-turbo-preview" since it allows inputs with 128K tokens:


```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.7);
$conf.Hash.elems
```




    22



---- 

## Themes

Here we extract the themes found in the speech and tabulate them (using the prompt ["ThemeTableJSON"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/ThemeTableJSON/)):


```raku
my $tblThemes = llm-synthesize(llm-prompt("ThemeTableJSON")($txtEN, "article", 50), e => $conf, form => sub-parser('JSON'):drop);

$tblThemes.&dimensions;
```




    (16 2)



Here we tabulate the found themes:


```raku
#% html
$tblThemes ==> data-translation(field-names=><theme content>)
```




<table border="1"><thead><tr><th>theme</th><th>content</th></tr></thead><tbody><tr><td>Introduction</td><td>President Joe Biden begins by reflecting on historical challenges to freedom and democracy, comparing them to current threats both domestically and internationally.</td></tr><tr><td>Foreign Policy and National Security</td><td>Biden addresses the situation in Ukraine, NATO&#39;s strength, and the necessity of bipartisan support to confront Putin and Russia&#39;s aggression.</td></tr><tr><td>Domestic Challenges and Democracy</td><td>He discusses the January 6th insurrection, the ongoing threats to democracy in the U.S., and the need for unity to defend democratic values.</td></tr><tr><td>Reproductive Rights</td><td>Biden criticizes the overturning of Roe v. Wade, shares personal stories to highlight the impact, and calls for legislative action to protect reproductive freedoms.</td></tr><tr><td>Economic Recovery and Policy</td><td>The President outlines his administration&#39;s achievements in job creation, economic growth, and efforts to reduce inflation, emphasizing a middle-out economic approach.</td></tr><tr><td>Infrastructure and Manufacturing</td><td>Biden highlights investments in infrastructure, clean energy, and manufacturing, including specific projects and acts that have contributed to job creation and economic development.</td></tr><tr><td>Healthcare</td><td>He details achievements in healthcare reform, including prescription drug pricing, expansion of Medicare, and initiatives focused on women&#39;s health research.</td></tr><tr><td>Housing and Education</td><td>The President proposes solutions for the housing crisis and outlines plans for improving education from preschool to college affordability.</td></tr><tr><td>Tax Reform and Fiscal Responsibility</td><td>Biden proposes tax reforms targeting corporations and the wealthy to ensure fairness and fund his policy initiatives, contrasting his approach with the previous administration.</td></tr><tr><td>Climate Change and Environmental Policy</td><td>He discusses significant actions taken to address climate change, emphasizing job creation in clean energy and conservation efforts.</td></tr><tr><td>Immigration and Border Security</td><td>Biden contrasts his immigration policy with his predecessor&#39;s, advocating for a bipartisan border security bill and a compassionate approach to immigration.</td></tr><tr><td>Voting Rights and Civil Rights</td><td>The President calls for the passage of voting rights legislation and addresses issues around diversity, equality, and the protection of fundamental rights.</td></tr><tr><td>Gun Violence Prevention</td><td>He shares personal stories to underscore the urgency of gun violence prevention, celebrating past achievements and calling for further action.</td></tr><tr><td>Foreign Conflicts</td><td>Biden addresses the conflict in the Middle East, emphasizing the need for humanitarian aid and a two-state solution for Israel and Palestine.</td></tr><tr><td>Global Leadership and Competition</td><td>The President discusses the U.S.&#39;s economic competition with China, American leadership on the global stage, and the importance of alliances.</td></tr><tr><td>Vision for America&#39;s Future</td><td>Biden concludes with an optimistic vision for America, focusing on unity, democracy, and a future where America leads by example.</td></tr></tbody></table>



------

## Most important or provocative statements

Here we find important or provocative statements in the speech via an LLM synthesis:


```raku
my $imp = llm-synthesize([
    "Give the most important or provocative statements in the following speech.\n\n", 
    $txtEN,
    "Give the results as a JSON array with subject-statement pairs.",
    llm-prompt('NothingElse')('JSON')
    ], e => $conf, form => sub-parser('JSON'):drop);

$imp.&dimensions
```




    (15 2)



Show the important or provocative statements in Markdown format:


```raku
#% html
$imp ==> data-translation(field-names => <subject statement>)
```




<table border="1"><thead><tr><th>subject</th><th>statement</th></tr></thead><tbody><tr><td>Global politics and security</td><td>Overseas, Putin of Russia is on the march, invading Ukraine and sowing chaos throughout Europe and beyond.</td></tr><tr><td>Support for Ukraine</td><td>But Ukraine can stop Putin if we stand with Ukraine and provide the weapons it needs to defend itself.</td></tr><tr><td>Domestic politics</td><td>A former American President actually said that, bowing down to a Russian leader.</td></tr><tr><td>NATO</td><td>Today, we’ve made NATO stronger than ever.</td></tr><tr><td>Democracy and January 6th</td><td>January 6th and the lies about the 2020 election, and the plots to steal the election, posed the gravest threat to our democracy since the Civil War.</td></tr><tr><td>Reproductive rights</td><td>Guarantee the right to IVF nationwide!</td></tr><tr><td>Economic recovery and policies</td><td>15 million new jobs in just three years – that’s a record!</td></tr><tr><td>Healthcare and prescription drug costs</td><td>Instead of paying $400 a month for insulin seniors with diabetes only have to pay $35 a month!</td></tr><tr><td>Education and tuition</td><td>Let’s continue increasing Pell Grants for working- and middle-class families.</td></tr><tr><td>Tax reform</td><td>It’s time to raise the corporate minimum tax to at least 21% so every big corporation finally begins to pay their fair share.</td></tr><tr><td>Gun violence prevention</td><td>I’m demanding a ban on assault weapons and high-capacity magazines!</td></tr><tr><td>Immigration</td><td>Send me the border bill now!</td></tr><tr><td>Climate action</td><td>I am cutting our carbon emissions in half by 2030.</td></tr><tr><td>Israel and Gaza conflict</td><td>Israel has a right to go after Hamas.</td></tr><tr><td>Vision for America&#39;s future</td><td>I see a future where the middle class finally has a fair shot and the wealthy finally have to pay their fair share in taxes.</td></tr></tbody></table>



-------

## Summary and recommendations

Here we get a summary and extract ideas, quotes, and recommendations from the speech:


```raku
my $sumIdea =llm-synthesize(llm-prompt("ExtractArticleWisdom")($txtEN), e => $conf);

text-stats($sumIdea)
```




    (chars => 6166 words => 972 lines => 95)



The result is rendered below.

<hr width="65%">


```raku
#% markdown
$sumIdea.subst(/ ^^ '#' /, '###', :g)
```




### SUMMARY

President Joe Biden delivered the State of the Union Address on March 7, 2024, focusing on the challenges and opportunities facing the United States. He discussed the assault on democracy, the situation in Ukraine, domestic policies including healthcare and economy, and the need for unity and progress in addressing national and international issues.

### IDEAS:

- The historical context of challenges to freedom and democracy, drawing parallels between past and present threats.
- The role of the United States in supporting Ukraine against Russian aggression.
- The importance of bipartisan support for national security and democracy.
- The need for America to take a leadership role in NATO and support its allies.
- The dangers of political figures undermining democratic values for personal or political gain.
- The connection between domestic policies and the strength of democracy, including reproductive rights and healthcare.
- The economic recovery and growth under the Biden administration, emphasizing job creation and infrastructure improvements.
- The focus on middle-class prosperity and the role of unions in economic recovery.
- The commitment to addressing climate change and promoting clean energy jobs.
- The significance of education, from pre-school access to college affordability, in securing America's future.
- The need for fair taxation and closing loopholes for the wealthy and corporations.
- The importance of healthcare reform, including lowering prescription drug costs and expanding Medicare.
- The commitment to protecting Social Security and Medicare from cuts.
- The approach to immigration reform and border security as humanitarian and security issues.
- The stance on gun violence prevention and the need for stricter gun control laws.
- The emphasis on America's resilience and optimism for the future.
- The call for unity in defending democracy and building a better future for all Americans.
- The vision of America as a land of possibilities, with a focus on progress and inclusivity.
- The acknowledgment of America's role in the world, including support for Israel and a two-state solution with Palestine.
- The importance of scientific research and innovation in solving major challenges like cancer.

### QUOTES:

- "Not since President Lincoln and the Civil War have freedom and democracy been under assault here at home as they are today."
- "America is a founding member of NATO the military alliance of democratic nations created after World War II to prevent war and keep the peace."
- "History is watching, just like history watched three years ago on January 6th."
- "You can’t love your country only when you win."
- "Inflation has dropped from 9% to 3% – the lowest in the world!"
- "The racial wealth gap is the smallest it’s been in 20 years."
- "I’ve been delivering real results in a fiscally responsible way."
- "Restore the Child Tax Credit because no child should go hungry in this country!"
- "No billionaire should pay a lower tax rate than a teacher, a sanitation worker, a nurse!"
- "We are the only nation in the world with a heart and soul that draws from old and new."

### HABITS:

- Advocating for bipartisan cooperation in Congress.
- Emphasizing the importance of education in personal growth and national prosperity.
- Promoting the use of clean energy and sustainable practices to combat climate change.
- Prioritizing healthcare reform to make it more affordable and accessible.
- Supporting small businesses and entrepreneurship as engines of economic growth.
- Encouraging scientific research and innovation, especially in healthcare.
- Upholding the principles of fair taxation and economic justice.
- Leveraging diplomacy and international alliances for global stability.
- Committing to the protection of democratic values and institutions.
- Fostering community engagement and civic responsibility.

### FACTS:

- The United States has welcomed Finland and Sweden into NATO, strengthening the alliance.
- The U.S. economy has created 15 million new jobs in three years, a record number.
- Inflation in the United States has decreased from 9% to 3%.
- More people have health insurance in the U.S. today than ever before.
- The racial wealth gap is the smallest it has been in 20 years.
- The United States is investing more in research and development than ever before.
- The Biden administration has made the largest investment in public safety ever through the American Rescue Plan.
- The murder rate saw the sharpest decrease in history last year.
- The United States is leading international efforts to provide humanitarian assistance to Gaza.
- The U.S. has revitalized its partnerships and alliances in the Pacific region.

### REFERENCES:

- NATO military alliance.
- Bipartisan National Security Bill.
- Chips and Science Act.
- Bipartisan Infrastructure Law.
- Affordable Care Act (Obamacare).
- Voting Rights Act.
- Freedom to Vote Act.
- John Lewis Voting Rights Act.
- PRO Act for worker's rights.
- PACT Act for veterans exposed to toxins.

### RECOMMENDATIONS:

- Stand with Ukraine and support its defense against Russian aggression.
- Strengthen NATO and support new member states.
- Pass the Bipartisan National Security Bill to enhance U.S. security.
- Guarantee reproductive rights nationwide and protect healthcare decisions.
- Continue economic policies that promote job creation and infrastructure development.
- Implement fair taxation for corporations and the wealthy to ensure economic justice.
- Expand Medicare and lower prescription drug costs for all Americans.
- Protect and strengthen Social Security and Medicare.
- Pass comprehensive immigration reform and secure the border humanely.
- Address climate change through significant investments in clean energy and jobs.
- Promote education access from pre-school to college to ensure a competitive workforce.
- Implement stricter gun control laws, including bans on assault weapons and universal background checks.
- Support a two-state solution for Israel and Palestine and work towards peace in the Middle East.
- Harness the promise of artificial intelligence while protecting against its perils.



-------

## Hidden and propaganda messages


 In this section we try to find is the speech apolitical and propaganda-free. 

**Remark:** We leave to the reader as an exercise to verify that both the overt and hidden messages found by the LLM below are explicitly stated in article.

Here we find the hidden and “propaganda” messages in the article:


```raku
my $propMess =llm-synthesize([llm-prompt("FindPropagandaMessage"), $txtEN], e => $conf);

text-stats($propMess)
```




    (chars => 6441 words => 893 lines => 83)



**Remark:** The prompt ["FindPropagandaMessage"](https://www.wolframcloud.com/obj/antononcube/DeployedResources/Prompt/FindPropagandaMessage/) has an explicit instruction to say that it is intentionally cynical. It is also, marked as being "For fun."

The LLM result is rendered below.

<hr width="65%">


```raku
#% markdown
$propMess.subst(/ ^^ '#' /, '###', :g).subst(/ ^^ (<[A..Z \h \']>+ ':') /, { "### {$0.Str} \n"}, :g)
```




OVERT MESSAGE

President Biden emphasizes democracy, support for Ukraine, and domestic advancements in his address.

HIDDEN MESSAGE

Biden seeks to consolidate Democratic power by evoking fear of Republican governance and foreign threats.

HIDDEN OPINIONS

- Democratic policies ensure national and global security effectively.
- Republican opposition jeopardizes both national unity and international alliances.
- Historical comparisons highlight current threats to democracy as unprecedented.
- Support for Ukraine is a moral and strategic imperative for global democracy.
- Criticism of the Supreme Court's decisions reflects a push for legislative action on contentious issues.
- Emphasis on job creation and economic policies aims to showcase Democratic governance success.
- Investments in infrastructure and technology are crucial for future American prosperity.
- Health care reforms and education investments underscore a commitment to social welfare.
- Climate change initiatives are both a moral obligation and economic opportunity.
- Immigration reforms are positioned as essential to American identity and values.

SUPPORTING ARGUMENTS and QUOTES

- Comparisons to past crises underscore the urgency of current threats.
- Criticism of Republican predecessors and Congress members suggests a need for Democratic governance.
- References to NATO and Ukraine highlight a commitment to international democratic principles.
- Mention of Supreme Court decisions and calls for legislative action stress the importance of Democratic control.
- Economic statistics and policy achievements are used to argue for the effectiveness of Democratic governance.
- Emphasis on infrastructure, technology, and climate investments showcases forward-thinking policies.
- Discussion of health care and education reforms highlights a focus on social welfare.
- The portrayal of immigration reforms reflects a foundational American value under Democratic leadership.

DESIRED AUDIENCE OPINION CHANGE

- See Democratic policies as essential for both national and global security.
- View Republican opposition as a threat to democracy and unity.
- Recognize the urgency of supporting Ukraine against foreign aggression.
- Agree with the need for legislative action on Supreme Court decisions.
- Appreciate the success of Democratic economic and infrastructure policies.
- Support Democratic initiatives on climate change as crucial for the future.
- Acknowledge the importance of health care and education investments.
- Value immigration reforms as core to American identity and values.
- Trust in Democratic leadership for navigating global crises.
- Believe in the effectiveness of Democratic governance for social welfare.

DESIRED AUDIENCE ACTION CHANGE

- Support Democratic candidates in elections.
- Advocate for legislative action on contentious Supreme Court decisions.
- Endorse and rally for Democratic economic and infrastructure policies.
- Participate in initiatives supporting climate change action.
- Engage in advocacy for health care and education reforms.
- Embrace and promote immigration reforms as fundamental to American values.
- Voice opposition to Republican policies perceived as threats to democracy.
- Mobilize for international solidarity, particularly regarding Ukraine.
- Trust in and amplify the successes of Democratic governance.
- Actively defend democratic principles both nationally and internationally.

MESSAGES

President Biden wants you to believe he is advocating for democracy and progress, but he is actually seeking to consolidate Democratic power and diminish Republican influence.

PERCEPTIONS

President Biden wants you to believe he is a unifier and protector of democratic values, but he's actually a strategic politician emphasizing Democratic successes and Republican failures.

ELLUL'S ANALYSIS

According to Jacques Ellul's "Propaganda: The Formation of Men's Attitudes," Biden's address exemplifies modern political propaganda through its strategic framing of issues, historical comparisons, and appeals to democratic ideals. Ellul would likely note the address's dual function: to solidify in-group unity (among Democratic supporters) and to subtly influence the broader public's perceptions of domestic and international challenges. The speech leverages crises as opportunities for reinforcing the necessity of Democratic governance, illustrating Ellul's observation that effective propaganda exploits existing tensions to achieve political objectives.

BERNAYS' ANALYSIS

Based on Edward Bernays' "Propaganda" and "Engineering of Consent," Biden's speech can be seen as an exercise in shaping public opinion towards Democratic policies and leadership. Bernays would recognize the sophisticated use of symbols (e.g., references to historical events and figures) and emotional appeals to construct a narrative that positions Democratic governance as essential for the nation's future. The speech's emphasis on bipartisan achievements and calls for legislative action also reflect Bernays' insights into the importance of creating a perception of consensus and societal progress.

LIPPMANN'S ANALYSIS

Walter Lippmann's "Public Opinion" offers a perspective on how Biden's address attempts to manufacture consent for Democratic policies by presenting a carefully curated version of reality. Lippmann would likely point out the strategic selection of facts, statistics, and stories designed to reinforce the audience's existing preconceptions and to guide them towards desired conclusions. The address's focus on bipartisan accomplishments and urgent challenges serves to create an environment where Democratic solutions appear both reasonable and necessary.

FRANKFURT'S ANALYSIS

Harry G. Frankfurt's "On Bullshit" provides a lens for criticizing the speech's relationship with truth and sincerity. Frankfurt might argue that while the address purports to be an honest assessment of the nation's state, it strategically blurs the line between truth and falsehood to serve political ends. The speech's selective presentation of facts and omission of inconvenient truths could be seen as indicative of a broader political culture where the distinction between lying and misleading is increasingly irrelevant.

**NOTE: This AI is tuned specifically to be cynical and politically-minded. Don't take it as perfect. Run it multiple times and/or go consume the original input to get a second opinion.**



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

[AA3] Anton Antonov,
["LLM aids for processing Putin’s State-Of-The-Nation speech"](https://rakuforprediction.wordpress.com/2024/03/03/llm-aids-for-processing-putins-state-of-the-nation-speech/)
(2024),
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA4] Anton Antonov,
["Comprehension AI Aids for “Can AI Solve Science?”](https://rakuforprediction.wordpress.com/2024/03/09/comprehension-ai-aids-for-can-ai-solve-science/),
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
