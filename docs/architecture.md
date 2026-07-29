# Architecture

## Architecture système

```
               ┌──────────────────────────────┐
               │     Application Flutter      │
               │   BLoC / GoRouter / GetIt    │
               └──────────┬───────────────────┘
                          │
              ┌───────────┴───────────────┐
              │ HTTPS        WebSocket    │
              ▼               ▼           ▼
       ┌──────────┐   ┌────────────┐   ┌────────┐
       │  Nginx   │   │  Chat WS   │   │Firebase│
       │Passerelle│   │ Socket.IO  │   │  Auth  │
       └────┬─────┘   └────────────┘   │  FCM   │
            │                          └────────┘
            ▼
    ┌──────────────────────────────────────┐
    │       Microservices (NestJS)         │
    ├──────────┬──────────┬───────┬────────┤
    │   auth   │   user   │  post │  feed  │
    ├──────────┼──────────┼───────┼────────┤
    │Firebase  │ Profils  │Posts  │Perso-  │
    │JWT       │Connexions│Comts  │nalisé  │
    │OTP       │Chat (WS) │Likes  │Fil     │
    │Mot de    │Tokens FCM│Emplois│Kafka   │
    │passe     │Blocage   │Hashtag│Consumer│
    ├──────────┼──────────┼───────┼────────┤
    │notif.    │ mailer   │ media │search  │
    ├──────────┼──────────┼───────┼────────┤
    │Push FCM  │ Email    │Upload │Type-   │
    │In-app    │(gRPC)    │Images │sense   │
    │Email     │          │Thumbor│Fulltext│
    │Kafka     │          │NSFW   │Géo-    │
    │Consumer  │          │Vérif. │recherch│
    ├──────────┼──────────┼───────┼────────┤
    │marketpl. │countrycty│dynlink│        │
    ├──────────┼──────────┼───────┼────────┤
    │Fiches    │Pays      │Liens  │        │
    │commerces │Villes    │profs  │        │
    │PostGIS   │PostGIS   │PlaySt.│        │
    │Galerie   │Plus      │       │        │
    └────┬─────┴────┬─────┴──┬────┴────────┘
         │          │        │
         ▼          ▼        ▼
   ┌───────────────────────────────┐
   │  PostgreSQL  │  Redis  │ Kafka│
   │  + PostGIS   │  Cache  │ KRaft│
   └───────────────────────────────┘
   ┌───────────────────────────────┐
   │ Typesense  │ Thumbor  │Docker │
   └───────────────────────────────┘
```

---

## Backend

### Architecture

Le backend est un monorepo de **11 microservices NestJS** + **1 service Python**, partageant une bibliothèque commune. Celle-ci contient toutes les entités de base de données, repositories, DTOs, guards — garantissant la cohérence entre les services.

Choix d'architecture clés :

- **Monorepo Nx** — une bibliothèque TypeScript partagée évite la duplication de code entre services
- **Infrastructure as code** — tous les services et leurs dépendances (PostgreSQL, Redis, Kafka, Typesense) sont définis dans Docker Compose
- **CI/CD** — GitHub Actions détecte les services modifiés et ne rebuild/déploie que ceux-ci

### Communication

Trois modes de communication inter-services :

| Mode | Protocole | Usage |
|---|---|---|
| **Asynchrone** | Kafka | Notifications, indexation recherche, recommandations |
| **Synchrone** | gRPC | Appels au service d'emails |
| **Temps réel** | WebSocket (Socket.IO) | Messagerie avec détection de présence |

### Base de données

PostgreSQL + PostGIS pour toutes les données spatiales :
- Localisation des utilisateurs et commerces avec calculs de distance précis
- Recherche des commerces à proximité
- Recherche de la ville la plus proche par coordonnées GPS

Redis est utilisé pour le cache et la mise sur liste noire des tokens JWT. La recherche full-text est déléguée à Typesense, un moteur de recherche dédié indexé en temps réel via Kafka.

### Pipeline CI/CD

```
Git push → Tests → Build images Docker → Push vers GHCR → Déploiement VPS
```

Détection intelligente des changements : seuls les services modifiés sont reconstruits et déployés.

---

## Frontend

### Architecture

Application Flutter organisée par fonctionnalités avec une architecture en couches :

```
features/<module>/
├── bloc/           # Gestion d'état
├── screens/        # Écrans
├── widgets/        # Widgets spécifiques
└── service/        # Logique métier
```

**Flux de données :** `UI → BLoC → Service → Repository → API`

### 16 modules fonctionnels

| Module | Responsabilité |
|---|---|
| **auth** | Connexion, inscription, OTP, mot de passe oublié, multi-comptes |
| **onboarding** | Parcours de bienvenue au premier lancement |
| **initializer** | Machine d'état au démarrage |
| **home** | Écran principal avec navigation par onglets |
| **feed** | Fil d'actualité social + offres d'emploi |
| **post** | CRUD publications, commentaires, likes, partages |
| **job_offer** | Création d'offres d'emploi, gestion des candidatures |
| **marketplace** | Fiches commerces |
| **chat** | Messagerie temps réel |
| **connexion** | Réseau professionnel |
| **notification** | Notifications push et in-app |
| **profile** | Affichage et édition du profil |
| **search** | Recherche full-text |
| **hashtag** | Hashtags tendances |
| **block** | Gestion des blocages |
| **setting** | Paramètres, mot de passe, suppression de compte |

### Gestion d'état

- **BLoC** pour les flux complexes (pagination du fil, authentification, chat)
- **Cubits** pour les états simples (badges, toggles)
- **Injection de dépendances** via GetIt (14 repositories, 22 services)
- **GoRouter** pour le routing (50+ routes nommées) avec redirections selon l'état d'authentification

### Réseau

- **Dio** HTTP client avec intercepteur JWT et gestion d'erreurs
- **Socket.IO** client WebSocket pour le chat temps réel
- **Firebase SDK** pour l'authentification, les notifications push, le crash reporting, l'attestation d'appareil

---

## Sécurité

- **Firebase Auth** pour l'identité (email/mot de passe, Google Sign-In)
- **JWT** pour l'authentification API interne
- **Firebase App Check** (Play Integrity) pour empêcher les abus API
- **Liste noire de tokens** via Redis à la déconnexion
- **Détection d'images NSFW** avant stockage
- **Validation des entrées** sur tous les endpoints

---

## Documentation

Chaque sous-projet dispose d'une documentation complète :

### Backend
| Document | Contenu |
|---|---|
| `README.md` | Stack technique, commandes, déploiement |
| `docs/architecture.md` | Vue d'ensemble architecture |
| `docs/features/*.md` | Documentation par fonctionnalité |
| `docs/roadmaps/*.md` | Roadmaps de développement |

### Frontend
| Document | Contenu |
|---|---|
| `README.md` | Présentation, installation |
| `docs/architecture.md` | Architecture Flutter |
| `docs/features/*.md` | Documentation des fonctionnalités |
| `docs/versions/*.md` | Notes de version |
