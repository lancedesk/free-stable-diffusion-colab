# Free Stable Diffusion on Google Colab

Run [AUTOMATIC1111's Stable Diffusion WebUI](https://github.com/AUTOMATIC1111/stable-diffusion-webui) for free on Google Colab — no local GPU required.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lancedesk/free-stable-diffusion-colab/blob/main/stable_diffusion_colab.ipynb)

---

## Features

- **Realistic Vision V6** — photorealistic model out of the box
- **One-click setup** — 3 cells, ~6 minutes, zero configuration
- **Pre-configured settings** — DPM++ SDE + Karras, 30 steps, CFG 5, Hires fix (768×512 → 1200×800)
- **ADetailer** — auto-fixes distorted hands & faces after every generation
- **Cloudflare Tunnel** — reliable public URL (replaces Gradio's unstable `--share`)
- **Multi-fallback cloning** — handles Stability-AI's private repo with automatic fork discovery
- **Python 3.12 compatible** — patches tokenizers build issues automatically
- **API enabled** — call from Python, curl, or any HTTP client

## Quick Start

1. Click the **Open in Colab** badge above, or go to [Google Colab](https://colab.research.google.com/) → **File → Upload notebook** → select `stable_diffusion_colab.ipynb`
2. **Runtime → Change runtime type** → Runtime type: **Python 3**, Hardware accelerator: **T4 GPU** → **Save**
3. **Runtime → Run all** — sit back for ~6 minutes while it installs, downloads the model, and launches
4. **Click the `trycloudflare.com` URL** that appears in the Cell 3 output → paste a prompt → Generate!

## Pre-configured Defaults

| Setting | Value |
|---------|-------|
| **Model** | **Realistic Vision V6** |
| Sampler | DPM++ SDE |
| Schedule | Karras |
| Steps | 30 |
| CFG Scale | 5 |
| Base resolution | 768 × 512 |
| Hires fix | Enabled → 1200 × 800 |
| Upscaler | R-ESRGAN 4x+ |
| Denoising | 0.40 |
| Batch count | 4 |
| ADetailer | Enabled (hands & faces) |

All settings are written to `ui-config.json` before launch — the WebUI opens ready to go.

## Fixing Hands & Faces

Stable Diffusion can still occasionally produce distorted hands. This notebook addresses it multiple ways:

1. **ADetailer extension** — pre-installed, auto-enabled. Detects hands/faces after generation and repaints them at higher detail using `hand_yolov8n.pt` and `face_yolov8n.pt`.

2. **Negative prompt** — always include:
   ```
   bad hands, extra fingers, fused fingers, too many fingers, missing fingers, malformed hands, bad anatomy
   ```

3. **Positive prompt** (for scenes showing hands):
   ```
   detailed hands, correct finger anatomy, five fingers
   ```

## API Usage

The `--api` flag is enabled by default. Once the WebUI is running:

```python
import requests, base64
from PIL import Image
from io import BytesIO

url = "https://your-url.trycloudflare.com/sdapi/v1/txt2img"

response = requests.post(url, json={
    "prompt": "mountain landscape at sunset, cinematic, 8k",
    "negative_prompt": "blurry, low quality, bad anatomy",
    "steps": 30,
    "cfg_scale": 5,
    "width": 768,
    "height": 512,
}).json()

img = Image.open(BytesIO(base64.b64decode(response["images"][0])))
img.save("output.png")
```

Full interactive API docs: `https://your-url.trycloudflare.com/docs`

## How It Works

### The private repo problem

The `Stability-AI/stablediffusion` GitHub repo (required by AUTOMATIC1111) went **private**. This notebook solves it with a 3-tier fallback strategy:

1. **Known public forks** — tries Hafiidz and camenduru forks first
2. **GitHub API** — queries fork list sorted by stars
3. **GitHub Search** — broader search as a last resort

After cloning, the notebook validates the repo structure (checks for `ldm.modules.midas`) and saves the working URL + commit hash. Cell 3 sets environment variables so `launch.py` skips all git operations (no fetch from private URLs).

### Other fixes baked in

| Issue | Fix |
|-------|-----|
| `tokenizers` build failure on Python 3.12 | Relaxes `transformers==4.30.2` pin to `>=4.30.2` |
| Gradio tunnel hangs | Uses Cloudflare Tunnel (`cloudflared`) instead of `--share` |
| Missing `taming` module | Auto-clones `CompVis/taming-transformers` and symlinks |
| PyTorch version conflicts | Uninstalls default Colab torch, installs matched cu121 set |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| All clone URLs fail | Find a public fork at [Stability-AI/stablediffusion/forks](https://github.com/Stability-AI/stablediffusion/forks) and add its URL to Cell 1 |
| Out of memory | Reduce to 512×512, disable Hires fix, or lower batch count |
| WebUI won't load | Ensure T4 GPU is enabled. Factory reset runtime → re-run all cells |
| Distorted hands | Check ADetailer is enabled. Add hand terms to negative prompt |
| Slow first image | Normal — first generation compiles CUDA kernels (~30-60 sec) |

**Full reset**: Runtime → Factory reset runtime → Change runtime type → T4 GPU → Run all cells

## Limitations

- Free Colab sessions last ~12 hours, then everything resets — **save your images**
- The `trycloudflare.com` URL changes every session
- T4 GPU has 15 GB VRAM — batch size >1 with Hires fix may OOM
- SD 1.5-based models cannot render legible text in images

## License

This notebook is provided as-is for educational purposes. It uses:
- [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) (AGPL-3.0)
- [Bing-su/adetailer](https://github.com/Bing-su/adetailer) (AGPL-3.0)
- [Realistic Vision V6](https://civitai.com/models/4201/realistic-vision-v60-b1) (CreativeML Open RAIL-M)
- [Stable Diffusion v1.5](https://huggingface.co/runwayml/stable-diffusion-v1-5) (CreativeML Open RAIL-M) — base architecture

---

**If this saved you time, give it a ⭐!**
