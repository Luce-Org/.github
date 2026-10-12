<div align="center">
  <img src="https://www.lucebox.com/lucebox-logo.png" alt="Lucebox" width="160" />

  <h3>Your Inference Workstation</h3>

  <p>
    Lucebox builds the Lucebox Zero 495, a plug-and-play workstation for local AI inference, and the open source engine that runs on it.
    Private, always on, and compatible with the OpenAI and Anthropic APIs.
  </p>

  <p>
    <a href="https://www.lucebox.com"><img alt="Website" src="https://img.shields.io/badge/website-lucebox.com-000?style=for-the-badge" /></a>
    <a href="https://huggingface.co/Lucebox"><img alt="Hugging Face" src="https://img.shields.io/badge/models-hugging_face-FFD21E?style=for-the-badge&logo=huggingface&logoColor=black" /></a>
    <a href="https://x.com/luceboxai"><img alt="X" src="https://img.shields.io/badge/follow-@luceboxai-1DA1F2?style=for-the-badge&logo=x" /></a>
    <a href="https://discord.gg/yHfswqZmJQ"><img alt="Discord" src="https://img.shields.io/badge/chat-discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" /></a>
    <a href="mailto:support@lucebox.com"><img alt="Email" src="https://img.shields.io/badge/contact-us-D14836?style=for-the-badge&logo=gmail&logoColor=white" /></a>
  </p>
</div>

---

## What is Lucebox

The Lucebox Zero 495 is an 11.97 L aluminium workstation that runs large open models on your desk. Inside: an AMD Radeon™ AI PRO R9700 (32 GB GDDR6, 640 GB/s) next to an AMD Ryzen™ AI MAX+ PRO 495 with 128 GB or 192 GB of LPDDR5X unified memory (273 GB/s), a 2 TB or 4 TB NVMe drive and a 1,000 W 80 PLUS Platinum SFX power supply. It has 10 GbE, Wi-Fi 7 and an optional 100 GbE cluster kit. It runs Ubuntu Server headless with the engine and tuned models already installed, and its OpenAI and Anthropic compatible API works with Claude Code, Codex, OpenCode, Hermes, OpenClaw, Pi and Open WebUI without changes.

**lucebox** is the open source inference engine behind it: speculative decoding, speculative prefill and custom kernels, tuned for each model and chip. It isn't tied to our hardware. It also runs on Strix Halo, the Radeon RX 7900 XT and XTX, NVIDIA cards from the RTX 2080 Ti to the RTX 5090, DGX Spark and Jetson AGX Thor.

## Why local

* **Privacy.** Prompts, files and weights stay on the machine. Once a model is loaded it runs fully offline, and the internet is only needed for updates.
* **Cost.** You pay for the hardware once and there is no per-token bill.
* **Speed.** Up to 227.8 tok/s on Qwen 3.8 27B on the R9700, and 300.9 tok/s in total across five clients.
* **Big models.** DeepSeek V4.1 Flash, a 383 GB model, runs across the R9700, the unified memory and the SSD, and the R9700 makes it up to 3x faster than the processor alone.
* **Open.** Apache 2.0 engine, GGUF weights, root access over SSH, no lock-in.

## Open source

### Inference engine

| Repository | Description | Stars | Forks |
|------------|-------------|-------|-------|
| [**lucebox**](https://github.com/Luce-Org/lucebox) | Speculative inference server for heterogeneous machines and consumer GPUs. OpenAI and Anthropic compatible HTTP server. | ![Stars](https://img.shields.io/github/stars/Luce-Org/lucebox?style=flat-square&label=) | ![Forks](https://img.shields.io/github/forks/Luce-Org/lucebox?style=flat-square&label=) |

### Engine optimizations

| Component | What it does | Measured result |
|-----------|--------------|-----------------|
| **DFlash2** | Speculative decoding with a block-diffusion drafter | 208.1 tok/s average, 227.8 peak, Qwen 3.8 27B on one R9700 (3.8x llama.cpp with the same drafter) |
| **DSpark** | Speculative decoding for DeepSeek V4 Flash | Up to 32.7 tok/s on DeepSeek V4 Flash 0731, Strix Halo |
| **PFlash** | Speculative prefill: a small drafter compresses long prompts before the target reads them | 6.1x prefill at 256K on Laguna XS 2.1 33B (411 s to 67.3 s), RTX 3090 |
| **KVFlash** | Keeps a small KV pool on the GPU and pages cold 64-token chunks to host RAM, bit-exact | 152.3 tok/s at 256K context with an 8K pool, RTX 3090 |
| **Spark** | Keeps the active MoE experts in VRAM and swaps the rest | Laguna XS.2 33B at about 100 tok/s in 14.6 GiB |
| **Three-tier experts** | Spreads a MoE model across the GPU, unified memory and the SSD | DeepSeek V4.1 Flash at 48.7 tok/s on code, R9700 + 192 GB Ryzen AI MAX+ PRO 495 |
| **Continuous batching** | Paged attention and a shared KV pool for concurrent requests | 300.9 tok/s total at 5 clients, Qwen 3.8 27B on R9700, up to 1.97x llama.cpp |
| **Tool prefix cache** | Restores the tool definitions agents resend every turn and processes only the new messages | 48x faster warm prefill (50.35 s to 1.04 s), Qwen 3.6 27B |
| **Megakernel** | All 24 layers of a small model fused into one persistent CUDA kernel | 413 tok/s and 1.87 tok/J on Qwen 3.5 0.8B, RTX 3090 |

## Quick start

Prebuilt images on GHCR track `main`. Put the target model in `server/models/` and its drafter in `server/models/draft/`, then run the image for your GPU:

```bash
# NVIDIA (CUDA 12+)
docker run --rm --gpus all -p 8000:8080 \
  -v "$PWD/server/models:/opt/lucebox-hub/server/models" \
  ghcr.io/luce-org/lucebox-hub:cuda12

# AMD (ROCm 6+)
docker run --rm --device /dev/kfd --device /dev/dri \
  --group-add video --group-add render --security-opt seccomp=unconfined \
  -p 8000:8080 -v "$PWD/server/models:/opt/lucebox-hub/server/models" \
  ghcr.io/luce-org/lucebox-hub:rocm
```

Then point any OpenAI or Anthropic compatible client at `http://localhost:8000/v1`. To build from source, follow [Run the Server](https://github.com/Luce-Org/lucebox#run-the-server).

## Benchmarks

| Model | Hardware | Result | Report |
|-------|----------|--------|--------|
| Qwen 3.8 27B + DFlash2 | R9700 | 227.8 tok/s peak, 208.1 HumanEval average | [qwen38-r9700](https://www.lucebox.com/blog/qwen38-r9700) |
| Qwen 3.8 27B, 5 clients | R9700 | 300.9 tok/s total | [continuous-batching](https://www.lucebox.com/blog/continuous-batching) |
| DeepSeek V4.1 Flash, 383 GB | R9700 + Ryzen AI MAX+ PRO 495, 192 GB | 48.7 tok/s on code, about 1,000 tok/s prefill | [deepseek-v41](https://www.lucebox.com/blog/deepseek-v41-lucebox-memory-hierarchy) |
| Qwen 3.8 Flash-Next | R9700 + Strix Halo | 75.5 tok/s decode, 1,820 tok/s prefill at 16K | [guide](https://github.com/Luce-Org/lucebox/blob/main/server/docs/QWEN4EXP.md) |
| Image questions, two models at once | R9700 + Strix Halo | 58 a minute, 3.2x a DGX Spark | [vision-llm-inference](https://www.lucebox.com/blog/vision-llm-inference) |
| Laguna XS 2.1 33B | RTX 3090 | 296 tok/s peak, 152 tok/s at 256K | [laguna-xs21](https://www.lucebox.com/blog/laguna-xs21) |
| Qwen 3.5 0.8B | RTX 3090 | 413 tok/s | [megakernel](https://www.lucebox.com/blog/megakernel) |

Each run uses its own prompts, quantization and settings, so the rows don't rank the hardware against each other.

## Supported hardware

* **AMD:** Radeon AI PRO R9700 (ROCm 7.2), Ryzen AI MAX+ 395 Strix Halo (ROCm 7.2), Ryzen AI MAX+ PRO 495, Radeon RX 7900 XT and XTX (ROCm 6+)
* **NVIDIA:** RTX 2080 Ti, 3090, 4090, 5090, V100, P40, DGX Spark (CUDA 12+), Jetson AGX Thor (CUDA 13)
* **Mixed devices:** one model across CUDA and HIP devices, such as R9700 + Strix Halo or 2x RTX 3090 with NVLink
* **OS (inference engine):** Linux, Windows through WSL2
* **OS (Lucebox):** Ubuntu Server, headless, pre-tuned

## Buy the hardware

The Lucebox Zero 495 ships with the engine and tuned models installed. It starts at $6,499 for a limited time, $500 off the $6,999 list price, with 128 GB and 2 TB. You can add 192 GB of memory (+$2,000), a 4 TB drive (+$250) and the 100 GbE cluster kit (+$799 per machine). Every machine has a 72-hour burn-in and a 1-year warranty on parts and labor. The first deliveries are expected in January 2027, and 192 GB machines in February 2027.

Order at [**lucebox.com**](https://www.lucebox.com).

## Resources

* [Product site](https://www.lucebox.com)
* [Hardware specs](https://www.lucebox.com/hardware)
* [Engine and results](https://www.lucebox.com/engine)
* [Lucebox Compared](https://www.lucebox.com/compare)
* [Blog](https://www.lucebox.com/blog)
* [DeepSeek V4.1 Flash on the Lucebox memory hierarchy](https://www.lucebox.com/blog/deepseek-v41-lucebox-memory-hierarchy)
* [Continuous batching](https://www.lucebox.com/blog/continuous-batching)
* [Vision LLM inference vs DGX Spark](https://www.lucebox.com/blog/vision-llm-inference)
* [Qwen 3.8 27B on the R9700](https://www.lucebox.com/blog/qwen38-r9700)
* [DeepSeek V4 Flash 0731 on Strix Halo](https://www.lucebox.com/blog/deepseek-v4-flash-0731)
* [Tool prefix caching](https://www.lucebox.com/blog/tool-prefix-cache)
* [KVFlash at 256K](https://www.lucebox.com/blog/kvflash)
* [Laguna XS 2.1 on a RTX 3090](https://www.lucebox.com/blog/laguna-xs21)
* [Lucebox in a container](https://www.lucebox.com/blog/docker)
* [Luce Spark](https://www.lucebox.com/blog/spark)
* [Client harnesses](https://www.lucebox.com/blog/client-harnesses)
* [PFlash speculative prefill](https://www.lucebox.com/blog/pflash)
* [DFlash on 27B](https://www.lucebox.com/blog/dflash27b)
* [Megakernel decode](https://www.lucebox.com/blog/megakernel)
* [Models on Hugging Face](https://huggingface.co/Lucebox)
* [Issue tracker](https://github.com/Luce-Org/lucebox/issues)

<p align="center">
  <a href="https://www.lucebox.com">Website</a> •
  <a href="https://github.com/Luce-Org/lucebox">GitHub</a> •
  <a href="https://huggingface.co/Lucebox">Hugging Face</a> •
  <a href="https://x.com/luceboxai">X</a> •
  <a href="https://discord.gg/yHfswqZmJQ">Discord</a>
</p>

<p align="center"><sub>Apache 2.0. Designed in Italy.</sub></p>
