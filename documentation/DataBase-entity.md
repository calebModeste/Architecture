# DATABASE - ENTITÉS

## Vue d'ensemble

Base de données pour une application de réservation type Airbnb avec gestion complète des locations, réservations, paiements et services additionnels.

---

## ENTITÉS

### 1. **USER**

> **Description**: Entité représentant tous les utilisateurs de l'application (clients, admins, super admins)

| Colonne name        | type        | relation/attr | Description                              |
| ------------------- | ----------- | ------------- | ---------------------------------------- |
| id                  | int         | PK            | Identifiant unique de l'utilisateur      |
| first_name          | string(100) |               | Prénom                                   |
| last_name           | string(100) |               | Nom                                      |
| email               | email()     | unique        | Email unique de l'utilisateur            |
| password            | string(255) |               | Mot de passe hashé                       |
| phone               | string(20)  |               | Numéro de téléphone                      |
| profile_picture     | string()    |               | URL de la photo de profil                |
| role_id             | int         | FK Role(id)   | Rôle de l'utilisateur                    |
| verification_status | enum        |               | État de vérification (pending, verified) |
| created_at          | timestamp   |               | Date de création du compte               |
| updated_at          | timestamp   |               | Date de dernière modification            |
| is_active           | boolean     |               | État du compte (actif/inactif)           |

---

### 2. **ROLE**

> **Description**: Entité définissant les rôles et permissions des utilisateurs

| Colonne name | type       | relation/attr | Description                              |
| ------------ | ---------- | ------------- | ---------------------------------------- |
| id           | int        | PK            | Identifiant du rôle                      |
| role_name    | string(30) | unique        | Nom du rôle (CLIENT, ADMIN, SUPER_ADMIN) |
| description  | text       |               | Description des permissions              |

---

### 3. **LOCATION**

> **Description**: Entité représentant les locations/propriétés disponibles à la réservation

| Colonne name     | type        | relation/attr       | Description                 |
| ---------------- | ----------- | ------------------- | --------------------------- |
| id               | int         | PK                  | Identifiant de la location  |
| owner_id         | int         | FK User(id)         | Propriétaire de la location |
| title            | string(255) |                     | Titre de la location        |
| description      | text        |                     | Description détaillée       |
| type_location_id | int         | FK TypeLocation(id) | Type de location            |
| nb_max_guests    | int         |                     | Nombre maximum de personnes |
| nb_rooms         | int         |                     | Nombre de chambres          |
| nb_bathrooms     | int         |                     | Nombre de salles de bain    |
| price_per_night  | decimal     |                     | Prix par nuit               |
| rating           | float       |                     | Note moyenne (0-5)          |
| latitude         | float       |                     | Latitude GPS                |
| longitude        | float       |                     | Longitude GPS               |
| address          | string(255) |                     | Adresse complète            |
| city             | string(100) |                     | Ville                       |
| postal_code      | string(20)  |                     | Code postal                 |
| country          | string(100) |                     | Pays                        |
| is_active        | boolean     |                     | Location disponible ou non  |
| created_at       | timestamp   |                     | Date de création            |
| updated_at       | timestamp   |                     | Date de modification        |

---

### 4. **TYPE_LOCATION**

> **Description**: Les différents types de location disponibles

| Colonne name | type        | relation/attr | Description                                      |
| ------------ | ----------- | ------------- | ------------------------------------------------ |
| id           | int         | PK            | Identifiant du type                              |
| type_name    | string(100) | unique        | Nom du type (Hôtel, Appartement, Villa, Chambre) |
| description  | text        |               | Description du type                              |

---

### 5. **AMENITIES** (Équipements)

> **Description**: Liste des équipements/commodités disponibles dans les locations

| Colonne name | type        | relation/attr | Description                              |
| ------------ | ----------- | ------------- | ---------------------------------------- |
| id           | int         | PK            | Identifiant de l'équipement              |
| name         | string(100) | unique        | Nom de l'équipement (WiFi, Cuisine, etc) |
| icon         | string()    |               | Icône représentant l'équipement          |
| description  | text        |               | Description de l'équipement              |

---

### 6. **LOCATION_AMENITIES** (Relation Many-to-Many)

> **Description**: Association entre locations et leurs équipements

| Colonne name | type | relation/attr              | Description |
| ------------ | ---- | -------------------------- | ----------- |
| id           | int  | PK                         | Identifiant |
| location_id  | int  | FK Location(id) - cascade  | Location    |
| amenity_id   | int  | FK Amenities(id) - cascade | Équipement  |

---

### 7. **RESERVATION**

> **Description**: Les réservations de locations par les clients

| Colonne name | type      | relation/attr   | Description                                     |
| ------------ | --------- | --------------- | ----------------------------------------------- |
| id           | int       | PK              | Identifiant de la réservation                   |
| location_id  | int       | FK Location(id) | Location réservée                               |
| user_id      | int       | FK User(id)     | Client effectuant la réservation                |
| check_in     | date      |                 | Date d'arrivée                                  |
| check_out    | date      |                 | Date de départ                                  |
| nb_guests    | int       |                 | Nombre de clients                               |
| total_price  | decimal   |                 | Prix total de la réservation                    |
| status       | enum      |                 | État (pending, confirmed, cancelled, completed) |
| notes        | text      |                 | Notes additionnelles du client                  |
| created_at   | timestamp |                 | Date de création de la réservation              |
| updated_at   | timestamp |                 | Date de modification                            |

---

### 8. **RESERVATION_SERVICE** (Relation Many-to-Many)

> **Description**: Services additionnels inclus dans une réservation

| Colonne name   | type    | relation/attr                | Description              |
| -------------- | ------- | ---------------------------- | ------------------------ |
| id             | int     | PK                           | Identifiant              |
| reservation_id | int     | FK Reservation(id) - cascade | Réservation              |
| service_id     | int     | FK Service(id) - cascade     | Service                  |
| quantity       | int     |                              | Quantité du service      |
| price          | decimal |                              | Prix du service appliqué |

---

### 9. **SERVICE**

> **Description**: Services supplémentaires disponibles (parking, restaurant, voiture, etc.)

| Colonne name | type        | relation/attr | Description                                |
| ------------ | ----------- | ------------- | ------------------------------------------ |
| id           | int         | PK            | Identifiant du service                     |
| name         | string(100) |               | Nom du service (Parking, Restaurant, etc.) |
| description  | text        |               | Description détaillée                      |
| base_price   | decimal     |               | Prix de base du service                    |
| type         | enum        |               | Type (optional, required, addon)           |
| is_active    | boolean     |               | Service disponible ou non                  |

---

### 10. **PAYMENT**

> **Description**: Gestion des paiements pour les réservations

| Colonne name   | type      | relation/attr      | Description                                 |
| -------------- | --------- | ------------------ | ------------------------------------------- |
| id             | int       | PK                 | Identifiant du paiement                     |
| reservation_id | int       | FK Reservation(id) | Réservation concernée                       |
| user_id        | int       | FK User(id)        | Utilisateur effectuant le paiement          |
| amount         | decimal   |                    | Montant payé                                |
| payment_method | enum      |                    | Méthode (carte, virement, portefeuille)     |
| status         | enum      |                    | État (pending, completed, failed, refunded) |
| transaction_id | string()  |                    | ID de la transaction externe                |
| payment_date   | timestamp |                    | Date du paiement                            |
| refund_date    | timestamp |                    | Date du remboursement (si applicable)       |

---

### 11. **CHECK_IN_OUT**

> **Description**: Gestion des entrées et sorties des clients

| Colonne name     | type      | relation/attr      | Description                           |
| ---------------- | --------- | ------------------ | ------------------------------------- |
| id               | int       | PK                 | Identifiant                           |
| reservation_id   | int       | FK Reservation(id) | Réservation concernée                 |
| user_id          | int       | FK User(id)        | Client                                |
| check_in_time    | timestamp |                    | Heure d'arrivée réelle                |
| check_out_time   | timestamp |                    | Heure de départ réelle                |
| check_in_status  | enum      |                    | État (pending, completed, no_show)    |
| check_out_status | enum      |                    | État (pending, completed)             |
| notes            | text      |                    | Notes du staff (état des lieux, etc.) |

---

### 12. **REVIEW**

> **Description**: Avis et évaluations des clients sur les locations

| Colonne name   | type        | relation/attr      | Description                 |
| -------------- | ----------- | ------------------ | --------------------------- |
| id             | int         | PK                 | Identifiant de l'avis       |
| reservation_id | int         | FK Reservation(id) | Réservation concernée       |
| location_id    | int         | FK Location(id)    | Location évaluée            |
| user_id        | int         | FK User(id)        | Auteur de l'avis            |
| rating         | int         |                    | Note (1-5 étoiles)          |
| title          | string(200) |                    | Titre de l'avis             |
| comment        | text        |                    | Contenu de l'avis           |
| created_at     | timestamp   |                    | Date de création            |
| is_verified    | boolean     |                    | Avis vérifié (après séjour) |

---

### 13. **BANK_ACCOUNT**

> **Description**: Comptes bancaires pour les paiements aux propriétaires

| Colonne name   | type        | relation/attr | Description                          |
| -------------- | ----------- | ------------- | ------------------------------------ |
| id             | int         | PK            | Identifiant                          |
| user_id        | int         | FK User(id)   | Propriétaire propriétaire du compte  |
| account_holder | string(200) |               | Titulaire du compte                  |
| iban           | string(34)  | unique        | IBAN du compte                       |
| bic            | string(11)  |               | Code BIC                             |
| is_verified    | boolean     |               | Compte vérifié ou non                |
| is_default     | boolean     |               | Compte par défaut pour les paiements |
| created_at     | timestamp   |               | Date de création                     |

---

### 14. **REPORT**

> **Description**: Rapports et statistiques pour les administrateurs

| Colonne name | type        | relation/attr | Description                               |
| ------------ | ----------- | ------------- | ----------------------------------------- |
| id           | int         | PK            | Identifiant du rapport                    |
| created_by   | int         | FK User(id)   | Admin ayant généré le rapport             |
| report_type  | enum        |               | Type (revenue, bookings, users, activity) |
| title        | string(255) |               | Titre du rapport                          |
| description  | text        |               | Description du rapport                    |
| data         | json        |               | Données du rapport (format JSON)          |
| generated_at | timestamp   |               | Date de génération                        |
| period_start | date        |               | Début de la période analysée              |
| period_end   | date        |               | Fin de la période analysée                |

---

### 15. **CALENDAR**

> **Description**: Calendrier des disponibilités des locations

| Colonne name      | type      | relation/attr   | Description                                 |
| ----------------- | --------- | --------------- | ------------------------------------------- |
| id                | int       | PK              | Identifiant                                 |
| location_id       | int       | FK Location(id) | Location concernée                          |
| availability_date | date      |                 | Date de la disponibilité                    |
| is_available      | boolean   |                 | Disponible (true) ou non (false)            |
| price_override    | decimal   |                 | Prix spécifique pour cette date (optionnel) |
| updated_at        | timestamp |                 | Date de dernière modification               |

---

## RELATIONS CLÉS

- **USER** ← FK → **ROLE** (N:1)
- **LOCATION** ← FK → **USER** (N:1 - owner_id)
- **LOCATION** ← FK → **TYPE_LOCATION** (N:1)
- **LOCATION_AMENITIES** ← FK → **LOCATION** (N:M)
- **LOCATION_AMENITIES** ← FK → **AMENITIES** (N:M)
- **RESERVATION** ← FK → **LOCATION** (N:1)
- **RESERVATION** ← FK → **USER** (N:1)
- **RESERVATION_SERVICE** ← FK → **RESERVATION** (N:M)
- **RESERVATION_SERVICE** ← FK → **SERVICE** (N:M)
- **PAYMENT** ← FK → **RESERVATION** (1:1 ou N:1)
- **PAYMENT** ← FK → **USER** (N:1)
- **CHECK_IN_OUT** ← FK → **RESERVATION** (1:1)
- **CHECK_IN_OUT** ← FK → **USER** (N:1)
- **REVIEW** ← FK → **RESERVATION** (1:1)
- **REVIEW** ← FK → **LOCATION** (N:1)
- **REVIEW** ← FK → **USER** (N:1)
- **BANK_ACCOUNT** ← FK → **USER** (N:1)
- **REPORT** ← FK → **USER** (N:1 - created_by)
