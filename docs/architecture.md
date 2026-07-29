# Architecture Overview

## System Architecture

```
               ┌──────────────────────────────┐
               │        Flutter App           │
               │    BLoC / GoRouter / GetIt   │
               └──────────┬───────────────────┘
                          │
              ┌───────────┴───────────────┐
              │ HTTPS        WebSocket    │
              ▼               ▼           ▼
       ┌──────────┐   ┌────────────┐   ┌────────┐
       │  Nginx   │   │  Chat WS   │   │Firebase│
       │ Gateway  │   │ Socket.IO  │   │  Auth  │
       └────┬─────┘   └────────────┘   │  FCM   │
            │                          └────────┘
            ▼
    ┌──────────────────────────────────────┐
    │      Microservices (NestJS)          │
    ├──────────┬──────────┬───────┬────────┤
    │   auth   │   user   │  post │  feed  │
    ├──────────┼──────────┼───────┼────────┤
    │Firebase  │ Profiles │ Posts │Person- │
    │JWT       │Connexions│Comts  │alized  │
    │OTP       │Chat (WS) │Likes  │Feed    │
    │Password  │FCM tokens│Jobs   │Kafka   │
    │Reset     │Blocking  │Hashtag│Consumer│
    ├──────────┼──────────┼───────┼────────┤
    │notif.    │ mailer   │ media │search  │
    ├──────────┼──────────┼───────┼────────┤
    │Push FCM  │ Email    │Image  │Type-   │
    │In-app    │(gRPC)    │Upload │sense   │
    │Email     │          │Thumbor│Fulltext│
    │Kafka     │          │NSFW   │Geo-    │
    │Consumer  │          │Check  │search  │
    ├──────────┼──────────┼───────┼────────┤
    │marketpl. │countrycty│dynlink│        │
    ├──────────┼──────────┼───────┼────────┤
    │Business  │Countries │Deep   │        │
    │Listings  │Cities    │Links  │        │
    │PostGIS   │PostGIS   │PlaySt.│        │
    │Gallery   │Nearest   │       │        │
    └────┬─────┴────┬─────┴──┬────┴────────┘
         │          │        │
         ▼          ▼        ▼
   ┌───────────────────────────────┐
   │  PostgreSQL  │  Redis  │ Kafka│
   │  + PostGIS   │  Cache  │ KRaft│
   └───────────────────────────────┘
   ┌───────────────────────────────┐
   │  Typesense   │ Thumbor  │Docker│
   └───────────────────────────────┘
```

---

## Backend

### Architecture

The backend is a monorepo of **11 NestJS microservices** + **1 Python service**, sharing a common library. A shared library contains all database entities, repositories, DTOs, guards, and decorators — ensuring consistency across services.

Key design decisions:

- **Nx monorepo** — shared TypeScript library prevents code duplication across services
- **Infrastructure as code** — all services and their dependencies (PostgreSQL, Redis, Kafka, Typesense) are defined in Docker Compose
- **CI/CD** — GitHub Actions detects which services changed and only rebuilds/deploys those

### Communication

Three inter-service communication patterns:

| Pattern | Protocol | Usage |
|---|---|---|
| **Async** | Kafka | Notifications, search indexing, feed recommendations |
| **Sync** | gRPC | Email service calls |
| **Real-time** | WebSocket (Socket.IO) | Chat messaging with presence detection |

### Database

PostgreSQL + PostGIS for all spatial data:
- User and business locations with accurate geo-distance calculations
- Nearby search for marketplace listings
- Nearest city reverse geocoding

Redis is used for caching and JWT token blacklisting. Full-text search is offloaded to Typesense, a dedicated search engine indexed in real-time via Kafka.

### CI/CD Pipeline

```
Git push → Tests → Build Docker images → Push to GHCR → Deploy to VPS
```

Smart change detection: the pipeline identifies which services were modified and only builds/deploys those, keeping deployments fast.

---

## Frontend

### Architecture

Feature-first Flutter app with a clean layered architecture:

```
features/<feature>/
├── bloc/           # State management
├── screens/        # UI screens
├── widgets/        # Feature-specific widgets
└── service/        # Business logic
```

**Data flow:** `UI → BLoC → Service → Repository → API`

### 16 Feature Modules

| Feature | Responsibility |
|---|---|
| **auth** | Login, register, OTP, forgot password, multi-account switching |
| **onboarding** | First-launch onboarding |
| **initializer** | Startup state machine |
| **home** | Main screen with tab navigation |
| **feed** | Social feed + job offers |
| **post** | Post CRUD, comments, likes, shares |
| **job_offer** | Job offer creation, applications |
| **marketplace** | Business listings |
| **chat** | Real-time messaging |
| **connexion** | Professional network |
| **notification** | Push & in-app notifications |
| **profile** | Profile display and editing |
| **search** | Full-text search |
| **hashtag** | Trending hashtags |
| **block** | User blocking |
| **setting** | Settings, password, account deletion |

### State Management

- **BLoC** for complex flows (feed pagination, auth, chat)
- **Cubits** for simple state (badge counts, toggles)
- **Dependency injection** via GetIt (14 repositories, 22 services)
- **GoRouter** for routing (50+ named routes) with auth-based redirects

### Networking

- **Dio** HTTP client with JWT auth interceptor and error handling
- **Socket.IO** WebSocket client for real-time chat
- **Firebase SDK** for auth, push notifications, crash reporting, app attestation

---

## Security

- **Firebase Auth** for identity (email/password, Google Sign-In)
- **JWT** for internal API authentication
- **Firebase App Check** (Play Integrity) to prevent API abuse
- **Token blacklisting** via Redis on logout
- **NSFW image detection** before storage
- **Input validation** on all endpoints

---

## Documentation

Each sub-project has comprehensive documentation:

### Backend
| Document | Content |
|---|---|
| `README.md` | Tech stack, commands, deployment |
| `docs/architecture.md` | Full architecture overview |
| `docs/features/*.md` | Per-feature docs |
| `docs/roadmaps/*.md` | Development phase roadmaps |

### Frontend
| Document | Content |
|---|---|
| `README.md` | Project overview, installation |
| `docs/architecture.md` | Flutter architecture |
| `docs/features/*.md` | Feature documentation |
| `docs/versions/*.md` | Release notes |
