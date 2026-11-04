---
title: "Packaging"
entangled:
  version: "2.5"
  namespace: packaging
---

jekyll-literate is one Ruby gem that does three jobs. It is a Jekyll theme:
Jekyll reads layouts, includes, Sass and assets from the root of the gem a book
names as its `theme:`. It is a Jekyll plugin: a book's Gemfile puts it in the
`jekyll_plugins` group, and Jekyll requires every gem in that group before it
builds. And it carries a command, `literate`, that tangles a book and then
weaves it.

## The gemspec

```ruby
#| file: gem/jekyll-literate.gemspec
# frozen_string_literal: true

Gem::Specification.new do |spec|
  <<identity>>

  <<contents>>

  <<dependencies>>
end
```

The gem is published only as a git repository, which a book's Gemfile names
with `git:`, so its version is not yet a promise to anyone; the lockfile of each
book records the commit it was built with.

```ruby
#| id: identity
spec.name = "jekyll-literate"
spec.version = "0.1.0"
spec.summary = "Weaves an Entangled literate program into a Jekyll book with a source browser"
spec.authors = ["Jonathan Snyder"]
spec.homepage = "https://github.com/jonsnyder01/jekyll-literate"
spec.required_ruby_version = ">= 3.1"
```

A theme is read from the gem's root, so the theme's directories are listed
beside `lib/`, along with the theme's own `_config.yml` of defaults.

```ruby
#| id: contents
# It is both a theme (layouts, includes, Sass, assets and a _config.yml of
# defaults, all read from the gem's root) and a plugin (lib/).
spec.files = Dir["{_config.yml,_includes/**/*,_layouts/**/*,_sass/**/*,assets/**/*,lib/**/*,exe/*}",
                 "LICENSE-tufte-jekyll", "README.md"]
spec.bindir = "exe"
spec.executables = ["literate"]
```

Jekyll brings kramdown and Rouge with it; the gem asks for the versions it was
written against, and for kramdown's GitHub-flavoured parser.

```ruby
#| id: dependencies
spec.add_dependency "jekyll", "~> 4.4"
spec.add_dependency "kramdown-parser-gfm", "~> 1.1"
spec.add_dependency "rouge", "~> 4.5"
# Reads the languages a book declares in entangled.toml.
spec.add_dependency "tomlrb", "~> 3.0"
# `jekyll serve` needs a web server; Ruby 3+ no longer bundles one.
spec.add_dependency "webrick", "~> 1.9"
```

Working on the gem itself only needs the gemspec's dependencies.

```ruby
#| file: gem/Gemfile
source "https://rubygems.org"

gemspec
```

## The plugin

Jekyll requires the gem by its name, which loads the weave and the source
browser.

```ruby
#| file: gem/lib/jekyll-literate.rb
# frozen_string_literal: true

# Loaded by Jekyll from a book's Gemfile (group :jekyll_plugins); see README.md.

require "jekyll"
require_relative "jekyll-literate/literate"
require_relative "jekyll-literate/source"
```

## The theme's defaults

Jekyll merges a theme's `_config.yml` under the book's own, except for its
built-in settings, which a theme cannot set.

```yaml
#| file: gem/_config.yml
# Defaults for every book; a book's own _config.yml overrides any of them.
# (Jekyll ignores a theme's settings for its own built-in keys, such as
# source, destination or defaults: those belong in the book's config.)

literate:
  # Where `entangled tangle` writes the tangled files, from the directory the
  # book is built in. The source browser lists it at /<tangle_dir>/.
  tangle_dir: src

sass:
  style: compressed
  sourcemap: never
  # The SCSS uses @import (pre-module-system Sass) for its partials.
  quiet_deps: true
  silence_deprecations: [import, global-builtin, color-functions, slash-div]
```

## The command

`literate` tangles first, because the source browser lists the tangled files,
then hands over to Jekyll in the same Ruby, so that Bundler's choice of gems
still holds. Its `mode` makes Entangled write it executable.

```ruby
#| file: gem/exe/literate
#| mode: "0755"
#!/usr/bin/env ruby
# frozen_string_literal: true

# Tangles the book with Entangled, then weaves it with Jekyll:
#   literate build          build into the configured destination
#   literate serve          preview at http://localhost:4000 with live reload
# Run from the book's root, where _config.yml is (any further arguments go to Jekyll).

command = ARGV.shift || "build"
# The source browser is built from the tangled files, so refresh them.
system("entangled", "tangle") or abort "literate: `entangled tangle` failed"
config = ARGV.include?("--config") ? [] : ["--config", "_config.yml"]
exec Gem.ruby, Gem.bin_path("jekyll", "jekyll"), command, *config, *ARGV
```
