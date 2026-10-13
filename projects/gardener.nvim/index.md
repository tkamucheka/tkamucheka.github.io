---
layout: project
title: gardener.nvim
project: gardener.nvim
---

Tree-sitter parser management for Neovim, built on the `main` branch of
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter), which
installs parsers but leaves starting them to the user. gardener:

* installs a list of parsers at startup, and the parser of any other
  filetype when a file of that type is first opened;
* registers parsers that nvim-treesitter does not provide, and keeps them
  registered across nvim-treesitter updates;
* detects the filetype of files that Neovim does not recognize, by extension
  or file name, for custom parsers;
* maps filetypes to the languages that parse them;
* starts highlighting, and optionally folds and indentation, in every buffer
  whose parser is installed, including a buffer whose parser was just
  installed.

## Installation

Requirements: Neovim 0.12 or later, and nvim-treesitter on its `main` branch,
with its requirements (`curl`, `tar`, the tree-sitter CLI, and a C compiler).

With [lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
{
  "tkamucheka/gardener.nvim",
  dependencies = { "nvim-treesitter/nvim-treesitter" },
  lazy = false,
  opts = {
    ensure_installed = { "lua", "vim", "vimdoc", "query" },
  },
}
```

With another plugin manager, install both plugins and call
`require("gardener").setup(opts)`.

## Custom Parsers

A parser that nvim-treesitter does not provide is registered with its
`install_info`, and optionally the filetypes, extensions, and file names that
use it:

```lua
opts = {
  custom_parsers = {
    dottr = {
      install_info = {
        url = "https://github.com/tkamucheka/tree-sitter-dottr",
        queries = "queries",
      },
      filetypes = { "dottr" },
      extensions = { "dotfile" },
    },
  },
}
```

Files ending in `.dotfile` are then detected as the filetype `dottr`, and
gardener installs and starts the `dottr` parser in them. All options are
described in the [README](https://github.com/tkamucheka/gardener.nvim#readme).
