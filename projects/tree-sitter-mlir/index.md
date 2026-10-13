---
layout: project
title: tree-sitter-mlir
project: tree-sitter-mlir
---

A [tree-sitter](https://tree-sitter.github.io/tree-sitter/) grammar for
[MLIR](https://mlir.llvm.org/), the Multi-Level Intermediate Representation.
The grammar follows the generic MLIR textual format of the
[MLIR Language Reference](https://mlir.llvm.org/docs/LangRef/), and handles
the custom operation formats of dialects without enumerating the dialects.

The existing parser works from a fixed list of dialects and their grammars.
This one asks whether a parser derived from MLIR's EBNF alone can handle
arbitrary dialects; the [blog post]({% post_url 2026-06-23-tree-sitter-mlir %})
tells the story. It has been tested, but not exhaustively.

## Language Coverage

* **Generic operations**: the `"opname"(args) {attrs} : functype` form.
* **Custom operations**: dialect-specific formats with arbitrary prefix
  tokens, type annotations, regions, and successor blocks.
* **Types**: builtin scalar and parametric types (`tensor`, `memref`,
  `vector`, `complex`, `tuple`), dialect types (`!namespace.name<...>`), type
  aliases, and function types.
* **Attributes**: all builtin attribute kinds (`dense`, `dense_resource`,
  `sparse`, `opaque`, `strided`, `affine_map`, `affine_set`, array,
  dictionary), dialect attributes, and attribute aliases.
* **SSA values**: `%id`, result lists, and block arguments with their types
  and attributes.
* **Regions and blocks**: entry blocks, labeled blocks with arguments, and
  block successors.
* **Symbol references**: `@name`, with nested `::` qualification.

## Queries

Highlighting, SSA value scopes for rename and go to definition, indentation,
folds, symbol tags, and text objects, in `queries/`.

## Installation

In Neovim with nvim-treesitter (`main` branch), register the parser in a
`User TSUpdate` autocommand:

```lua
vim.api.nvim_create_autocmd("User", {
  pattern = "TSUpdate",
  callback = function()
    require("nvim-treesitter.parsers").mlir = {
      install_info = {
        url = "https://github.com/tkamucheka/tree-sitter-mlir",
        queries = "queries",
        generate = true,
      },
    }
  end,
})
```

Bindings for Python and Rust, and the development workflow, are described in
the [README](https://github.com/tkamucheka/tree-sitter-mlir#readme).
