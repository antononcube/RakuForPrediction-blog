# Introdução às disputas de dados com Raku

**Versão 1.0***

Anton Antonov   
[RakuForPrediction at GitHub](https://github.com/antononcube/RakuForPrediction-book)   
Dezembro 2021   
Janeiro 2023

## Introdução

Uma das minhas missões de vida atual é a aceleração da próxima [inverno AI] (https://en.wikipedia.org/wiki/AI_winter) que se aproxima.
Eu decidi usar Raku para cumprir essa missão. (Bem, pelo menos no início...)

Também acho o termo Inteligência Artificial (IA) uma brilhante frase de marketing para extrair dinheiro do
Complexo militar americano e todos os tipos de investidores financeiros. Quando as pessoas que afirmam ser, digamos, "profissionais da ciência de dados"
usá-lo em ambientes profissionais, fico altamente desconfiado de seu "profissionalismo".
(Ou seja, eles são muito provavelmente alguns aspirantes a profissionais sem pistas).

De volta ao próximo inverno AI que se aproxima - aqui está meu plano:

1. Ensinar muitas pessoas [como ser impostores dos cientistas de dados] (https://github.com/antononcube/HowToBeADataScientistImpostor-book)

2. Produzir [ferramentas](https://conf.raku.org/talk/157) que facilitem os baristas com código AI

3. Esperar que um número suficiente de pessoas com AI atinja paredes de AI suficiente.

4. Próximo inverno de gripe aviária

5. Lucro

Veja a apresentação ["Raku for Prediction"](https://conf.raku.org/talk/157), [AAv2], que descreve meus esforços sobre o Ponto 2 usando Raku.

Uma versão longa do Ponto 3 é "esperar que investidores e gerentes de AI batam em muitas paredes usando uma grande,
massa sem talento de praticantes de AI".

O lucro no ponto 5 vem do "campo" ser "claro", portanto, menos competição para obter financiamento e investimentos.
(Mesmo após este esclarecimento, alguns ainda podem pensar que minha lista acima é muito parecida com a que foi feita pelos gnomos em
[Gnomos" do Parque Sul](https://en.wikipedia.org/wiki/Gnomes_(South_Park));
e por mim tudo bem).

A maioria dos cientistas de dados passa a maior parte de seu tempo fazendo aquisição de dados e disputas de dados.
Não a ciência dos dados, ou IA, ou qualquer trabalho "realmente aprendido".
Portanto, se eu estou falando sério sobre influenciar as curvas de evolução da IA, então eu devo levar a sério sobre influenciar
aquisição de dados e derivações de incantações de dados em diferentes linguagens de programação; [AAv2].
Uma vez que acredito firmemente que é bom comer ocasionalmente seu próprio alimento para cães,
Eu programei (nas últimas semanas) pacotes de dados em Raku.
Quem sabe, isso pode ser uma maneira de Rakunizar IA, e...
[para reformular Larry Wall](https://en.wikipedia.org/wiki/Raku_(programming_language)#História)
-- alguns usuários podem conseguir sua correção. (Veja também [FB1].)

Quanto à aquisição de dados -- eu tenho um
[Projeto do Motor de Aquisição de Dados](https://github.com/antononcube/Data-Acquisition-Engine-project),
[AAr1, AAv6],
que tem um agente conversador que usa a geração de código através de um pacote Raku, [AAp7].
A fim de discutir e exemplificar a disputa de dados, temos que utilizar certas funcionalidades de aquisição de dados.
Devido a isso, abaixo são dadas explicações e exemplos de utilização de pacotes Raku para a recuperação de dados populares,
bem conhecidos e para a geração de conjuntos de dados aleatórios.

Este documento é bastante técnico - os leitores podem simplesmente ler ou folhear a próxima seção e a seção
"Fazê-lo como um Cro" e seja feito. Alguns talvez queiram olhar e folhear a versão super-técnica
"Disputa de dados com Raku", [AA1].

**Observação:** Ocasionalmente o código abaixo pode ter a expressão Raku `===>encode-to-wl()`.
Isto é para a serialização de objetos Raku em
[Wolfram Language (WL)](https://en.wikipedia.org/wiki/Wolfram_Language)
expressões.
(Este documento foi escrito como um caderno [Mathematica](https://en.wikipedia.org/wiki/Wolfram_Mathematica)).

**Observação:** O público alvo deste documento consiste principalmente de pessoas expostas às culturas e cultos de Perl e Raku.
Mas a maior parte deste documento deve ser acessível e de interesse para os programadores ou cientistas de dados sem perspicácia.

**Observação:** Como pode ser visto na (longa) gravação da apresentação
["Agente conversador de aquisição de dados em vários idiomas (versão estendida)"](https://www.youtube.com/watch?v=KlEl2b8oxb8),
[AAv6], um agente de conversação de aquisição de dados Raku pode alavancar e utilizar grandemente as capacidades de negociação de dados Raku.

**Observação:** Sentenças com declarações, afirmações e códigos "universais", verificáveis ou reprodutíveis usam o "nós formamos".
As opiniões pessoais dos autores e as declarações de decisões utilizam o "formulário I".
Alternativamente, eu apenas usei o formulário que me pareceu mais fácil ou mais natural de escrever.

### Glossário

Este documento é uma tradução (do inglês para o búlgaro) do documento
["Introduction to Data Wrangling with Raku"](https://rakuforprediction.wordpress.com/2021/12/31/introduction-to-data-wrangling-with-raku/).

A tradução de alguns conceitos e frases do inglês para o portugues é, talvez, controversa.
(E, talvez, francamente irritante para alguns leitores.)
Por esse motivo, fiz um glossário de palavras e frases técnicas nesta subseção.

- Conjunto de dados : Dataset
- Ciência de Dados : Data Science
- Quadro de dados: Data frame
- Cientistas de dados: Data Scientists
- Análise de dados: Data Analysis
- Aquisição de dados: Data Acquisition
- Inteligência Artificial (IA): Artificial Intelligence (AI)
- Soquete : Socket
- Análise Semântica Latente (ASL): Latent Semantic Analysis (LSA)
- Matriz: Matrix
- Memoização: Memoization
- Testes de módulo: Unit tests
- Processamento de dados: Data Wrangling
- Expressão Regular: Regex
- Cientistas de dados autoproclamados: Data scientist impostors
- Linguagem específica de domínio: Domain-Specific Language (DSL)
- Caderno: Notebook
- Web: Web, (World Wide Web)
- Repositório: Repository


**Observação:** Consulte também a seção Reparar seu telefone para obter mais esclarecimentos. ("Notas do tradutor".)

------

## 楽-for ÷ with-楽

... também conhecido como ***"Raku-for vs. Raku "*** *(também, talvez, "diversão para" vs. "com diversão")*.

Em primeiro lugar, façamos a seguinte distinção:

- Raku ***for*** data wrangling significa usar Raku para facilitar o wrangling de dados em outras linguagens e sistemas de programação.

- Raku significa usar as estruturas de dados e a linguagem de programação de Raku para a disputa de dados.

Aqui está um exemplo de Raku ***for*** data wrangling -- código Python é gerado:

```perl6
dsl-translation("
carregar dados íris;
agrupar com a coluna Species;
mostrar dimensões"):code
```

O seguinte diagrama:

- Resume minhas atividades de disputa de dados
- Indica planos futuros com linhas tracejadas
- Representa os esforços “Raku-for” com o hexágono
- Representa os esforços “com-Raku” no canto inferior direito

![](https://github.com/antononcube/RakuForPrediction-book/raw/main/Diagrams/DSLs-Interpreter-for-Data-Wrangling-April-2022-state.png)

Para ilustrar a multilinguagem da abordagem, aqui está um exemplo de tradução
Especificações de transformação de dados em inglês para especificações de transformação de dados em búlgaro, inglês e espanhol:

```
# obj = example_dataset('iris')
# obj = obj.groupby([\"Species\"])
# print(obj.size())
```

```perl6
for <Bulgarian English Spanish> -> $l {
say '-' x 30;
say ToDSLCode("dsl target " ~ $l ~";
carregar dados íris;
agrupar com a coluna Species;
mostrar dimensões"):code
}
```

Como o diagrama acima indica, pretendo usar essa estrutura para narrar o código Raku de disputa de dados com linguagens naturais.
Além disso, é claro, traduza para outras linguagens de programação.

**Observação:** Por muito tempo usei o princípio "a roupa não tem imperador", [AAv1, CARH1], que,
obviamente pertence à abordagem "Raku-for-prediction". Ao dotar Raku com (i) capacidades de manipulação de dados, e
(ii) a capacidade de gerar código Raku de disputa de dados, eu diria que essas roupas podem ser usadas adequadamente.
E vice-versa - os filhos do sapateiro vão pular bem calçados.

------

## Datum fundamentalum

... *também conhecido como* ***"Estruturas e metodologia de dados"*** *(Também significa "base dada" em latim.)*

Vamos definir ou esboçar os conceitos básicos de nosso esforço de organização de dados Raku.

### Conjunto de dados vs quadro de dados

Aqui estão algumas definições intuitivas de conjuntos de dados e quadros de dados:

- Um ***conjunto de dados*** é uma tabela que, como estrutura de dados, é mais naturalmente interpretada como uma matriz de hashes, cada hash representando uma *linha* na tabela.

- Um **frame de dados** é uma tabela que, como estrutura de dados, é mais naturalmente interpretada como uma matriz de hashes, cada hash representando uma *coluna* na tabela.

O Mathematica usa conjuntos de dados. Os [pandas](https://pandas.pydata.org) de S, R e Python usam quadros de dados.

O sistema Raku apresentado neste documento usa conjuntos de dados. Aqui está um exemplo de um conjunto de dados com 3 linhas e 2 colunas:

```perl6
srand(128);
my $tbl=random-tabular-dataset(3,2).deepmap({ $_ ~~ Str ?? $_ !! round($_, 0.01) });
.say for |$tbl
```
```
# {controlling => -4.84, unlace => means}
# {controlling => 7.83, unlace => thyrotropin}
# {controlling => 11.92, unlace => parfait}
```

Aqui está como o quadro de dados correspondente teria sido estruturado:


```perl6
transpose($tbl)
```
```
# {controlling => [-4.84 7.83 11.92], unlace => [means thyrotropin parfait}]}
```

### Perspectiva minimalista

Não quero criar um tipo especial (classe) para conjuntos de dados ou quadros de dados -- quero usar as estruturas de dados Raku padrão.
(Pelo menos neste ponto dos meus esforços de disputa de dados Raku.)

Meus motivos são:

1. Os dados podem ser coletados e transformados com comandos embutidos típicos.

    - Ou seja, sem a adesão a uma determinada metodologia de transformação de dados ou pacotes dedicados.

2. O uso de estruturas integradas padrão é um tipo de decisão de “interface do usuário”.

3. A "experiência do usuário" pode ser alcançada com ou fornecida por outros paradigmas e pacotes de transformação.

Os pontos 2 e 3 são, obviamente, consequências do ponto 1.

### Estruturas de dados em Raku

As estruturas de dados nas quais nos concentramos são conjuntos de dados e, concretamente, no Raku, temos as seguintes representações de conjuntos de dados:

1. Matriz de hashes

1. Mistura de hashes

1. Matriz de matrizes

1. Hash de matrizes

A ordem das representações indica sua importância durante a implementação do Raku data wrangling
funcionalidades aqui apresentadas:

- As funcionalidades para os dois primeiros são primárias e possuem testes de unidade

- Além disso, acomodamos o uso dos dois últimos.

Quando a estrutura e a constelação de funcionalidades de disputa de dados amadurecem, todas as quatro estruturas de dados terão
tratamento correto e consistente.

### Usuários-alvo

Os usuários-alvo são cientistas de dados (tempo integral, meio período e novatos completos) que desejam:

- Faça disputa de dados de conjuntos de dados típicos de ciência de dados com Raku

- Saiba que seus esforços de disputa de dados Raku são reproduzidos com relativa facilidade em outras linguagens ou sistemas de programação

Alternativamente, podemos dizer que os usuários-alvo são:

1. Impostores de cientistas de dados

1. Código baristas

1. Cientistas de dados experientes que desejam acelerar seu trabalho

1. Cientistas de dados que desejam aprender Raku

1. Programadores Raku

### Fluxos de trabalho considerados

O fluxograma a seguir abrange os fluxos de trabalho de transformação de dados que consideramos:

![](https://raw.githubusercontent.com/antononcube/ConversationalAgents/master/ConceptualDiagrams/Tabular-data-transformation-workflows.jpg)

**Observação:** Vamos nos referir à metodologia representada pelo fluxograma acima como a
***Modelo de fluxos de trabalho de transformação de dados*** (DTWM).
Neste documento consideramos essa metodologia e o fluxograma como sinônimos.

Aqui estão algumas propriedades da metodologia/fluxograma:

- O fluxograma é para conjuntos de dados tabulares ou para listas (arrays) ou dicionários (hashes) de conjuntos de dados tabulares

- No fluxograma apenas o carregamento de dados e a análise resumida não são opcionais

- Todas as outras etapas são opcionais

- Transformações como junções internas são representadas pelo bloco “Combinar grupos”

- Supõe-se que em aplicações reais várias iterações (loops) devem ser executadas no fluxograma

No mundo da linguagem de programação R os blocos laranja representam o chamado padrão Split-Transform-Combine;
veja o artigo
["A estratégia Split-Apply-Combine para análise de dados"](https://www.jstatsoft.org/article/view/v040i01)
por Hadley Wickham, [[HW1](https://www.jstatsoft.org/article/view/v040i01)].

**Observação:** R era (e provavelmente ainda é) uma linguagem de programação bastante misteriosa, então a introdução explícita do
O padrão Split-Transform-Combine foi de grande ajuda para os programadores de R. Por outro lado, esse padrão é bastante antigo
e bem conhecido: é inerente ao SQL e é encontrado na programação paralela.
(Por exemplo, veja a função WL
[`ParallelCombine`](http://reference.wolfram.com/mathematica/ref/ParallelCombine.html),
[WRI2].)

Aqui está um exemplo simples de cenário de caso de uso:

1. Obtenha um conjunto de dados tabulares de um warehouse.

2. Resuma e examine o conjunto de dados e decida se ele não tem a forma e o conteúdo desejados.

    - Ou seja os dados devem ser disputados.

3. Selecione apenas as colunas que possuem os dados de interesse.

4. Filtre as linhas de acordo com um determinado critério operacional.

5. Divida as linhas -- ou seja, agrupe pelos valores em uma das colunas.

6. Transforme cada grupo combinando os valores de cada coluna de alguma forma.

    - Por exemplo, encontrar médias ou desvios padrão de colunas numéricas.

7. Combine os grupos transformados (em um conjunto de dados tabular “plano”).

8. Reformule os dados em formato longo e exporte-os.

A lista de etapas acima é apenas um fluxo de trabalho possível. Para exemplos e explicações mais detalhadas, consulte [AA1, AAv2, AAv3, AAv4].

### Operações fundamentais

Estas operações são apenas básicas:

- Seleção, renomeação e exclusão de colunas

- Seleção e exclusão de linha

- Transpondo

- Junções internas, esquerda e direita

- Agrupamento por critérios

(A transposição de dados tabulares ou de matriz completa também é uma operação de programação funcional básica.)

Na disputa de dados e na análise de dados, as três operações a seguir não são básicas, mas ainda são fundamentais:

- [Tabulação cruzada](https://en.wikipedia.org/wiki/Contingency_table)

- [Conversão de formato longo](https://en.wikipedia.org/wiki/Wide_and_narrow_data#Narrow)

- [Conversão de formato largo](https://en.wikipedia.org/wiki/Wide_and_narrow_data#Wide)

Veja [AA1, Wk1, Wk2, AAv1-AAv4] para mais detalhes.

**Observação:** O pacote
["Data::Reshapers"](https://github.com/antononcube/Raku-Data-Reshapers),
[AAp2], fornece todas as funções mencionadas nesta subseção.


------

## The size of the magic data does not determine how magic it is

** O tamanho dos dados mágicos não determina o quão mágico é **

*TBD...*

------

## GYOD

*TBT...*

------

## Data wrangling for dummies (a reference for the rest of us)

*TBT...*

------

## Doing it like a Cro

*TBT...*

------

## The one way to do it

*TBT...*

-------

## Heavy-brained instead of lighthearted

*TBT...*

-------

## A tale about the Wolf, the Ram, and the Raccoon

*TBT...*

------

## Too green to be Red

*TBT...*

------

## Making the future more evenly distributed

*TBT...*

--------


## Conserte o telefone

*TBT...*

-------

## Setup

*TBD...*

------

## References

### Articles, books

[AA1] Anton Antonov, "Data wrangling in Raku", (2021), RakuForPrediction-book at GitHub.

[AA2] Anton Antonov,
["Connecting Raku to Mathematica"](https://github.com/antononcube/RakuForPrediction-book/blob/main/Articles/Connecting-Mathematica-and-Raku.md),
(2021),
[RakuForPrediction-book at GitHub](https://github.com/antononcube/RakuForPrediction-book).

[AA3] Anton Antonov,
["Translating Mathematica expressions to High Performance Fortran"](https://library.wolfram.com/infocenter/MathSource/5143/HPF.pdf),
(1999),
HiPer'99, Tromsoe, Norway.

[AAn3] Anton Antonov,
["Translating Mathematica expression to High Performance Fortran"](https://www.wolframcloud.com/translating-mathematica-expression-to-high-performance-fortran--2018-10-10qgpsl/),
from the Notebook Archive (2004),
https://notebookarchive.org/2018-10-10qgpsl .

[CARH1] Charles Antony Richard Hoare,
(1980),
["The emperor's old clothes"](https://dl.acm.org/doi/10.1145/1283920.1283936)
ACM Turing award lectures January 2007 Year Awarded: 1980.
https://doi.org/10.1145/1283920.1283936.

[HW1] Hadley Wickham,
["The Split-Apply-Combine Strategy for Data Analysis"](https://www.jstatsoft.org/article/view/v040i01),
(2011),
[Journal of Statistical Software](https://www.jstatsoft.org).

[FB1] Federico Biancuzzi and Shane Warden,
(2009),
[Masterminds of Programming: Conversations with the Creators of Major Programming Languages](https://www.oreilly.com/library/view/masterminds-of-programming/9780596801670/).
ISBN 978-0596515171. See page. 385.

### Functions, packages, repositories

[AAp1] Anton Antonov,
[Data::Reshapers](https://modules.raku.org/dist/Data::Reshapers:cpan:ANTONOV),
(2021),
[Raku Modules](https://modules.raku.org).

[AAp2] Anton Antonov,
[Data::Summarizers](https://github.com/antononcube/Raku-Data-Summarizers),
(2021),
[Raku Modules](https://modules.raku.org).

[AAp3] Anton Antonov,
[Data::ExampleDatasets](https://github.com/antononcube/Raku-Data-ExampleDatasets),
(2021),
[Raku Modules](https://modules.raku.org).

[AAp4] Anton Antonov,
[Data::Generators](https://modules.raku.org/dist/Data::Generators:cpan:ANTONOV),
(2021),
[Raku Modules](https://modules.raku.org).

[AAp5] Anton Antonov,
[Mathematica::Serializer Raku package](https://github.com/antononcube/Raku-Mathematica-Serializer),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp6] Anton Antonov,
[DSL::English::DataQueryWorkflows Raku package](https://github.com/antononcube/Raku-DSL-English-DataQueryWorkflows),
(2020-2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAp7] Anton Antonov,
[DSL::English::DataAcquisitionWorkflows Raku package](https://github.com/antononcube/Raku-DSL-English-DataAcquisitionWorkflows),
(2021), [GitHub/antononcube](https://github.com/antononcube).

[AAp8] Anton Antonov,
[DSL::Utilities::ComprehensiveTranslation](https://github.com/antononcube/Raku-DSL-Shared-Utilities-ComprehensiveTranslation),
(2020-2021),
[GitHub/antononcube](https://github.com/antononcube).

[AAr1] Anton Antonov,
[Data Acquisition Engine project](https://github.com/antononcube/Data-Acquisition-Engine-project),
(2021),
[GitHub/antononcube](https://github.com/antononcube).

[FCOp1] Fernando Correa de Oliveira,
[Red](https://modules.raku.org/dist/Red:cpan:FCO),
(last updated 2021-11-22),
[Raku Modules](https://modules.raku.org).

[JSp1] Jonathan Stowe,
[XDG::BaseDirectory](https://modules.raku.org/dist/XDG::BaseDirectory:cpan:JSTOWE),
(last updated 2021-03-31),
[Raku Modules](https://modules.raku.org).

[WRI1] Wolfram Research, (2014),
[Dataset](https://reference.wolfram.com/language/ref/Dataset.html),
Wolfram Language function, https://reference.wolfram.com/language/ref/Dataset.html (updated 2021).

[WRI2] Wolfram Research, (2008),
[ParallelCombine](https://reference.wolfram.com/language/ref/ParallelCombine.html),
Wolfram Language function, https://reference.wolfram.com/language/ref/ParallelCombine.html (updated 2010).

[WRI3] Wolfram Research, (2016),
[URLRead](https://reference.wolfram.com/language/ref/URLRead.html),
Wolfram Language function, https://reference.wolfram.com/language/ref/URLRead.html.

### Presentation video recordings

[AAv1] Anton Antonov,
["Raku for Prediction"](https://conf.raku.org/talk/157),
(2021),
[The Raku Conference 2021](https://conf.raku.org).

[AAv2] Anton Antonov,
["Multi-language Data-Wrangling Conversational Agent"](https://www.youtube.com/watch?v=pQk5jwoMSxs),
(2020),
Wolfram Technology Conference 2020,
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv3] Anton Antonov,
["Data Transformation Workflows with Anton Antonov, Session #1"](https://www.youtube.com/watch?v=iXrXMQdXOsM),
(2020),
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv4] Anton Antonov,
["Data Transformation Workflows with Anton Antonov, Session #2"](https://www.youtube.com/watch?v=DWGgFsaEOsU),
(2020),
[YouTube/Wolfram](https://www.youtube.com/channel/UCJekgf6k62CQHdENWf2NgAQ).

[AAv5] Anton Antonov,
["Doing it like a Cro (Raku data wrangling Shortcuts demo)"](https://youtu.be/wS1lqMDdeIY),
(2021),
[YouTube/Anton.Antonov.Antonov](https://www.youtube.com/channel/UC5qMPIsJeztfARXWdIw3Xzw).

[AAv6] Anton Antonov,
["Multi language Data Acquisition Conversational Agent (extended version)"](https://www.youtube.com/watch?v=KlEl2b8oxb8),
(2021),
[YouTube/Anton.Antonov.Antonov](https://www.youtube.com/channel/UC5qMPIsJeztfARXWdIw3Xzw).
