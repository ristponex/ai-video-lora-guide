# AI動画LoRAガイド — AI動画生成のカスタムスタイル

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/ristponex/ai-video-lora-guide/pulls)

AI動画生成のためのLoRA（Low-Rank Adaptation）アダプターの使用に関する完全ガイド — ゼロからのトレーニングなしで、スタイル、キャラクター、ビジュアルの美学をカスタマイズできます。

[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](#) | [한국어](README.ko.md)

---

## 目次

- [LoRAとは？](#loraとは)
- [対応モデル](#対応モデル)
- [動画におけるLoRAの仕組み](#動画におけるloraの仕組み)
- [クイックスタート](#クイックスタート)
- [LoRAの探し方](#loraの探し方)
- [パラメータ詳細解説](#パラメータ詳細解説)
- [ユースケース](#ユースケース)
- [プロンプト + LoRAの相乗効果](#プロンプト--loraの相乗効果)
- [トラブルシューティング](#トラブルシューティング)
- [料金](#料金)
- [FAQ](#faq)

---

## LoRAとは？

**LoRA（Low-Rank Adaptation）**は、AIモデルをゼロから再トレーニングすることなくカスタマイズする技術です。数十億のパラメータを変更する代わりに、LoRAはモデルの動作を修正する小さなアダプター重みのセットを訓練します — カメラにレンズフィルターを追加するようなものです。

動画生成において、LoRAは以下を可能にします：
- **カスタムビジュアルスタイル** — アニメ、油絵、フィルム・ノワール、サイバーパンク
- **キャラクターの一貫性** — 複数の動画でキャラクターの外観を維持
- **品質向上** — 肌のテクスチャ、ライティング、ディテールの改善
- **芸術的制御** — 特定のアーティストスタイルや美学の適用

動画用LoRAは画像用より新しい技術です。画像LoRA（SDXL、Flux）は広く利用可能ですが、動画LoRAは主にWanモデルファミリーでサポートされている成長中のエコシステムです。

---

## 対応モデル

| モデル | タイプ | LoRAサポート | 価格 | プラットフォーム |
|-------|------|:------------:|:-----:|----------|
| ⭐ **Wan 2.2 Spicy LoRA** | 動画（I2V） | ✅ ハイ3 + ロー3 | $0.04/秒 | [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) |
| Wan 2.1（オープンソース） | 動画 | ✅ フル | GPU費用 | セルフホスト |
| Flux Dev LoRA | 画像 | ✅ | $0.012 | Atlas Cloud |
| Flux Kontext Dev LoRA | 画像（編集） | ✅ | $0.012 | Atlas Cloud |
| SDXL + LoRA | 画像 | ✅ フル | GPU費用 | セルフホスト |

> **動画LoRA**は現在、**Wanモデルファミリー**で最も充実したサポートがあります。他の動画モデル（Kling、Seedance、Vidu）はまだAPI経由でのカスタムLoRAをサポートしていません。

---

## 動画におけるLoRAの仕組み

### 2種類のLoRAスロット

Wan 2.2 Spicy LoRAは2種類のLoRAスロットを提供しています：

#### `high_noise_loras` — 強いスタイル影響

- 全体的な構図が決定される初期生成ステップに影響
- ビジュアルスタイル、カラーパレット、アートディレクションを変更
- 最適な用途：アニメ変換、絵画調スタイル、劇的な変換
- 最大：3つのLoRA

#### `low_noise_loras` — 微妙なリファインメント

- ディテールがリファインされる後期生成ステップに影響
- 構図を変えずに細かなディテールを調整
- 最適な用途：肌のテクスチャ、ライティング品質、カラーグレーディング
- 最大：3つのLoRA

### ビジュアルアナロジー

```
画像入力 → [ハイノイズLoRA: アートスタイル] → [モデル: モーション生成] → [ローノイズLoRA: ディテール] → 動画出力
                    ↑ 全体的な見た目                                            ↑ 細かなディテール
```

---

## クイックスタート

### ステップ1：APIキーを取得

[Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) → コンソール → APIキー → 作成でサインアップ。

```bash
export ATLASCLOUD_API_KEY="your-key"
```

### ステップ2：LoRAを見つける

[Civitai](https://civitai.com)で動画互換LoRAを閲覧。ダウンロードURLをコピー。

### ステップ3：生成

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

### ステップ4：ポーリング＆ダウンロード

```bash
# completedになるまでポーリング
curl -s "https://api.atlascloud.ai/api/v1/model/prediction/{prediction-id}" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY"

# ダウンロード
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

## LoRAの探し方

### Civitai（推奨）

プレビューと評価付きの最大のLoRAマーケットプレイスです。

- **URL**: https://civitai.com
- **検索のコツ**: "wan lora", "video lora", "anime lora", "realistic lora"
- **ダウンロードURLフォーマット**: `https://civitai.com/api/download/models/{version_id}`
- **フィルター**: Type → LoRA → Most Downloadedでソート

### HuggingFace

モデルカード付きのオープンソースLoRA。

- **URL**: https://huggingface.co/models?other=lora
- **検索**: "wan", "video generation", 具体的なスタイル名

### Tensor.Art

ビジュアルプレビュー付きのコミュニティギャラリー。

- **URL**: https://tensor.art
- **閲覧**: LoRAセクション、タイプでフィルター

### 互換性のコツ

- ✅ **Wan**アーキテクチャで訓練されたLoRAが最も良く動作
- ✅ 動画プレビュー画像/クリップがあるLoRAは動画で動作する可能性が高い
- ⚠️ 画像専用LoRA（SDXL、Flux）は動画生成と**互換性なし**
- ⚠️ ファイルサイズを確認 — 非常に大きなLoRA（500MB以上）はレイテンシーが増加する可能性あり

---

## パラメータ詳細解説

### ハイノイズを使うタイミング

| シナリオ | ハイノイズLoRA | 効果 |
|----------|:-:|--------|
| アニメに変換 | アニメスタイルLoRA | 完全なビジュアル変換 |
| 油絵調 | ペインティングLoRA | 筆跡テクスチャ、色調変更 |
| サイバーパンク美学 | ネオン/サイバーLoRA | カラーパレット + ライティング変更 |
| ビンテージフィルム | フィルムグレインLoRA | 全体的なカラーグレーディング + テクスチャ |

### ローノイズを使うタイミング

| シナリオ | ローノイズLoRA | 効果 |
|----------|:-:|--------|
| 肌のテクスチャ改善 | ディテールLoRA | 微妙なリアリズム強化 |
| 色補正 | カラーLoRA | 細かな色調整 |
| シャープなディテール | シャープニングLoRA | エッジ強化 |
| ライティング修正 | ライティングLoRA | 微妙なライティング改善 |

### 複数のLoRAの組み合わせ

```json
{
  "high_noise_loras": ["anime-style.safetensors", "vibrant-colors.safetensors"],
  "low_noise_loras": ["skin-detail.safetensors"]
}
```

**経験則：**
- まず**1つのLoRA**で結果を評価
- 必要な場合のみ2つ目を追加
- 相反するスタイル（例：アニメ + フォトリアリスティック）の混合は予測不能な結果を生む
- **少ないほうが良い** — 適切に選ばれた1つのLoRAは、中途半端な3つに勝ることが多い

---

## ユースケース

### 1. アニメスタイル

アニメLoRAを適用してフォトリアリスティックな入力をアニメ美学に変換。

```
Prompt: "Anime girl transformation, cel-shading, vibrant saturated colors, studio Ghibli inspired"
high_noise_loras: [anime-style-lora]
```

### 2. シネマティックリアリズム

映画品質のLoRAでリアリズムを強化。

```
Prompt: "Cinematic close-up, shallow depth of field, film grain, warm color grading"
low_noise_loras: [film-grain-lora, cinematic-lighting-lora]
```

### 3. アーティスティックフィルター

絵画調やスタイライズドなエフェクトを適用。

```
Prompt: "Oil painting come to life, thick brushstrokes, impressionist lighting"
high_noise_loras: [oil-painting-lora]
```

### 4. キャラクターの一貫性

キャラクターLoRAを使用して複数の動画で一貫した外観を維持。

```
Prompt: "The character walks through a garden, maintaining consistent face and body"
high_noise_loras: [character-specific-lora]
low_noise_loras: [face-detail-lora]
```

### 5. レトロ / ビンテージ

ビンテージフィルムの美学を適用。

```
Prompt: "1970s film aesthetic, warm grain, soft focus, golden tones"
high_noise_loras: [vintage-film-lora]
```

---

## プロンプト + LoRAの相乗効果

プロンプトとLoRAは互いを強化するように設計すべきです：

| LoRAスタイル | 良いプロンプトキーワード | 悪いプロンプトキーワード |
|------------|---------------------|---------------------|
| アニメ | "cel-shading, vibrant, anime, kawaii" | "photorealistic, raw photo" |
| 油絵 | "brushstrokes, canvas texture, impressionist" | "sharp, 8k, photography" |
| フィルム・ノワール | "dramatic shadows, high contrast, black and white" | "vibrant colors, sunny" |
| サイバーパンク | "neon, holographic, rain, dark atmosphere" | "natural, pastoral, warm" |

**プロンプトテンプレート：**

```
[被写体のアクション], [LoRAに合ったスタイルキーワード], [ライティング], [カメラ/構図]
```

---

## トラブルシューティング

| 問題 | 原因 | 解決策 |
|-------|-------|-----|
| スタイルが適用されない | 互換性のないLoRA | 別のLoRAを試す、アーキテクチャ互換性を確認 |
| ぼやけた出力 | 解像度が低すぎる | 480pの代わりに720pを使用 |
| アーティファクト/グリッチ | LoRAが多すぎる | 1-2つのLoRAに減らす |
| LoRAが弱すぎる | ローノイズスロットを使用 | ハイノイズスロットに移動 |
| LoRAが強すぎる | ハイノイズスロットを使用 | ローノイズスロットに移動 |
| スタイルの競合 | 相反するLoRA | 1つを削除し、マッチするスタイルを使用 |
| 生成が遅い | 大きなLoRAファイル | より小さなLoRA（200MB未満）を使用 |
| LoRA読み込みエラー | 不正なURLまたはフォーマット | URLがアクセス可能か確認、.safetensorsを使用 |
| モーションアーティファクト | 複雑なプロンプト + 多数のLoRA | プロンプトを簡素化、LoRA数を減らす |
| カラーバンディング | 低品質なソース画像 | より高解像度の入力画像を使用 |

---

## 料金

| オプション | コスト | 最適な用途 |
|--------|:----:|----------|
| Wan 2.2 Spicy Standard | $0.03/秒 | 一般的なNSFW、スタイルカスタマイズなし |
| **Wan 2.2 Spicy LoRA** | **$0.04/秒** | カスタムスタイル、アニメ、アーティスティック |
| Wan 2.6 I2V | $0.10-0.15/秒 | 1080p、音声、LoRAなし |
| セルフホスト（RunPod） | 〜$0.50/時間 GPU | 完全制御、任意のLoRA、無制限 |

[Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) で始めましょう。

---

## FAQ

<details>
<summary><b>Wan 2.2 Spicyで任意のLoRAを使用できますか？</b></summary>
すべてのLoRAが使えるわけではありません — Wanアーキテクチャと互換性がある必要があります。SDXLやFluxのLoRAは動作しません。Wanで訓練された、または動画互換とマークされたLoRAを探してください。
</details>

<details>
<summary><b>一度にいくつのLoRAを使用できますか？</b></summary>
最大6つ：ハイノイズ3 + ローノイズ3。ただし、1つから始めて必要な場合のみ追加してください。
</details>

<details>
<summary><b>LoRA版はNSFW専用ですか？</b></summary>
Wan 2.2 Spicy LoRAバリアントはNSFW向けに最適化されていますが、LoRA技術自体はどのスタイルにも使えます。SFWのアニメ、アーティスティック、シネマティックスタイルにも使用できます。
</details>

<details>
<summary><b>独自の動画LoRAを訓練できますか？</b></summary>
はい。Wan 2.1はオープンソース（Apache 2.0）です。kohya_ssなどのツールをLoRA訓練に使用できます。その後、カスタムLoRAをWan 2.2 Spicy LoRA APIで使用できます。
</details>

<details>
<summary><b>なぜLoRA版の方が高いのですか？</b></summary>
LoRAアダプターのロードと適用には推論時に追加のGPUメモリとコンピュートが必要で、コストが約33%増加します（秒あたり$0.04 vs $0.03）。
</details>

<details>
<summary><b>LoRAのファイルフォーマットは何にすべきですか？</b></summary>
`.safetensors` が標準フォーマットです。直接ダウンロードURLを提供してください — APIがLoRAファイルをフェッチします。
</details>

<details>
<summary><b>LoRAは生成速度に影響しますか？</b></summary>
わずかに。各LoRAが小さなオーバーヘッドを追加します。6つのLoRAの使用は1つの場合より遅くなります。通常、生成時間が10-30%増加する程度です。
</details>

<details>
<summary><b>LoRAの効果を安くプレビューできますか？</b></summary>
5秒480pのテストを生成してください（$0.20）。初期テストには8秒720p（$0.32）にコミットするよりはるかに安価です。
</details>

---

## コントリビューション

優れた動画LoRAを見つけましたか？共有したいコツがありますか？PRを開いてください！

---

## ライセンス

[MIT](LICENSE)
