# Notes on LaTeX

How ironic is it that notes on LaTeX are being made in Markdown?

## Document structure

* Every document must start by defining its document class:
  `\documentclass{...}`.
  - A document can use multiple document classes through a class like `combine`
    [2].
* The `documentclass` is followed by the preamble. The preamble is a section
  that contains definitions, macro/command calls and package imports that
  affect the whole document. These commands and macros should not be outputting
  any text.
* The text of the document is enclosed in the section defined by the
  `\begin{document}` and `\end{document}` commands.
  - A multi-class document will have multiple pairs of `\begin{document}` and
    `\end{document}`.
* The section after `\end{document}` and until the end of the input file is not
  typeset by LaTeX and therefore is a good place for comments or temporary
  text.
  - Putting text before the document section (in the preamble) results in
    error.

### Example

```tex
\documentclass{article}

\begin{document}
  Hello world.
\end{document}

After the document. Won't be typeset!
```

### Document class

The `\documentclass` can take optional arguments:
`\documentclass[option1,option2]{a_class}`.

### Packages

LaTeX packages can be imported with `\usepackage` command. The command can take
optional arguments: `\usepackage[option1,opton2]{a_package}`.

### Document environment

The document is, usually, split in the following sections:

1. Top matter: it defines the document meta data such as the title, the
   author etc. It usually ends with `\maketitle` which generates a title page
   or section.
1. Abstract: it can be enclosed in the `abstract` environment (in a pair of
   `\begin{abstract}` and `\end{abstract}` commands).
1. Sections: LaTeX provides 7 levels of document sectioning between the default
   classes. Custom document classes can provide arbitrary sectioning levels.
   LaTeX article can use `\part`, `\section`, `\subsection`, `\subsubsection`,
   `\paragraph` and `\subparagraph`.
   - LaTeX sections can be combined with customizable numbering (section 1 or
     chapter A, etc.).
1. Tables: table of contents `\tableofcontents`, list of figures
   `\listoffigures`, list of images etc.
1. Bibliography: within the `thebibliography` environment. See [3] for more.

#### Example

```tex
\documentclass{article}

\begin{document}
  % The top matter
  \title{Hello world}
  \author{John Doe}
  \maketitle

  % Section without number
  \section*{Introduction}
  Hello world.

  % First numbered section gets section number 1
  \section{Main}
  World hello.

  \subsection{Why?}
  Foo bar.
\end{document}
```

## References

1. https://en.wikibooks.org/wiki/LaTeX/Document_Structure
1. http://www.tex.ac.uk/FAQ-multidoc.html
1. https://en.wikibooks.org/wiki/LaTeX/Bibliography_Management
