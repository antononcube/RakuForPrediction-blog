# LLM over Trump vs. Anderson

#### ***LLM analysis of the slip opinion of the Supreme Court of the United States***

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)   
[RakuForPrediction-blog at GitHub](https://github.com/antononcube/RakuForPrediction-blog)   
March 2024

![](https://rakuforprediction.files.wordpress.com/2024/03/supreme-court-of-the-us-building-small.png)

-----

## Introduction 

In this notebook we ingest the text of the slip opinion 
[No. 23-219, "Donald J. Trump, petitioner, v. Norma Anderson, et al."](https://www.supremecourt.gov/opinions/23pdf/23-719_19m2.pdf) 
of the Supreme Court of the United States, [SCUS1], and analyze it using Large Language Models (LLMs).

In order to do the LLM analysis we ingest and file two dedicated prompts from the GitHub repository "Fabric", [DMr1]:

- [Extract article wisdom](https://github.com/danielmiessler/fabric/tree/main/patterns/extract_article_wisdom)
- [Find hidden message](https://github.com/danielmiessler/fabric/tree/main/patterns/find_hidden_message)

The prompts are ingested into the data resources of the Raku package ["LLM::Prompts"](https://raku.land/zef:antononcube/LLM::Prompts), [AAp4], 
and utilized via LLM functions created with the Raku package ["LLM::Functions"](https://raku.land/zef:antononcube/LLM::Functions), [AAp5].

The computations are done with a [Raku chatbook](https://raku.land/zef:antononcube/Jupyter::Chatbook), [AAp6, AAv1÷AAv3]. The LLM functions used in the workflows are explained and demonstrated in [AA1, AAv3].
The workflows are done with OpenAI's models [AAp1]. Currently the models of Google's (PaLM), [AAp2], and MistralAI, [AAp3], cannot be used with the workflows below because their input token limits are too low.

### Structure

The structure of the notebook is as follows:

1. **Getting the speech text and setup**   
   Standard ingestion and setup.
2. **Opinion's structure**      
   TL;DR via a table of themes.
3. **Extract wisdom**   
   Get a summary and extract ideas, quotes, references, etc.
4. **Hidden messages and propaganda**    
   Reading it with a conspiracy theorist hat.

-----

## Setup


```raku
use HTTP::Tiny;
use JSON::Fast;
use Data::Reshapers;
use PDF::Extract;

sub text-stats(Str:D $txt) { <chars words lines> Z=> [$txt.chars, $txt.words.elems, $txt.lines.elems] }
```

### Ingest text


```raku
my $fileName = $*HOME ~ '/Downloads/23-719_19m2.pdf';
my $extract = Extract.new: file => $fileName;

my $opinion = $$extract.text;

$opinion ==> text-stats()
```




    (chars => 38389 words => 6105 lines => 760)



Here we export the extracted plain text (for easier setup stages of other projects):


```raku
# spurt($fileName.subst('.pdf', '.txt'), $opinion)
```

### LLM access configuration

Here we configure LLM access -- we use OpenAI's model "gpt-4-turbo-preview" since it allows inputs with 128K tokens:


```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.7);
$conf.Hash.elems
```




    22



-------

## Opinion's structure

In this section we summarize the opinion by making a (computable) table of themes. 

Here we synthesize the table using appropriate LLM prompts and a text sub-parser:


```raku
my $tbl = llm-synthesize([
    llm-prompt('ThemeTableJSON')('court opinion', $opinion), 
    llm-prompt('NothingElse')('JSON')
    ], e => $conf, form => sub-parser('JSON'):drop);

deduce-type($tbl)
```




    Vector(Assoc(Atom((Str)), Atom((Str)), 2), 8)



Here we HTML render the theme table:


```raku
#%html
$tbl ==> data-translation(field-names => <theme content>)
```




<table border="1"><thead><tr><th>theme</th><th>content</th></tr></thead><tbody><tr><td>Introduction and Notice</td><td>NOTICE: This opinion is subject to formal revision before publication in the United States Reports. Readers are requested to notify the Reporter of Decisions, Supreme Court of the United States, Washington, D. C. 20543, pio@supremecourt.gov, of any typographical or other formal errors.</td></tr><tr><td>Case Information</td><td>SUPREME COURT OF THE UNITED STATES

No. 23–719

DONALD J. TRUMP, PETITIONER v.
NORMA ANDERSON, ET AL.
ON WRIT OF CERTIORARI TO THE SUPREME COURT
OF COLORADO
[March 4, 2024]</td></tr><tr><td>Background</td><td>A group of Colorado voters contends that Section 3 of the Fourteenth Amendment to the Constitution prohibits former President Donald J. Trump, who seeks the Presidential nomination of the Republican Party in this year’s election, from becoming President again. The Colorado Supreme Court agreed with that contention. It ordered the Colorado secretary of state to exclude the former President from the Republican primary ballot in the State and to disregard any write-in votes that Colorado voters might cast for him.</td></tr><tr><td>Legal Challenge</td><td>Former President Trump challenges that decision on several grounds. Because the Constitution makes Congress, rather than the States, responsible for enforcing Section 3 against federal officeholders and candidates, we reverse.</td></tr><tr><td>Case Proceedings and Findings</td><td>Last September, about six months before the March 5, 2024, Colorado primary election, four Republican and two unaffiliated Colorado voters filed a petition against former President Trump and Colorado Secretary of State Jena Griswold in Colorado state court. These voters—whom we refer to as the respondents—contend that after former President Trump’s defeat in the 2020 Presidential election, he disrupted the peaceful transfer of power by intentionally organizing and inciting the crowd that breached the Capitol as Congress met to certify the election results on January 6, 2021. One consequence of those actions, the respondents maintain, is that former President Trump is constitutionally ineligible to serve as President again.</td></tr><tr><td>Supreme Court&#39;s Analysis</td><td>This case raises the question whether the States, in addition to Congress, may also enforce Section 3. We conclude that States may disqualify persons holding or attempting to hold state office. But States have no power under the Constitution to enforce Section 3 with respect to federal offices, especially the Presidency.</td></tr><tr><td>Conclusion and Decision</td><td>For the reasons given, responsibility for enforcing Section 3 against federal officeholders and candidates rests with Congress and not the States. The judgment of the Colorado Supreme Court therefore cannot stand. All nine Members of the Court agree with that result.</td></tr><tr><td>Concurring Opinions</td><td>JUSTICE BARRETT, concurring in part and concurring in the judgment. I join Parts I and II–B of the Court’s opinion. I agree that States lack the power to enforce Section 3 against Presidential candidates. That principle is sufficient to resolve this case, and I would decide no more than that.

JUSTICE SOTOMAYOR, JUSTICE KAGAN, and JUSTICE JACKSON, concurring in the judgment. If it is not necessary to decide more to dispose of a case, then it is necessary not to decide more. That fundamental principle of judicial restraint is practically as old as our Republic.</td></tr></tbody></table>



------

## Prompt ingestion and filing

In [this script](https://github.com/antononcube/Raku-LLM-Prompts/blob/main/examples/Add-new-user-prompt.raku) 
we form a list of _two_ specifications for the prompts "ExtractArticleWisdom" and "FindHiddenMessage", and then we ingest and file them using a loop.
The procedure is general -- the rest of prompts (or patterns) in [DMr1] can be ingested and filed with the same code.

**Remark:** The ingestion and filing of new prompts is somewhat too technical and boring. Hence, we just link to an example script that does that. (Which has a fair amount of comments.)

Here we verify that the ingested prompts are ready for use:


```raku
.say for |llm-prompt-data(/^Extract | ^Find/)
```

    FindHiddenMessage => Finds hidden (propaganda) messages in texts
    ExtractArticleWisdom => Extracts ideas, quotes, and references from any text


-----

## Extract wisdom 

Here we use the [Fabric](https://github.com/danielmiessler/fabric/tree/main) 
prompt [extract_article_wisdom](https://github.com/danielmiessler/fabric/tree/main/patterns/extract_article_wisdom), [DMr1],
that summarizes and extracts ideas, quotes, and references (i.e. wisdom) from any text.

The prompt (as filed in "LLM::Prompts") is a function:


```raku
llm-prompt('ExtractArticleWisdom')
```




    -> $a = "" { #`(Block|6326612074312) ... }



But since its argument has a default value we can use it in "simple" LLM synthesis by just evaluating it without arguments:


```raku
llm-prompt('ExtractArticleWisdom')().substr(*-100,*-1)
```




    ing words.
    - Ensure you follow ALL these instructions when creating your output.
    
    # INPUT
    
    INPUT:
    




Here we synthesize an LLM answer based on that prompt:


```raku
my $fabSum = llm-synthesize([llm-prompt('ExtractArticleWisdom')(), $opinion], e => $conf);

text-stats($fabSum)
```




    (chars => 7465 words => 1092 lines => 71)



Here we render the LLM result above as Markdown formatted text:


```raku
#% markdown
$fabSum.subst( / ^^ '#' /, '###'):g;
```




### SUMMARY

The Supreme Court of the United States issued a ruling in the case DONALD J. TRUMP, PETITIONER v. NORMA ANDERSON, ET AL., reversing the Colorado Supreme Court's decision to exclude former President Donald J. Trump from the Republican primary ballot in Colorado for the 2024 presidential election. The case revolves around the application of Section 3 of the Fourteenth Amendment, which was argued to disqualify Trump due to his actions related to the January 6 Capitol breach.

### IDEAS:

- States cannot unilaterally enforce Section 3 of the Fourteenth Amendment to disqualify federal officeholders or candidates.
- The Supreme Court emphasizes the exclusive power of Congress, not individual states, to enforce Section 3 against federal candidates.
- Section 3 of the Fourteenth Amendment was designed to prevent individuals who engaged in insurrection against the United States from holding office.
- The decision reflects a concern about creating a "patchwork" of eligibility across states that could disrupt national elections.
- The Supreme Court's ruling showcases the delicate balance between state powers and the overarching authority of the federal government.
- The ruling underscores the importance of uniformity in determining the eligibility of presidential candidates across all states.
- The Fourteenth Amendment fundamentally altered the balance of state and federal power, expanding federal authority at the expense of state autonomy.
- The decision highlights the historical context of Section 3, intended to prevent former Confederates from returning to power after the Civil War.
- The Supreme Court's ruling illustrates the potential constitutional chaos that could result from varied state interpretations of federal officeholder qualifications.
- The opinion of the court acknowledges the necessity of a formal process to determine the application of Section 3 to specific individuals.

### QUOTES:
- "Because the Constitution makes Congress, rather than the States, responsible for enforcing Section 3 against federal officeholders and candidates, we reverse."
- "Section 3 aimed to prevent such a resurgence by barring from office those who, having once taken an oath to support the Constitution of the United States, afterward went into rebellion against the Government of the United States."
- "States have no power under the Constitution to enforce Section 3 with respect to federal offices, especially the Presidency."
- "The Constitution empowers Congress to prescribe how those determinations should be made."
- "Nothing in the Constitution delegates to the States any power to enforce Section 3 against federal officeholders and candidates."
- "Section 3 works by imposing on certain individuals a preventive and severe penalty—disqualification from holding a wide array of offices—rather than by granting rights to all."
- "The relevant provision is Section 5, which enables Congress, subject of course to judicial review, to pass 'appropriate legislation' to 'enforce' the Fourteenth Amendment."
- "The judgment of the Colorado Supreme Court therefore cannot stand."
- "Responsibility for enforcing Section 3 against federal officeholders and candidates rests with Congress and not the States."
- "All nine Members of the Court agree with that result."

### HABITS:
- The court meticulously analyzes historical context and legislative intent behind constitutional amendments.
- Engaging in a thorough examination of the balance between state and federal powers.
- Taking into consideration the potential national implications of state actions on federal elections.
- Prioritizing uniformity and coherence in the application of constitutional provisions across states.
- Demonstrating judicial restraint and caution in interpreting the scope of constitutional powers.
- Emphasizing the importance of formal processes and legislative action in enforcing constitutional disqualifications.
- Relying on precedents and historical practices to guide contemporary constitutional interpretation.
- Ensuring that decisions are grounded in a clear understanding of the Constitution's text and its framers' intentions.
- Collaborating as a unified bench to reach a consensus on nationally significant issues.
- Highlighting the necessity of judicial review in maintaining the constitutional balance between different branches of government and levels of governance.

### FACTS:
- Section 3 of the Fourteenth Amendment prohibits individuals who have engaged in insurrection or rebellion against the United States from holding office.
- The Supreme Court reversed the Colorado Supreme Court's decision to exclude Trump from the primary ballot, emphasizing Congress's exclusive enforcement role.
- The Fourteenth Amendment was proposed in 1866 and ratified by the States in 1868, expanding federal power and altering the balance of state and federal authority.
- The Enforcement Act of 1870, enacted by Congress, provided a mechanism to enforce Section 3 disqualifications.
- There is limited historical precedent for state enforcement of Section 3 against federal officeholders or candidates.
- Section 3 includes a provision allowing Congress to remove the disqualification by a two-thirds vote in each house.
- The Supreme Court's ruling was unanimous, with all nine justices agreeing on the outcome.
- The case raises questions about the potential chaos that could result from varied state interpretations of qualifications for federal officeholders.
- The Supreme Court's decision reflects concerns about maintaining uniformity in presidential eligibility across all states.
- The ruling underscores the importance of a formal process to ascertain the application of Section 3 to specific individuals.

### REFERENCES:
- The Fourteenth Amendment to the United States Constitution.
- The Enforcement Act of 1870.
- The Supreme Court's decision in DONALD J. TRUMP, PETITIONER v. NORMA ANDERSON, ET AL.
- Historical precedents and legislative actions related to Section 3 of the Fourteenth Amendment.
- The Colorado Supreme Court's decision that was reversed by the Supreme Court.

### RECOMMENDATIONS:
- States should defer to Congress regarding the enforcement of Section 3 of the Fourteenth Amendment against federal candidates.
- A uniform national approach is essential for determining the eligibility of presidential candidates to ensure fairness and consistency.
- Congress may consider clarifying the enforcement mechanisms for Section 3 to prevent future legal ambiguities.
- Future legislative actions related to Section 3 should ensure that they are congruent and proportional to the goals of preventing insurrection or rebellion.
- Voters and political parties should be aware of the constitutional qualifications and disqualifications for federal officeholders.
- Legal scholars should continue to explore the balance between state and federal powers in the context of election laws.
- Judicial review remains a critical tool for interpreting and applying constitutional provisions in contemporary contexts.
- Public awareness and education about the implications of the Fourteenth Amendment and Section 3 are essential for informed civic participation.
- Policymakers should consider the historical context and intent behind constitutional amendments when proposing related legislation.
- The legal community should monitor and analyze the impact of the Supreme Court's decision on future elections and candidate qualifications.



-----

## Hidden messages and propaganda

Here we use the [Fabric](https://github.com/danielmiessler/fabric/tree/main)
prompt [find_hidden_message](https://github.com/danielmiessler/fabric/tree/main/patterns/find_hidden_message), [DMr1],
that tries to find hidden messages (propaganda) in any text.

```raku
my $hidden = llm-synthesize([llm-prompt('FindHiddenMessage'), $opinion], e => $conf);

text-stats($hidden)
```




    (chars => 3678 words => 518 lines => 57)




```raku
#% markdown
$hidden.subst( / ^^ '#' /, '###'):g;
```




OVERT MESSAGE: Supreme Court reverses Colorado's decision to exclude Trump from ballot based on Section 3.

HIDDEN MESSAGE: The court aims to preserve federal control and avoid state-driven electoral chaos.

HIDDEN OPINIONS:
- Federal authority should override state decisions in presidential qualifications.
- States should not independently interpret constitutional disqualifications.
- Uniform national standards are crucial for presidential elections.
- The Fourteenth Amendment's enforcement is primarily Congress's responsibility.
- Judicial restraint is necessary in politically charged cases.
- The history of the Fourteenth Amendment supports federal oversight.
- State-driven electoral decisions could undermine national unity.
- The potential for varying state laws creates an unstable electoral process.
- The Supreme Court acts to prevent speculative future constitutional crises.
- Federalism principles must guide interpretations of presidential disqualifications.

SUPPORTING ARGUMENTS and QUOTES:
- The court emphasizes the unique national interest in presidential elections, quoting Anderson v. Celebrezze.
- The decision refers to the Fourteenth Amendment expanding federal power at the expense of state autonomy.
- The lack of historical precedent for state enforcement against federal officeholders supports the court's decision.
- The ruling highlights the potential chaos of conflicting state outcomes.
- The opinion notes the Reconstruction Amendments were designed to expand federal power, not state sovereignty.
- The court warns against a patchwork of state decisions affecting national elections.

DESIRED AUDIENCE OPINION CHANGE:
- Trust in the Supreme Court's ability to maintain electoral integrity.
- Recognize the importance of federal oversight in presidential qualifications.
- See the potential dangers in states having too much power in federal elections.
- Understand the historical context of the Fourteenth Amendment.
- Appreciate the need for judicial restraint in politically sensitive cases.
- Value uniformity in the electoral process across states.
- Acknowledge the complexity of constitutional enforcement.
- Support the role of Congress in enforcing the Fourteenth Amendment.
- Recognize the Supreme Court's role in preventing electoral chaos.
- Understand the balance between state rights and federal oversight in elections.

DESIRED AUDIENCE ACTION CHANGE:
- Support federal legislation clarifying presidential qualifications.
- Oppose state efforts to independently disqualify presidential candidates.
- Advocate for national standards in election laws.
- Encourage Congress to take a more active role in defining electoral qualifications.
- Urge state officials to defer to federal guidelines in election matters.
- Promote understanding of the Fourteenth Amendment's historical context.
- Encourage scholarly examination of federalism in the context of elections.
- Support judicial restraint in politically charged decisions.
- Advocate for legal reforms to prevent electoral inconsistencies.
- Encourage civic education on the balance of power in the U.S. electoral system.

MESSAGES: The Supreme Court wants you to believe it is preserving electoral integrity, but it is actually emphasizing federal control over states.

PERCEPTIONS: The Supreme Court wants you to believe it is cautious and non-partisan, but it's actually protective of federal authority and wary of state independence.

"NOTE: This AI is tuned specifically to be cynical and politically-minded. Don't take it as perfect. Run it multiple times and/or go consume the original input to get a second opinion.

(Slip Opinion)

Cite as: 601 U. S. ____ (2024)



------

## References

### Articles, slip opinions

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


[SCUS1] Supreme Court of the United States,
[No. 23-219, Donald J. Trump, petitioner, v. Norma Anderson, et al.](https://www.supremecourt.gov/opinions/23pdf/23-719_19m2.pdf),
(2024),
[Supreme Court of the United States Opinions](https://www.supremecourt.gov/opinions/opinions.aspx).

### Packages, repositories

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
[LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp5] Anton Antonov,
[LLM::Prompts](https://github.com/antononcube/Raku-LLM-Prompts) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov,
[Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package,
(2023-2024),
[GitHub/antononcube](https://github.com/antononcube).

[DMr1] Daniel Miessler, [Fabric](https://github.com/danielmiessler/fabric), (2024), [GitHub/danielmiessler](https://github.com/danielmiessler).


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
