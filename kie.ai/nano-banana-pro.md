# Kie.ai API — Google Nano Banana Pro (nano-banana-pro) — Detailed Spec

## 0) Basic Concepts
- **Asynchronous model**: you create a task → receive `taskId` → wait for a callback (recommended) or poll the status → receive `resultUrls`.
- **Unified "Market" framework** for tasks:
  - Create Task: `POST https://api.kie.ai/api/v1/jobs/createTask`
  - Query Task:  `GET  https://api.kie.ai/api/v1/jobs/recordInfo?taskId=...`

---

## 1) Authentication
### Header
- `Authorization: Bearer <YOUR_API_KEY>`
- `Content-Type: application/json` (for POST JSON)

---

## 2) Endpoints (Minimal Required Set)

### 2.1. Check Credit Balance
- `GET https://api.kie.ai/api/v1/chat/credit`
Response:
```json
{
  "code": 200,
  "msg": "success",
  "data": 100
}
```

> Note: Different sections of the documentation mention different endpoints for "credits", but `GET /api/v1/chat/credit` is explicitly described in the Common API.

---

## 3) Preparing Input Images (image_input)
For `nano-banana-pro`, the `image_input` parameter is an array (up to 8 images).

There are two working approaches:

### 3.1. Upload by URL (server downloads the file from your link)
- `POST https://kieai.redpandaai.co/api/v1/file/upload-url`
Body:
```json
{
  "url": "https://example.com/my-image.png"
}
```
Response (approximately):
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "url": "https://tempfile.aiquickdraw.com/...."
  }
}
```

### 3.2. Upload by Stream (multipart/form-data)
- `POST https://kieai.redpandaai.co/api/v1/file/upload-stream`
- Send the file as a multipart field `file`
Response (approximately):
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "url": "https://tempfile.aiquickdraw.com/...."
  }
}
```

### 3.3. Input Image Restrictions
- Formats: JPEG, PNG, WEBP
- Maximum single file size: 30MB
- Maximum files: 8

---

## 4) Create Task: nano-banana-pro

### 4.1. Endpoint
`POST https://api.kie.ai/api/v1/jobs/createTask`

### 4.2. Headers
```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

### 4.3. Request Body (JSON)
```json
{
  "model": "nano-banana-pro",
  "callBackUrl": "https://your-domain.com/api/callback",   // optional, but recommended
  "input": {
    "prompt": "string, required",
    "image_input": ["<image-url-1>", "<image-url-2>", "..."], // array, up to 8
    "aspect_ratio": "1:1 | 2:3 | 3:2 | 3:4 | 4:3 | 4:5 | 5:4 | 9:16 | 16:9 | 21:9 | auto",
    "resolution": "1K | 2K | 4K",
    "output_format": "png | jpg"
  }
}
```

### 4.4. Input Parameters (Detailed)
- **prompt** (string, required)
  - Text description of the desired result.
- **image_input** (array)
  - Links to images to be transformed/used as references.
  - In practice: use the URL obtained from the File Upload API (upload-url / upload-stream) to ensure the link is publicly accessible.
- **aspect_ratio** (string)
  - Values: `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`, `auto`
- **resolution** (string)
  - `1K`, `2K`, `4K`
- **output_format** (string)
  - `png`, `jpg`

### 4.5. Successful CreateTask Response
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "task_nano-banana-pro_1765178625768"
  }
}
```

### 4.6. Errors (Typical HTTP/Logic Codes)
(list of "standard" codes for Market)
- 401 Unauthorized (invalid/missing API key)
- 402 Insufficient Credits (not enough credits)
- 404 Not Found (endpoint/resource not found)
- 422 Validation Error (parameters failed validation)
- 429 Rate Limited (request limit exceeded)
- 455 Service Unavailable (maintenance)
- 500 Server Error
- 505 Feature Disabled

---

## 5) Getting Results: Polling (recordInfo)

### 5.1. Endpoint
`GET https://api.kie.ai/api/v1/jobs/recordInfo?taskId=<taskId>`

### 5.2. Response (Structure)
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "taskId": "task_12345678",
    "model": "nano-banana-pro",
    "state": "waiting|queuing|generating|success|fail",
    "param": "{...json string of original request...}",
    "resultJson": "{\"resultUrls\":[\"https://.../generated.png\"]}",
    "failCode": "",
    "failMsg": "",
    "completeTime": 1698765432000,
    "createTime": 1698765400000,
    "updateTime": 1698765432000
  }
}
```

### 5.3. State (Lifecycle)
- waiting
- queuing
- generating
- success
- fail

### 5.4. Where the Final URLs Are Located
- In `data.resultJson` — this is a **string** containing JSON.
- Inside: `{"resultUrls":[ "...", ... ]}`

---

## 6) Getting Results: Callback (callBackUrl)

### 6.1. General Mechanics
- Method: POST
- Content-Type: application/json
- Handler timeout: 15 seconds (it's recommended to respond quickly and offload heavy logic asynchronously).
- Important: the same `taskId` may arrive **multiple times** → processing should be **idempotent**.

### 6.2. Callback Payload Format (Typical Kie.ai Pattern)
Different products have slightly different fields, but the general template is:
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "taskId": "task12345",
    "info": {
      "resultUrls": ["https://.../result.png"]
      // sometimes the key is written as result_urls (snake_case) — account for this when parsing
    }
  }
}
```

Recommendation for reliability:
- Accept both key variants: `resultUrls` and `result_urls`.
- If the callback payload is insufficient/doesn't match — you can always fetch the result via `GET /api/v1/jobs/recordInfo`.

---

## 7) Download / "Direct" Links and URL Lifespan

### 7.1. If You Received a URL Pointing to Kie Temp Storage
Kie.ai provides an endpoint that returns a **direct download URL** for generated files:
- `POST https://api.kie.ai/api/v1/chat/download-url`

Body:
```json
{
  "url": "https://tempfile.aiquickdraw.com/.../generated.png"
}
```

Response:
```json
{
  "code": 200,
  "msg": "success",
  "data": "https://tempfile.aiquickdraw.com/.../direct-download?..."
}
```

Notes:
- The direct download URL may be **temporary** (the documentation indicates a short TTL — on the order of minutes).
- This method is applicable for URLs generated by Kie.ai.

Practical recommendation:
- Download files to your own storage immediately (S3/R2/GCS), and work with your own URLs in the application.

---

## 8) End-to-End Example (cURL)

### 8.1. (Optional) Upload an Image (if you already have a public link — you can skip this)
```bash
curl -X POST "https://kieai.redpandaai.co/api/v1/file/upload-url" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/source.png"}'
```

-> take data.url from the response (let's call it UPLOADED_URL)

### 8.2. Create a Task
```bash
curl -X POST "https://api.kie.ai/api/v1/jobs/createTask" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"nano-banana-pro",
    "callBackUrl":"https://your-domain.com/api/callback",
    "input":{
      "prompt":"Make it a cinematic poster, keep character identity stable, sharp typography.",
      "image_input":["UPLOADED_URL"],
      "aspect_ratio":"4:5",
      "resolution":"2K",
      "output_format":"png"
    }
  }'
```

-> you will receive data.taskId

### 8.3. Poll Status (if without callback)
```bash
curl -X GET "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=TASK_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

-> wait for state=success, parse data.resultJson, take resultUrls[0]

### 8.4. (Optional) Get Direct Download Link for Kie Temp URL
```bash
curl -X POST "https://api.kie.ai/api/v1/chat/download-url" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"RESULT_URL_FROM_KIE"}'
```

---

## 9) Practical Integration Notes
- Always make `taskId` processing idempotent (especially for callbacks).
- For callback handlers: respond with HTTP 200 quickly, and offload downloading/post-processing to a queue.
- In production, callbacks are preferred, with polling as a fallback.
- Don't store the API key on the client (browser/mobile). Proxy through your own backend.
