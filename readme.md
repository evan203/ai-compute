# AI Compute Research

## Models

Models in the roughly 90 to 160 GB size (after quantization), ranked by
[Artificial Analysis Intelligence Index](https://artificialanalysis.ai/models):

| Model | Intelligence Index | Quant for 128GB | Quant for 192GB |
| --------------- | --------------- | --------------- | --------------- |
| [GLM 5.3 Flash](https://huggingface.co/unsloth/GLM-5.3-Flash-GGUF) (320 B total, 18 B active)| 42 | UD-IQ1_S: 93.1 GB | UD-IQ4_XS: 157 GB |
| [Qwen 3.8 Flash Next](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF) (176 B total, 6 B active) | 40 | UD-IQ4_XS: 93.7 GB | UD-Q5_K_XL: 158 GB |
| [MiMo V2.6 Flash](https://huggingface.co/AesSedai/MiMo-V2.6-Flash-MOPD-GGUF) (309 B total, 15 B active) | 38 | BPW2.5: 90.1 GB | MXFP4: 162.9 GB |

## Hardware

Currently, I cant find any [DGX Spark](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/?superchip=GB10)s in stock

- [NVIDIA Blackwell GB10](https://docs.nvidia.com/dgx/dgx-spark/hardware.html) 128 GB LPDDR5x unified system memory, 256-bit interface, 4266 MHz, 273 GB/s bandwidth

I can find some AMD Ryzen options however:

- [AMD Ryzen™ AI Max+ PRO 495](https://www.amd.com/en/products/processors/laptop/ryzen-pro/ai-max-pro-400-series/amd-ryzen-ai-max-plus-pro-495.html): up to 192 GB 256-bit LPDDR5x-8533
- [AMD Ryzen™ AI Max+ 395](https://www.amd.com/en/products/processors/laptop/ryzen/ai-300-series/amd-ryzen-ai-max-plus-395.html): up to 128 GB 256-bit LPDDR5x-8000

Options for AMD Ryzen are:

- Max+ 395 - 128GB Memory (8000 MHz), 1TB SSD
  - $3,738: ([Framework Desktop](https://frame.work/desktop?slug=desktop-diy-amd-aimax300&tab=specs))
  - $3,500: [GMKtec EVO-X2](https://www.gmktec.com/products/amd-ryzen%E2%84%A2-ai-max-395-evo-x2-ai-mini-pc?variant=48746069262490)
- Max+ PRO 495 - 192GB Memory, 1TB SSD
  - $7,069: ([Framework Desktop](https://frame.work/desktop?slug=desktop-diy-amd-aimax400&tab=specs)) (pre-order, ships Nov)
  - $6,800: [GMKtec EVO-X5 Pro](https://www.gmktec.com/products/gmktec-evo-x5-pro-amd-ryzen-ai-max-pro-495-ai-mini-pc) (2TB SSD)
