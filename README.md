# jekyll-literate-book

The [jekyll-literate](https://github.com/jonsnyder01/jekyll-literate) gem written as a literate
program: the book in `book/` tangles every file of the gem that is code or configuration, and
the gem weaves the book. Edit the gem here, not in its own repository.

## Layout

```
book/                    the book source; the only thing you edit
gem/                     tangled by Entangled: naked by bin/dist, annotated by literate (gitignored)
gem-static/              the gem's files that are not tangled: fonts, licences, README, glossary, ADRs
dist/                    the gem as published: gem/ tangled naked, plus gem-static/ (gitignored)
bin/dist                 tangles naked, then assembles dist/
bin/compare              checks dist/ against a jekyll-literate checkout
bin/weave-dist           weaves the book with dist/ (Gemfile.dist), into build/dist-site/
_config.yml              this book's config (its source browser lists gem/)
entangled.toml           Entangled configuration, with the Ruby, SCSS and HTML languages
Gemfile                  the jekyll-literate that weaves this book
```

The book is tangled two ways. `literate build` tangles it with annotations, which the source
browser builds its cross-references from. The published gem must not carry them (in a layout
they would be HTML comments in every page, and above its front matter they would hide it from
Jekyll), so `bin/dist` tangles it naked. Both write `gem/`, each replacing the other's files. The terms used here (book source, chapter, fragment, …) are defined in
`gem-static/GLOSSARY.md`.

## Commands

Tools are as for any jekyll-literate book: Entangled 2.5.0 and Ruby 4.0 with Bundler (see the
gem's README).

```sh
bundle config set --local path vendor/bundle && bundle install   # first time

bin/dist                          # tangle naked, then assemble dist/
bin/compare ../jekyll-literate    # check dist/ against a jekyll-literate checkout
bin/weave-dist ../jekyll-literate # weave with dist/ laid over a jekyll-literate checkout
bundle exec literate serve        # tangle annotated, then weave and preview at http://localhost:4000
bundle exec literate build        # tangle annotated, then weave into build/site/
```

## Bootstrapping

The book is being written: chapters are added until it tangles every file of the gem. Until
then it is woven by the published jekyll-literate (the Gemfile's `git:` source), and
`bin/compare` reports how far it has come: every file the book tangles must match the published
one byte for byte, and the rest are listed as not in the book yet. While a file is not in the
book, change it in jekyll-literate directly; once it is, change it only here.

The Source workflow already weaves the book with the gem it tangles (`bin/weave-dist`), with
`dist/` laid over the published gem for the files not in the book yet; the weave succeeds or
the change is not published. When nothing is left, `bin/weave-dist` runs with no checkout
under `dist/`, the Gemfile switches to `path: "dist"` (so the Book workflow runs `bin/dist`
first), and the Source workflow on `main` pushes `dist/` to jekyll-literate after weaving with
it.

## CI

- **Source** (`.github/workflows/source.yml`, changes to the book, `gem-static/`, `bin/` or
  the configs): `bin/dist`, `bin/compare` against jekyll-literate's `main`, then
  `bin/weave-dist`.
- **Book** (`.github/workflows/book.yml`, changes to the book or its config): weaves the book
  with jekyll-literate's reusable workflow, which on `main` publishes it to GitHub Pages:
  https://jonsnyder01.github.io/jekyll-literate-book/.
