# Генериране на код чрез DSL примери

Раку пакетът [DSL::Examples](https://raku.land/zef:antononcube/DSL::Examples?utm_source=chatgpt.com), [AAp1],
е „даннов пакет“ с примери за превод на DSL команди към програмен код.

Послучай 24-ти май аз реших да обогатя този пакет с примери на български и руски езици.

DSL примерите в пакета са подходящи за
[LLM few-shot обучение](https://www.prompthub.us/blog/the-few-shot-prompting-guide?utm_source=chatgpt.com).
Подпрограмата `llm-example-function`, предоставена от
[LLM::Functions](https://github.com/antononcube/Raku-LLM-Functions?utm_source=chatgpt.com), [AAp3],
може ефективно да се използва за създаване на функции за превод, използващи тези примери.

Използването на такива LLM-функции за превод е показано по-долу.
Също и в презентацията
[Robust LLM pipelines (Mathematica, Python, Raku)](https://youtu.be/QOsVTCQZq_s?utm_source=chatgpt.com):

* [Кратко въведение](https://youtu.be/QOsVTCQZq_s?t=89&utm_source=chatgpt.com)
* [Подробни обяснения](https://www.youtube.com/watch?v=QOsVTCQZq_s&t=2840s&utm_source=chatgpt.com)

Подобни преводи — с много по-малко изчислителни ресурси — се постигат с
граматично-базирани DSL преводачи; вижте
[DSL::Translators](https://github.com/antononcube/Raku-DSL-Translators?utm_source=chatgpt.com), [AAp2].
Пакетът
[LLM::Resources](https://github.com/antononcube/Raku-LLM-Resources?utm_source=chatgpt.com), [AAp4],
съдържа LLM-графи за генериране на код, които използват DSL примерите от този пакет.

---

## Примери за използване

Получаване на всички примери:

```raku
use DSL::Examples;
use Data::TypeSystem;

dsl-examples()
    ==> deduce-type()
```

```text
# Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 17), Assoc(Atom((Str)), Atom((Str)), 14), Assoc(Atom((Str)), Atom((Str)), 32), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 27), Assoc(Atom((Str)), Atom((Str)), 6)]), 7), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 10), Assoc(Atom((Str)), Atom((Str)), 26), Assoc(Atom((Str)), Atom((Str)), 17), Assoc(Atom((Str)), Atom((Str)), 20)]), 4), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 15), Assoc(Atom((Str)), Atom((Str)), 23), Assoc(Atom((Str)), Atom((Str)), 33), Assoc(Atom((Str)), Atom((Str)), 20)]), 4), Assoc(Atom((Str)), Tuple([Assoc(Atom((Str)), Atom((Str)), 15), Assoc(Atom((Str)), Atom((Str)), 10), Assoc(Atom((Str)), Atom((Str)), 20), Assoc(Atom((Str)), Atom((Str)), 6)]), 4)]), 4)
```

Табулиране на всички езици за превод и наличните примери за работни потоци:

```raku, results=asis
use Data::Translators;
dsl-examples(from => 'English').map({ $_.key X $_.value.keys }).flat(1).map({ <language workflow> Z=> $_ })».Hash.sort.Array
==> to-dataset()
==> to-html(field-names => <language workflow>)
```

| language | workflow             |
| -------- | -------------------- |
| Python   | LSAMon               |
| Python   | QRMon                |
| Python   | SMRMon               |
| Python   | pandas               |
| R        | DataReshaping        |
| R        | LSAMon               |
| R        | QRMon                |
| R        | SMRMon               |
| Raku     | DataReshaping        |
| Raku     | LSAMon               |
| Raku     | SMRMon               |
| Raku     | TriesWithFrequencies |
| WL       | ClCon                |
| WL       | DataReshaping        |
| WL       | LSAMon               |
| WL       | QRMon                |
| WL       | SMRMon               |
| WL       | Tabular              |
| WL       | TriesWithFrequencies |

Обърнете внимание, че в `dsl-examples` се задава езикът, от който се превежда.
В момента пакетът съдържа DSL примери за български, английски и руски език (като изходни езици).

Получаване на примерите за сегменти от монадични конвейери за латентен семантичен анализ (**LSA**) в Python:

```raku
dsl-examples('Python', 'LSAMon')
    ==> deduce-type(:tally)
```

```text
# Assoc(Atom((Str)), Atom((Str)), 15)
```

Създаване на LLM функция-пример за превод на команди за изграждане на LSA конвейр:

```raku
use LLM::Functions;
my &llm-pipeline-segment = llm-example-function(dsl-examples()<WL><LSAMon>);
```

Изпълнение на LLM функцията върху списък от DSL команди:

```raku
my @commands = 
"use the dataset aAbstracts",
"make the document-term matrix without stemming",
"exract 40 topics using the method non-negative matrix factorization",
"show the topics";

@commands
.map({ .&llm-pipeline-segment })
.map({ .subst(/:i Output ':'?/):g })
.join("⟹\n")
```

```text
# LSAMonUnit[aAbstracts]⟹
# LSAMonMakeDocumentTermMatrix["StemmingRules"->{},"StopWords"->Automatic]⟹
# LSAMonExtractTopics["NumberOfTopics" -> 40, Method -> "NNMF"]⟹
# LSAMonEchoTopicsTable[]
```

Същият работен поток, зададен на български:

```raku
my &llm-pipeline-segment-bg = llm-example-function(dsl-examples(from => 'Bulgarian')<WL><LSAMon>);

my @commands = 
"използавай данните aAbstracts",
"направи документ-терм матрицата без да използаваш стъблата на думите",
"намери 40 теми ползвайки методата не-отрицателна матрична факторизация",
"покажи темите";

@commands
.map({ .&llm-pipeline-segment-bg })
.map({ .subst(/:i Output ':'?/):g })
.join("⟹\n")
```

```text
# LSAMonUnit[aAbstracts]⟹
# LSAMonMakeDocumentTermMatrix["StemmingRules"->{}]⟹
# LSAMonExtractTopics["NumberOfTopics"->40,Method->"NNMF"]⟹
# LSAMonEchoTopicsTable[]
```

---

## Детайли по реализацията

Съществуват няколко начина за организиране на DSL примерите по отношение на изходните езици:

| Тип                                                                                                  | Коментар                                    | Използва се в момента         |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------- | ----------------------------- |
| Отделен файл за всеки изходен език                                                                   | Удобно редактиране и усъвършенстване        | Да                            |
| Един файл с всички примери; изходният език е ключ за всеки работен поток                             | Може да бъде генериран от отделните файлове | Не                            |
| Запазване само на английски DSL примери и използване на речници за превод на командите към английски | Не обучава директно LLM с изходния език     | Речниците се пазят за справка |

---

## Препратки

### Пакети

[AAp1] Anton Antonov,
[DSL::Examples Raku package](https://github.com/antononcube/Raku-DSL-Examples?utm_source=chatgpt.com),
(2024-2026),
[GitHub/antononcube](https://github.com/antononcube?utm_source=chatgpt.com).

[AAp2] Anton Antonov,
[DSL::Translators Raku package](https://github.com/antononcube/Raku-DSL-Translators?utm_source=chatgpt.com),
(2020-2026),
[GitHub/antononcube](https://github.com/antononcube?utm_source=chatgpt.com).

[AAp3] Anton Antonov,
[LLM::Functions Raku package](https://github.com/antononcube/Raku-LLM-Functions?utm_source=chatgpt.com),
(2023-2026),
[GitHub/antononcube](https://github.com/antononcube?utm_source=chatgpt.com).

[AAp4] Anton Antonov,
[LLM::Resources Raku package](https://github.com/antononcube/Raku-LLM-Resources?utm_source=chatgpt.com),
(2026),
[GitHub/antononcube](https://github.com/antononcube?utm_source=chatgpt.com).

### Видеа

[AAv1] Anton Antonov,
[Robust LLM pipelines (Mathematica, Python, Raku)](https://youtu.be/QOsVTCQZq_s?utm_source=chatgpt.com),
(2024),
[YouTube/AAA4prediction](https://www.youtube.com/@AAA4prediction?utm_source=chatgpt.com).
