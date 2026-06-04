# Running Local AI on Colab

Jalankan **Ollama + Open WebUI** di Google Colab secara gratis, akses dari mana saja via Cloudflare Tunnel, dan integrasikan dengan **opencode CLI** sebagai AI coding assistant.

---

## Fitur

- **Ollama** — Jalankan model AI lokal (Qwen2.5-Coder, Llama 3.1, DeepSeek-R1, dll) di GPU Colab (T4)
- **Open WebUI** — Antarmuka web ChatGPT-like untuk chatting dengan model
- **Cloudflare Tunnel** — Akses publik tanpa registrasi, tanpa port forwarding, tanpa config DNS
- **Model Persistence** — Model otomatis tersimpan ke Google Drive agar tidak download ulang tiap sesi
- **opencode CLI Ready** — Tunnel API Ollama bisa langsung dipakai opencode sebagai provider AI

---

## Cara Pakai

### 1. Buka di Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/[username]/[repo]/blob/main/Colab_Ai.ipynb)

Atau upload manual `Colab_Ai.ipynb` ke [Google Colab](https://colab.research.google.com).

### 2. Jalankan Notebook

Jalankan 4 sel secara berurutan:

| Sel | Fungsi | Durasi |
|-----|--------|--------|
| **Sel 1** | Install dependency (Ollama, Open WebUI, cloudflared) | ~2-3 menit |
| **Sel 2** | Mount Drive, restore/download model, jalankan Ollama | ~5-15 menit (tergantung model) |
| **Sel 3** | Start Open WebUI + Cloudflare Tunnel (link publik) | ~2-3 menit |
| **Sel 4** | Tunnel API Ollama + tampilkan config opencode | ~30 detik |

> **Catatan:** Sel 3 dan 4 harus tetap berjalan — jangan matikan selnya selama masih dipakai.

### 3. Buka Open WebUI

Setelah Sel 3 selesai, akan muncul URL seperti:
```
https://something.trycloudflare.com
```
Klik link tersebut untuk membuka Open WebUI di browser.

---

## Integrasi dengan opencode CLI

Setelah Sel 4 jalan, notebook akan menampilkan config untuk `opencode.json`:

```json
{
  "endpoint": "https://something.trycloudflare.com/v1",
  "model": "qwen2.5-coder:7b",
  "api_key": "",
  "provider": "openai"
}
```

Simpan sebagai `opencode.json` di root project Anda, lalu:

```bash
opencode "refactor function calculateTotal"
```

> **Penting:** Tunnel Sel 4 harus tetap running agar opencode bisa terhubung.

---

## Model yang Didukung

| Model | Ukuran | Best For |
|-------|--------|----------|
| `qwen2.5-coder:7b` | 4.5 GB | Coding, debugging (default) |
| `qwen2.5-coder:14b` | 8.9 GB | **Vibe coding**, code generation tingkat lanjut |
| `deepseek-coder:14b` | 8.5 GB | **Vibe coding**, reasoning + coding hybrid |
| `deepseek-r1:7b` | 4.5 GB | Deep reasoning, chain-of-thought |
| `gemma4:12b` | — | **Vibe coding**, multimodal, instruction following |
| `llama3.1:8b` | 5 GB | Chat umum, asisten virtual |

> **Vibe coding:** Model 14B memberikan kualitas output yang jauh lebih baik untuk generate kode kompleks, refactoring, dan debugging — cocok untuk pairing dengan opencode CLI. Pastikan runtime Colab menggunakan GPU T4 (cukup untuk 14B dengan quantized).

Edit variabel `MODEL` di Sel 2 untuk mengganti model.

---

## Arsitektur

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

## Catatan Penting

- **GPU:** Colab free tier memberikan T4/NVIDIA GPU. Jika habis kuota, bisa pake runtime CPU (lambat).
- **Sesi:** Colab akan disconnect setelah idle ~90 menit. Jalankan ulang Sel 2→3→4 setelah reconnect.
- **Keamanan:** URL tunnel bersifat **public**. Siapa pun yang tahu URL bisa mengakses API Ollama Anda. Jangan bagikan.
- **Biaya:** Free tier Colab sudah cukup. Colab Pro/Pro+ memberi akses GPU prioritas & V100/A100.

---

## Troubleshooting

| Masalah | Solusi |
|---------|--------|
| `fuser: command not found` | Sudah di-fix — fallback ke `lsof` atau `ss` |
| Model download gagal | Coba ganti model yang lebih kecil, atau restart session |
| WebUI tidak bisa diakses | Pastikan Sel 3 masih running, cek URL tunnel |
| opencode connection refused | Tunnel Sel 4 harus aktif, pastikan endpoint pakai `/v1` |
| Drive not mounted | Izinkan akses Drive saat prompt muncul |

---

## Tech Stack

- [Ollama](https://ollama.com)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [Google Colab](https://colab.research.google.com)
- [opencode](https://opencode.ai)
