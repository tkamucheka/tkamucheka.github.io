---
layout: post
title: Running Ollama on Mixed AMD GPUs (RX 7700 XT + RX 6600)
id: 2026-02-08-ollama-dual-rocm-gpu
aliases: []
tags: [rocm, ollama, amd, gpu]
---

Lately I have been looking for more ways to integrate LLMs and agents into my workflow. Outside of the code I actually want to write, I am often faced with tasks I would categorize as laborious chores. This feels like a good fit for agents but I have also been thinking about privacy when using commercial offerings, so I have been exploring locally hosted small models.

This post documents how I got Ollama working on a system with two different AMD GPUs: an RX 7700 XT (supported) and an RX 6600 (nominally unsupported). The key detail is how `HSA_OVERRIDE_GFX_VERSION_%d`, being 1-indexed, behaves with multiple devices, and how that differs from other ROCm-related environment variables.

## Hardware and Symptoms

I run two GPUs in the same host:

- RX 7700 XT
- RX 6600

Initially, Ollama only initialized the 7700 XT. The 6600 was not picked up, despite ROCm being installed and functional for the primary GPU. I found a number of guides explaining how to override the GFX version for the RX 6600, but they assumed a single GPU. I needed to handle two different GPUs at once, with different GFX versions.

## What Actually Worked

The fix was to set the per-GPU override form of the variable. In single-GPU setups the variable is just `HSA_OVERRIDE_GFX_VERSION`. For multi-GPU setups it becomes `HSA_OVERRIDE_GFX_VERSION_[GPU]`, where the GPU index is 1-based in my testing. With two devices, the correct mapping was:

- `HSA_OVERRIDE_GFX_VERSION_1=11.0.1` for the RX 7700 XT (primary)
- `HSA_OVERRIDE_GFX_VERSION_2=10.3.0` for the RX 6600

This is the opposite of what I expected at first. Most ROCm device selection variables are 0-indexed (for example, `HIP_VISIBLE_DEVICES` and `ROCR_VISIBLE_DEVICES`). In my testing, `HSA_OVERRIDE_GFX_VERSION_%d` is 1-indexed. That distinction was the key to getting the second GPU online.

Based on my findings, this text in the Ollama docs is inaccurate:

> If you have multiple GPUs with different GFX versions, append the numeric device number to the environment variable to set them individually. For example, HSA_OVERRIDE_GFX_VERSION_0=10.3.0 and HSA_OVERRIDE_GFX_VERSION_1=11.0.0

I did not need to set `HIP_VISIBLE_DEVICES` or `ROCR_VISIBLE_DEVICES` for this configuration.

## Service and Shell Configuration

For systemd, I set the variables in an Ollama drop-in using `systemctl edit ollama.service`:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
Environment="OLLAMA_ORIGINS=*"
Environment="HSA_OVERRIDE_GFX_VERSION_1=11.0.1" # RX 7700 XT
Environment="HSA_OVERRIDE_GFX_VERSION_2=10.3.0" # RX 6000
```

For interactive shells, I used the same overrides in my profile:

```sh
# ROCm
export HSA_OVERRIDE_GFX_VERSION_1=11.0.1
export HSA_OVERRIDE_GFX_VERSION_2=10.3.0
```

## Validating Both GPUs Are Active

The relevant signal in `rocm-smi` is the device list and VRAM usage. On a working setup, both devices should be visible and reported with non-zero VRAM percentages under load. Here is a trimmed example of the output, with the columns that matter:

```
Device  Node  IDs           ...  Power  ...  VRAM%  GPU%
0       1     0x747e, 46094 ...  59.0W  ...  70%    1%
1       2     0x73ff, 35521 ...  39.0W  ...  69%    43%
```

## Sources

One comment on Ollama's Github suggested that `HSA_OVERRIDE_GFX_VERSION_%d` was the variable to look at but was not implemented. I tried it anyway and stumbled on the correct setting. The discussion was still useful for surfacing the variable and led to the right documentation.

A concise RX 6600 ROCm guide was also helpful for narrowing down the supported GFX versions and testing overrides. I used these references for context and verification:

- [AMD ROCm Installation Guide on RX6600 + Ollama](https://gist.github.com/furaar/ee05a5ef673302a8e653863b6eaedc90)
- [Running Ollama with an AMD Radeon 6600 XT](https://major.io/p/ollama-with-amd-radeon-6600xt/)
- [HSA_OVERRIDE_GFX_VERSION_0 while running on only one GPU #8473](https://github.com/ollama/ollama/issues/8473)

## Takeaway

Multi-GPU on Linux with AMD GPUs and ROCm is achievable, even when one device is an RX 6600 that is not yet officially supported. The key enabler in my setup was correctly applying per-device GFX overrides so both GPUs would initialize under Ollama. Next, I plan to integrate my Ollama backend with [99.nvim](https://github.com/ThePrimeagen/99). It is still in alpha, but I already like its different approach to AI-assisted coding.
