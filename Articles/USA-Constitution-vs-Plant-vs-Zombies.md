# US Constitution vs Plants vs Zombies

Anton Antonov   
[RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com)    
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book)   
February 2024

-------

## Introduction

In this notebook we ingest the text of the [US Constitution](https://en.wikipedia.org/wiki/Constitution_of_the_United_States), [Wk1], and use it as a part of different Large Language Model (LLM) requests, like, summaries and most important statements.

We also use LLM to derive analogies with the game ["Plants vs Zombies"](https://en.wikipedia.org/wiki/Plants_vs._Zombies), [Wk2].

The visualization of those analogies is an interesting task, and we apply OpenAI's model DALL-E 3 to generate (mostly) relevant images. (Following [AA2].)

At this point, the idea to apply LLMs to the US Constitution should have ocurred in many (human) heads. For example, see the GitHub repository ["ConstitutionGPT"](https://github.com/vdutts7/constitutionGPT), [VD7r1]. (Also see these [search results](https://github.com/topics/us-constitution).)

The US Constitution is popular, well known document, hence, it is expected LLMs to be able to "converse about it" without providing its full text in the prompts. Nevertheless, below we do ingest the full text of the US Constitution and include in our LLM interactions. That makes the presented LLM workflows (fairly) text independent.

**Remark:** Very similar LLM workflows are presented in the article ["LLM aids for processing of the first Carlson-Putin interview"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/), [AA3, AAn2].

------

## Setup

Here we load certain packages and define a text statistics function:


```raku
use HTTP::Tiny;
use JSON::Fast;

multi sub text-stats(Str:D $txt) { <chars words lines> Z=> [$txt.chars, $txt.words.elems, $txt.lines.elems] }
multi sub text-stats(Hash:D $txt) { text-stats($txt.Str); }
```




    &text-stats



Here we derive a configuration object for accessing OpenAI's LLM service with one of its latest ChatGPT models:


```raku
my $conf = llm-configuration('ChatGPT', model => 'gpt-4-turbo-preview', max-tokens => 4096, temperature => 0.2);
$conf.Hash.elems
```




    21



------

## Get text

Here we make an LLM request for the authoritative source of the USA constitution text:


```raku
llm-synthesize(["Give the most athoritative of URL for US constitution in plain text."])
```




    https://www.archives.gov/founding-docs/constitution



Here we get the HTML text of the US constitution transcript from the ["National Archives" site](https://www.archives.gov/founding-docs/):


```raku
my $url0 = 'https://www.archives.gov/founding-docs/constitution-transcript';
my $url1 = 'https://www.archives.gov/founding-docs/bill-of-rights-transcript';
my $txtUSACon0 = HTTP::Tiny.new.get($url0)<content>.decode;
$txtUSACon0 .= subst(/ \v+ /, "\n", :g);
say 'Constitution transcript   : ',text-stats($txtUSACon0);

my $txtUSACon1 = HTTP::Tiny.new.get($url1)<content>.decode;
$txtUSACon1 .= subst(/ \v+ /, "\n", :g);
say 'Bill of Rights transcript : ', text-stats($txtUSACon1);
```

    Constitution transcript   : (chars => 67406 words => 6634 lines => 765)
    Bill of Rights transcript : (chars => 43064 words => 3372 lines => 589)


Those variables with HTML texts, though. Instead of trying to figure out how to use Raku packages for stripping HTML code, we can just find plain text files of the US constitution from other sources. Here we get the text of the US constitution from https://www.usconstitution.net :


```raku
my $url = 'https://www.usconstitution.net/const.txt';
my $txtUSACon = HTTP::Tiny.new.get($url)<content>.decode;
$txtUSACon .= subst(/ \v+ /, "\n", :g);
$txtUSACon .= subst(/ ^ .* <?before 'We the People of the United States'>/);
text-stats($txtUSACon)
```




    (chars => 44674 words => 7620 lines => 698)



Here we get the US constitution content in JSON format from [MBr1] and then convert into a Raku data structure:


```raku
my $urlJSON = 'https://raw.githubusercontent.com/markbiek/constitution.json/master/constitution.json';
my $txtUSAConJSON = HTTP::Tiny.new.get($urlJSON)<content>.decode;
my %txtUSAConJSON = from-json($txtUSAConJSON);
text-stats(%txtUSAConJSON)
```




    (chars => 44211 words => 7514 lines => 120)



**Remark:** The repository [MBr1] refers to https://www.usconstitution.net/const.txt as a source to derive the JSON file from.


```raku
%txtUSAConJSON.keys
```

-------

## Summaries

Here is summary of the whole document:


```raku
#% markdown
llm-synthesize(["Summarize the document in less than 300 words:", $txtUSACon], e => $conf)
```




The document is the Constitution of the United States, which outlines the framework for the federal government, including its three branches: legislative, executive, and judicial. It begins with the Preamble, stating the purpose of the document to ensure justice, peace, defense, welfare, and liberty for the nation and its future generations. 

Article 1 establishes the legislative branch, Congress, divided into the Senate and House of Representatives, detailing their composition, powers, and operational rules. It grants Congress the power to legislate on various matters, including taxes, defense, commerce, and the creation of laws necessary for executing its powers.

Article 2 sets forth the executive branch, headed by the President, outlining the process for election, qualifications, powers, and duties, including serving as the Commander in Chief, making treaties with Senate approval, and appointing officials.

Article 3 establishes the judicial branch, led by the Supreme Court, and allows for lower courts. It defines the judiciary's powers, including jurisdiction over various cases and the trial process, especially for crimes and treason.

Articles 4 through 7 address the states' rights and relations, the process for amending the Constitution, the federal government's supremacy over state laws, and the ratification process of the Constitution.

The document also includes 27 amendments, the first ten known as the Bill of Rights, which protect individual freedoms such as speech, religion, and the press; the right to bear arms; protection against unreasonable searches and seizures; and rights to fair trials. Subsequent amendments address issues including the abolition of slavery (13th), citizenship and equal protection under the law (14th), voting rights regardless of race (15th), income tax (16th), direct election of senators (17th), prohibition and its repeal (18th and 21st), women's suffrage (19th), presidential term limits (22nd), voting age (26th), and congressional pay changes (27th).



Here we summarize Bill of Rights:


```raku
#% markdown
llm-synthesize(["Summarize in less than 300 words the Bill of Rights given in the following text:", $txtUSACon], e => $conf)
```




The Bill of Rights, the first ten amendments to the United States Constitution, guarantees fundamental civil rights and liberties. 

1. **First Amendment**: Protects freedom of religion, speech, press, assembly, and the right to petition the government.
2. **Second Amendment**: Ensures the right of individuals to keep and bear arms.
3. **Third Amendment**: Prohibits the quartering of soldiers in private homes without the owner's consent in peacetime.
4. **Fourth Amendment**: Guards against unreasonable searches and seizures, requiring any warrant to be judicially sanctioned and supported by probable cause.
5. **Fifth Amendment**: Provides the right to due process and prohibits self-incrimination and double jeopardy. It also contains the takings clause, which ensures that private property is not taken for public use without just compensation.
6. **Sixth Amendment**: Guarantees the rights of criminal defendants, including the right to a public trial without unnecessary delay, the right to a lawyer, the right to an impartial jury, and the right to know who your accusers are and the nature of the charges and evidence against you.
7. **Seventh Amendment**: Provides for the right to trial by jury in certain civil cases, according to common law.
8. **Eighth Amendment**: Prohibits excessive fines and excessive bail, as well as cruel and unusual punishment.
9. **Ninth Amendment**: Asserts that the enumeration of specific rights in the Constitution does not exclude other rights possessed by the people.
10. **Tenth Amendment**: States that the powers not delegated to the United States by the Constitution, nor prohibited by it to the States, are reserved to the States respectively, or to the people.

These amendments were ratified on December 15, 1791, and form a critical foundation of American freedoms.



-------

## Structures and relationships

### Powers and restrictions per governmental branch

From reading the summaries above we can come up with LLM requests to obtain a structured (e.g. tabular) outline of document's content. Here is one a such request, in which the powers and restrictions of the governmental branches are tabulated: 


```raku
#% markdown
llm-synthesize(["Cross-tabulate the powers and restrictions three branches: legislative, executive, and judicial. The table rows should correspond to the branches.", $txtUSACon], e => $conf)
```




Given the extensive text provided, which encompasses the Constitution of the United States and its amendments, a comprehensive cross-tabulation of the powers and restrictions of the three branches of government (Legislative, Executive, and Judicial) as outlined in the Constitution would be quite detailed. Below is a simplified version focusing on key powers and restrictions for each branch.

| Branch        | Powers                                                                                          | Restrictions                                                                                   |
|---------------|-------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Legislative   | - Make laws<br>- Tax and spend for the defense & general welfare<br>- Declare war<br>- Regulate commerce<br>- Establish lower federal courts<br>- Impeach and remove President and federal judges | - Cannot suspend the writ of habeas corpus except in cases of rebellion or invasion<br>- No ex post facto laws or bills of attainder<br>- No title of nobility |
| Executive     | - Enforce laws<br>- Act as Commander-in-Chief of the armed forces<br>- Make treaties (with Senate approval)<br>- Appoint federal officers and judges (with Senate approval)<br>- Grant pardons (except in cases of impeachment) | - Cannot make laws<br>- Treaties and major appointments require Senate approval<br>- Subject to impeachment and removal by Congress                  |
| Judicial      | - Interpret laws<br>- Review the constitutionality of laws<br>- Cases involving laws and treaties of the U.S., ambassadors, maritime law, disputes between states or between a state and the federal government | - Judges serve during "good behavior" but can be impeached and removed<br>- Cannot enforce its decisions, relies on the executive branch<br>- Cannot make or enact laws |

This table simplifies a complex system of checks and balances designed to ensure no single branch of the U.S. government gains too much power. Each branch has specific roles and can influence or limit the powers of the others, maintaining a balance that supports the principles of democracy and the rule of law.



### Amendments vs Articles

In this subsection we use LLM requests to tabulate the relationships between the concepts in the Constitution and the Bill of Rights.


```raku
#% html
my $k=1;
my $articles = %txtUSAConJSON<articles>.map({ "Article {$k++}:\n" ~ $_}).join("\n\n");

my $l=1;
my $amendments = %txtUSAConJSON<amendments>.map({ "Amendment {$l++}:\n" ~ $_}).join("\n\n");

my $res = llm-synthesize([
    "Make a Morphological Analysis table of relationships between the articles and amendments.",
    "The rows are the amendments; the columns are the articles.",
    "A table cells corresponding to a relationship should have a concise description of that relationship.",
    "Articles:\n" ~ $articles,
    "Amendments:\n" ~ $amendments,
    llm-prompt('NothingElse')('HTML table')
], e => $conf);

$res.subst(/^ '```html' | '```' $/):g
```





<table border="1">
  <tr>
    <th>Amendments / Articles</th>
    <th>Article 1</th>
    <th>Article 2</th>
    <th>Article 3</th>
    <th>Article 4</th>
    <th>Article 5</th>
    <th>Article 6</th>
  </tr>
  <tr>
    <td>Amendment 1</td>
    <td>Establishes limits on Congress' legislative powers</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 2</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 3</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 4</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 5</td>
    <td></td>
    <td></td>
    <td>Defines rights in criminal cases, due process</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 6</td>
    <td></td>
    <td></td>
    <td>Defines rights to a fair trial</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 7</td>
    <td></td>
    <td></td>
    <td>Defines rights in civil cases</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 8</td>
    <td></td>
    <td></td>
    <td>Defines rights against excessive bail, fines, and cruel punishment</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 9</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 10</td>
    <td>Reserves powers not delegated to the federal government to the states or people</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 11</td>
    <td></td>
    <td></td>
    <td>Limits judicial power in suits against states</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 12</td>
    <td></td>
    <td>Modifies the procedure for electing the President and Vice President</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 13</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 14</td>
    <td>Defines citizenship, contains the Privileges or Immunities Clause, the Due Process Clause, the Equal Protection Clause</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 15</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 16</td>
    <td>Grants Congress the power to tax income</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 17</td>
    <td>Changes the process for electing Senators</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 18</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 19</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 20</td>
    <td></td>
    <td>Changes the dates of congressional and presidential terms</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 21</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 22</td>
    <td></td>
    <td>Limits the President to two terms</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 23</td>
    <td></td>
    <td>Grants the District of Columbia electors in the Electoral College</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 24</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 25</td>
    <td></td>
    <td>Details the succession to the Presidency and the Vice Presidency</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 26</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Amendment 27</td>
    <td>Delays laws affecting Congressional salary from taking effect until after the next election of Representatives</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>




------

## Most important statements

Here we define a function for extracting most important elements for text:


```raku
my &fImp = llm-function({"Which are the top $^a most $^b of the following document:\n\n$^c"}, e => $conf)
```




    -> **@args, *%args { #`(Block|4971098499408) ... }



Here are the most important statements:


```raku
#% markdown
&fImp(3, 'important statements', $txtUSACon)
```




Given the extensive nature of the document you've provided, which is essentially the United States Constitution along with its amendments, identifying the "top 3 most important statements" can be subjective and dependent on the criteria used for importance. However, considering the foundational principles and rights it establishes, the following can be considered among the most significant statements:

1. **Preamble to the Constitution**: "We the People of the United States, in Order to form a more perfect Union, establish Justice, insure domestic Tranquility, provide for the common defence, promote the general Welfare, and secure the Blessings of Liberty to ourselves and our Posterity, do ordain and establish this Constitution for the United States of America." This statement encapsulates the purpose and guiding principles behind the establishment of the Constitution and the government it creates.

2. **First Amendment**: "Congress shall make no law respecting an establishment of religion, or prohibiting the free exercise thereof; or abridging the freedom of speech, or of the press; or the right of the people peaceably to assemble, and to petition the Government for a redress of grievances." This amendment is crucial as it protects fundamental freedoms such as religion, speech, press, assembly, and petition, which are essential to a democratic society.

3. **Fourteenth Amendment, Section 1**: "All persons born or naturalized in the United States, and subject to the jurisdiction thereof, are citizens of the United States and of the State wherein they reside. No State shall make or enforce any law which shall abridge the privileges or immunities of citizens of the United States; nor shall any State deprive any person of life, liberty, or property, without due process of law; nor deny to any person within its jurisdiction the equal protection of the laws." This amendment addresses citizenship rights and equal protection under the law and is foundational to civil rights in the United States.

These statements are pivotal as they lay the groundwork for the governance, rights, and freedoms of the United States, impacting not only the legal framework but also the societal and cultural ethos of the nation.



----- 

## Most important amendments

Most important amendments:


```raku
#% markdown
&fImp(3, 'important amendments', $txtUSACon)
```




Determining the "top 3 most important amendments" from the United States Constitution can be subjective and depends on the criteria used for evaluation (e.g., impact on civil liberties, structural changes to government, etc.). However, based on their broad implications for civil rights and liberties, the following three amendments are often highlighted for their significance:

1. **First Amendment**: Protects several fundamental rights, including freedom of speech, religion, press, assembly, and the right to petition the government for a redress of grievances. This amendment is foundational to the American democratic system and the protection of individual liberties.

2. **Fourteenth Amendment**: Addresses citizenship rights and equal protection under the law and was pivotal in extending civil rights protections. Its clauses, particularly the Equal Protection Clause and the Due Process Clause, have been the basis for many landmark Supreme Court decisions that have expanded civil rights and liberties.

3. **Nineteenth Amendment**: Prohibits the states and the federal government from denying the right to vote to citizens of the United States on the basis of sex. This amendment marked a significant milestone in the women's suffrage movement in the United States, ensuring women the right to vote and advancing gender equality.

These amendments are among the most cited and have had profound impacts on American society, law, and governance.



------

## Analogies with "Plants vs Zombies"


In this section we use LLMs to explain the elements of US constitution to people who are familiar with the game ["Plants vs Zombies"](https://en.wikipedia.org/wiki/Plants_vs._Zombies), [Wk2].


```raku
#% markdown
llm-synthesize(["Explain the Bill of Rights described in the following text to someone who has played 'Plants vs Zombies':", $txtUSACon], e => $conf)
```




Alright, let's break this down into something more relatable to a "Plants vs. Zombies" player. Imagine the United States Constitution and its Bill of Rights as the ultimate game guide for running a country, where the goal is to ensure a balanced, fair, and free society, much like strategizing to protect your garden from zombies.

**Preamble (Introduction):** This is like the game's intro screen, setting the stage. It tells us why the Constitution was created: to make a better "team" (union), ensure peace at home, provide defense, improve everyone's well-being, and guarantee freedom for now and the future.

**Article 1: Legislative Branch (Congress)** - Think of Congress as two different types of plants in your garden, the Senate and the House of Representatives, each with unique powers and responsibilities. They make the rules (laws) of the game, deciding how resources are distributed, how to defend against threats, and how to keep the garden (country) running smoothly.

**Article 2: Executive Branch (President and Administration)** - This is like choosing your main plant or hero, the President, who executes the laws made by Congress, leads the military, and deals with other gardens (countries). The President has a team (administration) to help manage specific areas, like defense or agriculture.

**Article 3: Judicial Branch (Supreme Court and other Federal Courts)** - Imagine this as the game's referees or rule-checkers. They interpret the laws, settle disputes, and ensure that everything aligns with the game guide (Constitution). They make sure everyone plays fair.

**Bill of Rights (First 10 Amendments):** These are like the essential power-ups or protections for the players (citizens) to ensure they can play the game freely and fairly.

1. **Freedom Power-Up:** You can speak, write, gather, and believe freely.
2. **Defense Boost:** You have the right to hold onto and use tools (in this case, weapons) for protection.
3. **Privacy Shield:** Soldiers can't just crash at your place without permission.
4. **Anti-Spyware:** Your personal space and belongings can't be searched or taken without a good reason.
5. **Fair Play Protocol:** You have rights in legal situations, like not having to incriminate yourself and being treated fairly by the law.
6. **Rapid Response:** In legal battles, you're guaranteed a quick and public trial, with a chance to defend yourself.
7. **Trial by Jury:** For serious disputes in court, you can have a jury decide.
8. **No Cheat Codes:** Punishments can't be overly harsh or unfair.
9. **Hidden Levels:** Just because a right isn't listed here doesn't mean you don't have it.
10. **Local Control:** Powers not given to the federal government belong to the states or the people.

The rest of the amendments (11-27) are like updates or expansions to the game, adding new features, fixing bugs, or adjusting the gameplay to make sure it stays fair and fun for everyone playing.

So, just like in "Plants vs. Zombies," where you need to know your plants, zombies, and strategies to win, the Constitution and the Bill of Rights lay out the rules, roles, and protections to ensure the game of governance is played effectively and fairly for all citizens.



------ 

## Supporting illustrations

Let us try to generate the illustration images that correspond to the USA Constitution vs Plants vs Zombies analogies.
First attempt with "simple" prompt:


```raku
#% dalle, size=landscape
Make a illustration of the analogies between the USA Constitution and the game "Plants vs Zombies."
```

Here we request the image of DALL-E’s response to be given as a URL and the result of the chatbook cell to be given in JSON format:


```raku
#% dalle, model=dall-e-3, size=landscape, style=vivid, response-format=url, format=json
Create an image of a large, ornate, classical room that resembles the interior of a government assembly hall, with rows of desks and American flags hanging from the balconies.
The floor of the hall is made of tiles that are pages of the USA Constitution.
In the center of the room, place a giant cartoon-style plant character from the game "Plants vs Zombies", with a large green head that resembles a simplified, friendly face. 
Surround the plant with multiple miniature versions of the same character, all engaged in chaotic activity with human figures in various formal and informal attire. 
The humans and the plant characters appear to be having an epic confrontation, with some characters attacking the giant wielding weapons, and others gesturing energetically. 
```




{
  "data": [
    {
      "revised_prompt": "Create an image depicting a spacious, richly decorated space reminiscent of a traditional legislative hall, with staggered seats and a number of sheets of cloth patterned in the red, white, and blue of the American flag draping from overhead structures. The flooring of the chamber is constructed from stone plates, each engraved with sections of the United States Constitution. In the epicenter of the room, position a colossal, cartoonish flora entity akin to a character from a strategic horticulture-themed game, characterized by a large green head that mirrors a simplified, affable countenance. Surround this entity with numerous, smaller iterations of the same character, all caught in various stages of frenzied activity with multiethnic men and women in a combination of formal suits and casual clothing. The scene captures an intense standoff between the humans and plant entities, with some figures wielding weapon-like props and others gesticulating with vigor.",
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-KbuLSsqssXAPQFZORGWZzuN0/user-Ss9QQAmz9L5UJDcmKnhxnRoT/img-5tql5Og2vVPQMxiQASDiQKi6.png?st=2024-02-24T19%3A03%3A01Z&se=2024-02-24T21%3A03%3A01Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2024-02-24T10%3A28%3A55Z&ske=2024-02-25T10%3A28%3A55Z&sks=b&skv=2021-08-06&sig=xX3PpROBT/fyvqEAvqPrP8tTuQ913iDOgAyVrI55F/I%3D"
    }
  ],
  "created": 1708804981
}



Here we display the image directly as a Markdown image link (using the command paste of “Clipboard“):


```raku
#% markdown
my $url = from-json(paste)<data>.head<url>;
"![]($url)"
```

-------

## Meta cells

Here we use a DALL-E meta cell to see how many images were generated in this session:


```raku
#% dalle meta
elems
```




    2



Here we export all images generated during the sessions into file names with the prefix “USConstitutionVsPlantsVsZombies":


```raku
#% dalle export, index=all, prefix=USConstitutionVsPlantsVsZombies
```

------

## References

### Articles

[AA1] Anton Antonov, ["Workflows with LLM functions"](https://rakuforprediction.wordpress.com/2023/08/01/workflows-with-llm-functions/) , (2023), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[AA2] Anton Antonov, ["Day 21 - Using DALL-E models in Raku"](https://raku-advent.blog/2023/12/21/day-22-using-dall-e-models-in-raku/) , (2023), [Raku Advent Calendar blog for 2023](https://raku-advent.blog/2023).

[AA3] Anton Antonov, ["LLM aids for processing of the first Carlson-Putin intervie"](https://rakuforprediction.wordpress.com/2024/02/12/llm-aids-for-processing-of-the-first-carlson-putin-interview/), (2024), [RakuForPrediction at WordPress](https://rakuforprediction.wordpress.com).

[Wk1] Wikipedia entry, ["Constitution of the United States"](https://en.wikipedia.org/wiki/Constitution_of_the_United_States).

[Wk2] Wikipedia entry, ["Plants vs Zombies"](https://en.wikipedia.org/wiki/Plants_vs._Zombies).

### Notebooks

[AAn1] Anton Antonov, ["Workflows with LLM functions (in Raku)"](https://community.wolfram.com/groups/-/m/t/2982320), (2024), [Wolfram Community](https://community.wolfram.com/).

[AAn2] Anton Antonov, ["LLM aids for processing of the first Carlson-Putin interview"](https://community.wolfram.com/groups/-/m/t/3121333), (2024), [Wolfram Community](https://community.wolfram.com/).



### Packages

[AAp1] Anton Antonov, [WWW::OpenAI](https://github.com/antononcube/Raku-WWW-OpenAI) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp2] Anton Antonov, [WWW::PaLM](https://github.com/antononcube/Raku-WWW-PaLM) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp3] Anton Antonov, [WWW::MistralAI](https://github.com/antononcube/Raku-WWW-MistralAI) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp4] Anton Antonov, [WWW::MermaidInk](https://github.com/antononcube/Raku-WWW-MermaidInk) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp5] Anton Antonov, [LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp6] Anton Antonov, [Jupyter::Chatbook](https://github.com/antononcube/Raku-Jupyter-Chatbook) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

[AAp7] Anton Antonov, [Image::Markup::Utilities](https://github.com/antononcube/Raku-Image-Markup-Utilities) Raku package, (2023), [GitHub/antononcube](https://github.com/antononcube) .

### Repositories

[MBr1] Mark Biek, 
[constitution.json](https://github.com/markbiek/constitution.json),
(2016-2018),
[GitHub/markbiek](https://github.com/markbiek).

[VD7r1] vdutts7,
[Constitution GPT](https://github.com/vdutts7/constitutionGPT),
(2023),
[GitHub/vdutts7](https://github.com/vdutts7).

### Videos

[AAv1] Anton Antonov, ["Jupyter Chatbook LLM cells demo (Raku)"](https://www.youtube.com/watch?v=cICgnzYmQZg) (2023), [YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction) .

[AAv2] Anton Antonov, ["Jupyter Chatbook multi cell LLM chats teaser (Raku)"](https://www.youtube.com/watch?v=wNpIGUAwZB8) , (2023), [YouTube/@AAA4Prediction](https://www.youtube.com/@AAA4prediction) .

[AAv3] Anton Antonov ["Integrating Large Language Models with Raku"](https://www.youtube.com/watch?v=-OxKqRrQvh0) , (2023), [YouTube/@therakuconference6823](https://www.youtube.com/@therakuconference6823) .
