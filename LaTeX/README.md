# LaTeX

A few handy commands for LaTeX that help solve recurring problems.

## Todo comments

This creates commands for leaving notes on the pdf which show off to the side, instead of occupying space in the text itself. This allows for leaving notes without messing with the spacing of the document, which makes it more difficult to measure the page count at any given time. It's also easier to visualize.

First, import the appropriate package:

```
\usepackage{todonotes}
```

Then, create new commands as required:

```
\newcommand{\cfm}[1]{\todo[size=\tiny,linecolor=blue,backgroundcolor=green!25,bordercolor=black]{FM: #1}}
```

`cfm` can be replaced with something meaningful, like `neno` or `marcelo` (as we'd used with `\textcolor` before). The coloring can also be modified if desired. Finally, the name of the person leaving the comment can be left in the content, such as replacing `{FM: #1}` with `{Neno: #1}` or `{Marcelo: #1}`. The result will look like this:

![todo_comment.jpg](./todo_comment.jpg?raw=true "Todo Comment")