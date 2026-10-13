---
layout: project
title: vscode-spar
project: vscode-spar
---

Visual Studio Code support for SPAR assembly (`*.spar`), the assembly
language of the [SPAR](https://github.com/SPAR-Public) processor:

* highlighting of comments, strings, opcodes, tensor types, registers,
  labels, numbers, and operators;
* semantic highlighting, including affine map names (`#tile`) and the
  dimension and symbol identifiers of maps (`d0`, `s0`);
* snippets for definitions, loops, and common operations;
* comment toggling, bracket matching, and auto-closing of brackets and
  quotes.

For Neovim, see [tree-sitter-spar](../tree-sitter-spar/).

## Installation

Requirements: VS Code 1.90 or later, and Node.js to build the extension
package:

```sh
git clone https://github.com/tkamucheka/vscode-spar.git
cd vscode-spar
npm run package:vsix
code --install-extension spar-assembly-0.0.1.vsix
```

The snippets are listed in the
[README](https://github.com/tkamucheka/vscode-spar#readme).
