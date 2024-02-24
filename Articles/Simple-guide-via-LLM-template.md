# Simple guide 

### ... LLM-based template

```perl6, results=hide, echo=FALSE
# In case those packages are not loaded by the LLM code chunks 
use WWW::PaLM;
use WWW::OpenAI;
```


## Simply put

```palm, format=values, temperature=0.75, max-tokens=800, results=asis, output-prompt=NONE, echo=FALSE
Generate a 7 steps outline list -- using Arabic numerals with bold prefixes per item -- for R being a programming language that:
1) Is arcane
2) Has all the features of a design by a committee 
3) Cannot suffer the LISP curse.
```

```perl6, results=asis, output-prompt=NONE, echo=FALSE
# Expansions generation script

# Assuming that the list items are numbered with Arabic or Roman numerals
my $txt = _.trim;
my $txtExpanded = do for $txt.split(/ ^^ [ \d+ | <[IVXLC]>+ ] /, :v, :skip-empty)>>.Str.rotor(2) -> $p {
    my $res = "-" x 6; 
    $res ~= "\n"; 
    $res ~= "\n## {$p[0]}.";
    my $start = '';
    if $p[1] ~~ / '**'  (.*?) '**' | '<b>'  (.*?) '</b>' / {
        $start = $0.Str;
        $res ~= ' ' ~ $start.subst( / <punct>+ $$/, '');
    };
    $res ~= "\n\n>... {$p[1].subst(/'**' $start '**'/, '').subst( / ^^ <punct>+ /, '')}"; 
    #$res ~= "\n\n", openai-completion( "Expand upon: {$p[1]}", temperature => 1.45, max-tokens => 400, format=>'values' );
    $res ~= "\n\n", palm-generate-text( "Expand upon: {$p[1]}", temperature => 0.75, max-tokens => 400, format=>'values' );
}

$txtExpanded.join("\n\n") 
```
