---
layout: project
title: tree-sitter-dottr
project: tree-sitter-dottr
---

A [tree-sitter](https://tree-sitter.github.io/) grammar for
[Dottr](../dottr/) manifests (`*.dotfile`), with queries for:

* highlighting, including the scripts of `shell` blocks, highlighted as
  bash, zsh, or PowerShell;
* folds and indentation;
* text objects for units, blocks, conditionals, and variables.

Diagnostics, navigation, completion, and formatting come from the Dottr
language server, `dottr-lsp`.

## Neovim

With [gardener.nvim](../gardener.nvim/) and
[lazy.nvim](https://github.com/folke/lazy.nvim), gardener detects `*.dotfile`
files, installs the parser when a manifest is first opened, and starts
highlighting, folds, and indentation:

```lua
{
  "tkamucheka/gardener.nvim",
  dependencies = { "nvim-treesitter/nvim-treesitter" },
  lazy = false,
  opts = {
    ensure_installed = { "bash", "zsh", "powershell" },
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
    manage_folds = true,
    manage_indent = true,
  },
}
```

The `bash`, `zsh`, and `powershell` parsers highlight the scripts of `shell`
blocks. Setup without gardener, the language server, and the text objects
are described in the
[README](https://github.com/tkamucheka/tree-sitter-dottr#readme).

## Queries

| File | Purpose |
|------|---------|
| `highlights.scm` | Highlighting, with Neovim's standard capture names |
| `injections.scm` | Shell and PowerShell scripts, without the indentation of the block |
| `folds.scm` | Folds for units, blocks, branches, and scripts |
| `indents.scm` | Indentation |
| `textobjects.scm` | Text objects |
| `locals.scm`, `tags.scm` | Scopes, definitions, and references |
