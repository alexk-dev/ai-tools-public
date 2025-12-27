# 🤖 AI Tools Specs

> My personal (but public) collection of API specifications and integration notes for various AI services I use.

---

## 📂 What's Inside

A curated library of detailed API documentation, quick-reference guides, and integration patterns for generative AI platforms.

| Provider | Service | Description |
|----------|---------|-------------|
| [Kie.ai](./kie.ai/) | **Video** | |
| | [Veo 3.1](./kie.ai/video/veo3.1.md) | Text-to-video & image-to-video generation |
| | [Runway](./kie.ai/video/runway.md) | Video generation (Gen4 + Aleph) |
| | [Luma](./kie.ai/video/luma.md) | AI-powered video modification |
| [Kie.ai](./kie.ai/) | **Image** | |
| | [Nano Banana Pro](./kie.ai/image/nano-banana-pro.md) | Image generation/transformation API |
| | [4o Image](./kie.ai/image/4o-image.md) | GPT-Image-1 text-to-image & editing |
| | [Flux Kontext](./kie.ai/image/flux-kontext.md) | Image generation & editing (Pro/Max) |
| [Kie.ai](./kie.ai/) | **Music** | |
| | [Suno](./kie.ai/music/suno.md) | AI music generation (V3.5/V4/V4.5/V5) |

---

## 💰 Kie.ai Pricing

> **1 credit = $0.005 USD**

| Model | Variant | Credits | Price (USD) |
|-------|---------|--------:|------------:|
| **Veo 3.1** | Fast | 60 | $0.30 |
| | Quality | 250 | $1.25 |
| **Runway** | Aleph (video-to-video) | 110 | $0.55 |
| | Gen-3 Alpha (5s, 720p) | 12 | $0.06 |
| | Gen-3 Alpha (10s, 720p) | 30 | $0.15 |
| | Gen-3 Alpha (5s, 1080p) | 30 | $0.15 |
| **Luma** | Dream Machine (5s, 720p) | 28 | $0.14 |
| **4o Image** | Text-to-Image | 6 | $0.03 |
| **Flux Kontext** | Pro | 5 | $0.025 |
| | Max | 10 | $0.05 |
| **Nano Banana** | Pro 1K/2K | 18 | $0.09 |
| | Pro 4K | 24 | $0.12 |
| | Edit / Text-to-Image | 4 | $0.02 |
| **Suno** | Generate / Extend Music | 12 | $0.06 |
| | Generate Lyrics | 0.4 | $0.002 |
| | Multi-Stem Separation | 50 | $0.25 |

---

## 🎯 Purpose

- **Quick Reference** — Copy-paste-ready endpoints, headers, and request bodies
- **Integration Notes** — Gotchas, best practices, and real-world tips
- **Living Documentation** — Updated as APIs evolve or new services are added

---

## 🚀 Usage

Each directory contains markdown files with:
- Authentication details
- Endpoint specifications  
- Request/response examples (cURL)
- Webhook/callback handling
- Error codes and edge cases

---

## 📝 License

This is a personal reference repository. Feel free to use these notes for your own integrations.

---

<p align="center">
  <i>Built with ☕ and curiosity</i>
</p>
