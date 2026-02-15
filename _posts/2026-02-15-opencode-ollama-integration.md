---
id: 2026-02-15-opencode-ollama-integration
aliases:
  - Useful OpenCode Ollama Integration
tags:
  - OpenCode
  - Ollama
  - LLM
  - Agentic Coding
  - Context Window
  - Custom Models
author: Tendayi Kamucheka
category: blog
date: February 15, 2026
layout: post
title: Optimizing Local LLM Context for Agentic Coding with Ollama and OpenCode
---

My journey of integrating Large Language Models (LLMs) into my daily development workflow continues to evolve. Recently, I’ve been focused on bridging the gap between **Ollama**, my preferred local LLM platform, and **OpenCode**. While the promise of a local, private coding agent is enticing, the reality of the initial setup was, frankly, underwhelming.

If you’ve followed the standard guides and arrived at the `ollama launch opencode` stage only to find your "agent" is more of a "waiter that forgets the order," this post is for you.

## The Symptoms of a "Lobotomized" Model

You might have noticed that even with a powerful model, the agentic features in OpenCode often fail in bizarre ways. In my testing, I encountered several "red flags" that indicated the integration wasn't working as intended:

- **The JSON Cliff**: The model starts responding with a short JSON snippet and then abruptly goes silent.
- **Reasoning Loops**: Getting stuck in `<thinking>...</thinking>` or repeating the same suggestion—often claiming you haven't specified a task when you clearly have.
- **Permission Paralysis**: The model fails to execute basic commands like `ls` or `git`, or can't read files from the filesystem despite having explicit permissions in the Ollama configuration.

## The Context Window Trap

The culprit isn't usually the model's intelligence, but its memory. By default, Ollama uses a **context window of 4096 tokens**.

When using an agentic editor like OpenCode, that window is shared between the conversation history, the list of available tools, the system prompt, and your actual code. If you want to diagnose this yourself, stop your Ollama service and run `ollama serve` in a terminal. When you trigger a prompt in OpenCode, you’ll likely see a "200" response followed by a warning that your prompt was truncated to fit the context window.

## The Solution: Creating a Custom Agent Model

Setting the context size in `opencode.json` isn't enough; that only tells the editor when to start compacting the conversation. To actually fix this, you need to create a custom model in Ollama with an expanded context parameter.

---

### Step 1: Create a Modelfile

Run the following in your terminal to define a larger context (I recommend starting significantly higher than 4096):

```bash
echo "FROM [base-model]\nPARAMETER num_ctx [bigger-model-size]" > Modelfile
```

### Step 2: Build and Launch

```bash
ollama create qwen3-agent -f Modelfile
ollama launch opencode --model qwen3-agent
```

---

## My Setup

I’m currently running this setup with **20GB of VRAM**.

* **Base Model**: `qwen3:14b` (~9GB on disk).
* **Custom Model**: `qwen3-agent` with **40,960 tokens** of context.
* **Performance**: This setup consumes about 16GB of VRAM. A major perk of Ollama is that it automatically handled the split across my two GPUs without any manual ROCm tweaking.

### Keep in Mind

* **VRAM is King**: Larger context windows demand more memory. You are limited by your hardware.
* **Model Ceilings**: Every model has a hard limit. I found 40,960 to be the "sweet spot" for `qwen3:14b`; anything higher and the model would default back to that value anyway.

## Final Thoughts

While this local setup is great for saving on API costs for simple file manipulations and shell commands, it still doesn't quite replace the heavy lifting of a remote model like Gemini or Claude for massive codebases. However, it’s a massive step forward for privacy-conscious development.

**In my next post**: I had a mind-blowing interaction with Gemini Pro’s new personalization settings this morning that I can’t wait to break down. Stay tuned!
