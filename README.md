# Running Local AI on Colab

Run **Ollama + Open WebUI** on Google Colab for free, access from anywhere via ngrok, and integrate with **opencode CLI** as your AI coding assistant.

---

## Features

- **Ollama** — Run local AI models (Qwen2.5-Coder, Llama 3.1, DeepSeek-R1, Gemma 4, etc.) on Colab's GPU (T4)
- **Open WebUI** — ChatGPT-like web interface for chatting with models
- **ngrok Tunnel** — Public access without port forwarding (free account required)
- **Model Persistence** — Models auto-save to Google Drive so you don't re-download every session
- **opencode CLI Ready** — Ollama API tunnel works directly with opencode as your AI provider

---

## Usage

### 1. Open in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Samsulmaarif01/Running-Local-AI-on-Colab/blob/main/Colab_Ai.ipynb)

Or manually upload `Colab_Ai.ipynb` to [Google Colab](https://colab.research.google.com).

### 2. Run the Notebook

Execute the 3 cells in order:

| Cell | Function | Duration |
|------|----------|----------|
| **Cell 1** | Install dependencies (Ollama, Open WebUI, pyngrok) | ~2-3 min |
| **Cell 2** | Mount Drive, restore/download model, start Ollama | ~5-15 min (depends on model) |
| **Cell 3+4** | Start Open WebUI + ngrok tunnel (single cell) | ~2-3 min |

> **Note:** Cell 3+4 must keep running — don't stop it while in use.

### 3. Open Open WebUI

After Cell 3+4 runs, a URL like this will appear:
```
https://something.ngrok-free.app
```
Click the link to open Open WebUI in your browser.

---

## opencode CLI Integration

Cell 3+4 also displays the config for `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "Ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (Colab)",
      "options": {
        "baseURL": "https://something.ngrok-free.app/ollama/v1"
      },
      "models": {
        "QwenCoder7B": {
          "name": "qwen2.5-coder:7b"
        }
      }
    }
  }
}
```

Save it as `opencode.json` in your project root, then:

```bash
opencode "refactor function calculateTotal"
```

> **Important:** The ngrok tunnel must stay running for opencode to connect.
> **Tip:** Run `/models` in opencode to see all available models from Colab.

---

## Supported Models

| Model | Size | Best For |
|-------|------|----------|
| `qwen2.5-coder:7b` | 4.7 GB | Coding, debugging (default) |
| `qwen2.5-coder:14b` | 9.0 GB | **Vibe coding**, advanced code generation |
| `huihui_ai/qwen2.5-coder-abliterate:14b-instruct-q4_k_m` | 9.0 GB | Uncensored coding (abliterated) |
| `supergoatscriptguy/mythos-sec:24b` | 14 GB | Pentest/CTF/bug bounty uncensored |
| `dolphin-llama3:8b` | 4.7 GB | Uncensored general/agentic, function calling |
| `deepseek-coder:14b` | 8.5 GB | **Vibe coding**, reasoning + coding hybrid |
| `deepseek-r1:7b` | 4.7 GB | Deep reasoning, chain-of-thought |
| `gemma4:12b` | 6.6 GB | **Vibe coding**, multimodal, instruction following |
| `llama3.1:8b` | 4.9 GB | General chat, virtual assistant |

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
│  │       ngrok Tunnel          │            │
│  │   (single tunnel :8081)     │            │
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

> Ollama API is accessed through WebUI's built-in proxy at `/ollama/v1` — only one ngrok tunnel needed.

---

## Important Notes

- **GPU:** Colab free tier provides a T4/NVIDIA GPU. If quota runs out, you can use CPU runtime (slow).
- **Session:** Colab disconnects after ~90 minutes of idle. Re-run Cells 2→3+4 after reconnecting.
- **Security:** The tunnel URL is **public**. Anyone with the URL can access your Ollama API. Keep it private.
- **ngrok Auth:** ngrok free tier requires signing up at [dashboard.ngrok.com](https://dashboard.ngrok.com/signup) for an auth token. It's free.
- **Cost:** Colab free tier is sufficient. Colab Pro/Pro+ gives priority GPU access & V100/A100.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `fuser: command not found` | Already fixed — falls back to `lsof` or `ss` |
| Model download fails | Try a smaller model, or restart the session |
| WebUI inaccessible | Make sure Cell 3+4 is running, check the ngrok URL |
| opencode connection refused | ngrok tunnel must be active, endpoint includes `/ollama/v1` |
| ngrok auth error | Sign up at dashboard.ngrok.com and paste your auth token when prompted |
| Drive not mounted | Grant Drive access when prompted |

---

## Tech Stack

- [Ollama](https://ollama.com)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [ngrok](https://ngrok.com)
- [Google Colab](https://colab.research.google.com)
- [opencode](https://opencode.ai)
