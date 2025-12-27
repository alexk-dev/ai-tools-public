# KIE — Luma API (Video Modification) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/modify/generate -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/modify/record-info?taskId=...
3) When successFlag=1 -> take response.resultUrls[]

------------------------------------------------------------

## 1) MODIFY VIDEO
`POST https://api.kie.ai/api/v1/modify/generate`

Transform existing videos with AI-powered modifications.

**Body:**
```json
{
  "prompt": "A futuristic cityscape at night with towering glass spires reaching into a starry sky. Neon lights in blue and purple illuminate the buildings while flying vehicles glide silently between the structures.",
  "videoUrl": "https://public-url/input-video.mp4",
  "watermark": "YourBrand",
  "callBackUrl": "https://your-domain.com/luma-callback"
}
```

**Key params:**
- `prompt` (required): Describe the transformation or enhancement
- `videoUrl` (required): Source video URL (HTTPS, publicly accessible)
- `watermark`: Custom watermark text
- `callBackUrl`: Webhook URL (recommended)

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "774d9a7dd608a0e49293903095e45a4c" }
}
```

------------------------------------------------------------

## 2) POLLING STATUS + RESULTS
`GET https://api.kie.ai/api/v1/modify/record-info?taskId=<TASK_ID>`

**Response (completed):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "774d9a7dd608a0e49293903095e45a4c",
    "successFlag": 1,
    "response": {
      "resultUrls": ["https://example.com/modified-video.mp4"]
    }
  }
}
```

**successFlag:**
- 0 = GENERATING (in progress)
- 1 = SUCCESS (completed)
- 2 = CREATE_TASK_FAILED
- 3 = GENERATE_FAILED
- 4 = CALLBACK_FAILED (generation succeeded but callback failed)

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
    "taskId": "774d9a7dd608a0e49293903095e45a4c",
    "resultUrls": ["https://.../modified-video.mp4"]
  }
}
```

**Failure callback:**
```json
{
  "code": 400,
  "msg": "Generation failed",
  "data": null
}
```

------------------------------------------------------------

## KEY FEATURES
- **Video Transformation**: Apply AI-powered modifications to existing footage
- **Style Transfer**: Transform video style (e.g., realistic to anime)
- **Scene Enhancement**: Add elements, change lighting, atmosphere
- **Watermark Support**: Add custom branding to output videos

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

**Prompt Engineering:**
- Describe the desired transformation, not the original video content.
- Include atmosphere and mood: "dramatic", "ethereal", "vibrant".
- Specify lighting changes: "golden hour", "neon glow", "soft shadows".
- Add scene elements: "add falling snow", "include floating particles".

**Performance Optimization:**
- Use callbacks instead of frequent polling.
- Implement proper error handling and retry logic.
- Consider video size and complexity for processing time estimates.

**Important Limitations:**
- Input video must be publicly accessible via HTTPS.
- Video URLs remain accessible for limited time after generation.
- Download/store important outputs promptly.

**Production Tips:**
- Implement webhook endpoints for async processing.
- Ensure callback URLs are publicly accessible and secure.
- Log all requests and responses for debugging.
- Don't store API key on frontend; proxy through backend.
