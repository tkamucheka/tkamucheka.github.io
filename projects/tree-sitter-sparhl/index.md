---
layout: project
title: tree-sitter-sparhl
project: tree-sitter-sparhl
---

A [tree-sitter](https://tree-sitter.github.io/) grammar and highlighting
queries for SPARHL programs. A SPARHL program defines numbers and typed
tensors, such as matrices and vectors, and applies register instructions to
them:

```
; Activation function test:

r = 4
c = 4
m1 = matrix<41,4> [0.27070464, 0.42842184, ...]
v1 = vector<r> [0.39126588, 0.75008642, 0.29998157, 0.59304644]
v2 = vector<r>

LOADM R1, m1
LOADV R2, v1
MULT R3, R1, R2
ACCUMULATE R3
ACTIVATION <RELU> R3
STOREV v2, R3
```

The repository contains the grammar, its corpus tests, highlighting queries
(`queries/highlights.scm`), and an example program for an MNIST network
(`examples/mnist.spar`).
