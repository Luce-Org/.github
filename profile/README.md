<div align="center">
  <img src="https://www.lucebox.com/lucebox-logo.png" alt="Lucebox" width="160" />

  <h3>The Personal Computer for AI Agents</h3>

  <p>
    Lucebox builds plug-and-play hardware and the open inference stack that powers it.
    Local-first, OpenAI/Anthropic-compatible, 4 to 6x faster than competing boxes at the same price.
  </p>

  <p>
    <a href="https://www.lucebox.com"><img alt="Website" src="https://img.shields.io/badge/website-lucebox.com-000?style=for-the-badge" /></a>
    <a href="https://x.com/luceboxai"><img alt="X" src="https://img.shields.io/badge/follow-@luceboxai-1DA1F2?style=for-the-badge&logo=x" /></a>
    <a href="https://discord.gg/yHfswqZmJQ"><img alt="Discord" src="https://img.shields.io/badge/chat-discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" /></a>
    <a href="mailto:davide@lucebox.com"><img alt="Email" src="https://img.shields.io/badge/contact-us-D14836?style=for-the-badge&logo=gmail&logoColor=white" /></a>
  </p>
</div>

---

## What is Lucebox

Lucebox is a 9.56L aluminum box that runs frontier open models locally at speeds people thought needed cloud GPUs. Inside: an RTX 3090 (24GB GDDR6X) paired with an AMD Ryzen AI MAX+ 395 APU (128GB unified LPDDR5X), 2TB NVMe, Corsair 750W 80+ Gold. Outside: a single power cable and an OpenAI/Anthropic-compatible endpoint reachable from Claude Code, Codex, OpenCode, Hermes, OpenClaw, Open WebUI, and Ollama in roughly one minute from unboxing.

The speed comes from this org. **lucebox-hub** is the open inference engine: custom CUDA kernels, speculative decoding (DFlash), speculative prefill compression (PFlash), and persistent megakernels. It runs on any RTX 30/40/50, on Strix Halo, and on Radeon 7900 XTX, not just on our hardware.

## Why local

* **Privacy.** Prompts and weights never leave the device. Default-fit for legal, medical, finance, and any team where the data is the moat.
* **Cost.** $4,900 once, then zero per token. Replaces $200 to $2,000 per month in cloud API spend for sustained agent workloads.
* **Throughput.** Up to 207 tok/s on Qwen3.5-27B and 134 tok/s at 128K context, matching or beating cloud latency on a desk.
* **Open.** Apache 2.0 inference stack, GGUF models, no vendor lock-in.

## Open Source

### Inference Engine

| Repository | Description | Stars | Forks |
|------------|-------------|-------|-------|
| [**lucebox-hub**](https://github.com/Luce-Org/lucebox-hub) | Fast LLM speculative inference server for consumer hardware. DFlash + PFlash + Megakernel. OpenAI/Anthropic compatible HTTP server. | ![Stars](https://img.shields.io/github/stars/Luce-Org/lucebox-hub?style=flat-square&label=) | ![Forks](https://img.shields.io/github/forks/Luce-Org/lucebox-hub?style=flat-square&label=) |
| [**llama.cpp-dflash-ggml**](https://github.com/Luce-Org/llama.cpp-dflash-ggml) | llama.cpp fork with DFlash speculative decode and ggml DDTree integration. Upstream-tracking. | ![Stars](https://img.shields.io/github/stars/Luce-Org/llama.cpp-dflash-ggml?style=flat-square&label=) | ![Forks](https://img.shields.io/github/forks/Luce-Org/llama.cpp-dflash-ggml?style=flat-square&label=) |

### Lucebox Inference Optimizations

| Component | What it does | Speedup |
|-----------|--------------|---------|
| **DFlash** | Speculative decode with draft model + tree verification (DDTree) | 3 to 5x on 27B |
| **PFlash** | Block-sparse speculative prefill, register-resident FA-2 kernels | ~5.6x on long context, 5.4x at 128K |
| **Megakernel** | Fused 24-layer persistent CUDA kernel for small drafts | ~2x on 0.8B (413 tok/s) |

## Quick Start

```bash
git clone --recurse-submodules https://github.com/Luce-Org/lucebox-hub
cd lucebox-hub
cmake -B server/build -S server -DCMAKE_CUDA_ARCHITECTURES=86
cmake --build server/build --target dflash_server -j

./server/build/dflash_server \
  model.gguf \
  --draft draft.gguf \
  --port 8000
```

Then point any OpenAI-compatible client at `http://localhost:8000/v1`.

## Benchmarks

| Model | Hardware | Throughput | Method |
|-------|----------|-----------|--------|
| Qwen3.5-27B AWQ | RTX 3090 | 207 tok/s | DFlash + DDTree |
| Qwen3.6-27B Q4_K_M | RTX 3090 | 134 tok/s @ 128K | PFlash sliding target_feat |
| Laguna-XS.2 33B | RTX 3090 | 5.4x @ 128K | PFlash |
| Qwen3.5-0.8B | RTX 3090 | 413 tok/s | bf16 Megakernel |
| gfx1151 iGPU (Strix Halo) | Ryzen AI MAX+ 395 | 26.85 tok/s | HIP, 2.23x vs llama.cpp HIP |

## Supported Hardware

* **NVIDIA:** RTX 3090, RTX 4090, RTX 5090, RTX 2080 Ti (CUDA 12+)
* **AMD:** Ryzen AI MAX+ 395 Strix Halo (HIP / ROCm 6+), RX 7900 XTX (HIP)
* **OS (inference engine):** Linux, Windows
* **OS (Lucebox appliance):** Linux, pre-tuned

## Buy the Hardware

The plug-and-play Lucebox ships pre-tuned with the full stack loaded. $4,900, one year warranty, refurbished and fully serviced RTX 3090.

→ [**lucebox.com**](https://www.lucebox.com)

## Resources

* [Product Site](https://www.lucebox.com)
* [Blog](https://www.lucebox.com/blog)
* [DFlash on 27B](https://www.lucebox.com/blog/dflash27b)
* [PFlash Speculative Prefill](https://www.lucebox.com/blog/pflash)
* [Megakernel Decode](https://www.lucebox.com/blog/megakernel)
* [Laguna-XS.2 @128K](https://www.lucebox.com/blog/laguna)
* [Gemma vs DeepSeek](https://www.lucebox.com/blog/gemma-vs-deepseek)
* [Client Harnesses](https://www.lucebox.com/blog/client-harnesses)
* [AMD Strix Halo Notes](https://www.lucebox.com/blog/amd)
* [eGPU Myth](https://www.lucebox.com/blog/egpu-myth)
* [Issue Tracker](https://github.com/Luce-Org/lucebox-hub/issues)

<p align="center">
  <a href="https://www.lucebox.com">Website</a> •
  <a href="https://github.com/Luce-Org/lucebox-hub">GitHub</a> •
  <a href="https://x.com/luceboxai">X</a> •
  <a href="https://discord.gg/yHfswqZmJQ">Discord</a>
</p>

<p align="center"><sub>Apache 2.0. Built in Italy.</sub></p>
