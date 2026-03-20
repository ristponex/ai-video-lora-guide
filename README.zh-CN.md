# AI 视频 LoRA 指南 — AI 视频生成的自定义风格

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/ristponex/ai-video-lora-guide/pulls)

使用 LoRA（低秩适应）适配器进行 AI 视频生成的完整指南 — 无需从头训练即可自定义风格、角色和视觉美学。

[English](README.md) | [简体中文](#) | [日本語](README.ja.md) | [한국어](README.ko.md)

---

## 目录

- [什么是 LoRA？](#什么是-lora)
- [支持的模型](#支持的模型)
- [LoRA 如何用于视频](#lora-如何用于视频)
- [快速入门](#快速入门)
- [查找 LoRA](#查找-lora)
- [参数深入解析](#参数深入解析)
- [使用场景](#使用场景)
- [提示词 + LoRA 协同](#提示词--lora-协同)
- [故障排除](#故障排除)
- [定价](#定价)
- [常见问题](#常见问题)

---

## 什么是 LoRA？

**LoRA（Low-Rank Adaptation，低秩适应）** 是一种无需从头重新训练即可自定义 AI 模型的技术。LoRA 不是修改数十亿个参数，而是训练一小组适配器权重来改变模型的行为 — 就像给相机加一个滤镜。

对于视频生成，LoRA 可以实现：
- **自定义视觉风格** — 动漫、油画、黑色电影、赛博朋克
- **角色一致性** — 在多个视频中保持角色的外观一致
- **质量增强** — 改善皮肤纹理、光线、细节
- **艺术控制** — 应用特定的艺术家风格或美学

视频 LoRA 比图像 LoRA 更新。虽然图像 LoRA（SDXL、Flux）已广泛可用，但视频 LoRA 是一个不断发展的生态系统，主要由 Wan 模型家族支持。

---

## 支持的模型

| 模型 | 类型 | LoRA 支持 | 价格 | 平台 |
|------|------|:---------:|:----:|------|
| ⭐ **Wan 2.2 Spicy LoRA** | 视频 (I2V) | ✅ 3 高 + 3 低 | $0.04/秒 | [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) |
| Wan 2.1（开源） | 视频 | ✅ 完整 | GPU 成本 | 自托管 |
| Flux Dev LoRA | 图像 | ✅ | $0.012 | Atlas Cloud |
| Flux Kontext Dev LoRA | 图像（编辑） | ✅ | $0.012 | Atlas Cloud |
| SDXL + LoRA | 图像 | ✅ 完整 | GPU 成本 | 自托管 |

> **视频 LoRA** 目前由 **Wan 模型家族**提供最佳支持。其他视频模型（Kling、Seedance、Vidu）尚不支持通过 API 使用自定义 LoRA。

---

## LoRA 如何用于视频

### 两种类型的 LoRA 插槽

Wan 2.2 Spicy LoRA 提供两种类型的 LoRA 插槽：

#### `high_noise_loras` — 强风格影响

- 影响早期生成步骤，此时决定整体构图
- 改变视觉风格、配色方案、美术指导
- 最适合：动漫转换、油画风格、戏剧性变换
- 最大数量：3 个 LoRA

#### `low_noise_loras` — 细微改善

- 影响后期生成步骤，此时改善细节
- 调整精细细节而不改变构图
- 最适合：皮肤纹理、光线质量、色彩调整
- 最大数量：3 个 LoRA

### 可视化类比

```
图像输入 → [高噪声 LoRA: 艺术风格] → [模型: 生成动作] → [低噪声 LoRA: 细节] → 视频输出
                  ↑ 整体外观                                       ↑ 精细细节
```

---

## 快速入门

### 第1步：获取 API 密钥

在 [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) 注册 → 控制台 → API 密钥 → 创建。

```bash
export ATLASCLOUD_API_KEY="your-key"
```

### 第2步：找到一个 LoRA

在 [Civitai](https://civitai.com) 浏览视频兼容的 LoRA。复制下载 URL。

### 第3步：生成

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

### 第4步：轮询和下载

```bash
# 轮询直到完成
curl -s "https://api.atlascloud.ai/api/v1/model/prediction/{prediction-id}" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY"

# 下载
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

## 查找 LoRA

### Civitai（推荐）

最大的 LoRA 市场，带预览和评分。

- **网址**：https://civitai.com
- **搜索技巧**："wan lora"、"video lora"、"anime lora"、"realistic lora"
- **下载 URL 格式**：`https://civitai.com/api/download/models/{version_id}`
- **筛选**：类型 → LoRA → 按下载量排序

### HuggingFace

开源 LoRA，带模型卡片。

- **网址**：https://huggingface.co/models?other=lora
- **搜索**："wan"、"video generation"、特定风格名称

### Tensor.Art

社区画廊，带视觉预览。

- **网址**：https://tensor.art
- **浏览**：LoRA 版块，按类型筛选

### 兼容性技巧

- ✅ 在 **Wan** 架构上训练的 LoRA 效果最好
- ✅ 带视频预览图片/片段的 LoRA 更可能适用于视频生成
- ⚠️ 仅适用于图像的 LoRA（SDXL、Flux）与视频生成**不兼容**
- ⚠️ 检查文件大小 — 非常大的 LoRA（>500MB）可能增加延迟

---

## 参数深入解析

### 何时使用高噪声

| 场景 | 高噪声 LoRA | 效果 |
|------|:-:|------|
| 转换为动漫 | 动漫风格 LoRA | 完全视觉变换 |
| 油画效果 | 绘画 LoRA | 笔触纹理、色彩偏移 |
| 赛博朋克美学 | 霓虹/赛博 LoRA | 配色方案 + 光线变化 |
| 复古胶片 | 胶片颗粒 LoRA | 整体色彩调整 + 纹理 |

### 何时使用低噪声

| 场景 | 低噪声 LoRA | 效果 |
|------|:-:|------|
| 更好的皮肤纹理 | 细节 LoRA | 细微的写实感增强 |
| 色彩校正 | 色彩 LoRA | 精细的色彩调整 |
| 更清晰的细节 | 锐化 LoRA | 边缘增强 |
| 光线修正 | 光线 LoRA | 细微的光线改善 |

### 组合多个 LoRA

```json
{
  "high_noise_loras": ["anime-style.safetensors", "vibrant-colors.safetensors"],
  "low_noise_loras": ["skin-detail.safetensors"]
}
```

**经验法则：**
- 从**一个 LoRA** 开始，评估效果
- 只在需要时添加第二个
- 混合冲突的风格（例如动漫 + 写实）会产生不可预测的结果
- **少即是多** — 一个精心选择的 LoRA 通常比三个普通的效果更好

---

## 使用场景

### 1. 动漫风格

应用动漫 LoRA 将写实输入转换为动漫美学。

```
Prompt: "Anime girl transformation, cel-shading, vibrant saturated colors, studio Ghibli inspired"
high_noise_loras: [anime-style-lora]
```

### 2. 电影写实

使用电影级 LoRA 增强写实感。

```
Prompt: "Cinematic close-up, shallow depth of field, film grain, warm color grading"
low_noise_loras: [film-grain-lora, cinematic-lighting-lora]
```

### 3. 艺术滤镜

应用油画或风格化效果。

```
Prompt: "Oil painting come to life, thick brushstrokes, impressionist lighting"
high_noise_loras: [oil-painting-lora]
```

### 4. 角色一致性

使用角色 LoRA 在多个视频中保持一致的外观。

```
Prompt: "The character walks through a garden, maintaining consistent face and body"
high_noise_loras: [character-specific-lora]
low_noise_loras: [face-detail-lora]
```

### 5. 复古/怀旧

应用复古胶片美学。

```
Prompt: "1970s film aesthetic, warm grain, soft focus, golden tones"
high_noise_loras: [vintage-film-lora]
```

---

## 提示词 + LoRA 协同

你的提示词和 LoRA 应该相互配合：

| LoRA 风格 | 好的提示词关键词 | 不好的提示词关键词 |
|----------|----------------|------------------|
| 动漫 | "cel-shading, vibrant, anime, kawaii" | "photorealistic, raw photo" |
| 油画 | "brushstrokes, canvas texture, impressionist" | "sharp, 8k, photography" |
| 黑色电影 | "dramatic shadows, high contrast, black and white" | "vibrant colors, sunny" |
| 赛博朋克 | "neon, holographic, rain, dark atmosphere" | "natural, pastoral, warm" |

**提示词模板：**

```
[主体动作], [匹配 LoRA 的风格关键词], [光线], [镜头/构图]
```

---

## 故障排除

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| 风格未应用 | LoRA 不兼容 | 尝试其他 LoRA，检查架构兼容性 |
| 输出模糊 | 分辨率过低 | 使用 720p 代替 480p |
| 伪影/故障 | LoRA 太多 | 减少到 1-2 个 LoRA |
| LoRA 效果太弱 | 使用了低噪声插槽 | 改用高噪声插槽 |
| LoRA 效果太强 | 使用了高噪声插槽 | 改用低噪声插槽 |
| 风格冲突 | LoRA 之间互相矛盾 | 移除一个，使用匹配的风格 |
| 生成速度慢 | LoRA 文件太大 | 使用较小的 LoRA（<200MB） |
| 加载 LoRA 出错 | URL 错误或格式问题 | 验证 URL 是否可访问，使用 .safetensors 格式 |
| 动作伪影 | 复杂提示词 + 多个 LoRA | 简化提示词，减少 LoRA 数量 |
| 色带效应 | 源图像质量低 | 使用更高分辨率的输入图像 |

---

## 定价

| 选项 | 费用 | 最适合 |
|------|:----:|--------|
| Wan 2.2 Spicy 标准版 | $0.03/秒 | 通用 NSFW，无风格自定义 |
| **Wan 2.2 Spicy LoRA** | **$0.04/秒** | 自定义风格、动漫、艺术 |
| Wan 2.6 I2V | $0.10-0.15/秒 | 1080p，音频，无 LoRA |
| 自托管 (RunPod) | ~$0.50/小时 GPU | 完全掌控，任意 LoRA，无限制 |

在 [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) 开始使用。

---

## 常见问题

<details>
<summary><b>任何 LoRA 都可以用于 Wan 2.2 Spicy 吗？</b></summary>
不是所有 LoRA 都行 — 它需要与 Wan 架构兼容。SDXL 或 Flux 的 LoRA 不能使用。请寻找专门在 Wan 上训练或标记为视频兼容的 LoRA。
</details>

<details>
<summary><b>一次可以使用多少个 LoRA？</b></summary>
最多6个：3个高噪声 + 3个低噪声。但建议从1个开始，仅在需要时添加更多。
</details>

<details>
<summary><b>LoRA 版本只能用于 NSFW 吗？</b></summary>
Wan 2.2 Spicy LoRA 变体针对 NSFW 进行了优化，但 LoRA 技术本身适用于任何风格。你也可以将其用于 SFW 动漫、艺术、电影风格。
</details>

<details>
<summary><b>我可以训练自己的视频 LoRA 吗？</b></summary>
可以。Wan 2.1 是开源的（Apache 2.0）。使用 kohya_ss 等工具进行 LoRA 训练。然后通过 Wan 2.2 Spicy LoRA API 使用你的自定义 LoRA。
</details>

<details>
<summary><b>为什么 LoRA 版本更贵？</b></summary>
加载和应用 LoRA 适配器在推理过程中需要额外的 GPU 内存和计算，成本增加约 33%（$0.04 vs $0.03 每秒）。
</details>

<details>
<summary><b>LoRA 应该用什么文件格式？</b></summary>
`.safetensors` 是标准格式。提供一个直接下载 URL — API 会获取 LoRA 文件。
</details>

<details>
<summary><b>LoRA 会影响生成速度吗？</b></summary>
会稍有影响。每个 LoRA 都会增加少量开销。使用6个 LoRA 会比使用1个慢。通常生成时间增加 10-30%。
</details>

<details>
<summary><b>可以低成本预览 LoRA 效果吗？</b></summary>
生成一个5秒 480p 的测试（$0.20）。比直接使用8秒 720p（$0.32）便宜得多，适合初步测试。
</details>

---

## 贡献

发现了优秀的视频 LoRA？有技巧想分享？欢迎提交 PR！

---

## 许可证

[MIT](LICENSE)
