# Running Local AI on Colab

Run **Ollama + Open WebUI** on Google Colab for free, access from anywhere via **two** Cloudflare tunnels, and integrate with **opencode CLI** / **Claude Code** / any OpenAI-compatible client.

---

## Features

- **Ollama** — Run local AI models (Qwen2.5-Coder, Llama 3.1, DeepSeek-R1, Gemma 4, etc.) on Colab's GPU (T4)
- **Open WebUI** — ChatGPT-like web interface for chatting with models
- **Two Cloudflare Tunnels** — One for WebUI browser access, one direct to Ollama API
- **Model Persistence** — Models auto-save to Google Drive so you don't re-download every session
- **opencode / Claude Code / Cline / Continue Ready** — Direct Ollama tunnel works with any OpenAI-compatible client

---

## Usage

### 1. Open in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Samsulmaarif01/Running-Local-AI-on-Colab/blob/main/Colab_Ai.ipynb)

Or manually upload `Colab_Ai.ipynb` to [Google Colab](https://colab.research.google.com).

### 2. Run the Notebook

Execute the 3 cells in order:

| Cell | Function | Duration |
|------|----------|----------|
| **Cell 1** | Install dependencies (Ollama, Open WebUI, system deps) | ~2-3 min |
| **Cell 2** | Mount Drive, restore/download model, start Ollama | ~5-15 min (depends on model) |
| **Cell 3** | Start WebUI + **two** Cloudflare tunnels, test, print configs | ~2-3 min |

> Cell 3 must keep running — it monitors both tunnels. All access stops if this cell is interrupted.

### 3. Open Open WebUI

After Cell 3 runs, a URL like this will appear:
```
https://something.trycloudflare.com
```
Click the link to open Open WebUI in your browser.

---

## opencode CLI Integration

Cell 3 prints the config for `opencode.json` using the **direct Ollama tunnel**:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "Ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (Colab)",
      "options": {
        "baseURL": "https://ollama-tunnel.trycloudflare.com/v1"
      },
      "models": {
        "qwen2.5-coder:7b": {
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

> **Important:** The Ollama tunnel must stay running for opencode to connect.
> **Tip:** Run `/models` in opencode to see all available models from Colab.

---

## Claude Code Router Integration

Cell 3 also prints config for Claude Code Router using the **WebUI tunnel** (Anthropic Messages API). Since `WEBUI_AUTH=False`, no real API key is needed:

```bash
export ANTHROPIC_BASE_URL="https://webui-tunnel.trycloudflare.com/api"
export ANTHROPIC_API_KEY="not-needed"
```

Or add to `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://webui-tunnel.trycloudflare.com/api",
    "ANTHROPIC_AUTH_TOKEN": "not-needed"
  }
}
```

---

## Any OpenAI-Compatible Client

The direct Ollama tunnel works with **any** OpenAI-compatible tool (Cline, Continue, etc.):

```
baseURL: https://ollama-tunnel.trycloudflare.com/v1
model:   qwen2.5-coder:7b
```

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
┌───────────────────────────────────────────────────┐
│                  Google Colab                      │
│                                                    │
│  ┌──────────┐         ┌──────────────┐            │
│  │  Ollama  │◄────────│  Open WebUI  │            │
│  │ :11434   │         │ :8081        │            │
│  └────┬─────┘         └──────┬───────┘            │
│       │                     │                     │
│  ┌────▼──────────┐   ┌──────▼─────────┐          │
│  │ Tunnel A     │   │ Tunnel B       │          │
│  │ cloudflared  │   │ cloudflared    │          │
│  │ → Ollama     │   │ → WebUI        │          │
│  └────┬──────────┘   └──────┬─────────┘          │
│       │                     │                    │
│  ┌────▼─────────────────────▼─────────┐          │
│  │         Google Drive               │          │
│  │       (model storage)              │          │
│  └─────────────────────────────────────┘          │
└───────────────────┬───────────────────────────────┘
                    │
         ┌──────────┴───────────────────┐
         │                              │
┌────────▼────────┐          ┌─────────▼──────────┐
│ Ollama Tunnel   │          │  WebUI Tunnel      │
│ trycloudflare   │          │  trycloudflare     │
│                 │          │                    │
│ opencode        │          │ Browser (WebUI)    │
│ Cline / Continue│          │ Claude Code Router │
│ Any OpenAI-     │          │ Anthropic API      │
│ compatible tool │          │                    │
└─────────────────┘          └────────────────────┘
```

> Two separate Cloudflare tunnels: one directly to Ollama (`:11434`) for OpenAI-compatible API access, and one to WebUI (`:8081`) for browser UI and Anthropic Messages API.

---

## Colab Environment

- **GPU:** Colab free tier provides a T4/NVIDIA GPU. If quota runs out, you can use CPU runtime (slower).
- **Session:** Colab disconnects after ~90 minutes of idle. Just re-run Cell 2 → Cell 3 after reconnecting.
- **Security:** Both tunnel URLs are **public**. Anyone with the URLs can access your Ollama API. Keep them private.
- **Cost:** Colab free tier is sufficient. Colab Pro/Pro+ gives priority GPU access & V100/A100.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `fuser: command not found` | Already fixed — falls back to `lsof` or `ss` |
| Model download fails | Try a smaller model, or restart the session |
| WebUI inaccessible | Make sure Cell 3 is running, check the WebUI tunnel URL |
| opencode connection refused | Ollama tunnel must be active, copy the URL from Cell 3 output |
| Tunnel fails to start | Cloudflare trycloudflare may be rate-limited; wait a few minutes and retry |
| Drive not mounted | Grant Drive access when prompted |

---

## Tech Stack

- [Ollama](https://ollama.com)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [Google Colab](https://colab.research.google.com)
- [opencode](https://opencode.ai)
