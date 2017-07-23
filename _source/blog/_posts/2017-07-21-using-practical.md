---
title: "Practical the Jekyll Template & Theme"
---

## Layouts

Practial has two classes of layouts: **default** and **wiki**.

### default layout

default layout is used for normal pages including blog posts. The left panel contains the main navigation menu.

### wiki layout

wiki layout is designed for documentation use. The left panel becomes the index of whole wiki. So people can navigate through wiki pages easily.

## Using Markdown

Practical uses kramdown for markdown.

### Using LaTeX

Practical supports LaTeX rendering through either MathJax or KaTeX.

This feature is disabled by default. To enable, set `features.latex` to `mathjax` or `katex` instead of `false`.

Usually, inline and display math are using different markups. for example `$` for inline and `$$` for display. kramdown doesn't hornor this as it can automatically differentiate inline and display from input. Simply surround all math with a pair of `$$`. If a math block itself is a paragraph, it becomes a display math, otherwise it becomes inline. 