# Ollama Colab

Run a large language model on a free Google Colab GPU and expose it as a public API endpoint — accessible from anywhere via a secure tunnel.

Built as a self-teaching project exploring LLM inference, tunneling, and remote API access.

---

## What it does

The notebook (`Ollama_Colab_v2_4.ipynb`) automates the full setup on a Colab instance:

1. Verifies GPU availability
2. Installs [Ollama](https://ollama.com) and tunnel dependencies
3. Pulls your chosen model (Qwen 3 or Qwen 2.5 Coder)
4. Starts the Ollama server
5. Opens a public tunnel (ngrok or Cloudflare)
6. Tests the endpoint with a sample prompt
7. Keeps the server alive for the duration of the session

Once running, the endpoint speaks the standard Ollama HTTP API — compatible with any client that supports it (curl, Python `requests`, VS Code extensions, etc.).

---

## Supported models

| Model | Tag | Size | Speed on T4 | Notes |
|---|---|---|---|---|
| Qwen 3 32B | `qwen3:32b` | ~20 GB | Moderate | Best quality |
| Qwen 3 14B | `qwen3:14b` | ~9 GB | Fast | Good balance — recommended for free tier |
| Qwen 3 8B | `qwen3:8b` | ~5 GB | Fastest | Quick experiments |
| Qwen 2.5 Coder 32B | `qwen2.5-coder:32b` | ~20 GB | Moderate | Code-specialised |
| Qwen 2.5 Coder 14B | `qwen2.5-coder:14b` | ~9 GB | Fast | Code-specialised |

---

## Prerequisites

### 1. Google Colab runtime

Change the runtime to **T4 GPU** before running:  
`Runtime → Change runtime type → T4 GPU → Save`

### 2. Tunnel credentials (one of the following)

**Option A — Cloudflare Tunnel** *(recommended)*  
Provides a stable, fixed hostname with no session limits.

1. Sign up at [one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. Go to **Networks → Tunnels → Create a tunnel**
3. Copy the tunnel token
4. In Colab, open **Secrets** (key icon in the sidebar)
5. Add a secret: `cloudflare_token` = `<your-token>`

**Option B — ngrok**  
Simpler to set up, but the URL rotates every ~2 hours on the free tier.

1. Sign up at [ngrok.com](https://ngrok.com)
2. Copy your authtoken from the [dashboard](https://dashboard.ngrok.com/get-started/your-authtoken)
3. In Colab Secrets, add: `ngrok_authtoken` = `<your-token>`

The notebook auto-detects whichever secret is present. If both are set, Cloudflare takes priority.

---

## Usage

### Running the notebook

Open `Ollama_Colab_v2_1.ipynb` in Google Colab and run the cells in order (Steps 1–9). The public URL is printed at the end of Step 7.

### Making requests

Replace `https://your-url` with the URL from Step 7.

**curl**
```bash
# Chat
curl -X POST https://your-url/api/chat \
  -H 'Content-Type: application/json' \
  -d '{"model": "qwen3:14b", "messages": [{"role": "user", "content": "Explain recursion"}]}'

# Single-turn generate
curl -X POST https://your-url/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model": "qwen3:14b", "prompt": "Hello, world!"}'

# Extended thinking mode (Qwen 3)
curl -X POST https://your-url/api/chat \
  -H 'Content-Type: application/json' \
  -d '{"model": "qwen3:14b", "messages": [{"role": "user", "content": "/think Solve this step by step: ..."}]}'
```

**Python**
```python
import requests

response = requests.post(
    "https://your-url/api/chat",
    json={
        "model": "qwen3:14b",
        "messages": [{"role": "user", "content": "Write a Flask API"}],
    }
)
print(response.json()["message"]["content"])
```

**VS Code (Continue / CodeGPT extension)**

1. Install the extension
2. Set provider to **Ollama**
3. Set base URL to your tunnel URL
4. Select your model (e.g. `qwen3:14b`)

---

## Project structure

```
.
├── Ollama_Colab_v2_4.ipynb   # Main notebook (current version)
├── Ollama_Colab_v2_3.ipynb   # Previous version
├── Ollama_Colab_v2_2.ipynb   # Previous version
├── Ollama_Colab_v2_1.ipynb   # Previous version
└── README.md
```

### Notebook steps

| Step | Cell | What it does |
|---|---|---|
| 1 | Install | Verifies GPU, installs Ollama, pyngrok, cloudflared |
| 2 | Configure | Model selection dropdown |
| 3 | Imports | Loads libraries, detects tunnel type from Colab Secrets |
| 4 | Helpers | `stream_output` and `test_endpoint` utility functions |
| 5 | Ollama server | Starts `ollama serve`, waits for readiness, checks for early exit |
| 6 | Pull model | Downloads the selected model with live progress |
| 7 | Tunnel | Opens ngrok or Cloudflare Tunnel, prints public URL |
| 8 | Test | Sends a sample prompt, prints the response |
| 9 | Keep-alive | Blocks to keep the server running; shuts down cleanly on interrupt |

---

## Limitations

- **Colab free tier**: Sessions last a few hours; daily GPU quota applies. The `qwen3:14b` model is the most practical choice here.
- **Colab Pro**: Sessions up to 24 hours with L4 GPU access.
- **ngrok free tier**: URL rotates every ~2 hours. Update your client config after each rotation.
- **Security**: The tunnel exposes your Ollama instance publicly with no authentication. Avoid running sensitive prompts on shared or public networks. Consider adding a reverse proxy with auth if you need persistent access.

---

## Changelog

### v2.4 (current)
- Added **live Ollama model browser** (Step 2) — scrapes `ollama.com/library` at runtime
- Filter models by capability (tools, vision, thinking, embedding, code, cloud)
- Search by name or description
- Selecting a model fetches its available tags live from `ollama.com/library/<model>/tags`
- Tag picker (radio buttons) lets you choose the exact variant (e.g. `14b`, `q4_K_M`, `latest`)
- Clicking **✅ Use this model** sets `MODEL_NAME` — Step 3 validates it is set before proceeding
- Tag results are cached to avoid repeated network requests when switching between models
- The hardcoded model dropdown is removed — any model in the Ollama library is now selectable

### v2.3
- Added **tunnel selection dropdown** in Step 3 (Colab form UI)
- Default is **Cloudflare quick tunnel** — no credentials needed, just run
- Selecting a named tunnel or ngrok loads the token from Colab Secrets and shows a clear error if the secret is missing, rather than silently falling back
- Removed auto-detection logic in favour of explicit user choice

### v2.2
- Added **Cloudflare quick tunnel** as a zero-credential fallback (no secrets needed)
- Tunnel priority: `cloudflare_token` → `ngrok_authtoken` → quick tunnel (auto-detected)
- Added `OLLAMA_KEEP_ALIVE=-1` — model stays loaded in VRAM, eliminating cold-start delays between requests
- Added `OLLAMA_CONTEXT_LENGTH=16384` — larger context window than the Ollama default
- Added `OLLAMA_HOST=0.0.0.0` — ensures the tunnel can reach the server
- Switched all HTTP calls from `urllib.request` to `requests` library
- Added RAM check alongside GPU check (from `psutil`)
- Fixed quick-tunnel stdout blocking bug — URL is parsed then stdout handed off to a background thread so the cell completes cleanly
- Added `qwen2.5-coder:7b` to model dropdown

### v2.1
- Fixed Cloudflare Tunnel token handling — now uses `cloudflared tunnel run --token` (correct invocation)
- Fixed `shell=True` + list args conflict in tunnel subprocess
- Removed dead 60-second wait loop in Cloudflare setup
- Added early-exit detection if `ollama serve` crashes on startup
- Added `_bg_processes` registry — Step 9 now actually terminates all background processes on shutdown
- Replaced `stdout` pipe race condition with single-reader background threads
- `MODEL_SIZES` dict replaces fragile string-matching size estimates
- `import re` moved to top-level imports; `json`/`json_mod` alias removed
- `for line in process.stdout` loop replaces manual `readline()` + `poll()` pattern

### v2.0
- Dual-tunnel support (ngrok + Cloudflare auto-detection)
- Qwen 3 model support
- Colab Secrets integration

---

## Learning goals

This project was built to explore:

- Running local LLMs on cloud GPU hardware
- The Ollama API and how LLM servers work
- Tunneling techniques for exposing local services publicly
- Python subprocess management and background threading
- Google Colab as a free compute platform

---

## Resources

- [Ollama documentation](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [Qwen 3 model page](https://ollama.com/library/qwen3)
- [Cloudflare Tunnel docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [ngrok documentation](https://ngrok.com/docs)
