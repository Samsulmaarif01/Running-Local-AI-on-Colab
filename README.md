# Running Local AI on Colab

Run **Ollama + Open WebUI** on Google Colab for free, access from anywhere via Cloudflare Tunnel, and integrate with **opencode CLI** as your AI coding assistant.

---

## Features

- **Ollama** — Run local AI models (Qwen2.5-Coder, Llama 3.1, DeepSeek-R1, Gemma 4, etc.) on Colab's GPU (T4)
- **Open WebUI** — ChatGPT-like web interface for chatting with models
- **Cloudflare Tunnel** — Public access without registration, port forwarding, or DNS config
- **Model Persistence** — Models auto-save to Google Drive so you don't re-download every session
- **opencode CLI Ready** — Ollama API tunnel works directly with opencode as your AI provider

---

## Usage

### 1. Open in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/[username]/[repo]/blob/main/Colab_Ai.ipynb)

Or manually upload `Colab_Ai.ipynb` to [Google Colab](https://colab.research.google.com).

### 2. Run the Notebook

Execute the 4 cells in order:

| Cell | Function | Duration |
|------|----------|----------|
| **Cell 1** | Install dependencies (Ollama, Open WebUI, cloudflared) | ~2-3 min |
| **Cell 2** | Mount Drive, restore/download model, start Ollama | ~5-15 min (depends on model) |
| **Cell 3** | Start Open WebUI + Cloudflare Tunnel (public link) | ~2-3 min |
| **Cell 4** | Ollama API tunnel + display opencode config | ~30 sec |

> **Note:** Cells 3 and 4 must keep running — don't stop them while in use.

### 3. Open Open WebUI

After Cell 3 completes, a URL like this will appear:
```
https://something.trycloudflare.com
```
Click the link to open Open WebUI in your browser.

---

## opencode CLI Integration

After Cell 4 runs, the notebook will display the config for `opencode.json`:

```json
{
  "endpoint": "https://something.trycloudflare.com/v1",
  "model": "qwen2.5-coder:7b",
  "api_key": "",
  "provider": "openai"
}
```

Save it as `opencode.json` in your project root, then:

```bash
opencode "refactor function calculateTotal"
```

> **Important:** Cell 4's tunnel must stay running for opencode to connect.

---

## Supported Models

| Model | Size | Best For |
|-------|------|----------|
| `qwen2.5-coder:7b` | 4.5 GB | Coding, debugging (default) |
| `qwen2.5-coder:14b` | 8.9 GB | **Vibe coding**, advanced code generation |
| `deepseek-coder:14b` | 8.5 GB | **Vibe coding**, reasoning + coding hybrid |
| `deepseek-r1:7b` | 4.5 GB | Deep reasoning, chain-of-thought |
| `gemma4:12b` | — | **Vibe coding**, multimodal, instruction following |
| `llama3.1:8b` | 5 GB | General chat, virtual assistant |

> **Vibe coding:** 14B models deliver significantly better output quality for complex code generation, refactoring, and debugging — ideal for pairing with opencode CLI. Make sure to use GPU T4 runtime (sufficient for 14B with quantization).

Edit the `MODEL` variable in Cell 2 to switch models.

---

## Architecture

```
┌──────────────────────────────────────────────┐
│                 Google Colab                  │
│                                               │
│  ┌──────────┐    ┌──────────────┐            │
│  │  Ollama  │◄───│  Open WebUI  │            │
│  │ :11434   │    │ :8081        │            │
│  └────┬─────┘    └──────┬───────┘            │
│       │                 │                    │
│  ┌────▼─────────────────▼───────┐            │
│  │   Cloudflare Tunnel x2      │            │
│  │   (daemon thread)           │            │
│  └──────────┬──────────────────┘            │
│             │                                │
│  ┌──────────▼──────────┐                    │
│  │   Google Drive      │                    │
│  │   (model storage)   │                    │
│  └─────────────────────┘                    │
└──────────────┬───────────────────────────────┘
               │
        🌐 Public URL
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────┐         ┌─────▼─────┐
│Browser │         │ opencode  │
│(WebUI) │         │ CLI       │
└────────┘         └───────────┘
```

---

## Important Notes

- **GPU:** Colab free tier provides a T4/NVIDIA GPU. If quota runs out, you can use CPU runtime (slow).
- **Session:** Colab disconnects after ~90 minutes of idle. Re-run Cells 2→3→4 after reconnecting.
- **Security:** The tunnel URL is **public**. Anyone with the URL can access your Ollama API. Keep it private.
- **Cost:** Colab free tier is sufficient. Colab Pro/Pro+ gives priority GPU access & V100/A100.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `fuser: command not found` | Already fixed — falls back to `lsof` or `ss` |
| Model download fails | Try a smaller model, or restart the session |
| WebUI inaccessible | Make sure Cell 3 is running, check the tunnel URL |
| opencode connection refused | Cell 4 tunnel must be active, endpoint must include `/v1` |
| Drive not mounted | Grant Drive access when prompted |

---

## Tech Stack

- [Ollama](https://ollama.com)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [Google Colab](https://colab.research.google.com)
- [opencode](https://opencode.ai)
