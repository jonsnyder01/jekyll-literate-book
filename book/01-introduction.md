---
title: "Introduction"
---

jekyll-literate is a literate programming tool: a book written as Markdown
chapters is tangled into files by [Entangled](https://entangled.github.io/) and
woven into a website by [Jekyll](https://jekyllrb.com) with this gem's help. This
book is jekyll-literate itself, written that way. Every file of the gem that is
code or configuration is tangled from a fragment in one of its chapters; only the
fonts, the licences and the gem's prose (its README, glossary and decision
records) are kept as they are.

## How this book becomes the gem

The book is tangled two ways. To be woven, it is tangled into `gem/` with
annotations: comment lines that record which fragment wrote each line. This
book's source browser is built from them, so every listing links back to its
fragments. The gem as published must not carry them, though: in a layout they
would be HTML comments in every page built with the theme, and above a layout's
front matter they would hide it from Jekyll. So to make the gem, `bin/dist`
tangles the book naked (`entangled tangle -a naked`), without annotations, and
adds the files that are not tangled to make `dist/`. A build on `main` will push
`dist/` to the jekyll-literate repository, which holds only what this book
builds.

While the book is being written it is woven by the published jekyll-literate,
and `bin/compare` checks `dist/` against it: every file the book tangles must
match the published one byte for byte, and the rest are listed as not in the
book yet. Before anything is published the book is also woven by `dist/`,
laid over the published gem until the book tangles all of it, so that a change
that breaks the weave never reaches jekyll-literate.

## Reading the code

Each fragment has a file header with the full path of the file its code is in,
from this book's root: `gem/lib/jekyll-literate.rb` is published as
`lib/jekyll-literate.rb`. A named fragment shows its name beside its code,
`⟨name⟩ ≡`, or `⟨name⟩ +≡` for an appended fragment, and a reference to it
appears in another fragment's code as `⟨name⟩`. Names are scoped to their
chapter.
