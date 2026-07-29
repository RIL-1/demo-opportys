# Technical Highlights

The design decisions, architectural patterns, and engineering challenges that demonstrate production-level software craftsmanship.

---

## 1. Microservices Architecture in Production

A monorepo of **11 NestJS microservices** + **1 Python service** orchestrated via Docker Compose on a VPS. Each service is independently deployable, scalable, and owns its data domain.

Key decisions:
- **Nx monorepo** — a shared TypeScript library prevents code duplication across services
- **Smart CI/CD** — GitHub Actions detects which services changed and only rebuilds/deploys those, keeping deployments fast
- **Container images** pushed to GitHub Container Registry, pulled on VPS via SSH

---

## 2. Three Communication Patterns

The system uses three different inter-service communication patterns, each chosen for the right job:

| Pattern | Use Case | Why |
|---|---|---|
| **Kafka (async)** | Notifications, search indexing, feed recommendations | Decoupled, fault-tolerant, replayable |
| **gRPC (sync)** | Mailer service calls | Type-safe, fast |
| **Socket.IO (real-time)** | Chat messaging | Persistent connection, low latency |

Events flow between services without tight coupling. If a consumer is down, Kafka retains messages until it recovers.

---

## 3. Real-Time Chat with Socket.IO

A production-grade chat system with:

- JWT-authenticated WebSocket connections
- Message read receipts and typing indicators
- Online/offline presence detection
- Image attachment support in messages
- REST fallback for message history
- Auto-reconnection with exponential backoff

---

## 4. Full-Text & Geo-Search with Typesense

Instead of using the database's built-in full-text search, the system uses a dedicated search engine:

- **Real-time indexing** via Kafka — when content is created or updated, a Kafka message triggers re-indexing
- **Faceted search:** Filter by type, activity field, hashtag, offer type
- **Geo-search:** Filter by location with radius and sort by proximity
- **Autocomplete** for search suggestions
- **Search history** per user

---

## 5. PostGIS Spatial Queries

Leveraging PostgreSQL + PostGIS for all location-based features:
- User and business locations with accurate geo-distance calculations
- Nearby marketplace search
- Nearest city reverse geocoding
- High-performance spatial indexing

---

## 6. Image Pipeline with NSFW Detection

A multi-step image processing pipeline:

```
Upload → File validation → NSFW detection → Storage → Thumbor resize
```

- **Safe Image Checker** — a standalone Python microservice using NudeNet to classify images before they reach the database
- **Thumbor** — on-demand image resizing with signed URLs (prevents hotlinking)
- **Custom Flutter cache manager** for optimized image loading

---

## 7. Push Notifications with Firebase Cloud Messaging

A comprehensive notification system:
- Push notifications delivered even when the app is closed
- In-app notifications served via REST API
- Email notifications sent via transactional email service
- Per-user notification preferences
- Cron-driven digests (daily post activity, profile completion reminders, re-engagement)

---

## 8. Flutter Architecture at Scale

The Flutter app follows production-grade patterns:
- **Feature-first architecture** — 16 feature modules with clear separation of concerns
- **22 services, 14 repositories** registered via dependency injection
- **50+ routes** with auth-based redirects and deep link support
- **BLoC pattern** with event transformers handling race conditions
- **Dio interceptors** for JWT auth and error handling
- **Multi-platform** — Android, iOS, Web, Linux, macOS, Windows

---

## 9. CI/CD Pipeline

A fully automated delivery pipeline:

```
Push to main → Tests → Build Docker images → Push to registry → Deploy to VPS
```

- Smart change detection (only rebuilds changed services)
- Monthly container registry cleanup
- Full Docker Compose test environment

---

## 10. Security

Multiple layers of defense:
- **Firebase App Check** (Play Integrity) — only requests from the genuine app reach the API
- **JWT authentication** with token blacklisting on logout
- **Input validation** on every endpoint
- **NSFW detection** before image storage
- **HTTPS** in production
- **Account deletion** available directly from the app

---

## Summary

Opportys is a **full-production social network** with:

- A microservices backend handling real-time messaging, search, notifications, and geo-spatial data
- A cross-platform mobile app with 50+ screens and complex state management
- Automated CI/CD from commit to production
- Security best practices at every layer
- A real user base on Google Play

This project demonstrates proficiency across the full stack: **Flutter, NestJS, PostgreSQL, Kafka, Redis, Typesense, Docker, CI/CD, and cloud deployment.**
