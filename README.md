<p align="center">
  <img src="images/screenshots/onboarding.jpg" alt="Opportys" width="200">
</p>

<h1 align="center">Opportys</h1>

<p align="center">
  <b>Professional Social Network for African Talent</b>
  <br>
  Connect. Share. Seize Opportunities.
</p>

<p align="center">
  <a href="https://play.google.com/store/apps/details?id=com.opportys">
    <img src="https://img.shields.io/badge/Google_Play-414141?logo=google-play&logoColor=white" alt="Google Play">
  </a>
  <img src="https://img.shields.io/badge/Flutter-02569B?logo=flutter" alt="Flutter">
  <img src="https://img.shields.io/badge/NestJS-E0234E?logo=nestjs" alt="NestJS">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Apache_Kafka-231F20?logo=apache-kafka" alt="Kafka">
  <img src="https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Redis-DC382D?logo=redis&logoColor=white" alt="Redis">
  <img src="https://img.shields.io/badge/Firebase-FFCA28?logo=firebase&logoColor=black" alt="Firebase">
  <img src="https://img.shields.io/badge/License-MIT-green" alt="MIT License">
</p>

---

## Overview

**Opportys** is a mobile-first professional social network connecting professionals, freelancers, students, and craftspeople across Africa. Users can publish opportunities, discover posts, build their professional network, and exchange in real-time — all within a single platform.

> **Status:** Live on Google Play | Actively maintained

---

## Screenshots

| Onboarding | Home Feed | Opportunities | Comments |
|---|---|---|---|
| ![onboarding](images/screenshots/onboarding.jpg) | ![home](images/screenshots/home.jpg) | ![plan](images/screenshots/plan-opportunite.jpg) | ![comments](images/screenshots/appercu-commentaire.jpg) |

| Marketplace | Profile | Connections | Create Post |
|---|---|---|---|
| ![marketplace](images/screenshots/commerce-services.jpg) | ![profile](images/screenshots/mon-profil.jpg) | ![connexions](images/screenshots/com.opportys-connexions.jpg) | ![publish](images/screenshots/publier-post.jpg) |

---

## Key Features

### Social Feed
- Personalized feed with post and job offer discovery
- Post creation with text, images, and video
- Like, comment (threaded), share
- Hashtag support with trending detection
- @mentions with autocomplete

### Professional Networking
- Bilateral connection system (request/accept)
- Profile with professional details, activity fields, location
- Profile view tracking and statistics
- Connection suggestions

### Real-Time Messaging
- WebSocket chat via Socket.IO
- Message read receipts and typing indicators
- Image attachment support
- Online/offline presence detection

### Job Offers & Opportunities
- Multi-step job offer creation wizard
- CV/resume upload and application management
- Activity field categorization
- Location-based filtering

### Marketplace
- Business listings (products and/or services)
- Geo-tagged businesses with PostGIS
- Gallery, logo, cover image, opening hours
- Nearby search with radius filtering

### Search
- Full-text search powered by Typesense
- Faceted filtering (type, activity field, hashtag, offer type)
- Geo-search with proximity sorting
- Search history with autocomplete

### Notifications
- Push notifications via Firebase Cloud Messaging
- In-app notification feed
- Email notifications
- Configurable per-type notification settings

### Security
- Firebase Authentication + JWT
- Firebase App Check (Play Integrity)
- NSFW image detection
- Token blacklisting via Redis

---

## Tech Stack

### Backend — 12 Microservices

| Service | Technology | Responsibility |
|---|---|---|
| **auth** | NestJS | Firebase token verification, JWT issuance, OTP, password reset |
| **user** | NestJS + Socket.IO | Profiles, social graph, real-time chat |
| **post** | NestJS | Posts, comments, likes, job offers, hashtags |
| **feed** | NestJS | Personalized feed & recommendations |
| **notification** | NestJS | Push, email, in-app notifications |
| **mailer** | NestJS (gRPC) | Transactional emails |
| **media** | NestJS | Image/PDF upload, validation, Thumbor resize |
| **search** | NestJS | Typesense full-text + geo-search |
| **marketplace** | NestJS | Business listings with geo-location |
| **countrycity** | NestJS | Geographic reference data (cities/countries) |
| **dynamiclink** | NestJS | Deep link generation for Play Store |
| **safe-image-checker** | Python FastAPI | NSFW detection |

### Frontend

| Layer | Technology |
|---|---|
| **Framework** | Flutter (Android, iOS, Web, Desktop) |
| **State Management** | BLoC + Cubit |
| **Routing** | GoRouter |
| **DI** | GetIt |
| **Networking** | Dio + Socket.IO |
| **Maps** | OpenStreetMap (flutter_map) |

### Infrastructure & DevOps

| Component | Details |
|---|---|
| **Database** | PostgreSQL + PostGIS |
| **Cache** | Redis |
| **Async Messaging** | Apache Kafka |
| **Search Engine** | Typesense |
| **Container Runtime** | Docker + Docker Compose |
| **CI/CD** | GitHub Actions → GHCR → VPS |
| **Image Server** | Thumbor (on-demand resize) |
| **Auth Provider** | Firebase Auth |

### Architecture Diagram

```
Flutter App
    │
    ▼ HTTPS / WebSocket
  Nginx Gateway
    │
    ├──► auth
    ├──► user
    ├──► post
    ├──► feed
    ├──► notification ──► gRPC ──► mailer
    ├──► media
    ├──► search
    ├──► marketplace
    ├──► countrycity
    └──► dynamiclink
           │
    ┌──────┴────────┐
    │ PostgreSQL    │
    │ Redis         │
    │ Kafka         │
    └───────────────┘
```

---

## Communication Patterns

| Pattern | Protocol | Usage |
|---|---|---|
| **REST** | HTTP/HTTPS | CRUD operations between app and services |
| **Async** | Kafka | Notifications, search indexing, feed recommendations |
| **Sync** | gRPC | Email service calls |
| **Real-time** | WebSocket (Socket.IO) | Chat messaging with presence detection |

---

## Project Structure

```
backend/                         # Backend monorepo (Nx + NestJS)
├── apps/
│   ├── auth/         service
│   ├── user/         service
│   ├── post/         service
│   ├── feed/         service
│   ├── notification/ service
│   ├── mailer/       service (gRPC)
│   ├── media/        service
│   ├── search/       service
│   ├── marketplace/  service
│   ├── countrycity/  service
│   └── dynamiclink/  service
├── libs/
│   └── shared/       # Shared entities, repositories, DTOs, guards
├── docs/             # Architecture, deployment, features
├── docker-compose.yaml
└── .github/workflows/

frontend/                        # Flutter app
├── lib/
│   ├── features/      # 16 feature modules
│   ├── blocs/         # Shared BLoCs
│   ├── repositories/  # API repositories
│   ├── services/      # Business services
│   ├── widgets/       # Reusable widgets
│   └── di/            # Dependency injection
├── docs/
└── pubspec.yaml

safe-image-checker/              # NSFW detection service
├── app.py             # FastAPI service
├── Dockerfile
└── docker-compose.yml
```

---

## Key Metrics

- **11 NestJS microservices** + 1 Python service
- **43 database entities**
- **16 Flutter feature modules**
- **50+ named routes**
- **14 repositories, 22 services**
- **CI/CD** with smart change detection

---

## Learn More

| Resource | Link |
|---|---|
| **Google Play** | [com.opportys](https://play.google.com/store/apps/details?id=com.opportys) |
| **Architecture** | [docs/architecture.md](docs/architecture.md) |
| **Technical Highlights** | [docs/tour-de-force.md](docs/tour-de-force.md) |

---

## License

[MIT](LICENSE) © 2026 Richard LABITE
