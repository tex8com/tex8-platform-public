# TEX8 Platform — Public Overview

**TEX8 LLP** is a UK-registered software company building a production-grade SaaS platform for websites, e-commerce, AI-powered services, and a next-generation AI marketplace.

This repository contains a public technical and product overview — no source code is published here.

> **Author:** Roland Kohlhuber  
> **Company:** TEX8 LLP · OC458784  
> **Website:** [tex8.com](https://tex8.com) · [solutions.tex8.com](https://solutions.tex8.com)

---

## Platform Overview

TEX8 is a microservices platform where all core services are written in **Rust (Axum)** for performance and reliability, backed by a **Node.js** product layer and a **React Native** mobile app. A key design principle is **on-premise AI**: all inference (embeddings, speech, image, video) runs on self-hosted hardware — no third-party AI API dependency for core features.

```
TEX8 Platform
├── TEX8 Web Solutions         ← Live SaaS product (websites & shops)
├── Auth Service               ← OAuth, RS256 JWT, TOTP 2FA, Google & Apple Sign-In
├── AI Assistant Service       ← Streaming chatbot, RAG, LLM-driven frontend actions
├── Search Service             ← Hybrid semantic search, multi-node embedding pool
├── Content Service            ← Media & file management (MinIO)
├── Notification Service       ← Email, WhatsApp, Telegram, Instagram, Messenger
├── Embedding Service          ← On-premise Qwen3-Embedding-8B (gRPC)
├── AI Runtime Service         ← GPU job orchestration & worker management
├── AI GPU Worker              ← On-premise STT, TTS, image generation, video generation
└── TEX8 AI Marketplace        ← Phase 2: AI-native marketplace + LLM-controlled mobile app
```

---

## Products

### TEX8 Web Solutions
**Status: Live** · [solutions.tex8.com](https://solutions.tex8.com)

A fully managed, subscription-based platform that delivers professional websites and online shops to small and medium-sized businesses — zero upfront cost for the customer.

**Core features:**
- Multi-lingual website with dynamic page routing (German + English, i18n built in)
- Full e-commerce system (products, variants, categories, cart, checkout, orders)
- Customer self-service portal (OAuth login, order history, subscription management)
- Admin dashboard (content, shop settings, orders, analytics, subscription controls)
- AI-powered semantic search for product discovery
- Integrated AI chatbot (optional upgrade)
- Multi-tenant architecture — one deployment serves many independent shops
- Custom domain support per tenant

**Multi-Tenancy Architecture:**

Tenant resolution follows a **5-priority detection chain** per request: custom domain lookup → subdomain parsing → `X-Shop-Id` header → query parameter → environment singleton. Custom domain lookups use a 5-minute in-memory cache to minimize database hits.

Critically, tenant context is propagated using **Node.js AsyncLocalStorage** — not globals or middleware-chained parameters. This eliminates race conditions in concurrent async request handling. Every MongoDB model applies tenant filtering automatically via a Mongoose plugin with pre-find hooks and compound indexes, so no query can accidentally leak data across tenants.

**Session Architecture:**

Sessions use a **dual-layer persistence model**:
- Primary: **Dragonfly (Redis)** — fast synchronous read/write
- Backup: **MongoDB** — async writes in the background, fire-and-forget
- On Redis miss or failure: transparent fallback to MongoDB
- Result: zero data loss even if Redis restarts, with no performance penalty

**Security Middleware Stack:**
- CSRF protection via **double-submit cookie pattern** (64-byte tokens, `__Host-` prefix in production, `HttpOnly`, `Lax SameSite`) — automatically bypassed for Bearer token auth (API clients have no CSRF surface)
- Helmet.js: CSP, HSTS, X-Frame-Options, X-Content-Type-Options
- Structured **API logging** (method, endpoint, response time, user ID, IP — bulk-inserted to MongoDB)
- Separate **audit log** collection (create/update/delete actions with change deltas, IP, user-agent)
- **Error log** collection with stack traces and request context for debugging

**Payment & Subscription Infrastructure:**

TEX8 Web Solutions includes a production-grade payment and subscription system built on **Stripe** and **Revolut** — not just one-time payments, but full recurring billing with lifecycle management.

- **One-time payments** via Stripe and Revolut (card, saved payment methods)
- **Recurring subscriptions** on both providers — monthly billing, plan upgrades/downgrades
- **Revolut MIT** (Merchant Initiated Transactions) — platform charges saved cards server-side without requiring customer re-authentication each billing cycle
- **Stripe subscription lifecycle** — handles 18+ webhook events: `invoice.payment_succeeded/failed`, `customer.subscription.updated/deleted`, fraud warnings (`radar.early_fraud_warning`), disputes (`charge.dispute.created/closed`), reviews, and more
- **Webhook-first confirmation** — orders stay `pending_payment` after the API response; the frontend polls a status endpoint while the webhook triggers confirmation in the background. Robust against network interruptions and page reloads
- **Subscription plan hierarchy** — the platform tracks the customer's highest active plan across all subscriptions and syncs it into their JWT in real time, so feature-gating is always accurate
- **Contract term enforcement** — minimum subscription months are enforced server-side; cancellation or pause during a lock-in period is rejected with remaining months calculated dynamically
- **Quota tracking** — per-feature monthly limits (AI searches, tokens, API calls, storage) checked and incremented atomically in Redis before each request

**Tech Stack:** Node.js, Express, EJS, MongoDB, Dragonfly/Redis, Docker

---

### Auth Service
**Status: Production-Ready**

Centralized authentication for all TEX8 services and customer websites. Built in Rust for performance and correctness.

**Authentication methods:**
- **Email + password** with secure bcrypt hashing
- **Google Sign-In** (OAuth2 with server-side token exchange)
- **Apple Sign-In** (JWT-based, private key cryptography)
- Email verification and password reset flows

**Two-Factor Authentication (TOTP):**
- RFC 4648 Base32 secret generation
- SHA1-based TOTP (6-digit codes, 30-second windows)
- QR code generation in both SVG and base64 PNG (for flexible integration)
- 10 backup codes per user (SHA-256 hashed, unambiguous character set)

**JWT Architecture:**
- **RS256 asymmetric signing** — private key signs, public key shared via JWKS endpoint
- All downstream Rust services validate tokens independently without calling the auth service
- Subscription plan and feature flags embedded in JWT claims — no round-trip needed for access control
- Refresh token rotation with revocation support

**Rate limiting by attack surface:**
| Surface | Limit |
|---------|-------|
| Login attempts | 5/min per email, 10/min per IP |
| 2FA validation | 5 attempts per token per 10 min |
| OAuth init | 5 attempts per IP |
| Password reset | 3 per email per hour |
| Registration | 3 accounts per IP per hour |

**Tech Stack:** Rust (Axum), MongoDB, Redis

---

### AI Assistant Service
**Status: Production-Ready**

An intelligent chatbot service embeddable into any customer website on the TEX8 platform — with long-term memory, streaming responses, and LLM-driven frontend control.

**Capabilities:**
- Natural language Q&A (FAQ, products, business hours, policies)
- **Server-Sent Events (SSE)** streaming — token-by-token output with quota enforcement *during* the stream, not just before it
- **RAG pipeline**: shop-specific knowledge base → Weaviate vector search → context injection
- **FIFO context buffer** (configurable, default 50 messages, MongoDB-backed) with automatic archiving and LLM-generated summarization of archived history
- Long-term conversation memory via Weaviate
- Per-shop knowledge base management via admin API with automatic vector sync
- Free tier (10 messages) + subscription-gated full access
- Multipart upload support (images and PDFs in chat)

**LLM-Driven Frontend Actions:**

AI responses can include structured JSON **action commands** alongside text, enabling the LLM to directly drive frontend behavior:

| Category | Actions |
|----------|---------|
| Navigation | `push`, `replace`, `new_tab` (with scroll target) |
| E-Commerce | `add_to_cart`, `remove_from_cart`, `apply_coupon`, `checkout` |
| Forms | `fill_form`, `submit_form` |
| Custom | extensible action framework |

This means the AI doesn't just answer questions — it can guide users through flows, populate forms, and trigger cart actions autonomously.

**Tech Stack:** Rust (Axum), MongoDB, Weaviate, OpenRouter (cloud LLM, model-agnostic), MinIO

---

### Search Service
**Status: Production-Ready**

Hybrid semantic and full-text search with a distributed embedding pool and intelligent caching.

**Search capabilities:**
- **Semantic vector search** via Weaviate (intent-aware, not keyword-matching)
- **Full-text fallback** for edge cases
- Instant suggestions and autocomplete (per-shop keyword tracking via Redis ZSET)
- JWT-authenticated, shop-scoped results

**Embedding pool:**
- Maintains connections to **multiple embedding service nodes**
- Tracks **latency per node** using atomic counters (no locks)
- Routes each request to the **lowest-latency available node**
- Automatic failover to next-best node on failure
- Health checking across all nodes

**Caching:**
- **7-day embedding cache** in Dragonfly (SHA-256 query hashing, bincode serialization for compact storage)
- Cache statistics endpoint for monitoring
- Per-shop result caching with TTL

**Tech Stack:** Rust (Axum), Weaviate, Redis/Dragonfly, Embedding Service (on-premise gRPC)

---

### Embedding Service
**Status: Production-Ready**

On-premise ML embedding service — all vectors computed locally, no data leaves the infrastructure.

- Model: **Qwen3-Embedding-8B** (self-hosted, quantized)
- Also includes: **Qwen3-Reranker** for result re-ranking
- Transport: gRPC
- Used for: search indexing, RAG memory, product vectorization
- VRAM-aware: warm/evict semantics — stays active while GPU headroom allows

**Tech Stack:** Python (gRPC), self-hosted GPU server

---

### Notification Service
**Status: Production-Ready**

Multi-channel notification infrastructure:

| Channel | Type |
|---------|------|
| **Email** | Transactional (orders, password reset, payment links) via SMTP |
| **WhatsApp** | Webhook + messaging (Meta API) |
| **Telegram** | Bot + webhook |
| **Instagram** | Webhook (Meta API) |
| **Facebook Messenger** | Webhook (Meta API) |

- Tera templating engine for email rendering
- Per-channel template storage with variable interpolation
- Database logging of all sent notifications

**Tech Stack:** Rust (Axum), Lettre (SMTP), webhook integrations

---

### Content Service
**Status: Production-Ready**

Centralized media and file management for all customer websites.

- Image, video, and PDF upload and retrieval
- **MinIO** (S3-compatible) object storage
- Per-shop scoping and access control
- Referenced by AI assistant (multipart chat), product images, and admin uploads

**Tech Stack:** Rust (Axum), MinIO

---

### AI Runtime Service + AI GPU Worker
**Status: In Development**

The AI Runtime Service is TEX8's GPU job orchestration layer. It manages a fleet of AI GPU Workers and routes inference jobs to available hardware — locally, on the TEX8 server, or on rented GPU capacity (Vast.ai).

**AI GPU Worker — on-premise inference lanes:**

| Lane | Model | Notes |
|------|-------|-------|
| **STT** | Qwen3-ASR-1.7B | Speech-to-text |
| **TTS** | Qwen3-TTS-1.7B | Text-to-speech |
| **Embedding** | Qwen3-Embedding-8B | Warm lane, auto-evict/restore |
| **Image** | Qwen-Image / Qwen-Image-Edit | 8-bit quantized |
| **Video** | WAN 2.2 (T2V, I2V, Animate) | Text-to-video, image-to-video, animation — 8-bit |
| **FFmpeg** | ffmpeg-video-edit | Video processing & composition |

**Design principles:**
- Worker registers with runtime, advertises capabilities, pulls jobs via internal API
- VRAM-aware scheduler: only one heavy lane (image/video) in GPU memory at a time; embedding lane auto-evicts and re-activates around heavy jobs
- Model prefetch at container startup: no cold-start penalty for production traffic
- Deployable locally (Apple Silicon), on the TEX8 server, or on Vast.ai GPU rentals
- mTLS between runtime and workers

**Tech Stack:** Rust (worker binary + runtime service), Python (model scripts), Docker

---

### TEX8 AI Marketplace + Mobile App *(Phase 2)*
**Status: In Development**

A next-generation AI-native marketplace platform with a fully LLM-controlled React Native mobile app for iOS and Android.

**Platform:**
- Multi-vendor product listings with AI-powered semantic search
- Personalized recommendations
- Fast local delivery infrastructure
- All TEX8 Rust microservices as cloud backend (auth, search, AI assistant, payments, content)

**React Native App — LLM-first architecture:**

The mobile app is built with an LLM-first control model. Rather than hardcoded UI flows, the app exposes its navigation, screen state, and actions to an LLM agent — enabling the AI to drive the user experience, fill forms, trigger searches, and respond to context in natural language.

- **TypeScript** throughout — fully typed interfaces for every screen, action, and service
- **Local LLM module** — on-device model inference integrated directly into the app; supported tasks run without a cloud round-trip
- **LLM-controlled navigation** — the agent navigates screens, triggers actions, and responds to user intent dynamically
- **AI chat layer** — connected to both local inference and TEX8 cloud services, with context sharing between them
- **SQLite + async storage** — local persistence for offline capability and instant startup
- **i18n from day one** — full internationalization module built into the architecture
- **OAuth integration** — seamless login via TEX8 Auth Service (JWT, token refresh)
- **Modular architecture** — clean separation of `core`, `modules`, `screens`, `services`, `navigation`
- Built and maintained end-to-end by a single developer: UI/UX design → native iOS & Android → AI integration → backend wiring

**Tech Stack:** React Native (TypeScript), on-device LLM inference, TEX8 Rust microservices, SQLite

---

## On-Premise AI Stack

TEX8 runs its own AI infrastructure — no dependency on OpenAI, Google, or any external inference provider for core services.

| Capability | Model | Where |
|-----------|-------|-------|
| Text embeddings | Qwen3-Embedding-8B | Self-hosted GPU server |
| Reranking | Qwen3-Reranker | Self-hosted GPU server |
| Speech-to-text | Qwen3-ASR-1.7B | Self-hosted GPU worker |
| Text-to-speech | Qwen3-TTS-1.7B | Self-hosted GPU worker |
| Image generation | Qwen-Image / Qwen-Image-Edit (8-bit) | Self-hosted GPU worker |
| Video generation | WAN 2.2 T2V / I2V / Animate (8-bit) | Self-hosted GPU worker |
| LLM (cloud fallback) | OpenRouter | Cloud, model-agnostic |
| Mobile LLM | On-device inference | React Native app |

---

## Infrastructure

| Component | Role |
|-----------|------|
| **MongoDB** | Primary database (all services) |
| **Dragonfly / Redis** | Caching, sessions, rate limiting, quota tracking |
| **Weaviate** | Vector database (semantic search, RAG long-term memory) |
| **MinIO** | S3-compatible object storage (images, files, media) |
| **Docker Compose** | 3-phase orchestrated startup with health checks |
| **Nginx** | Reverse proxy, SSL termination |
| **Self-hosted GPU server** | On-premise AI inference |

**Startup sequence** — services come up in three ordered phases:
1. Infrastructure (MongoDB, Dragonfly, Weaviate, MinIO) with health checks
2. Rust microservices (auth, search, AI assistant, notifications, content)
3. Main product (web-solutions) after auth service is confirmed healthy

---

## Tech Stack Summary

| Layer | Technology |
|-------|------------|
| **Microservices** | Rust (Axum), async Tokio runtime |
| **Customer Websites** | Node.js, Express, EJS |
| **Mobile App** | React Native (TypeScript) |
| **On-premise AI** | Python, gRPC, Qwen3 family, WAN 2.2 |
| **LLM (cloud)** | OpenRouter (model-agnostic) |
| **Vector DB** | Weaviate |
| **Primary DB** | MongoDB |
| **Cache / Sessions** | Dragonfly / Redis |
| **Object Storage** | MinIO (S3-compatible) |
| **Payments** | Stripe, Revolut (subscriptions + MIT) |
| **Deployment** | Docker, Nginx |

---

## Business Model

TEX8 Web Solutions operates on a **subscription model**:

- Customers receive a professionally built website or online shop
- Monthly subscription covers hosting, maintenance, updates, and support
- Optional upgrades: AI chatbot, semantic search, e-commerce module, custom domain
- Multi-tenant platform: one deployment serves many independent customers

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
