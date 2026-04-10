# TEX8 Platform — Public Overview

**TEX8 LLP** is a UK-registered software company building a subscription-based SaaS platform for websites, e-commerce, AI-powered services, and a next-generation AI marketplace.

This repository contains a public technical and product overview — no source code is published here.

> **Author:** Roland Kohlhuber  
> **Company:** TEX8 LLP · OC458784  
> **Website:** [tex8.com](https://tex8.com) · [solutions.tex8.com](https://solutions.tex8.com)

---

## Platform Overview

TEX8 is built as a microservices architecture where a suite of Rust-based backend services powers multiple customer-facing products — from websites and online shops to AI assistants and semantic search.

```
TEX8 Platform
├── TEX8 Web Solutions         ← Live SaaS product (websites & shops)
├── AI Assistant Service       ← Intelligent chatbot for customer websites
├── Search Service             ← Semantic vector search
├── Auth Service               ← OAuth, JWT, sessions, 2FA
├── Content Service            ← Media & file management
├── Notification Service       ← Email, WhatsApp, Telegram, Instagram
├── Embedding Service          ← ML embeddings (gRPC)
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
- LLM provider: **OpenRouter** (model-agnostic)

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

**Tech Stack:** Rust (Axum), Weaviate, Redis, custom embedding service

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

High-performance ML embedding service used internally by the Search and AI Assistant services.

- Model: **Qwen3-Embedding-8B**
- Transport: gRPC
- Used for: semantic search indexing, RAG memory, product vectorization

**Tech Stack:** Python (gRPC / FastAPI), GPU-ready

---

### TEX8 AI Marketplace *(Phase 2)*
**Status: In Development**

A next-generation AI-native marketplace platform with:
- Multi-vendor product listings
- AI-powered semantic search and personalized recommendations
- Fast local delivery infrastructure
- React Native mobile app (iOS & Android)

This product leverages all existing TEX8 platform services (auth, search, AI assistant, payments, content) as its cloud backend.

**Tech Stack:** React Native (TypeScript), all TEX8 Rust microservices

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

---

## Tech Stack Summary

| Layer | Technology |
|-------|------------|
| **Microservices** | Rust (Axum framework) |
| **Customer Websites** | Node.js, Express, EJS |
| **Mobile App** | React Native (TypeScript) |
| **AI / ML** | Python, gRPC, Qwen3-Embedding-8B, OpenRouter |
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
