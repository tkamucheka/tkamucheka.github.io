---
id: 2026-06-23-tree-sitter-mlir
aliases:
  - A Dialect-Agnostic Tree-sitter Grammar for MLIR
tags:
  - mlir
  - tree-sitter
  - neovim
  - compilers
  - tooling
author: Tendayi Kamucheka
category: blog
date: June 23, 2026
layout: post
title: "A Dialect-Agnostic Tree-sitter Grammar for MLIR"
---

If you spend a meaningful amount of time working with MLIR in Neovim, you've probably noticed that syntax highlighting mostly exists for upstream dialects. Custom dialects are generally unsupported and treated almost like plaintext. 

The canonical tree-sitter grammar for MLIR, [artagnon/tree-sitter-mlir](https://github.com/artagnon/tree-sitter-mlir), exists and does useful work, but it takes a dialect-enumeration approach: it knows about `func`, `arith`, `scf`, and a handful of others, and handles their syntax as special cases. I run two custom dialects in my own compiler work, and neither was on the list. Highlighting would go patchy — some keywords recognised, others not, depending on how much of the grammar the parser could infer from context.

I kept coming back to the same question: *can MLIR's own EBNF specification be sufficient to parse arbitrary dialect IR, without enumerating dialects?* I believe the answer is yes, and [tree-sitter-mlir](https://github.com/tkamucheka/tree-sitter-mlir) is my attempt at proving it.

Fair warning before we go further: I'm a PhD candidate finishing a dissertation on an MLIR compiler. This entire project was vibe-coded with Claude in the spare minutes before bed. I've done enough testing to be cautiously optimistic, but not enough to be fully confident. Use accordingly, and please open issues when you find the cracks.

---

## The Hard Problem: Custom Operations

MLIR's generic operation form is easy to parse:

```mlir
"arith.addi"(%0, %1) : (i32, i32) -> i32
```

Quoted name, parenthesised operands, colon, function type. Unambiguous. Any grammar student could write a parser for it.

The *custom* operation form is another matter entirely:

```mlir
%result = arith.addi %a, %b : i32
%out = scf.for %iv = %lb to %ub step %step iter_args(%x = %init) -> f32 {
  ...
}
memref.store %val, %buf[%i0, %i1] : memref<?x?xf32>, index, index
```

Each dialect invents its own textual syntax. The parser sees a stream of tokens with no structural delimiter telling it where one operation ends and the next begins. If you're looking at `%result = foo.bar %x, %y`, the tokens `%x`, `,`, `%y` could be operands to this op, or they could be the start of the *next* op's result list after a boundary you haven't found yet.

The upstream grammar handles this by knowing what each dialect's syntax looks like. I wanted to handle it without that knowledge.

---

## The Two-Path GLR Structure

The insight that makes a general parser possible is that MLIR's custom op syntax always terminates in one of a small number of *unambiguous terminal constructs*:

- A **type annotation** — `: some_type` or `-> some_type`
- A **region** — `{ ... }`
- A **block successor** — `^bb0`

Everything before the terminal is a prefix — a sequence of tokens that are part of *this* operation. Once you spot the terminal, you know where the operation ends.

This gives the grammar a two-path structure for the operation body:

**Path 1 — `with_terminal`:** Zero or more prefix tokens, followed by one of the terminal constructs above. This handles the vast majority of real-world ops.

**Path 2 — `safe_prefix_only`:** One or more tokens that are structurally safe to consume — they cannot be mistaken for the start of the next operation's result list or name. This handles the rare ops that emit no type annotation and open no region.

The grammar runs these two paths in parallel using GLR (Generalised LR) parsing and uses dynamic precedence to pick the right one when both could match.

---

## Why GLR?

Tree-sitter supports GLR mode, which allows the parser to maintain multiple parse hypotheses simultaneously and resolve them as more tokens arrive. This is essential for MLIR because several constructs are genuinely ambiguous until you've seen more of the input:

- `{` could open a **region** or a **dictionary attribute**. You can't know which until you see what's inside.
- A dotted bare identifier like `foo.bar` could be an **op name** or an **attribute key**. Context resolves it.
- `!MyType` could be a **type alias** or the start of a **dialect type** — depends on whether an angle body follows.

Rather than paper over these ambiguities with lookahead hacks, the grammar declares them honestly and lets dynamic precedence sort them out. There are thirteen declared conflict sets in total, all of them genuine structural ambiguities in the MLIR spec, not artefacts of a sloppy grammar.

One non-obvious design decision worth calling out: several dialects (e.g. `memref.reinterpret_cast`) use a `key: [values]` syntax for named offset/size/stride lists. Admitting a bare `:` token into the prefix list would cause the parser to maintain a "colon-as-prefix" hypothesis in parallel with "colon-as-type-annotation-terminal," and when that hypothesis eventually dies, error recovery spans across operation boundaries — cascading failures. Instead, the grammar handles `key: [...]` with a dedicated rule that treats the whole construct as a single unit.

---

## What You Get

Six [tree-sitter query files](https://github.com/tkamucheka/tree-sitter-mlir/blob/main/docs/queries.md) cover the standard editor feature surface:

| File | What it enables |
|---|---|
| `highlights.scm` | Syntax highlighting — dialect prefixes, op mnemonics, SSA values, types, attributes, keywords |
| `locals.scm` | SSA value scoping — rename, go-to-definition, reference highlighting |
| `indents.scm` | Auto-indent inside regions and attribute dictionaries |
| `folds.scm` | Code folding for regions, dicts, and `dense<...>` literals |
| `tags.scm` | Symbol index — `func.func`, `module`, type/attr alias definitions |
| `textobjects.scm` | Structural motions — select a function, a block, a parameter |

The highlight scheme distinguishes dialect prefixes (`func.` coloured as `@module`) from op mnemonics (`func` coloured as `@function.call`), SSA definitions from uses, and structural keywords (`module`, `dense`, `affine_map`) from bare identifiers. In practice this means you can tell at a glance what a token *is* rather than having to mentally parse the surrounding context.

---

## Installation (Neovim)

With [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) on the `main` branch and Lazy.nvim:

```lua
{
  "nvim-treesitter/nvim-treesitter",
  lazy = false,
  build = ":TSUpdate",
  config = function()
    vim.api.nvim_create_autocmd("User", {
      pattern = "TSUpdate",
      callback = function()
        require("nvim-treesitter.parsers").mlir = {
          install_info = {
            url = "https://github.com/tkamucheka/tree-sitter-mlir",
            files = { "src/parser.c" },
            queries = "queries",
            generate = true,
          },
        }
      end,
    })
    require("nvim-treesitter").setup()
  end,
}
```

Then `:TSInstall mlir` (or let `auto_install` handle it on first open).

For Python and Rust bindings, and for using the grammar programmatically, see the [README](https://github.com/tkamucheka/tree-sitter-mlir).

---

## State of the Grammar

The test suite covers 87 corpus tests drawn from a range of real-world MLIR dialects — `arith`, `func`, `scf`, `affine`, `memref`, `linalg`, `llvm`, `vector`, among others — and all pass. The grammar handles the full generic operation format, all builtin type and attribute forms, affine maps, dense literals, and the custom op prefix patterns that appear in these dialects.

What I'm less confident about: exotic dialect syntax I haven't encountered, and edge cases in the GLR disambiguation under pathological inputs. The grammar is permissive by design, it will parse things that aren't valid MLIR, so it won't catch semantic errors. It's a syntax tree, not a verifier.

If you hit a parse failure or a miscoloured token on real IR, please [open an issue](https://github.com/tkamucheka/tree-sitter-mlir/issues). That's the kind of testing I can't do alone.

One open question I keep returning to: would a hybrid approach make sense — use the upstream dialect-enumeration grammar for known dialects, and fall back to this general parser for everything else? It would likely yield more precise parse trees for the covered dialects. The cost is significant: you'd need to enumerate every upstream dialect, track new ones as they're merged, and keep two grammars in sync. For now, the general parser handles everything well enough that I'm not sure the maintenance burden is worth it, but I haven't ruled it out.

---

## Acknowledgements

Thanks to [artagnon/tree-sitter-mlir](https://github.com/artagnon/tree-sitter-mlir) for source snippets and test cases that helped validate the grammar, and to Claude for being an unusually patient pair programmer at midnight.
