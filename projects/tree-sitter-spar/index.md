---
layout: project
title: tree-sitter-spar
project: tree-sitter-spar
---

A [tree-sitter](https://tree-sitter.github.io/) grammar for SPAR assembly
(`*.spar`), the assembly language of the
[SPAR](https://github.com/SPAR-Public) processor, with highlighting queries.
It supersedes tree-sitter-sparhl, the grammar of the earlier SPARHL
language.

```
; Multiply a matrix by a vector and add a vector.
v1 = vector<8> [1,2,3,4,5,6,7,8]
v2 = vector<8> [1,2,3,4,5,6,7,8]
out = vector<8,8>

LOADM     R1, m
LOADROWV  R2, v1
MULT      R3, R1, R2
REDUCE    R4, R3
LOADCOLV  R5, v2
ADD       R6, R4, R5
STORECOLV R6, out
```

## Language Coverage

* **Definitions** of numbers and typed tensors, with literals that may span
  lines, and `#include` of other files.
* **Instructions**: load, store, arithmetic, reduction, transposition, and
  activation operations on registers and tensors, with labels.
* **Indexed operands**: `m[i,j][tile,tile][1,1]`.
* **Structured loops**: `FOR ... TO ... STEP ... END`.
* **Affine maps** and bindings: `#tile = MAP<d0, s0 -> FLOORDIV(d0, s0)>` and
  `n = MIN #tile(i, 4)`.

## Neovim

With [gardener.nvim](../gardener.nvim/), gardener detects `*.spar` files,
installs the parser when a file is first opened, and starts highlighting:

```lua
opts = {
  custom_parsers = {
    spar = {
      install_info = {
        url = "https://github.com/tkamucheka/tree-sitter-spar",
        queries = "queries",
      },
      extensions = { "spar" },
    },
  },
}
```

Setup with nvim-treesitter alone, and the bindings for C, Go, Node.js,
Python, Rust, and Swift, are described in the
[README](https://github.com/tkamucheka/tree-sitter-spar#readme).
