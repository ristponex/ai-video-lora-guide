# AI 비디오 LoRA 가이드 — AI 비디오 생성을 위한 커스텀 스타일

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/ristponex/ai-video-lora-guide/pulls)

AI 비디오 생성을 위한 LoRA (Low-Rank Adaptation) 어댑터 사용 완전 가이드 — 처음부터 훈련하지 않고도 스타일, 캐릭터, 비주얼 미학을 커스터마이즈합니다.

[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](#)

---

## 목차

- [LoRA란?](#lora란)
- [지원 모델](#지원-모델)
- [비디오에서 LoRA가 작동하는 방식](#비디오에서-lora가-작동하는-방식)
- [빠른 시작](#빠른-시작)
- [LoRA 찾기](#lora-찾기)
- [파라미터 심층 분석](#파라미터-심층-분석)
- [사용 사례](#사용-사례)
- [프롬프트 + LoRA 시너지](#프롬프트--lora-시너지)
- [문제 해결](#문제-해결)
- [가격](#가격)
- [FAQ](#faq)

---

## LoRA란?

**LoRA (Low-Rank Adaptation)**는 AI 모델을 처음부터 재훈련하지 않고 커스터마이즈하는 기술입니다. 수십억 개의 파라미터를 수정하는 대신, LoRA는 모델의 동작을 변경하는 소규모 어댑터 가중치 세트를 훈련합니다 — 카메라에 렌즈 필터를 추가하는 것과 같습니다.

비디오 생성에서 LoRA가 가능하게 하는 것:
- **커스텀 비주얼 스타일** — 애니메이션, 유화, 필름 누아르, 사이버펑크
- **캐릭터 일관성** — 여러 비디오에서 캐릭터의 외모 유지
- **품질 향상** — 피부 질감, 조명, 디테일 개선
- **예술적 제어** — 특정 아티스트 스타일이나 미학 적용

비디오용 LoRA는 이미지용보다 더 새로운 기술입니다. 이미지 LoRA (SDXL, Flux)는 널리 이용 가능하지만, 비디오 LoRA는 주로 Wan 모델 제품군에서 지원하는 성장하는 생태계입니다.

---

## 지원 모델

| 모델 | 유형 | LoRA 지원 | 가격 | 플랫폼 |
|------|------|:---------:|:----:|--------|
| ⭐ **Wan 2.2 Spicy LoRA** | 비디오 (I2V) | ✅ high 3개 + low 3개 | $0.04/초 | [Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide) |
| Wan 2.1 (오픈 소스) | 비디오 | ✅ 전체 | GPU 비용 | 셀프 호스팅 |
| Flux Dev LoRA | 이미지 | ✅ | $0.012 | Atlas Cloud |
| Flux Kontext Dev LoRA | 이미지 (편집) | ✅ | $0.012 | Atlas Cloud |
| SDXL + LoRA | 이미지 | ✅ 전체 | GPU 비용 | 셀프 호스팅 |

> **비디오 LoRA**는 현재 **Wan 모델 제품군**에서 가장 잘 지원됩니다. 다른 비디오 모델 (Kling, Seedance, Vidu)은 아직 API를 통한 커스텀 LoRA를 지원하지 않습니다.

---

## 비디오에서 LoRA가 작동하는 방식

### 두 가지 유형의 LoRA 슬롯

Wan 2.2 Spicy LoRA는 두 가지 유형의 LoRA 슬롯을 제공합니다:

#### `high_noise_loras` — 강한 스타일 영향

- 전체 구도가 결정되는 초기 생성 단계에 영향
- 비주얼 스타일, 색상 팔레트, 아트 디렉션 변경
- 적합한 용도: 애니메이션 변환, 회화풍 스타일, 드라마틱한 변환
- 최대: 3개 LoRA

#### `low_noise_loras` — 미세한 보정

- 디테일이 다듬어지는 후기 생성 단계에 영향
- 구도 변경 없이 미세한 디테일 조정
- 적합한 용도: 피부 질감, 조명 품질, 색 보정
- 최대: 3개 LoRA

### 시각적 비유

```
이미지 입력 → [High-Noise LoRA: 아트 스타일] → [모델: 모션 생성] → [Low-Noise LoRA: 디테일] → 비디오 출력
                    ↑ 전체 외관                                            ↑ 미세 디테일
```

---

## 빠른 시작

### 1단계: API 키 발급

[Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide)에서 가입 → 콘솔 → API Keys → 생성.

```bash
export ATLASCLOUD_API_KEY="your-key"
```

### 2단계: LoRA 찾기

[Civitai](https://civitai.com)에서 비디오 호환 LoRA를 찾으세요. 다운로드 URL을 복사합니다.

### 3단계: 생성

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

### 4단계: 폴링 및 다운로드

```bash
# completed가 될 때까지 폴링
curl -s "https://api.atlascloud.ai/api/v1/model/prediction/{prediction-id}" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY"

# 다운로드
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

## LoRA 찾기

### Civitai (추천)

미리보기와 평점이 있는 최대 LoRA 마켓플레이스.

- **URL**: https://civitai.com
- **검색 팁**: "wan lora", "video lora", "anime lora", "realistic lora"
- **다운로드 URL 형식**: `https://civitai.com/api/download/models/{version_id}`
- **필터**: Type → LoRA → 다운로드 순 정렬

### HuggingFace

모델 카드가 있는 오픈 소스 LoRA.

- **URL**: https://huggingface.co/models?other=lora
- **검색**: "wan", "video generation", 특정 스타일명

### Tensor.Art

비주얼 미리보기가 있는 커뮤니티 갤러리.

- **URL**: https://tensor.art
- **찾아보기**: LoRA 섹션, 유형별 필터

### 호환성 팁

- ✅ **Wan** 아키텍처에서 훈련된 LoRA가 가장 잘 작동
- ✅ 비디오 미리보기 이미지/클립이 있는 LoRA가 비디오 생성에 적합할 가능성이 높음
- ⚠️ 이미지 전용 LoRA (SDXL, Flux)는 비디오 생성과 **호환되지 않음**
- ⚠️ 파일 크기 확인 — 매우 큰 LoRA (>500MB)는 지연 시간을 증가시킬 수 있음

---

## 파라미터 심층 분석

### High-Noise 사용 시기

| 시나리오 | High-Noise LoRA | 효과 |
|----------|:-:|--------|
| 애니메이션으로 변환 | 애니메이션 스타일 LoRA | 완전한 비주얼 변환 |
| 유화 느낌 | 회화 LoRA | 붓질 질감, 색상 변화 |
| 사이버펑크 미학 | 네온/사이버 LoRA | 색상 팔레트 + 조명 변화 |
| 빈티지 필름 | 필름 그레인 LoRA | 전체 색 보정 + 질감 |

### Low-Noise 사용 시기

| 시나리오 | Low-Noise LoRA | 효과 |
|----------|:-:|--------|
| 피부 질감 개선 | 디테일 LoRA | 미세한 사실감 향상 |
| 색상 보정 | 색상 LoRA | 미세한 색상 조정 |
| 더 선명한 디테일 | 샤프닝 LoRA | 엣지 향상 |
| 조명 수정 | 조명 LoRA | 미세한 조명 개선 |

### 여러 LoRA 조합

```json
{
  "high_noise_loras": ["anime-style.safetensors", "vibrant-colors.safetensors"],
  "low_noise_loras": ["skin-detail.safetensors"]
}
```

**경험 법칙:**
- **하나의 LoRA**로 시작하여 결과를 평가
- 필요한 경우에만 두 번째 추가
- 상충하는 스타일 혼합 (예: 애니메이션 + 포토리얼리스틱)은 예측 불가능한 결과 생성
- **적을수록 좋음** — 잘 선택한 하나의 LoRA가 보통 평범한 세 개보다 나음

---

## 사용 사례

### 1. 애니메이션 스타일

애니메이션 LoRA를 적용하여 포토리얼리스틱 입력을 애니메이션 미학으로 변환.

```
Prompt: "Anime girl transformation, cel-shading, vibrant saturated colors, studio Ghibli inspired"
high_noise_loras: [anime-style-lora]
```

### 2. 시네마틱 리얼리즘

필름 품질 LoRA로 사실감 향상.

```
Prompt: "Cinematic close-up, shallow depth of field, film grain, warm color grading"
low_noise_loras: [film-grain-lora, cinematic-lighting-lora]
```

### 3. 아티스틱 필터

회화풍 또는 스타일화된 효과 적용.

```
Prompt: "Oil painting come to life, thick brushstrokes, impressionist lighting"
high_noise_loras: [oil-painting-lora]
```

### 4. 캐릭터 일관성

캐릭터 LoRA를 사용하여 여러 비디오에서 일관된 외모 유지.

```
Prompt: "The character walks through a garden, maintaining consistent face and body"
high_noise_loras: [character-specific-lora]
low_noise_loras: [face-detail-lora]
```

### 5. 레트로 / 빈티지

빈티지 필름 미학 적용.

```
Prompt: "1970s film aesthetic, warm grain, soft focus, golden tones"
high_noise_loras: [vintage-film-lora]
```

---

## 프롬프트 + LoRA 시너지

프롬프트와 LoRA는 서로를 강화해야 합니다:

| LoRA 스타일 | 좋은 프롬프트 키워드 | 나쁜 프롬프트 키워드 |
|------------|---------------------|---------------------|
| 애니메이션 | "cel-shading, vibrant, anime, kawaii" | "photorealistic, raw photo" |
| 유화 | "brushstrokes, canvas texture, impressionist" | "sharp, 8k, photography" |
| 필름 누아르 | "dramatic shadows, high contrast, black and white" | "vibrant colors, sunny" |
| 사이버펑크 | "neon, holographic, rain, dark atmosphere" | "natural, pastoral, warm" |

**프롬프트 템플릿:**

```
[피사체 동작], [LoRA에 맞는 스타일 키워드], [조명], [카메라/구도]
```

---

## 문제 해결

| 문제 | 원인 | 해결 방법 |
|------|------|----------|
| 스타일이 적용되지 않음 | 호환되지 않는 LoRA | 다른 LoRA를 시도하고 아키텍처 호환성 확인 |
| 흐릿한 출력 | 해상도가 너무 낮음 | 480p 대신 720p 사용 |
| 아티팩트/글리치 | LoRA가 너무 많음 | 1-2개 LoRA로 줄이기 |
| LoRA가 너무 미묘함 | low-noise 슬롯 사용 중 | high-noise 슬롯으로 이동 |
| LoRA가 너무 강함 | high-noise 슬롯 사용 중 | low-noise 슬롯으로 이동 |
| 스타일 충돌 | 상충하는 LoRA | 하나를 제거하고 매칭되는 스타일 사용 |
| 느린 생성 | 큰 LoRA 파일 | 더 작은 LoRA 사용 (<200MB) |
| LoRA 로딩 오류 | 잘못된 URL 또는 형식 | URL 접근 가능 여부 확인, .safetensors 사용 |
| 모션 아티팩트 | 복잡한 프롬프트 + 많은 LoRA | 프롬프트 단순화, LoRA 수 줄이기 |
| 색상 밴딩 | 저품질 소스 이미지 | 더 높은 해상도의 입력 이미지 사용 |

---

## 가격

| 옵션 | 비용 | 적합한 용도 |
|------|:----:|------------|
| Wan 2.2 Spicy Standard | $0.03/초 | 일반 NSFW, 스타일 커스터마이즈 불필요 |
| **Wan 2.2 Spicy LoRA** | **$0.04/초** | 커스텀 스타일, 애니메이션, 아티스틱 |
| Wan 2.6 I2V | $0.10-0.15/초 | 1080p, 오디오, LoRA 없음 |
| 셀프 호스팅 (RunPod) | ~$0.50/시간 GPU | 완전한 제어, 모든 LoRA, 무제한 |

[Atlas Cloud](https://www.atlascloud.ai?ref=JPM683&utm_source=github&utm_campaign=ai-video-lora-guide)에서 시작하세요.

---

## FAQ

<details>
<summary><b>Wan 2.2 Spicy에 아무 LoRA나 사용할 수 있나요?</b></summary>
아무 LoRA나 사용할 수는 없습니다 — Wan 아키텍처와 호환되어야 합니다. SDXL이나 Flux LoRA는 작동하지 않습니다. Wan에서 훈련되었거나 비디오 호환으로 표시된 LoRA를 찾으세요.
</details>

<details>
<summary><b>한 번에 몇 개의 LoRA를 사용할 수 있나요?</b></summary>
최대 6개: high-noise 3개 + low-noise 3개. 하지만 1개로 시작하여 필요한 경우에만 추가하세요.
</details>

<details>
<summary><b>LoRA 버전은 NSFW 전용인가요?</b></summary>
Wan 2.2 Spicy LoRA 변형은 NSFW에 최적화되어 있지만 LoRA 기술 자체는 모든 스타일에 작동합니다. SFW 애니메이션, 아티스틱, 시네마틱 스타일에도 사용할 수 있습니다.
</details>

<details>
<summary><b>자체 비디오 LoRA를 훈련할 수 있나요?</b></summary>
네. Wan 2.1은 오픈 소스(Apache 2.0)입니다. kohya_ss 같은 도구를 사용하여 LoRA 훈련을 할 수 있습니다. 그런 다음 커스텀 LoRA를 Wan 2.2 Spicy LoRA API와 함께 사용하세요.
</details>

<details>
<summary><b>LoRA 버전이 왜 더 비싼가요?</b></summary>
LoRA 어댑터를 로드하고 적용하려면 추론 시 추가 GPU 메모리와 컴퓨팅이 필요하며, 비용이 약 33% 증가합니다 (초당 $0.04 vs $0.03).
</details>

<details>
<summary><b>LoRA의 파일 형식은 무엇이어야 하나요?</b></summary>
`.safetensors`가 표준 형식입니다. 직접 다운로드 URL을 제공하세요 — API가 LoRA 파일을 가져옵니다.
</details>

<details>
<summary><b>LoRA가 생성 속도에 영향을 미치나요?</b></summary>
약간 있습니다. 각 LoRA가 소량의 오버헤드를 추가합니다. 6개 LoRA를 사용하면 1개보다 느려집니다. 일반적으로 생성 시간이 10-30% 증가합니다.
</details>

<details>
<summary><b>LoRA의 효과를 저렴하게 미리 볼 수 있나요?</b></summary>
5초 480p 테스트를 생성하세요 ($0.20). 초기 테스트에 8초 720p ($0.32)를 투자하는 것보다 훨씬 저렴합니다.
</details>

---

## 기여하기

훌륭한 비디오 LoRA를 발견하셨나요? 공유할 팁이 있나요? PR을 보내주세요!

---

## 라이선스

[MIT](LICENSE)
