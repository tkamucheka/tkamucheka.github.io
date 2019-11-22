---
layout: post
title: VSCode C/C++ Plugin Formatting Fails 
date: November 21, 2019
author: Tendayi Kamucheka
category: blog
---

Lately, I've been doing a lot of programming in C++ for my work. I have also switched my editor of choice from Atom to VS Code. One of my favorite plugins on VS Code is C/C++ [`ms-vscode.cpptools`](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools). This is plugin offers great features like code auto-formatting, IntelliSense (code suggestions), and syntax highlighting for C and C++. These features are reminiscent of working in Visual Studio and for a text editor, that's plenty.

Since Ubuntu 19.04, there is an annoying bug in the extension where it fails to auto-format code on a fresh install of Ubuntu. I have experienced this with both Ubuntu 19.04 and 19.10.

Thankfully, the cause of this bug is described here: [Formatting Failed C/C++ No info in Output window · Issue #3271 · Microsoft/vscode-cpptools · GitHub](https://github.com/Microsoft/vscode-cpptools/issues/3271). Basically, the plugin looks for __libtinfo.so.5__ which is not present on Ubuntu 19.04 and after. Auto formatting fails when the plugin fails to locate the required lib. A solution is described in the comments on the same issue page. Seems sym-linking __libtinfo.so.6__ to __libtinfo.so.5__ solves the issue.

``` bash
$ sudo apt install libncurses5-dev
$ sudo ln -s /usr/lib/x86_64-linux/libncurses.so.6 \
 /usr/lib/x86_64-linux/libncurses.so.5
```

Everything should work fine after doing this. Happy coding!