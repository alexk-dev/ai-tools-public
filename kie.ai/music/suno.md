# KIE — Suno API (Music Generation) — Simple Copy-Paste Doc

## BASE
- Base URL: https://api.kie.ai
- Auth header: Authorization: Bearer <API_KEY>
- Content-Type: application/json (for POST)

## MAIN FLOW (async)
1) POST /api/v1/generate -> returns taskId
2) Either:
   - wait for webhook (callBackUrl), OR
   - poll GET /api/v1/generate/record-info?taskId=...
3) When status=SUCCESS -> take response.sunoData[].audioUrl

------------------------------------------------------------

## 1) GENERATE MUSIC
`POST https://api.kie.ai/api/v1/generate`

**Body (simple mode — customMode: false):**
```json
{
  "prompt": "A calm and relaxing piano track with soft melodies",
  "customMode": false,
  "instrumental": true,
  "model": "V4_5",
  "callBackUrl": "https://your-domain.com/suno-callback"
}
```

**Body (custom mode — customMode: true):**
```json
{
  "prompt": "[Verse 1]\nWalking through the city lights...\n[Chorus]\nWe are the dreamers...",
  "style": "Pop, Mysterious, Vocal, Drum and Bass",
  "title": "City Dreams",
  "customMode": true,
  "instrumental": false,
  "model": "V4_5",
  "callBackUrl": "https://your-domain.com/suno-callback"
}
```

**Key params:**
- `prompt` (required): Text description or lyrics
  - Non-custom mode: max 500 chars
  - Custom mode (V3_5 & V4): max 3000 chars
  - Custom mode (V4_5, V4_5PLUS, V5): max 5000 chars
- `customMode`: false (simple) or true (advanced with style & title)
- `instrumental`: true (no vocals) or false (with vocals/lyrics)
- `model`: "V3_5" | "V4" | "V4_5" | "V4_5PLUS" | "V5"
- `style`: music style tags (custom mode only)
  - V3_5 & V4: max 200 chars
  - V4_5, V4_5PLUS & V5: max 1000 chars
- `title`: song title (custom mode only)
- `callBackUrl`: webhook URL (recommended)

**AI Models:**
| Model | Description |
|-------|-------------|
| V3_5 | Best for structured songs with clear verse/chorus patterns |
| V4 | Superior vocal quality |
| V4_5 | Faster generation with smart prompt handling |
| V4_5PLUS | Highest quality, longest tracks (up to 8 mins) |
| V5 | Fastest generation with superior musicality |

**Response:**
```json
{
  "code": 200,
  "msg": "success",
  "data": { "taskId": "5c79****be8e" }
}
```

------------------------------------------------------------

## 2) POLLING STATUS + RESULTS
`GET https://api.kie.ai/api/v1/generate/record-info?taskId=<TASK_ID>`

**Response (success):**
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "5c79****be8e",
    "status": "SUCCESS",
    "response": {
      "sunoData": [
        {
          "id": "e231****8cadc7dc",
          "audioUrl": "https://example.cn/****.mp3",
          "streamAudioUrl": "https://example.cn/****",
          "imageUrl": "https://example.cn/****.jpeg",
          "prompt": "A calm and relaxing piano track",
          "title": "Peaceful Piano",
          "tags": "calm, relaxing, piano",
          "duration": 198.44,
          "createTime": "2025-01-01 00:00:00"
        }
      ]
    }
  }
}
```

**Status values:**
- PENDING: Task queued
- TEXT_SUCCESS: Lyrics generated
- FIRST_SUCCESS: First track ready
- SUCCESS: All tracks completed
- CREATE_TASK_FAILED: Task creation failed
- GENERATE_AUDIO_FAILED: Audio generation failed
- SENSITIVE_WORD_ERROR: Content policy violation

------------------------------------------------------------

## 3) EXTEND MUSIC
`POST https://api.kie.ai/api/v1/extend/generate`

**Body:**
```json
{
  "taskId": "original_task_id",
  "audioId": "audio_id_from_sunoData",
  "prompt": "Continue with a more energetic chorus",
  "continueAt": 120,
  "model": "V4_5",
  "callBackUrl": "https://your-domain.com/extend-callback"
}
```

------------------------------------------------------------

## 4) GENERATE LYRICS
`POST https://api.kie.ai/api/v1/lyrics/generate`

**Body:**
```json
{
  "prompt": "A song about summer love at the beach",
  "callBackUrl": "https://your-domain.com/lyrics-callback"
}
```

------------------------------------------------------------

## 5) ADDITIONAL ENDPOINTS

### Upload & Cover Audio
`POST https://api.kie.ai/api/v1/cover/generate`
Transform uploaded audio into different musical styles.

### Add Instrumental
`POST https://api.kie.ai/api/v1/instrumental/generate`
Generate instrumental accompaniment for uploaded audio.

### Add Vocals
`POST https://api.kie.ai/api/v1/vocals/generate`
Add vocal singing to uploaded audio files.

### Separate Vocals
`POST https://api.kie.ai/api/v1/vocal-removal/generate`
Isolate vocals and instrumentals from music.

### Convert to WAV
`POST https://api.kie.ai/api/v1/wav/generate`
Convert audio to high-quality WAV format.

### Create Music Video
`POST https://api.kie.ai/api/v1/mp4/generate`
Create video with waveform visualization.

### Boost Music Style (V4_5 models)
`POST https://api.kie.ai/api/v1/style/generate`
Enhance style descriptions for better generation.

------------------------------------------------------------

## 6) WEBHOOK (callBackUrl)
KIE sends POST JSON to your callBackUrl.

- Method: POST
- Timeout: 15 seconds
- Handle idempotency by taskId

**Success callback:**
```json
{
  "code": 200,
  "data": {
    "callbackType": "complete",
    "data": [
      {
        "audio_url": "https://.../track.mp3",
        "stream_audio_url": "https://...",
        "image_url": "https://.../cover.jpeg",
        "title": "Generated Song",
        "duration": 180.5
      }
    ]
  }
}
```

------------------------------------------------------------

## ERROR CODES
| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Content policy violation |
| 401 | Unauthorized (invalid API key) |
| 402 | Insufficient credits |
| 422 | Validation error |
| 429 | Rate limited |
| 500 | Server error |

------------------------------------------------------------

## BEST PRACTICES
- Be specific about genre, mood, and instruments in prompts.
- Use V4_5 or V5 for faster generation.
- Use V4_5PLUS for highest quality and longest tracks.
- Use callbacks as primary, polling as fallback.
- Cache generated content — files expire after 14 days.
- Don't store API key on frontend; proxy through backend.
