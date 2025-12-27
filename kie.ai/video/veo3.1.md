# KIE — Veo 3.1 API (veo3 / veo3_fast) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/veo/generate  -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/veo/record-info?taskId=...
3) When success -> take resultUrls[] (mp4 links)
4) Optional (only if eligible): GET /api/v1/veo/get-1080p-video?taskId=...&index=0
5) Optional: extend with POST /api/v1/veo/extend

------------------------------------------------------------

## 1) CREATE VIDEO TASK
`POST https://api.kie.ai/api/v1/veo/generate`

**Body (text-to-video):**
```json
{
  "prompt": "Cinematic shot of a dinosaur astronaut walking on red sand, dust particles, shallow depth of field",
  "model": "veo3_fast",
  "aspectRatio": "16:9",
  "seeds": 12345,
  "enableTranslation": true,
  "watermark": "MyBrand",
  "callBackUrl": "https://your-domain.com/veo-callback"
}
```

**Body (image-to-video; 1–2 images):**
```json
{
  "prompt": "The image comes alive: subtle camera push-in, wind, dust, cinematic lighting",
  "imageUrls": ["https://public-url/image1.jpg", "https://public-url/image2.jpg"],
  "model": "veo3_fast",
  "generationType": "FIRST_AND_LAST_FRAMES_2_VIDEO",
  "aspectRatio": "9:16",
  "callBackUrl": "https://your-domain.com/veo-callback"
}
```

**Body (reference/material-to-video; ONLY veo3_fast + ONLY 16:9 + 1–3 images):**
```json
{
  "prompt": "Turn the references into a coherent cinematic scene, smooth motion, consistent style",
  "imageUrls": ["https://public-url/ref1.jpg", "https://public-url/ref2.jpg"],
  "model": "veo3_fast",
  "generationType": "REFERENCE_2_VIDEO",
  "aspectRatio": "16:9",
  "callBackUrl": "https://your-domain.com/veo-callback"
}
```

**Key params:**
- `prompt` (required)
- `model`: "veo3" (quality) OR "veo3_fast" (fast). Default usually veo3_fast.
- `imageUrls`: optional array of public image URLs (KIE must be able to fetch them)
- `generationType`: TEXT_2_VIDEO | FIRST_AND_LAST_FRAMES_2_VIDEO | REFERENCE_2_VIDEO
- `aspectRatio`: "16:9" | "9:16" | "Auto"
- `seeds`: int 10000..99999 (optional)
- `enableTranslation`: boolean (default true)
- `watermark`: string (optional)
- `callBackUrl`: string (recommended)
- `enableFallback`: deprecated (better NOT to send)

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "veo_task_xxx" }
}
```

------------------------------------------------------------

## 2) POLLING STATUS + RESULTS
`GET https://api.kie.ai/api/v1/veo/record-info?taskId=<TASK_ID>`

**Response (example):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "veo_task_xxx",
    "successFlag": 0,
    "fallbackFlag": false,
    "response": {
      "resultUrls": ["https://.../video.mp4"],
      "originUrls": ["https://.../original.mp4"],
      "resolution": "1080p"
    },
    "errorCode": null,
    "errorMessage": ""
  }
}
```

**successFlag:**
- 0 = generating (in progress)
- 1 = success
- 2 = failed
- 3 = generation failed

**Notes:**
- `originUrls` usually only matters if aspectRatio != 16:9 (consider the field optional).
- Documentation mentions a record storage limitation of ~14 days: old taskIds may return 422/record is null.

------------------------------------------------------------

## 3) GET 1080P VIDEO (HD UPGRADE)
`GET https://api.kie.ai/api/v1/veo/get-1080p-video?taskId=<TASK_ID>&index=0`

**Rules:**
- Only works for aspectRatio = 16:9
- Requires additional processing (~2 minutes). If not ready — retry later.
- If fallbackFlag=true — this endpoint may be unavailable (consider it "not supported").

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "resultUrl": "https://.../1080p.mp4" }
}
```

------------------------------------------------------------

## 4) EXTEND VIDEO
`POST https://api.kie.ai/api/v1/veo/extend`

**Body:**
```json
{
  "taskId": "veo_task_xxx",
  "prompt": "Continue: the astronaut dinosaur stops, looks at the horizon, rover approaches, wind intensifies",
  "seeds": 12345,
  "watermark": "MyBrand",
  "callBackUrl": "https://your-domain.com/veo-extend-callback"
}
```

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "veo_extend_task_yyy" }
}
```

**Important:**
- Per documentation: videos that already have 1080p generation/upgrade in progress cannot be extended. Do extend BEFORE 1080p.

------------------------------------------------------------

## 5) WEBHOOK (callBackUrl)
KIE sends POST JSON to your callBackUrl.

- Method: POST
- Timeout on your server: 15 seconds
- If delivery fails repeatedly, KIE may stop retrying.
- Handle idempotency by taskId (same task may be sent more than once).

**Success callback (example):**
```json
{
  "code": 200,
  "msg": "Veo3.1 video generated successfully.",
  "data": {
    "taskId": "veo_task_xxx",
    "info": {
      "resultUrls": ["https://.../video.mp4"],
      "originUrls": ["https://.../original.mp4"],
      "resolution": "1080p"
    },
    "fallbackFlag": false
  }
}
```

**Failure callback:** code often 400/422/500/501 with msg explaining the reason
(e.g. policy flagged prompt, failed to fetch image URL, unsafe image, etc.)

------------------------------------------------------------

## BEST PRACTICES
- Do NOT put API key on frontend. Call KIE from your backend.
- Make imageUrls public and stable (no hotlink protection / short-lived signed URLs).
- Download/store outputs to your own storage quickly (links can expire).
- Use callback as primary, polling as fallback.
