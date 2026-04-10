# TEX8 Platform — Public Overview

**TEX8 LLP** is a UK-registered software company building a subscription-based SaaS platform for websites, e-commerce, AI-powered services, and a next-generation AI marketplace.

This repository contains a public technical and product overview — no source code is published here.

> **Author:** Roland Kohlhuber  
> **Company:** TEX8 LLP · OC458784  
> **Website:** [tex8.com](https://tex8.com) · [solutions.tex8.com](https://solutions.tex8.com)

---

## Platform Overview

TEX8 is built as a microservices architecture where a suite of Rust-based backend services powers multiple customer-facing products — from websites and online shops to AI assistants and semantic search.

A key design principle is **on-premise AI**: all inference (embeddings, speech, image generation, video generation) runs on self-hosted hardware — no third-party AI API dependency for core features.

```
TEX8 Platform
├── TEX8 Web Solutions         ← Live SaaS product (websites & shops)
├── AI Assistant Service       ← Intelligent chatbot for customer websites
├── Search Service             ← Semantic vector search
├── Auth Service               ← OAuth, JWT, sessions, 2FA
├── Content Service            ← Media & file management
├── Notification Service       ← Email, WhatsApp, Telegram, Instagram
├── Embedding Service          ← On-premise ML embeddings (gRPC)
├── AI Runtime Service         ← GPU job orchestration & worker management
├── AI GPU Worker              ← On-premise inference: STT, TTS, image, video
└── TEX8 AI Marketplace        ← Phase 2: AI-native marketplace + mobile app
```

---

## Products

### TEX8 Web Solutions
**Status: Live** · [solutions.tex8.com](https://solutions.tex8.com)

A fully managed, subscription-based platform that delivers professional websites and online shops to small and medium-sized businesses — with zero upfront cost for the customer.

**What's included:**
- Multi-lingual website with dynamic page routing
- Full e-commerce system (products, categories, cart, checkout, orders)
- Customer portal with OAuth login, order history, and subscription management
- Admin dashboard for content, shop settings, orders, and analytics
- Payment integrations: **Stripe** and **Revolut** (including subscriptions and webhooks)
- AI-powered semantic search for product discovery
- Integrated AI chatbot (optional upgrade)
- Multi-tenant architecture — one platform, many shops
- Custom domain support

**Tech Stack:**
- Node.js + Express (backend & server-side rendering with EJS)
- MongoDB (data persistence)
- Redis / Dragonfly (caching & sessions)
- Docker (containerized deployment)

---

### AI Assistant Service
**Status: Production-Ready**

An intelligent chatbot service that can be embedded into any customer website on the TEX8 platform.

**Capabilities:**
- Natural language Q&A (FAQ, product info, business hours, policies)
- Product search and recommendations
- Long-term conversation memory via **Weaviate** (vector DB + RAG)
- FIFO context buffer (50 messages, MongoDB-backed)
- Conversation summarization
- Per-shop knowledge base management via admin API
- Free tier (10 messages) + subscription-gated full access
- LLM provider: **OpenRouter** (model-agnostic, cloud) — with on-premise inference path planned

**Tech Stack:** Rust (Axum), MongoDB, Weaviate, OpenRouter

---

### Search Service
**Status: Production-Ready**

Semantic product and content search using vector embeddings and Weaviate.

**Features:**
- Multi-lingual semantic search (intent-aware, not just keyword)
- Instant suggestions and autocomplete
- Redis-based rate limiting (per-shop sliding window)
- Graceful shutdown, request timeouts, health checks
- Integration: JWT-authenticated, shop-scoped

**Tech Stack:** Rust (Axum), Weaviate, Redis, Embedding Service (on-premise)

---

### Auth Service
**Status: Production-Ready**

Centralized authentication and authorization for all TEX8 services and customer websites.

**Features:**
- OAuth 2.0 flows (login, register, token refresh)
- JWT issuance and revocation
- Session management
- 2FA support
- Subscription-aware JWT claims (plan, features)
- Shared across all platform services

**Tech Stack:** Rust (Axum), MongoDB, Redis

---

### Notification Service
**Status: Production-Ready**

Multi-channel notification infrastructure supporting:
- **Email** (transactional: orders, password reset, payment links)
- **WhatsApp** (webhook + messaging)
- **Telegram** (bot + webhook)
- **Instagram** (webhook)

**Tech Stack:** Rust (Axum), webhook-based integrations

---

### Content Service
**Status: Production-Ready**

Centralized media and file management for all customer websites.

- Image, video, and PDF uploads
- MinIO-backed object storage
- Per-shop scoping and access control

**Tech Stack:** Rust (Axum), MinIO

---

### Embedding Service
**Status: Production-Ready**

High-performance, **on-premise** ML embedding service. All embeddings are computed locally — no data leaves the infrastructure.

- Model: **Qwen3-Embedding-8B** (self-hosted, quantized)
- Transport: gRPC
- Used for: semantic search indexing, RAG memory, product vectorization
- Warm/evict VRAM semantics — stays active while GPU headroom allows

**Tech Stack:** Python (gRPC), runs on self-hosted GPU server

---

### AI Runtime Service + AI GPU Worker
**Status: In Development**

The AI Runtime Service is TEX8's GPU job orchestration layer. It manages a fleet of AI GPU Workers and routes inference jobs to available hardware — locally, on the TEX8 server, or on rented GPU capacity (e.g. Vast.ai).

**AI GPU Worker capabilities (on-premise inference):**

| Lane | Model | Description |
|------|-------|-------------|
| **STT** | Qwen3-ASR-1.7B | Speech-to-text, on-premise |
| **TTS** | Qwen3-TTS-1.7B | Text-to-speech, on-premise |
| **Embedding** | Qwen3-Embedding-8B | Vector embeddings, warm lane |
| **Image** | Qwen-Image / Qwen-Image-Edit | Image generation & editing, 8-bit |
| **Video** | WAN 2.2 (T2V, I2V, Animate) | Text-to-video, image-to-video, animation, 8-bit |
| **FFmpeg** | ffmpeg-video-edit | Video processing & composition |

**Design:**
- Worker registers with the runtime, advertises capabilities, pulls jobs
- VRAM-aware scheduling: only one heavy lane (image/video) in GPU memory at a time
- Embedding lane can be evicted and automatically re-activated after heavy jobs
- Model prefetch at startup: no cold-start penalty in production
- Deployable locally (Docker), on the TEX8 server, or on cloud GPU rentals

**Tech Stack:** Rust (worker binary + runtime), Python (model scripts), Docker

---

### TEX8 AI Marketplace + Mobile App *(Phase 2)*
**Status: In Development**

A next-generation AI-native marketplace platform with a fully LLM-controlled React Native mobile app for iOS and Android.

**Platform:**
- Multi-vendor product listings
- AI-powered semantic search and personalized recommendations
- Fast local delivery infrastructure
- All TEX8 Rust microservices as cloud backend (auth, search, AI assistant, payments, content)

**React Native App — what makes it different:**

The mobile app is built with LLM-first control in mind. Rather than hardcoded UI flows, the app exposes its navigation, actions, and state to an LLM agent — enabling the AI to drive the user experience, fill forms, trigger searches, and respond to context dynamically.

- **TypeScript** throughout — typed interfaces for every screen and action
- **Local LLM module** — on-device model inference integrated directly into the app (no cloud round-trip for supported tasks)
- **LLM-controlled navigation** — the agent can navigate screens, trigger actions, and respond to user intent in natural language
- **AI chat layer** — chat interface connected to both local inference and TEX8 cloud services
- **SQLite + async storage** — local persistence for offline capability and fast startup
- **Multi-language support** — i18n module built in from day one
- **OAuth integration** — seamless login via TEX8 Auth Service
- **Marketplace screens** — product discovery, AI-powered marketplace chat, seller interaction
- **Modular architecture** — clear separation of core, modules, screens, services, and navigation
- Built and maintained by a single developer end-to-end (design → native iOS/Android → AI integration → backend wiring)

**Tech Stack:** React Native (TypeScript), llama.cpp / on-device inference, TEX8 Rust microservices, SQLite

---

## On-Premise AI Stack

TEX8 runs its own AI infrastructure — no dependency on OpenAI, Google, or any external inference provider for core services.

| Capability | Model | Where |
|-----------|-------|-------|
| Text embeddings | Qwen3-Embedding-8B | Self-hosted GPU server |
| Speech-to-text | Qwen3-ASR-1.7B | Self-hosted GPU worker |
| Text-to-speech | Qwen3-TTS-1.7B | Self-hosted GPU worker |
| Image generation | Qwen-Image (8-bit) | Self-hosted GPU worker |
| Video generation | WAN 2.2 T2V / I2V (8-bit) | Self-hosted GPU worker |
| LLM (cloud fallback) | OpenRouter | Cloud (model-agnostic) |
| Mobile LLM | On-device inference | React Native app |

---

## Infrastructure

| Component | Role |
|-----------|------|
| **MongoDB** | Primary database (all services) |
| **Dragonfly / Redis** | Caching, sessions, rate limiting |
| **Weaviate** | Vector database (semantic search, RAG memory) |
| **MinIO** | Object storage (images, files) |
| **Docker Compose** | Service orchestration |
| **Nginx** | Reverse proxy, SSL termination |
| **Self-hosted GPU server** | On-premise AI inference |

---

## Tech Stack Summary

| Layer | Technology |
|-------|------------|
| **Microservices** | Rust (Axum framework) |
| **Customer Websites** | Node.js, Express, EJS |
| **Mobile App** | React Native (TypeScript) |
| **On-premise AI** | Python, gRPC, Qwen3 family, WAN 2.2 |
| **LLM (cloud)** | OpenRouter (model-agnostic) |
| **Vector DB** | Weaviate |
| **Primary DB** | MongoDB |
| **Cache / Sessions** | Dragonfly / Redis |
| **Object Storage** | MinIO |
| **Payments** | Stripe, Revolut |
| **Deployment** | Docker, Nginx |

---

## Business Model

TEX8 Web Solutions operates on a **subscription model**:

- Customers get a professionally built website or online shop
- Monthly subscription covers hosting, maintenance, updates, and support
- Optional upgrades: AI chatbot, semantic search, e-commerce module, custom domain
- Multi-tenant platform: one deployment serves many customers

---

## Screenshots

*Coming soon — Screenshots and UI previews will be added to the `/screenshots` folder.*

---

## Contact

**Roland Kohlhuber**  
Founder & CTO, TEX8 LLP  
[tex8.com](https://tex8.com)

---

*This repository is a public overview. Source code is private.*
