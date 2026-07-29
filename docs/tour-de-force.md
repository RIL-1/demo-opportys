# Points forts techniques

Décisions d'architecture, patterns et défis techniques démontrant un savoir-faire de niveau production.

---

## 1. Architecture microservices en production

Un monorepo de **11 microservices NestJS** + **1 service Python** orchestré via Docker Compose sur un VPS. Chaque service est déployable et scalable indépendamment.

Choix clés :
- **Monorepo Nx** — une bibliothèque TypeScript partagée évite la duplication de code entre services
- **CI/CD intelligent** — GitHub Actions détecte les services modifiés et ne rebuild/déploie que ceux-ci
- **Images Docker** poussées vers GitHub Container Registry, déployées sur VPS via SSH

---

## 2. Trois modes de communication

Le système utilise trois modes de communication inter-services, chacun choisi pour le bon usage :

| Mode | Cas d'usage | Pourquoi |
|---|---|---|
| **Kafka (async)** | Notifications, indexation recherche, recommandations | Découplé, tolérant aux pannes, replayable |
| **gRPC (sync)** | Appels au service d'emails | Type-safe, rapide |
| **Socket.IO (temps réel)** | Messagerie instantanée | Connexion persistante, faible latence |

Les événements circulent entre services sans couplage fort. Si un consommateur est indisponible, Kafka conserve les messages jusqu'à son rétablissement.

---

## 3. Chat temps réel avec Socket.IO

Un système de messagerie de niveau production :

- Connexions WebSocket authentifiées par JWT
- Accusés de lecture et indicateurs de saisie
- Détection de présence en ligne/hors ligne
- Envoi d'images dans les messages
- Fallback REST pour l'historique
- Reconnexion automatique avec backoff exponentiel

---

## 4. Recherche full-text et géographique avec Typesense

Plutôt que d'utiliser la recherche full-text de la base de données, le système utilise un moteur de recherche dédié :

- **Indexation temps réel** via Kafka — à la création/modification d'un contenu, un message Kafka déclenche la réindexation
- **Recherche par facettes :** filtre par type, domaine d'activité, hashtag, type d'offre
- **Recherche géographique :** filtre par localisation avec rayon et tri par proximité
- **Autocomplétion** pour les suggestions de recherche
- **Historique de recherche** par utilisateur

---

## 5. Requêtes spatiales avec PostGIS

PostgreSQL + PostGIS pour toutes les fonctionnalités de localisation :
- Localisation des utilisateurs et commerces avec calculs de distance précis
- Recherche de commerces à proximité
- Recherche de la ville la plus proche par coordonnées GPS
- Indexation spatiale haute performance

---

## 6. Pipeline d'images avec détection NSFW

Un pipeline de traitement d'images en plusieurs étapes :

```
Upload → Validation fichier → Détection NSFW → Stockage → Redimensionnement Thumbor
```

- **Safe Image Checker** — un microservice Python autonome utilisant NudeNet pour classer les images avant stockage
- **Thumbor** — redimensionnement d'images à la demande avec URLs signées
- **Cache manager Flutter personnalisé** pour le chargement optimisé des images

---

## 7. Notifications push avec Firebase Cloud Messaging

Un système de notification complet :
- Notifications push délivrées même lorsque l'application est fermée
- Notifications in-app servies via API REST
- Notifications email via service transactionnel
- Préférences de notification par utilisateur
- Digest quotidiens automatisés (activité des publications, rappels de complétion de profil, réengagement)

---

## 8. Architecture Flutter à l'échelle

L'application Flutter suit des patterns de niveau production :
- **Architecture feature-first** — 16 modules fonctionnels avec séparation claire des responsabilités
- **22 services, 14 repositories** enregistrés via injection de dépendances
- **50+ routes** avec redirections selon l'authentification et liens profonds
- **Pattern BLoC** avec transformers d'événements gérant les race conditions
- **Intercepteurs Dio** pour l'authentification JWT et la gestion d'erreurs
- **Multi-plateforme** — Android, iOS, Web, Linux, macOS, Windows

---

## 9. Pipeline CI/CD

Un pipeline de livraison entièrement automatisé :

```
Push → Tests → Build images Docker → Registry → Déploiement VPS
```

- Détection intelligente des changements (seuls les services modifiés sont rebuild)
- Nettoyage mensuel du registry de conteneurs
- Environnement de test complet avec Docker Compose

---

## 10. Sécurité

Plusieurs couches de défense :
- **Firebase App Check** (Play Integrity) — seules les requêtes provenant de l'application authentique atteignent l'API
- **Authentification JWT** avec mise sur liste noire des tokens à la déconnexion
- **Validation des entrées** sur chaque endpoint
- **Détection NSFW** avant stockage des images
- **HTTPS** en production
- **Suppression de compte** disponible directement depuis l'application

---

## Résumé

Opportys n'est pas un projet d'étude ou un tutoriel. C'est un **réseau social en production** avec :

- Un backend microservices gérant la messagerie temps réel, la recherche, les notifications et les données géospatiales
- Une application mobile multiplateforme avec 50+ écrans et une gestion d'état complexe
- Un CI/CD automatisé du commit à la production
- Les bonnes pratiques de sécurité à chaque couche
- Une base d'utilisateurs réelle sur Google Play

Ce projet démontre une maîtrise de la stack complète : **Flutter, NestJS, PostgreSQL, Kafka, Redis, Typesense, Docker, CI/CD et déploiement cloud.**
