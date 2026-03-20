# AI Video LoRA Guide — Custom Styles for AI Video Generation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/ristponex/ai-video-lora-guide/pulls)

A complete guide to using LoRA (Low-Rank Adaptation) adapters for AI video generation — customize styles, characters, and visual aesthetics without training from scratch.

[English](#) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

---

## Table of Contents

- [What is LoRA?](#what-is-lora)
- [Supported Models](#supported-models)
- [How LoRA Works for Video](#how-lora-works-for-video)
- [Quick Start](#quick-start)
- [Finding LoRAs](#finding-loras)
- [Parameter Deep Dive](#parameter-deep-dive)
- [Use Cases](#use-cases)
- [Prompt + LoRA Synergy](#prompt--lora-synergy)
- [Troubleshooting](#troubleshooting)
- [Pricing](#pricing)
- [FAQ](#faq)

---

## What is LoRA?

**LoRA (Low-Rank Adaptation)** is a technique for customizing AI models without retraining them from scratch. Instead of modifying billions of parameters, LoRA trains a small set of adapter weights that modify the model's behavior — like adding a lens filter to a camera.

For video generation, LoRA enables:
- **Custom visual styles** — anime, oil painting, film noir, cyberpunk
- **Character consistency** — maintain a character's appearance across videos
- **Quality enhancement** — improve skin texture, lighting, details
- **Artistic control** — apply specific artist styles or aesthetics

LoRA for video is newer than for images. While image LoRAs (SDXL, Flux) are widely available, video LoRAs are a growing ecosystem primarily supported by the Wan model family.

---

## Supported Models

| Model | Type | LoRA Support | Price | Platform |
|-------|------|:------------:|:-----:|----------|
| ⭐ **Wan 2.2 Spicy LoRA** | Video (I2V) | ✅ 3 high + 3 low | $0.04/s | [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) |
| Wan 2.1 (Open Source) | Video | ✅ Full | GPU cost | Self-hosted |
| Flux Dev LoRA | Image | ✅ | $0.012 | Atlas Cloud |
| Flux Kontext Dev LoRA | Image (edit) | ✅ | $0.012 | Atlas Cloud |
| SDXL + LoRA | Image | ✅ Full | GPU cost | Self-hosted |

> **Video LoRA** is currently best supported by the **Wan model family**. Other video models (Kling, Seedance, Vidu) do not yet support custom LoRAs via API.

---

## How LoRA Works for Video

### Two Types of LoRA Slots

Wan 2.2 Spicy LoRA provides two types of LoRA slots:

#### `high_noise_loras` — Strong Style Influence

- Affects early generation steps where overall composition is determined
- Changes visual style, color palette, art direction
- Best for: anime conversion, painterly styles, dramatic transformations
- Max: 3 LoRAs

#### `low_noise_loras` — Subtle Refinement

- Affects later generation steps where details are refined
- Adjusts fine details without changing composition
- Best for: skin texture, lighting quality, color grading
- Max: 3 LoRAs

### Visual Analogy

```
Image Input → [High-Noise LoRA: Art Style] → [Model: Generate Motion] → [Low-Noise LoRA: Detail] → Video Output
                    ↑ overall look                                            ↑ fine details
```

---

## Quick Start

### Step 1: Get an API Key

Sign up at [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) → Console → API Keys → Create.

```bash
export ATLASCLOUD_API_KEY="your-key"
```

### Step 2: Find a LoRA

Browse [Civitai](https://civitai.com) for video-compatible LoRAs. Copy the download URL.

### Step 3: Generate

```bash
curl -s -X POST "https://api.atlascloud.ai/api/v1/model/generateVideo" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "alibaba/wan-2.2-spicy/image-to-video-lora",
    "image": "https://example.com/photo.jpg",
    "prompt": "Graceful movement, anime style, vibrant colors",
    "resolution": "720p",
    "duration": 5,
    "high_noise_loras": ["https://civitai.com/api/download/models/12345"]
  }'
```

### Step 4: Poll & Download

```bash
# Poll until completed
curl -s "https://api.atlascloud.ai/api/v1/model/prediction/{prediction-id}" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY"

# Download
curl -o output.mp4 "VIDEO_URL"
```

### Python

```python
import urllib.request, json, os, time

API_KEY = os.environ["ATLASCLOUD_API_KEY"]
BASE = "https://api.atlascloud.ai/api/v1"

def generate_with_lora(image_url, prompt, high_loras=None, low_loras=None):
    payload = {
        "model": "alibaba/wan-2.2-spicy/image-to-video-lora",
        "image": image_url,
        "prompt": prompt,
        "resolution": "720p",
        "duration": 5,
    }
    if high_loras:
        payload["high_noise_loras"] = high_loras
    if low_loras:
        payload["low_noise_loras"] = low_loras

    data = json.dumps(payload).encode()
    req = urllib.request.Request(
        f"{BASE}/model/generateVideo", data=data,
        headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    )
    pred_id = json.loads(urllib.request.urlopen(req).read())["data"]["id"]

    while True:
        req = urllib.request.Request(
            f"{BASE}/model/prediction/{pred_id}",
            headers={"Authorization": f"Bearer {API_KEY}"},
        )
        result = json.loads(urllib.request.urlopen(req).read())
        if result["data"]["status"] in ("completed", "succeeded"):
            return result["data"]["outputs"][0]
        if result["data"]["status"] == "failed":
            raise Exception(result["data"]["error"])
        time.sleep(5)

video_url = generate_with_lora(
    "https://example.com/photo.jpg",
    "Anime style transformation, vibrant colors, dynamic camera",
    high_loras=["https://civitai.com/api/download/models/12345"],
)
print(f"Video: {video_url}")
```

---

## Finding LoRAs

### Civitai (Recommended)

The largest LoRA marketplace with previews and ratings.

- **URL**: https://civitai.com
- **Search tips**: "wan lora", "video lora", "anime lora", "realistic lora"
- **Download URL format**: `https://civitai.com/api/download/models/{version_id}`
- **Filter**: Type → LoRA → sort by Most Downloaded

### HuggingFace

Open-source LoRAs with model cards.

- **URL**: https://huggingface.co/models?other=lora
- **Search**: "wan", "video generation", specific style names

### Tensor.Art

Community gallery with visual previews.

- **URL**: https://tensor.art
- **Browse**: LoRA section, filter by type

### Compatibility Tips

- ✅ LoRAs trained on **Wan** architecture work best
- ✅ LoRAs with video preview images/clips are more likely to work for video
- ⚠️ Image-only LoRAs (SDXL, Flux) are **not compatible** with video generation
- ⚠️ Check file size — very large LoRAs (>500MB) may increase latency

---

## Parameter Deep Dive

### When to Use High-Noise

| Scenario | High-Noise LoRA | Effect |
|----------|:-:|--------|
| Convert to anime | Anime style LoRA | Complete visual transformation |
| Oil painting look | Painting LoRA | Brushstroke texture, color shift |
| Cyberpunk aesthetic | Neon/cyber LoRA | Color palette + lighting change |
| Vintage film | Film grain LoRA | Overall color grading + texture |

### When to Use Low-Noise

| Scenario | Low-Noise LoRA | Effect |
|----------|:-:|--------|
| Better skin texture | Detail LoRA | Subtle realism enhancement |
| Color correction | Color LoRA | Fine color adjustment |
| Sharper details | Sharpening LoRA | Edge enhancement |
| Lighting fix | Lighting LoRA | Subtle lighting improvement |

### Combining Multiple LoRAs

```json
{
  "high_noise_loras": ["anime-style.safetensors", "vibrant-colors.safetensors"],
  "low_noise_loras": ["skin-detail.safetensors"]
}
```

**Rules of thumb:**
- Start with **one LoRA** and evaluate results
- Add a second only if needed
- Mixing conflicting styles (e.g., anime + photorealistic) produces unpredictable results
- **Less is more** — one well-chosen LoRA often beats three mediocre ones

---

## Use Cases

### 1. Anime Style

Apply anime LoRA to convert photorealistic input to anime aesthetic.

```
Prompt: "Anime girl transformation, cel-shading, vibrant saturated colors, studio Ghibli inspired"
high_noise_loras: [anime-style-lora]
```

### 2. Cinematic Realism

Enhance realism with film-quality LoRA.

```
Prompt: "Cinematic close-up, shallow depth of field, film grain, warm color grading"
low_noise_loras: [film-grain-lora, cinematic-lighting-lora]
```

### 3. Artistic Filters

Apply painterly or stylized effects.

```
Prompt: "Oil painting come to life, thick brushstrokes, impressionist lighting"
high_noise_loras: [oil-painting-lora]
```

### 4. Character Consistency

Use character LoRA for consistent appearance across multiple videos.

```
Prompt: "The character walks through a garden, maintaining consistent face and body"
high_noise_loras: [character-specific-lora]
low_noise_loras: [face-detail-lora]
```

### 5. Retro / Vintage

Apply vintage film aesthetics.

```
Prompt: "1970s film aesthetic, warm grain, soft focus, golden tones"
high_noise_loras: [vintage-film-lora]
```

---

## Prompt + LoRA Synergy

Your prompt and LoRA should reinforce each other:

| LoRA Style | Good Prompt Keywords | Bad Prompt Keywords |
|------------|---------------------|---------------------|
| Anime | "cel-shading, vibrant, anime, kawaii" | "photorealistic, raw photo" |
| Oil painting | "brushstrokes, canvas texture, impressionist" | "sharp, 8k, photography" |
| Film noir | "dramatic shadows, high contrast, black and white" | "vibrant colors, sunny" |
| Cyberpunk | "neon, holographic, rain, dark atmosphere" | "natural, pastoral, warm" |

**Prompt template:**

```
[Subject action], [LoRA-matching style keywords], [lighting], [camera/composition]
```

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Style not applied | Incompatible LoRA | Try a different LoRA, check architecture compatibility |
| Blurry output | Resolution too low | Use 720p instead of 480p |
| Artifacts/glitches | Too many LoRAs | Reduce to 1-2 LoRAs |
| LoRA too subtle | Using low-noise slot | Move to high-noise slot |
| LoRA too strong | Using high-noise slot | Move to low-noise slot |
| Style conflicts | Conflicting LoRAs | Remove one, use matching styles |
| Slow generation | Large LoRA files | Use smaller LoRAs (<200MB) |
| Error loading LoRA | Bad URL or format | Verify URL is accessible, use .safetensors |
| Motion artifacts | Complex prompt + many LoRAs | Simplify prompt, reduce LoRA count |
| Color banding | Low-quality source image | Use higher resolution input image |

---

## Pricing

| Option | Cost | Best For |
|--------|:----:|----------|
| Wan 2.2 Spicy Standard | $0.03/s | General NSFW, no style customization |
| **Wan 2.2 Spicy LoRA** | **$0.04/s** | Custom styles, anime, artistic |
| Wan 2.6 I2V | $0.10-0.15/s | 1080p, audio, no LoRA |
| Self-hosted (RunPod) | ~$0.50/hr GPU | Full control, any LoRA, unlimited |

Get started at [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide).

---

## FAQ

<details>
<summary><b>Can I use any LoRA with Wan 2.2 Spicy?</b></summary>
Not any LoRA — it needs to be compatible with the Wan architecture. SDXL or Flux LoRAs won't work. Look for LoRAs specifically trained on Wan or marked as video-compatible.
</details>

<details>
<summary><b>How many LoRAs can I use at once?</b></summary>
Up to 6: 3 high-noise + 3 low-noise. But start with 1 and add more only if needed.
</details>

<details>
<summary><b>Is the LoRA version NSFW only?</b></summary>
The Wan 2.2 Spicy LoRA variant is optimized for NSFW but the LoRA technique itself works for any style. You can use it for SFW anime, artistic, cinematic styles too.
</details>

<details>
<summary><b>Can I train my own video LoRA?</b></summary>
Yes. Wan 2.1 is open source (Apache 2.0). Use tools like kohya_ss for LoRA training. Then use your custom LoRA with the Wan 2.2 Spicy LoRA API.
</details>

<details>
<summary><b>Why is the LoRA version more expensive?</b></summary>
Loading and applying LoRA adapters requires additional GPU memory and compute during inference, adding ~33% to the cost ($0.04 vs $0.03 per second).
</details>

<details>
<summary><b>What file format should LoRAs be?</b></summary>
`.safetensors` is the standard format. Provide a direct download URL — the API will fetch the LoRA file.
</details>

<details>
<summary><b>Do LoRAs affect generation speed?</b></summary>
Slightly. Each LoRA adds a small overhead. Using 6 LoRAs will be slower than using 1. The effect is usually 10-30% increase in generation time.
</details>

<details>
<summary><b>Can I preview a LoRA's effect cheaply?</b></summary>
Generate a 5s 480p test ($0.20). Much cheaper than committing to 8s 720p ($0.32) for initial testing.
</details>

---

## Contributing

Found a great video LoRA? Have tips to share? Open a PR!

---

## License

[MIT](LICENSE)
