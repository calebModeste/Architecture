# Architecture 4-Tier — Application de Réservation (type Airbnb)

## Vue d'ensemble

L'application est structurée selon un modèle **4-Tier (4 couches)**, qui sépare clairement les responsabilités entre la présentation, la logique applicative, les services métier et les données. Cette séparation améliore la **maintenabilité**, la **scalabilité** et la **sécurité** du système.

```
┌─────────────────────────────────────────────────────┐
│               TIER 1 — PRÉSENTATION                 │
│                     Next.js                         │
├─────────────────────────────────────────────────────┤
│             TIER 2 — APPLICATION / API              │
│               Node.js + Express.js                  │
├─────────────────────────────────────────────────────┤
│            TIER 3 — SERVICES MÉTIER                 │
│         Stripe | Nodemailer | Mapbox                │
├─────────────────────────────────────────────────────┤
│               TIER 4 — DONNÉES                      │
│         PostgreSQL (Supabase) | Storage             │
└─────────────────────────────────────────────────────┘
```

---

## TIER 1 — Couche Présentation

> **Rôle** : Interface utilisateur, rendu des pages, interactions client.

**Technologie** : `Next.js`

### Responsabilités

- Affichage des pages (SSR, SSG, CSR selon le besoin)
- Gestion de la navigation et des routes
- Formulaires, validations côté client
- Consommation de l'API REST (Tier 2)
- Gestion de l'état global (session, panier, filtres)

### Modules UI

| Module                | Acteurs concernés  |
| --------------------- | ------------------ |
| Page d'accueil        | CLIENT             |
| Recherche / Filtres   | CLIENT             |
| Page de location      | CLIENT             |
| Tunnel de réservation | CLIENT             |
| Tableau de bord       | ADMIN, SUPER ADMIN |
| Gestion des locations | ADMIN              |
| Gestion des admins    | SUPER ADMIN        |

### Interactions

```
Utilisateur → Next.js (pages/composants) → HTTP Request → Tier 2
Next.js ← HTTP Response ← Tier 2
```

---

## TIER 2 — Couche Application / API

> **Rôle** : Traitement des requêtes, logique de contrôle, routage, authentification, autorisation.

**Technologie** : `Node.js` + `Express.js`

### Responsabilités

- Exposition des endpoints REST
- Authentification (JWT / sessions)
- Contrôle des accès selon les rôles (CLIENT, ADMIN, SUPER_ADMIN)
- Validation et sanitization des données entrantes
- Orchestration des appels vers le Tier 3 (services) et le Tier 4 (données)

### Routes principales

| Groupe de routes | Méthodes               | Rôles autorisés    |
| ---------------- | ---------------------- | ------------------ |
| `/auth`          | POST                   | PUBLIC             |
| `/locations`     | GET, POST, PUT, DELETE | CLIENT, ADMIN      |
| `/reservations`  | GET, POST, PUT         | CLIENT, ADMIN      |
| `/payments`      | POST, GET              | CLIENT, ADMIN      |
| `/reviews`       | GET, POST              | CLIENT             |
| `/checkinout`    | POST, PUT              | ADMIN              |
| `/services`      | GET, POST, PUT         | ADMIN              |
| `/reports`       | GET                    | ADMIN, SUPER ADMIN |
| `/admin`         | GET, POST, PUT, DELETE | SUPER ADMIN        |
| `/config`        | GET, PUT               | SUPER ADMIN        |

### Flux de traitement

```
Requête → Middleware Auth → Middleware Role → Controller → Service (Tier 3) / Repo (Tier 4) → Réponse
```

---

## TIER 3 — Couche Services Métier

> **Rôle** : Intégration des services externes, logique métier complexe, notifications, paiements, géolocalisation.

### Services intégrés

---

### 3.1 — Stripe (Paiement)

**Usage** : Traitement sécurisé des paiements pour les réservations.

| Fonctionnalité | Description                                 |
| -------------- | ------------------------------------------- |
| Payment Intent | Création du paiement lors de la réservation |
| Remboursement  | Annulation avec remboursement automatique   |
| Webhooks       | Confirmation asynchrone du paiement         |
| Compte Connect | Versements aux propriétaires (BANK_ACCOUNT) |

```
Tier 2 (Controller Paiement)
  → Stripe API (créer PaymentIntent)
  ← Stripe Webhook (confirmer / échouer)
  → Mise à jour statut PAYMENT en DB (Tier 4)
```

---

### 3.2 — Nodemailer (Notifications Email)

**Usage** : Envoi d'emails transactionnels à chaque étape clé.

| Événement             | Destinataire   | Contenu                           |
| --------------------- | -------------- | --------------------------------- |
| Inscription           | CLIENT         | Email de bienvenue + vérification |
| Réservation confirmée | CLIENT         | Récapitulatif de réservation      |
| Paiement reçu         | CLIENT + ADMIN | Confirmation de paiement          |
| Annulation            | CLIENT         | Notification d'annulation         |
| Check-in proche       | CLIENT         | Rappel J-1                        |
| Nouveau compte admin  | ADMIN          | Identifiants de connexion         |

```
Tier 2 (Controller) → Nodemailer Service → SMTP Server → Email Client
```

---

### 3.3 — Mapbox (Géolocalisation)

**Usage** : Affichage et recherche des locations sur une carte interactive.

| Fonctionnalité         | Description                                |
| ---------------------- | ------------------------------------------ |
| Carte interactive      | Affichage des locations avec markers       |
| Géocodage              | Conversion adresse ↔ coordonnées (lat/lng) |
| Recherche géographique | Filtrer les locations par zone / rayon     |
| Calcul distance        | Distance entre le client et la location    |

```
Next.js (Tier 1) → Mapbox SDK (client-side)
Tier 2 → Mapbox API (géocodage serveur-side)
```

---

## TIER 4 — Couche Données

> **Rôle** : Persistance, stockage et récupération de toutes les données de l'application.

### 4.1 — Base de données relationnelle

**Technologie** : `PostgreSQL` via `Supabase`

Contient les **15 entités** du modèle de données :

| Groupe                  | Entités                                                |
| ----------------------- | ------------------------------------------------------ |
| Utilisateurs & Accès    | USER, ROLE                                             |
| Locations               | LOCATION, TYPE_LOCATION, AMENITIES, LOCATION_AMENITIES |
| Réservations            | RESERVATION, RESERVATION_SERVICE, SERVICE              |
| Transactions            | PAYMENT, CHECK_IN_OUT                                  |
| Feedback & Rapports     | REVIEW, REPORT                                         |
| Planification & Finance | CALENDAR, BANK_ACCOUNT                                 |

**Fonctionnalités Supabase utilisées**

| Fonctionnalité     | Usage                                          |
| ------------------ | ---------------------------------------------- |
| Auth (optionnel)   | Gestion des sessions utilisateurs              |
| Row Level Security | Sécurité au niveau des lignes (accès par rôle) |
| Realtime           | Mise à jour en temps réel du calendrier        |
| Edge Functions     | Logique serveur légère (webhooks Stripe)       |
| PostgREST          | API auto-générée sur les tables                |

---

### 4.2 — Stockage de fichiers

**Technologie** : `Supabase Storage`

| Bucket      | Contenu                           | Accès  |
| ----------- | --------------------------------- | ------ |
| `avatars`   | Photos de profil des utilisateurs | Privé  |
| `locations` | Photos des propriétés             | Public |
| `documents` | Documents légaux, contrats        | Privé  |
| `reports`   | Exports PDF des rapports admin    | Privé  |

---

## Flux Complet — Exemple : Réservation d'une Location

```
[CLIENT - Next.js]
  1. Recherche de locations (filtres + carte Mapbox)
  2. Sélection d'une location → voir disponibilité (CALENDAR)
  3. Formulaire de réservation (dates, nb guests, services)
        ↓ POST /reservations
[API - Express.js]
  4. Validation des données entrantes
  5. Vérification disponibilité (CALENDAR en DB)
  6. Création de la RESERVATION (statut: pending)
  7. Appel Stripe → création PaymentIntent
        ↓ Stripe API
[SERVICE - Stripe]
  8. Retourne client_secret au frontend
        ↓ client_secret
[CLIENT - Next.js]
  9. Formulaire paiement Stripe (carte bancaire)
  10. Confirmation côté Stripe
        ↓ Webhook Stripe → POST /webhooks/stripe
[API - Express.js]
  11. Réception webhook → vérification signature
  12. Mise à jour RESERVATION (statut: confirmed)
  13. Création PAYMENT (statut: completed)
  14. Envoi email confirmation via Nodemailer
        ↓ Email SMTP
[CLIENT]
  15. Email de confirmation reçu ✓
```

---

## Sécurité par Couche

| Tier   | Mécanisme de sécurité                                             |
| ------ | ----------------------------------------------------------------- |
| Tier 1 | HTTPS, validation client, protection CSRF (Next.js)               |
| Tier 2 | JWT, middleware de rôles, rate limiting, sanitization inputs      |
| Tier 3 | Clés API secrètes (env variables), vérification signatures Stripe |
| Tier 4 | Row Level Security (RLS), connexion chiffrée, backup auto         |

---

## Résumé des Technologies

| Tier | Nom             | Technologie                   |
| ---- | --------------- | ----------------------------- |
| 1    | Présentation    | Next.js                       |
| 2    | Application/API | Node.js + Express.js          |
| 3    | Services Métier | Stripe · Nodemailer · Mapbox  |
| 4    | Données         | PostgreSQL + Supabase Storage |
