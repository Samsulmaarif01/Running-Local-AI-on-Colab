# AGENTS.md — Running Local AI on Colab

This repo is a **single Colab notebook** that runs Ollama + Open WebUI on Google Colab with **two** Cloudflare tunnels. All work happens inside the notebook — nothing to install locally.

---

## Repo structure

```
Colab_Ai.ipynb   ← the only file that matters
README.md        ← overview (prefer notebook for ground truth)
```

---

## Run order (must be sequential)

Run cells in the notebook in order. Each depends on the prior.

| Cell | What it does | Wait time |
|------|-------------|-----------|
| **1** | Install Ollama, Open WebUI, system deps | ~2–3 min |
| **2** | Start Ollama, pull model, mount Drive, backup | ~5–15 min (download depends on model) |
| **3** | Start WebUI + **two** Cloudflare tunnels, test, print configs | ~2–3 min |

**Cell 3 must keep running** — it monitors WebUI + both tunnels. Stopping it kills access.

---

## Model

Edit the `MODEL` variable at the top of **Cell 2** to switch. Default is `qwen2.5-coder:7b`.

---

## Tunnel (cloudflared)

The notebook uses **Cloudflare tunnel** (`cloudflared`). Cell 3 downloads cloudflared, starts **two** tunnels:

| Tunnel | Port | Purpose |
|--------|------|---------|
| **WebUI** | `:8081` | Browser UI + Anthropic Messages API (for Claude Code Router) |
| **Ollama** | `:11434` | Direct OpenAI-compatible API (for opencode, Cline, Continue, etc.) |

---

## opencode.json (gitignored)

The Ollama tunnel URL from Cell 3 goes into `opencode.json`. The file is in `.gitignore` because the URL is ephemeral and secret. The notebook prints the exact JSON config.

---

## Model persistence

Models are backed up to Google Drive:  
`/content/drive/MyDrive/ollama_models/`  
On reconnect, Cell 2 restores from Drive so you don't re-download.

---

## Colab quirks

- **Idle disconnect** after ~90 min. Re-run Cell 2 → Cell 3 on reconnect.
- **GPU**: T4 on free tier. 14B models fit with quantization. CPU runtime works (slow).
- **Auth**: WebUI starts with `WEBUI_AUTH=False` — login is disabled, open access.
- **Ports**: WebUI on `:8081`, Ollama on `:11434` (internal only).
- **State**: Nothing persists between Colab sessions except Drive-backed models.

---

## Local environment

Nothing to install, build, lint, typecheck, or test locally. All work happens inside the Colab notebook.
