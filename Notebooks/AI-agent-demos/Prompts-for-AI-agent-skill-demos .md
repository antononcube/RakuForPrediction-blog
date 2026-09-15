# Prompts for AI-agent skill demos 

---

## LLM::Graph

See the notebook ["LLM-graph-creation-demo.ipynb"](./LLM-graph-creation-demo.ipynb).

```text
Create an LLM-graph for "progressive disclosure" ingestion of an AI-agent skill.

- The input of the graph is a (user) command and a skill directory 
- The LLM-graph has a dispatcher / classifier as a top node
    - The classification labels are: 'skill-info', 'skill-description', and 'code generation' 
- Next level of the graph has a nodes for each classification label
- The bottom note formulates the final result
```

---

## Math::NumberTheory

See the notebook ["Number-theory-computaions.ipynb"](./Number-theory-computaions.ipynb).

````text
In the notebook "./Number-theory-computaions.ipynb" make a Raku cell that is equivalent to the following Wolfram Language code:

```wolfram
ArrayPlot[Table[CarmichaelLambda[j + 5 k], {j, 1, 100}, {k, 1, 100}], ColorFunction -> "AvocadoColors"]
```
````

````text
Nice. In the last cell of the notebook code this neat example:

```wolfram
(*Plot MangoldtLambda for the sum of two squares:*)

ArrayPlot[Table[MangoldtLambda[a^2 + b^2], {a, 50}, {b, 50}], ColorFunction -> "BlueGreenYellow"]
```
````

````text
Ok. Use the Ulam spiral utility function of that package and code:

```wolfram
ArrayPlot[MangoldtLambda[ulam[101]], ColorFunction -> "Rainbow"]
```
````

````text
In the Raku notebook "Number-theory-computations.ipynb" make a neat example of Mangoldt lambda with  colorful matrix plot.
````

---

## JavaScript::Google::Charts

See the notebook ["JavaScript-Google-Charts-demo.ipynb"](./JavaScript-Google-Charts-demo.ipynb).

```text
In the last section of the Raku notebook "JavaScript-Google-Charts-demo.ipynb" generate  data representing to ten USA most populated cites: coordinates and population and area size. Then make a corresponding bubble chart using Google Charts.
```

```text
Turn the plot into dark mode, please.
```

```text
Make a separate plot in which the area sizes and the populations are combined. Can this done via a bubble chart?
```

```text
In the "Number theory" section create a sunflower distribution of integers from 1 to 5000 assign colors to them according to mod 5 and use Google Charts bubble chat to plot them. The bubble sizes and colors should reflect the mod 5 values.
```

```text
Nice! Remove the legend bar on top of the plot, and make plot margins much smaller, say 10px.
```