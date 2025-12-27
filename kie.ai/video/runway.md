# KIE — Runway API (Gen4 + Aleph) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/runway/generate -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/runway/record-detail?taskId=...
3) When state="success" -> take videoInfo.videoUrl

------------------------------------------------------------

## RUNWAY GEN4 — VIDEO GENERATION

### 1) CREATE VIDEO TASK
`POST https://api.kie.ai/api/v1/runway/generate`

**Body (text-to-video):**
```json
{
  "prompt": "A fluffy orange cat dancing energetically in a colorful room with disco lights",
  "duration": 5,
  "quality": "720p",
  "aspectRatio": "16:9",
  "waterMark": "",
  "callBackUrl": "https://your-domain.com/runway-callback"
}
```

**Body (image-to-video):**
```json
{
  "prompt": "The character starts walking forward with confidence",
  "imageUrl": "https://public-url/character.jpg",
  "duration": 5,
  "quality": "720p",
  "aspectRatio": "16:9",
  "callBackUrl": "https://your-domain.com/runway-callback"
}
```

**Key params:**
- `prompt` (required): Describe actions, movements, camera angles
- `imageUrl`: Source image URL (optional, for image-to-video)
- `duration`: 5 (5 seconds) | 10 (10 seconds, only 720p)
- `quality`: "720p" (all durations) | "1080p" (only 5-second videos)
- `aspectRatio`: "16:9" | "9:16" | "1:1" | "4:3" | "3:4"
- `waterMark`: Custom watermark text
- `callBackUrl`: Webhook URL (recommended)

**Quality & Duration Compatibility:**
| Duration | 720p | 1080p |
|----------|------|-------|
| 5 sec | ✅ | ✅ |
| 10 sec | ✅ | ❌ |

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "ee603959-debb-48d1-98c4-a6d1c717eba6" }
}
```

### 2) POLLING STATUS
`GET https://api.kie.ai/api/v1/runway/record-detail?taskId=<TASK_ID>`

**Response (success):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "ee603959-debb-48d1-98c4-a6d1c717eba6",
    "state": "success",
    "generateTime": "2023-08-15 14:30:45",
    "videoInfo": {
      "videoId": "485da89c-7fca-4340-8c04-101025b2ae71",
      "videoUrl": "https://file.com/k/xxxxxxx.mp4",
      "imageUrl": "https://file.com/m/xxxxxxxx.png"
    },
    "expireFlag": 0
  }
}
```

**State values:**
- wait: Queued
- queueing: In queue
- generating: Processing
- success: Completed
- fail: Failed

### 3) EXTEND VIDEO
`POST https://api.kie.ai/api/v1/runway/extend`

**Body:**
```json
{
  "taskId": "original_task_id",
  "prompt": "The cat continues dancing with more energy and spins around",
  "quality": "720p",
  "callBackUrl": "https://your-domain.com/extend-callback"
}
```

------------------------------------------------------------

## RUNWAY ALEPH — VIDEO-TO-VIDEO

### 1) CREATE ALEPH TASK
`POST https://api.kie.ai/api/v1/aleph/generate`

Transforms existing video footage using AI.

**Body:**
```json
{
  "prompt": "Transform into a dreamy watercolor painting style with soft flowing movements",
  "videoUrl": "https://public-url/input-video.mp4",
  "aspectRatio": "16:9",
  "seed": 123456,
  "referenceImage": "https://public-url/style-reference.jpg",
  "waterMark": "YourBrand",
  "uploadCn": false,
  "callBackUrl": "https://your-domain.com/aleph-callback"
}
```

**Key params:**
- `prompt` (required): Describe style transformation, camera movements
- `videoUrl` (required): Source video URL (max 10MB, HTTPS)
- `aspectRatio`: "16:9" | "9:16" | "4:3" | "3:4" | "1:1" | "21:9"
- `seed`: Reproducibility seed (integer)
- `referenceImage`: Style reference image URL
- `waterMark`: Custom watermark (1-20 chars recommended)
- `uploadCn`: Use China storage (for users in China)
- `callBackUrl`: Webhook URL (recommended)

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "ee603959-debb-48d1-98c4-a6d1c717eba6" }
}
```

### 2) POLLING ALEPH STATUS
`GET https://api.kie.ai/api/v1/aleph/record-detail?taskId=<TASK_ID>`

Same response structure as Gen4 polling.

------------------------------------------------------------

## WEBHOOK (callBackUrl)
KIE sends POST JSON to your callBackUrl.

- Method: POST
- Timeout: 15 seconds
- Handle idempotency by taskId

**Success callback:**
```json
{
  "code": 200,
  "data": {
    "taskId": "ee603959-...",
    "videoInfo": {
      "videoUrl": "https://.../video.mp4",
      "imageUrl": "https://.../thumbnail.png"
    }
  }
}
```

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
| 451 | Failed to fetch image/video |
| 500 | Server error |

------------------------------------------------------------

## BEST PRACTICES

**Prompt Engineering for Videos:**
- Focus on actions and movements, not static descriptions
- Include temporal elements: "gradually", "suddenly", "slowly"
- Describe camera movements: "zoom in", "pan left", "tracking shot"
- Add lighting/atmosphere: "golden hour", "dramatic shadows"

**Performance:**
- Use callbacks instead of frequent polling.
- Choose 720p for longer videos (10s) or when file size matters.
- Use 1080p for high-quality short videos (5s only).
- Implement exponential backoff for retries.

**Aleph-specific:**
- Max input video size: 10MB
- Describe transformations, not what's already in the video.
- Use reference images for consistent style transfer.
- Consider camera movements in prompts.

**Storage:**
- Video URLs remain accessible for 14 days.
- expireFlag: 0 = active, 1 = expired.
- Download/store important videos before expiration.
- Don't store API key on frontend; proxy through backend.
