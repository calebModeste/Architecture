# Système de Gestion Hôtelière - Architecture complète

## Table des matières

1. [Présentation du projet](#présentation-du-projet)
2. [Fonctionnalités principales](#fonctionnalités-principales)
3. [Architecture technique](#architecture-technique)
4. [Modèle de données](#modèle-de-données)
5. [Rôles et permissions](#rôles-et-permissions)
6. [Technologies utilisées](#technologies-utilisées)
7. [Flux de réservation](#flux-de-réservation)
8. [Sécurité](#sécurité)
9. [Installation et déploiement](#installation-et-déploiement)

---

## Présentation du projet

### Vue d'ensemble
Le système de gestion hôtelière est une application web complète permettant de gérer de manière centralisée toutes les opérations liées à la location de biens immobiliers tels que des chambres, studios et appartements.

Cette application a été conçue pour répondre aux besoins modernes de digitalisation du secteur hôtelier en remplaçant les méthodes traditionnelles par une solution automatisée, sécurisée et accessible en ligne.

### Objectifs principaux
- **Automatisation** des processus de réservation et de paiement
- **Centralisation** des données pour une meilleure gestion
- **Sécurisation** des transactions et des informations personnelles
- **Amélioration** de l'expérience utilisateur
- **Optimisation** de la communication entre les différents acteurs

### Problèmes résolus
- Élimination des réservations manuelles et des erreurs associées
- Prévention des doubles réservations
- Mise à jour en temps réel des disponibilités
- Structuration du suivi des paiements
- Accélération de la communication client-gestionnaire

---

## Fonctionnalités principales

### 1. Authentification et sécurité
- **Gestion multi-rôles** : Client, Admin, Super Admin
- **Protection des mots de passe** par chiffrement
- **Sessions sécurisées** avec JWT
- **Prévention des accès non autorisés**
- **Récupération de mot de passe**

### 2. Gestion des biens immobiliers
- **Ajout/Modification/Suppression** de locations
- **Informations détaillées** : type, prix, localisation, images, description
- **Gestion des disponibilités** en temps réel
- **Association d'équipements** (WiFi, cuisine, parking, etc.)

### 3. Système de réservation
- **Vérification automatique** des disponibilités
- **Processus de validation** par les administrateurs
- **Statuts de réservation** : En attente, Confirmée, Annulée, Terminée
- **Prévention des conflits** de réservation

### 4. Gestion des paiements
- **Paiements en ligne** sécurisés via Stripe
- **Enregistrement** de chaque transaction
- **Suivi transparent** des paiements
- **Remboursements** automatiques en cas d'annulation

### 5. Tableau de bord et statistiques
- **Visualisation** des réservations totales
- **Suivi des revenus** générés
- **Taux d'occupation** des biens
- **Indicateurs de performance** globaux

### 6. Avis et évaluations
- **Système de notation** (1-5 étoiles)
- **Commentaires détaillés** sur les locations
- **Amélioration continue** des services
- **Renforcement de la confiance** entre utilisateurs

---

## Architecture technique

### Architecture 4-Tier

```
TIER 1 - Présentation
    Next.js
    (Interface utilisateur)

TIER 2 - Application/API  
    Node.js + Express.js
    (Logique métier, authentification)

TIER 3 - Services Métier
    Stripe | Nodemailer | Mapbox
    (Paiements, emails, géolocalisation)

TIER 4 - Données
    PostgreSQL | File Storage
    (Base de données, fichiers)
```

### TIER 1 - Couche Présentation
**Technologie** : `Next.js`

**Responsabilités** :
- Affichage des pages (SSR, SSG, CSR)
- Gestion de la navigation et des routes
- Formulaires et validations côté client
- Consommation de l'API REST
- Gestion de l'état global

**Modules UI** :
- Page d'accueil et recherche
- Pages de locations détaillées
- Tunnel de réservation
- Tableaux de bord admin
- Interface de gestion

### TIER 2 - Couche Application/API
**Technologie** : `Node.js + Express.js`

**Responsabilités** :
- Exposition des endpoints REST
- Authentification et autorisation
- Validation des données
- Orchestration des services externes

**Routes principales** :
| Route | Méthodes | Rôles autorisés |
|-------|----------|----------------|
| `/auth` | POST | PUBLIC |
| `/locations` | GET, POST, PUT, DELETE | CLIENT, ADMIN |
| `/reservations` | GET, POST, PUT | CLIENT, ADMIN |
| `/payments` | POST, GET | CLIENT, ADMIN |
| `/reviews` | GET, POST | CLIENT |
| `/reports` | GET | ADMIN, SUPER ADMIN |
| `/admin` | CRUD | SUPER ADMIN |

### TIER 3 - Services Métier

#### Stripe (Paiements)
- **Payment Intent** : Création des paiements
- **Remboursements** : Automatiques en cas d'annulation
- **Webhooks** : Confirmation asynchrone
- **Compte Connect** : Versements aux propriétaires

#### Nodemailer (Notifications)
- **Emails transactionnels** à chaque étape clé
- **Inscription** : Email de bienvenue et vérification
- **Réservation** : Récapitulatif et confirmation
- **Paiement** : Notification de réception
- **Rappels** : Check-in proche

#### Mapbox (Géolocalisation)
- **Carte interactive** avec markers
- **Géocodage** : Conversion adresse/coordonnées
- **Recherche géographique** par zone/rayon
- **Calcul des distances**

### TIER 4 - Couche Données
**Technologie** : `PostgreSQL direct avec Node.js`

#### Base de données
15 entités structurées en plusieurs groupes :
- **Utilisateurs & Accès** : USER, ROLE
- **Locations** : LOCATION, TYPE_LOCATION, AMENITIES
- **Réservations** : RESERVATION, SERVICE, RESERVATION_SERVICE
- **Transactions** : PAYMENT, CHECK_IN_OUT
- **Feedback & Rapports** : REVIEW, REPORT
- **Planification** : CALENDAR, BANK_ACCOUNT

#### Fonctions PostgreSQL utilisées

| Fonctionnalité     | Usage                                          |
| ------------------ | ---------------------------------------------- |
| Connexions poolées | Gestion des connexions via pg                  |
| Transactions       | ACID pour les opérations critiques             |
| Indexation         | Optimisation des requêtes                      |
| Triggers           | Logique métier en base de données              |
| Views              | Requêtes complexes pré-définies                |

---

## Modèle de données

### Entités principales

#### USER
Gestion de tous les utilisateurs du système avec rôles spécifiques.

| Champ | Type | Description |
|-------|------|-------------|
| id | INT | Identifiant unique |
| first_name | VARCHAR(100) | Prénom |
| last_name | VARCHAR(100) | Nom |
| email | EMAIL | Email unique |
| password | VARCHAR(255) | Mot de passe hashé |
| role_id | INT | Référence au rôle |
| verification_status | ENUM | État de vérification |
| is_active | BOOLEAN | État du compte |

#### LOCATION
Représentation des biens immobiliers disponibles.

| Champ | Type | Description |
|-------|------|-------------|
| id | INT | Identifiant unique |
| owner_id | INT | Propriétaire (FK User) |
| title | VARCHAR(255) | Titre de la location |
| description | TEXT | Description détaillée |
| type_location_id | INT | Type de location |
| price_per_night | DECIMAL | Prix par nuit |
| latitude/longitude | FLOAT | Coordonnées GPS |
| address | VARCHAR(255) | Adresse complète |
| is_active | BOOLEAN | Disponibilité |

#### RESERVATION
Gestion des réservations clients.

| Champ | Type | Description |
|-------|------|-------------|
| id | INT | Identifiant unique |
| location_id | INT | Location réservée |
| user_id | INT | Client |
| check_in/check_out | DATE | Dates de séjour |
| total_price | DECIMAL | Prix total |
| status | ENUM | État de la réservation |

#### PAYMENT
Traitement des paiements sécurisés.

| Champ | Type | Description |
|-------|------|-------------|
| id | INT | Identifiant unique |
| reservation_id | INT | Réservation concernée |
| amount | DECIMAL | Montant payé |
| payment_method | ENUM | Méthode de paiement |
| status | ENUM | État du paiement |
| transaction_id | STRING | ID transaction externe |

### Relations clés
- **USER** 1:N **LOCATION** (propriétaire)
- **USER** 1:N **RESERVATION** (client)
- **LOCATION** 1:N **RESERVATION**
- **RESERVATION** 1:1 **PAYMENT**
- **LOCATION** M:N **AMENITIES**

---

## Rôles et permissions

### CLIENT
**Fonctionnalités** :
- Authentification et gestion de profil
- Consultation des locations disponibles
- Réservation de biens
- Paiement en ligne sécurisé
- Donner des avis et évaluations

### ADMIN
**Fonctionnalités** :
- Authentification avancée
- Gestion complète des locations
- Tableau de bord avec statistiques
- Gestion des réservations
- Création des comptes bancaires
- Gestion des clients

### SUPER ADMIN
**Fonctionnalités** :
- Gestion des administrateurs
- Configuration du système
- Consultation des rapports globaux
- Supervision de l'ensemble du système
- Maintenance et paramètres avancés

---

## Technologies utilisées

### Frontend
- **Next.js** : Framework React moderne
- **TypeScript** : Typage strict
- **Tailwind CSS** : Styling utility-first
- **React Hook Form** : Gestion des formulaires

### Backend
- **Node.js** : Runtime JavaScript
- **Express.js** : Framework web
- **TypeScript** : Typage côté serveur
- **JWT** : Authentification

### Base de données
- **PostgreSQL** : SGBD relationnel
- **Prisma** : ORM et migrations
- **pg** : Driver PostgreSQL pour Node.js

### Services externes
- **Stripe** : Paiements sécurisés
- **Nodemailer** : Envoi d'emails
- **Mapbox** : Cartographie et géolocalisation

### Infrastructure
- **Vercel** : Hébergement frontend
- **Railway/Heroku** : Hébergement backend
- **PostgreSQL** : Base de données hébergée
- **AWS S3/Cloudinary** : Stockage de fichiers

---

## Flux de réservation complet

### Processus étape par étape

1. **Recherche** (Client)
   - Filtres et recherche géographique
   - Affichage sur carte Mapbox
   - Consultation des disponibilités

2. **Sélection** (Client)
   - Visualisation détaillée de la location
   - Vérification du calendrier
   - Choix des dates et services

3. **Réservation** (Client)
   - Formulaire de réservation
   - Création en base (statut: pending)
   - Redirection vers paiement

4. **Paiement** (Client/Stripe)
   - Formulaire Stripe sécurisé
   - Validation du paiement
   - Confirmation webhook

5. **Validation** (Système)
   - Mise à jour statut réservation
   - Enregistrement paiement
   - Envoi email confirmation

6. **Gestion** (Admin)
   - Tableau de bord des réservations
   - Gestion des check-in/out
   - Suivi des revenus

### Diagramme de séquence simplifié

```
Client -> Next.js: Recherche locations
Next.js -> API: GET /locations
API -> DB: Query locations
DB -> API: Results
API -> Next.js: JSON response
Next.js -> Client: Affichage résultats

Client -> Next.js: Réservation
Next.js -> API: POST /reservations
API -> Stripe: Create PaymentIntent
Stripe -> API: client_secret
API -> Next.js: Response
Next.js -> Client: Formulaire paiement

Client -> Stripe: Paiement
Stripe -> API: Webhook confirmation
API -> DB: Update reservation status
API -> Nodemailer: Send email
Nodemailer -> Client: Email confirmation
```

---

## Sécurité

### Par couche

#### Tier 1 (Présentation)
- **HTTPS** obligatoire
- **Protection CSRF** (Next.js)
- **Validation client**
- **Sanitization inputs**

#### Tier 2 (API)
- **JWT** pour l'authentification
- **Middleware de rôles**
- **Rate limiting**
- **Validation serveur**

#### Tier 3 (Services)
- **Clés API secrètes**
- **Variables environnement**
- **Signatures webhook** (Stripe)
- **Connexions sécurisées**

#### Tier 4 (Données)
- **Sécurité PostgreSQL** (rôles, permissions)
- **Connexion chiffrée** (SSL)
- **Backups automatiques** (pg_dump)
- **Audit logs** (triggers PostgreSQL)

### Mesures spécifiques
- **Chiffrement** des mots de passe (bcrypt)
- **Tokens** à expiration limitée
- **Vérification** email obligatoire
- **Logs** de sécurité détaillés
- **Monitoring** des activités suspectes

---

## Installation et déploiement

### Prérequis
- Node.js 18+
- PostgreSQL 14+
- Comptes services externes (Stripe, Mapbox, Supabase)

### Configuration locale

1. **Clonage du projet**
```bash
git clone [repository-url]
cd hotel-management-system
```

2. **Installation dépendances**
```bash
npm install
# Frontend
cd frontend && npm install
# Backend  
cd ../backend && npm install
```

3. **Configuration environnement**
```bash
# .env.local (Frontend)
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your_stripe_key

# .env (Backend)
DATABASE_URL=postgresql://username:password@host:port/database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=hotel_management
DB_USER=postgres
DB_PASSWORD=your_password
STRIPE_SECRET_KEY=your_stripe_secret
EMAIL_HOST=smtp_provider
EMAIL_USER=your_email
EMAIL_PASS=your_password
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_S3_BUCKET=your_bucket_name
```

4. **Base de données**
```bash
# Migration Prisma
npx prisma migrate dev
# Seed data
npx prisma db seed
```

5. **Démarrage**
```bash
# Backend
npm run dev

# Frontend (autre terminal)
cd frontend && npm run dev
```

### Déploiement production

#### Frontend (Vercel)
```bash
vercel --prod
```

#### Backend (Railway/Heroku)
```bash
# Configuration variables d'environnement
# Build et déploiement automatique
```

#### Base de données (PostgreSQL)
- Installation PostgreSQL local ou distant
- Configuration de la base de données
- Exécution des migrations Prisma
- Configuration des rôles et permissions

#### Stockage fichiers (AWS S3/Cloudinary)
- Création buckets S3 ou compte Cloudinary
- Configuration des clés d'accès
- Setup des permissions CORS

### Monitoring et maintenance

#### Logs et métriques
- **Application logs** : Winston/Morgan
- **Error tracking** : Sentry
- **Performance** : Vercel Analytics
- **Uptime** : UptimeRobot

#### Backups
- **Base de données** : pg_dump automatisé ou AWS RDS backups
- **Storage** : Versioning S3 ou backups Cloudinary
- **Code** : Git versioning

#### Mises à jour
- **Dépendances** : npm audit & update
- **Sécurité** : Patchs réguliers
- **Fonctionnalités** : Déploiement progressif

---

## Documentation complémentaire

### Schémas disponibles
- `schemas/architecture/` : Architecture 4-tier
- `schemas/class/` : Diagramme de classes complet
- `schemas/diagram action/` : Diagrammes d'activité par fonction
- `schemas/diagram cas usage/` : Cas d'utilisation
- `schemas/diagram sequance/` : Séquences système

### Documentation technique
- `documentation/DataBase-entity.md` : Modèle de données détaillé
- `documentation/architecture-4-tier.md` : Architecture technique
- `documentation/role-fonction.md` : Rôles et permissions
- `documentation/TECHNO.md` : Stack technique

---

## Licence et contribution

Ce projet est développé dans le cadre d'un TP de conception d'application. Pour toute question ou contribution, veuillez contacter l'équipe de développement.

---

**Version** : 1.0.0  
**Dernière mise à jour** : Avril 2026  
**Auteurs** : Équipe de développement IPSSI