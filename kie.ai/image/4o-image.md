# KIE — 4o Image API (GPT-Image-1) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/gpt4o-image/generate -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/gpt4o-image/record-info?taskId=...
3) When successFlag=1 -> take response.result_urls[]

------------------------------------------------------------

## 1) CREATE IMAGE TASK
`POST https://api.kie.ai/api/v1/gpt4o-image/generate`

**Body (text-to-image):**
```json
{
  "prompt": "A serene mountain landscape at sunset with a lake reflecting the orange sky, photorealistic style",
  "size": "1:1",
  "nVariants": 1,
  "callBackUrl": "https://your-domain.com/4o-callback"
}
```

**Body (image editing with mask):**
```json
{
  "prompt": "Replace the masked area with a beautiful garden",
  "filesUrl": ["https://public-url/original.jpg"],
  "maskUrl": "https://public-url/mask.png",
  "size": "3:2",
  "callBackUrl": "https://your-domain.com/4o-callback"
}
```

**Body (image variants):**
```json
{
  "prompt": "Create artistic variations of this image",
  "filesUrl": ["https://public-url/source.jpg"],
  "nVariants": 3,
  "size": "1:1",
  "callBackUrl": "https://your-domain.com/4o-callback"
}
```

**Key params:**
- `prompt` (required): Text description of desired image
- `size`: Aspect ratio — "1:1" | "3:2" | "2:3"
- `filesUrl`: Array of input image URLs (for editing/variants)
- `maskUrl`: Mask image URL for inpainting (white = edit area)
- `nVariants`: Number of images to generate (1-4)
- `isEnhance`: Enable prompt enhancement (boolean)
- `enableFallback`: Enable fallback to alternative model
- `fallbackModel`: "FLUX_MAX" | "GPT_IMAGE_1"
- `callBackUrl`: Webhook URL (recommended)

**Supported input formats:** .jpg, .jpeg, .png, .webp, .jfif

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "task_4o_abc123" }
}
```

------------------------------------------------------------

## 2) POLLING STATUS + RESULTS
`GET https://api.kie.ai/api/v1/gpt4o-image/record-info?taskId=<TASK_ID>`

**Response (in progress):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "task_4o_abc123",
    "successFlag": 0,
    "progress": "0.50",
    "response": null,
    "createTime": "2024-01-15 10:30:00"
  }
}
```

**Response (completed):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "task_4o_abc123",
    "successFlag": 1,
    "progress": "1.00",
    "response": {
      "result_urls": ["https://example.com/generated-image.png"]
    },
    "completeTime": "2024-01-15 10:35:00",
    "createTime": "2024-01-15 10:30:00"
  }
}
```

**successFlag:**
- 0 = Generating (in progress)
- 1 = Success (completed)
- 2 = Failed

------------------------------------------------------------

## 3) GET DOWNLOAD URL
`POST https://api.kie.ai/api/v1/gpt4o-image/download-url`

For solving cross-domain download issues.

**Body:**
```json
{
  "url": "https://tempfile.aiquickdraw.com/.../generated.png"
}
```

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": "https://tempfile.aiquickdraw.com/.../direct-download?..."
}
```

**Note:** Download URLs expire after 20 minutes.

------------------------------------------------------------

## 4) WEBHOOK (callBackUrl)
KIE sends POST JSON to your callBackUrl.

- Method: POST
- Timeout: 15 seconds
- Handle idempotency by taskId

**Success callback:**
```json
{
  "code": 200,
  "data": {
    "taskId": "task_4o_abc123",
    "info": {
      "result_urls": ["https://.../image.png"]
    }
  }
}
```

**Failure callback:**
```json
{
  "code": 400,
  "msg": "Generation failed, please try again or contact support",
  "data": null
}
```

------------------------------------------------------------

## KEY FEATURES
- **Text-to-Image**: Generate from text descriptions
- **Image Editing**: Edit specific areas using masks
- **Image Variants**: Generate creative variations
- **Accurate Text Rendering**: Renders text in images accurately
- **Style Control**: Supports various artistic styles
- **Consistent Characters**: Maintains character consistency

------------------------------------------------------------

## ERROR CODES
| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad request / Content policy violation |
| 401 | Unauthorized (invalid API key) |
| 402 | Insufficient credits |
| 422 | Validation error |
| 429 | Rate limited |
| 500 | Server error |

------------------------------------------------------------

## BEST PRACTICES
- Use detailed, specific descriptions in prompts.
- Include style descriptors: "photorealistic", "watercolor", "digital art".
- Specify color, lighting, and composition requirements.
- Use high-quality input images for editing.
- Ensure mask images accurately mark editing areas (white = edit).
- Enable fallback for service reliability.
- Download images promptly — URLs expire after 14 days.
- Download URLs (from download-url endpoint) expire after 20 minutes.
- Don't store API key on frontend; proxy through backend.
