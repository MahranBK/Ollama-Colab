# Ollama Colab

Run a large language model on a free Google Colab GPU and expose it as a public API endpoint — accessible from anywhere via a secure tunnel.

Built as a self-teaching project exploring LLM inference, tunneling, and remote API access.

---

## What it does

The notebook (`Ollama_Colab_v2_4.ipynb`) automates the full setup on a Colab instance:

1. Verifies GPU availability
2. Installs [Ollama](https://ollama.com) and tunnel dependencies
3. Pulls your chosen model from the entire Ollama library  
4. Starts the Ollama server
5. Opens a public tunnel (ngrok or Cloudflare)
6. Tests the endpoint with a sample prompt
7. Keeps the server alive for the duration of the session

Once running, the endpoint speaks the standard Ollama HTTP API — compatible with any client that supports it (curl, Python `requests`, VS Code extensions, etc.).

---

## Supported models

The v2.4 notebook includes a **live model browser** that lets you select from the entire Ollama library. Popular models include:

| Model | Tag | Size | Speed on T4 | Speed on A100 | Notes |
| --- | --- | --- | --- | --- | --- |
| DeepSeek R1 14B | `deepseek-r1:14b` | ~9 GB | Fast | Very Fast | Best reasoning for T4 |
| Qwen 2.5 14B | `qwen2.5:14b` | ~9 GB | Fast | Very Fast | Good balance — recommended |
| Qwen 2.5 Coder 14B | `qwen2.5-coder:14b` | ~9 GB | Fast | Very Fast | Code-specialised |
| Llama 3.1 8B | `llama3.1:8b` | ~5 GB | Fast | Very Fast | General purpose |
| DeepSeek R1 32B | `deepseek-r1:32b` | ~20 GB | Slow (offload) | Fast | Best reasoning, fits in A100 |
| DeepSeek R1 70B | `deepseek-r1:70b` | ~40 GB | N/A | Fast | Best reasoning, requires A100 |
| Mistral 7B | `mistral:7b` | ~4 GB | Fast | Very Fast | Good all-around performance |

---

## Prerequisites

### 1. Google Colab runtime

**Free tier**: Change the runtime to **T4 GPU** before running:  
`Runtime → Change runtime type → T4 GPU → Save`

**Colab Pro+ (2026)**: Offers access to **A100 GPU** (40GB VRAM), enabling much larger models:  
`Runtime → Change runtime type → A100 GPU → Save`

### 2. Tunnel credentials (one of the following)

**Option A — Cloudflare Tunnel** *(recommended)*  
Provides a stable, fixed hostname with no session limits.

1. Sign up at [one.dash.cloudflare.com](https://one.dash.cloudflare.com)
2. Go to **Networks → Tunnels → Create a tunnel**
3. Copy the tunnel token
4. In Colab, open **Secrets** (key icon in the sidebar)
5. Add a secret: `cloudflare_token` = `<your-token>`

**Option B — ngrok**  
Simpler to set up, but has significant 2026 free tier limitations:

⚠️ **2026 Free Tier Restrictions:**

- **1 GB/month bandwidth cap** - Very limited for LLM API usage
- **Interstitial warning pages** - Browser traffic shows ngrok warnings
- **URL rotation** - Changes every ~2 hours

1. Sign up at [ngrok.com](https://ngrok.com)
2. Copy your authtoken from the [dashboard](https://dashboard.ngrok.com/get-started/your-authtoken)
3. In Colab Secrets, add: `ngrok_authtoken` = `<your-token>`

💡 **Recommendation**: Use Cloudflare Tunnel instead - no bandwidth limits or warning pages.

The notebook uses a dropdown to select tunnel method. Cloudflare quick tunnel is the default and requires no credentials.

---

## Usage

### Running the notebook

Open `Ollama_Colab_v2_4.ipynb` in Google Colab and run the cells in order (Steps 1–9). The public URL is printed at the end of Step 7.

---

## Limitations

- **Colab free tier**: Sessions last a few hours; daily GPU quota applies. Models up to 14B are ideal.
- **T4 VRAM**: T4 has 16GB. Models > 14B (like 32B) will offload to system RAM and run significantly slower.
- **A100 VRAM**: A100 has 40GB. Can comfortably run 70B+ models with full GPU acceleration. Available in Colab Pro+ (2026).
- **ngrok free tier (2026)**: 1 GB/month bandwidth cap is very restrictive for LLM API usage; warning pages interrupt browser traffic.
- **Security**: The tunnel exposes your Ollama instance publicly with no authentication. In 2026, this creates significant security risks:
  - **Unauthorized inference**: Anyone can use your GPU quota for model inference
  - **Cost abuse**: Malicious actors could run expensive operations at your expense
  - **Prompt injection**: Vulnerable to adversarial prompt attacks
  - **Tool exploitation**: If using models with tool access, could trigger unauthorized actions
  - **Data exfiltration**: Models with internet access could be manipulated to extract data

  **Recommendations:**
  - Use only on trusted networks or for short testing periods
  - Consider adding authentication (API key, OAuth, or reverse proxy)
  - Monitor usage and costs regularly
  - Disable tool access for public deployments
  - Never use with models containing sensitive information

## Ollama 2026 Features

The notebook supports the latest Ollama capabilities (v0.14.0+):

- **Anthropic API Compatibility**: Enables Claude Code integration and Anthropic-style API clients
- **Structured Outputs (JSON Schema)**: Generate valid JSON responses with schema validation for API consumers
- **Web Search Plugin**: Enable web search capabilities via Ollama integrations (requires additional setup)

These features can be enabled through the Ollama API endpoints exposed by the notebook.

---

## Learning goals

This project was built to explore:

- Running local LLMs on cloud GPU hardware
- The Ollama API and how LLM servers work
- Tunneling techniques for exposing local services publicly
- Python subprocess management and background threading
- Google Colab as a free compute platform
