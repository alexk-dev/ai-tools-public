# KIE — Flux Kontext API (Pro / Max) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/flux/kontext/generate -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/flux/kontext/record-info?taskId=...
3) When successFlag=1 -> take response.resultImageUrl

------------------------------------------------------------

## 1) GENERATE OR EDIT IMAGE
`POST https://api.kie.ai/api/v1/flux/kontext/generate`

**Body (text-to-image):**
```json
{
  "prompt": "A serene mountain landscape at sunset with a lake reflecting the orange sky",
  "aspectRatio": "16:9",
  "model": "flux-kontext-pro",
  "callBackUrl": "https://your-domain.com/flux-callback"
}
```

**Body (image editing):**
```json
{
  "prompt": "Add colorful hot air balloons floating in the sky",
  "inputImage": "https://public-url/landscape.jpg",
  "aspectRatio": "16:9",
  "model": "flux-kontext-max",
  "callBackUrl": "https://your-domain.com/flux-callback"
}
```

**Key params:**
- `prompt` (required): Text description for generation or editing instructions
- `model`: "flux-kontext-pro" (standard) | "flux-kontext-max" (enhanced quality)
- `aspectRatio`: "21:9" | "16:9" | "4:3" | "1:1" | "3:4" | "9:16" | "16:21"
- `inputImage`: URL of image to edit (optional, for editing mode)
- `outputFormat`: "jpeg" | "png" (default: jpeg)
- `promptUpsampling`: AI enhances prompt for better results (boolean)
- `safetyTolerance`: 0-6 for generation, 0-2 for editing (default: 2)
- `enableTranslation`: Auto-translate non-English prompts (default: true)
- `watermark`: Custom watermark text
- `uploadCn`: Use China storage (boolean, for users in China)
- `callBackUrl`: Webhook URL (recommended)

**Models:**
| Model | Description |
|-------|-------------|
| flux-kontext-pro | Standard model, balanced performance |
| flux-kontext-max | Enhanced model for complex scenes and higher quality |

**Aspect Ratios:**
| Ratio | Use Case |
|-------|----------|
| 21:9 | Cinematic/ultra-wide displays |
| 16:9 | HD video, desktop wallpapers |
| 4:3 | Traditional displays |
| 1:1 | Social media posts |
| 3:4 | Magazine layouts |
| 9:16 | Mobile/vertical content |
| 16:21 | Mobile app splash screens |

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "task_flux_abc123" }
}
```

------------------------------------------------------------

## 2) POLLING STATUS + RESULTS
`GET https://api.kie.ai/api/v1/flux/kontext/record-info?taskId=<TASK_ID>`

**Response (completed):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "task_flux_abc123",
    "successFlag": 1,
    "response": {
      "resultImageUrl": "https://example.com/generated.png"
    }
  }
}
```

**successFlag:**
- 0 = GENERATING (in progress)
- 1 = SUCCESS (completed)
- 2 = CREATE_TASK_FAILED
- 3 = GENERATE_FAILED

------------------------------------------------------------

## 3) WEBHOOK (callBackUrl)
KIE sends POST JSON to your callBackUrl.

- Method: POST
- Timeout: 15 seconds
- Handle idempotency by taskId

**Success callback:**
```json
{
  "code": 200,
  "data": {
    "taskId": "task_flux_abc123",
    "info": {
      "resultImageUrl": "https://.../generated.png"
    }
  }
}
```

------------------------------------------------------------

## KEY FEATURES
- **Text-to-Image**: Generate images from detailed text prompts
- **Image Editing**: Transform existing images with natural language
- **Subject Consistency**: Strong subject and style consistency
- **Multiple Aspect Ratios**: Support for various formats
- **Prompt Enhancement**: AI-powered prompt improvement
- **Non-Destructive Editing**: Preserves image quality

------------------------------------------------------------

## ERROR CODES
| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad request |
| 401 | Unauthorized (invalid API key) |
| 402 | Insufficient credits |
| 422 | Validation error |
| 429 | Rate limited |
| 500 | Server error |

------------------------------------------------------------

## BEST PRACTICES
- Use flux-kontext-max for complex scenes requiring high detail.
- Use flux-kontext-pro for faster, cost-effective generation.
- Enable promptUpsampling for short/simple prompts.
- For editing, describe what to change rather than describing existing content.
- Use appropriate aspect ratios for your target platform.
- Ensure input images are publicly accessible URLs.
- Use callbacks instead of frequent polling.
- Download/store outputs quickly — links can expire.
- Don't store API key on frontend; proxy through backend.
