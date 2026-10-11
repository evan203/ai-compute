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

## Proposal: Club Inference Cluster for HiveMind (~$15-20k)

Goal: always-on, multi-user inference for our free agentic product HiveMind, serving ~20 students in our free open-access tier with agentic hardware assistants to automate parts of their life.

We propose serving multiple independent `llama.cpp` instances across 3 servers (2x 192GB + 1x 128GB) behind a single authenticated gateway.

### Recommended BOM (~$18.6k)

- 2x GMKtec EVO-X5 Pro (Max+ PRO 495, 192GB, 2TB SSD) @ $6,800 = $13,600
  - Runs 157-163GB quants: GLM-5.3-Flash UD-IQ4_XS 157 GB, Qwen3.8-Flash-Next UD-Q5_K_XL 158 GB, MiMo-V2.6-Flash MXFP4 162.9 GB
- 1x GMKtec EVO-X2 (Max+ 395, 128GB, 1TB SSD) @ $3,500 = $3,500
  - Runs 90-94GB quants: GLM-5.3-Flash UD-IQ1_S 93.1 GB, Qwen3.8-Flash-Next UD-IQ4_XS 93.7 GB, MiMo-V2.6-Flash BPW2.5 90.1 GB
- Infra (~$1,500):
  - 8-port 2.5GbE managed switch
  - 1500VA UPS + smart PDU
  - Locking 12U cabinet / rack shelves
  - Cables, spare SSD, warranty extensions
- **Total: ~$18,600**

Heterogeneous sizing is intentional: 192GB nodes fit the largest Q4/Q5 quants, 128GB nodes fit the efficient Q1/IQ4 quants. Each node is 120-180W, <800W total for 3 nodes on standard 120V/15A - no special power or cooling.

Uniform alternative if preferred: 4x EVO-X2 128GB = $14,000 + $1,500 infra = ~$15,500, but cannot fit 157GB+ quants.

### Software Architecture

```text
HiveMind agents (20 per-user API keys)
  -> LiteLLM gateway (auth, rate-limit, load-balance, fallback)
    -> node1 192GB: llama-server :8080 (e.g. GLM-5.3 Q4, ctx 32k)
    -> node2 192GB: llama-server :8080 (e.g. Qwen3.8 Q5, ctx 64k)
    -> node3 128GB: llama-server :8080 (e.g. MiMo BPW2.5 / Qwen IQ4, ctx 32k)
```

- Base OS: Ubuntu 24.04 LTS
- Inference: `llama-server` in Docker, OpenAI-compatible `/v1/chat/completions`
- No cross-node sharding: each node is an independent replica holding a full model. Gateway does round-robin + fallback. If one node dies, others absorb load degraded.
- Gateway: LiteLLM on one node on a VM, per-student API keys, quotas (e.g. 30 req/min, 200k tok/user/day), audit logs
- HiveMind points to `https://hivemind.cs.wisc.edu/v1`
- Ops: Ansible playbooks in git, `unattended-security-upgrades`, monthly maintenance window, Prometheus/Grafana + Uptime-Kuma + Loki, log rotation

### Physical Location Options

Propose both, let school choose. Recommended: pilot in club room, then move to datacenter.

Option A - Datacenter hosted (lowest risk for IT):

- 3x mini-PCs shelf-mounted, ~4U + 2x UPS sockets
- IT provides rack space, power, cooling, asset tag
- IT holds host-level access if VMs are used, club operates inside
- Locked, monitored, backed power

Option B - Club room in CS building (club bare-metal managed):

- Locked cabinet, smart PDU + UPS, <45dB (no server fans)
- Needs 1x wired drop + power strip only
- Faculty sponsor holds key, 2-3 named student SREs operate
- No inbound ports required (see Networking)

### Network / Admin / Compliance Options

Networking (university network constraints: MAC/802.1X registration, no open inbound ports):

- Outbound-only mesh: nodes join Tailscale/WireGuard tailnet + Caddy reverse proxy with university DNS (`hivemind.cs.university.edu`)
- No SSH open to internet. SSH only over Tailscale + campus VLAN
- Single HTTPS API surface with per-user API keys
- Ask IT for: 1x VLAN, 3-4x static DHCP reservations, 1x DNS A record, Tailscale/WireGuard approval, log retention sign-off (propose 30d)

Admin (two models matching physical options):

- IT-administered: IT holds Proxmox/host root and network, club gets sudo inside 1 VM per physical box with iGPU passthrough. IT patches host, club patches guest via Ansible. Preferred if school wants full control.
- Club-administered bare-metal: faculty advisor + 2 leads hold root, full handover doc each semester, keys rotated on graduation, monthly patch window, status page IT can view. Preferred for speed/hands-on learning.
- Either way: Ubuntu 24.04 LTS, Docker images pinned, Ansible in git, monitoring visible to IT, IT kill-switch via gateway

Compliance / Security:

- Auth: campus SSO -> gateway-issued per-student API key, rate limits + quotas, abuse prevention (no crypto, no external resale, HiveMind free-tier only)
- Privacy: prompts not used for training, 30d retention for abuse debugging only, no PII in logs, all inference on-prem with no third-party API fallback without opt-in
- Governance: faculty advisor (owner), 2-3 student SREs (operators), IT liaison (host/network). Acceptable Use Policy signed by 20 users. Semester handover + incident runbook.

### Asks for University

1. Approve ~$18.6k capital: 2x 192GB + 1x 128GB + network/power/lock
2. Decide hosting: pilot in club room (Room XXX, jack Y) vs direct to datacenter (need U/power/VLAN survey)
3. Assign IT contact for MAC registration, DNS, Tailscale approval, VM-vs-bare-metal policy
4. Approve MOU: university owns assets, faculty-sponsored student operation, AUP + quotas for 20 free-tier users
5. Lifecycle: 3-yr warranty, 1 cold-spare SSD, <1hr re-image per node, e-waste via IT after 4-5yr
