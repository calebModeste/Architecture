# Rapport Technique — Cluster de Bases de Données

**Projet :** Application de réservation hôtelière (type Airbnb)
**Stack :** Next.js · Node.js/Express · PostgreSQL (Supabase)
**Date :** Avril 2026

---

## 1. Pourquoi un cluster ?

Un serveur unique représente un point de défaillance unique. Le cluster répond à trois risques :

- **Disponibilité** : si le serveur tombe, l'application reste en ligne via un réplica
- **Performance** : les lectures (catalogue, avis) sont distribuées sur les réplicas
- **Durabilité** : perte de données impossible grâce aux backups et à la réplication

---

## 2. Modélisation des Tables

### Tables et domaines

L'application repose sur **15 tables** réparties en 4 domaines :

| Domaine     | Tables                                                          |
| ----------- | --------------------------------------------------------------- |
| Identité    | USER, ROLE                                                      |
| Catalogue   | LOCATION, TYPE_LOCATION, AMENITIES, LOCATION_AMENITIES, SERVICE |
| Transaction | RESERVATION, RESERVATION_SERVICE, PAYMENT, CHECK_IN_OUT         |
| Post-séjour | REVIEW, CALENDAR, BANK_ACCOUNT, REPORT                          |

Toutes les tables ont une clé primaire `id SERIAL` et les relations sont assurées par des clés étrangères avec contrainte d'intégrité.

### Index principaux

```sql
CREATE UNIQUE INDEX idx_user_email       ON USER(email);
CREATE INDEX idx_location_city           ON LOCATION(city);
CREATE INDEX idx_location_geo            ON LOCATION(latitude, longitude);
CREATE INDEX idx_reservation_location    ON RESERVATION(location_id);
CREATE INDEX idx_reservation_dates       ON RESERVATION(check_in, check_out);
CREATE UNIQUE INDEX idx_payment_tx       ON PAYMENT(transaction_id);
CREATE INDEX idx_calendar_location_date  ON CALENDAR(location_id, availability_date);
```

### Normalisation (3NF)

| Règle                                  | Application dans le projet                                                                     |
| -------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **1NF** — pas de valeur multiple       | Les équipements sont dans `LOCATION_AMENITIES`, pas en colonne `"wifi,piscine"`                |
| **2NF** — dépendance à toute la clé    | Dans `RESERVATION_SERVICE`, le `price` dépend de `(reservation_id, service_id)`, pas d'un seul |
| **3NF** — pas de dépendance transitive | `rating` dans LOCATION est dénormalisé intentionnellement (évite un `AVG()` à chaque requête)  |

---

## 3. Technique de Réplication

### Comparaison

| Technique          | Avantage                | Inconvénient                               |
| ------------------ | ----------------------- | ------------------------------------------ |
| **Synchrone**      | Aucune perte de données | Latence plus élevée                        |
| **Asynchrone**     | Rapide                  | Risque de perte si crash avant réplication |
| **Semi-synchrone** | Compromis               | Plus complexe                              |

### Choix : réplication mixte selon le domaine (théorème CAP)

**Données critiques → synchrone (CP)** : PAYMENT, RESERVATION, CHECK_IN_OUT

> Un paiement perdu = argent perdu. La latence supplémentaire est acceptable.

**Données non critiques → asynchrone (AP)** : LOCATION, CALENDAR, REVIEW, REPORT

> Un avis affiché avec 1 seconde de délai n'a aucun impact métier.

Configuration PostgreSQL :

```sql
-- postgresql.conf
synchronous_standby_names = 'FIRST 1 (replica1, replica2)'
synchronous_commit = on
```

---

## 4. Configuration du Cluster

### Choix : Actif-Passif

```
Application
    │
    ├─ Écritures ──→ PRIMARY (Read/Write)
    │                    │ WAL sync  → REPLICA 1 (Read-Only)
    │                    └ WAL async → REPLICA 2 (Standby/Failover)
    │
    └─ Lectures ───→ REPLICA 1 (via PgBouncer)
```

| Nœud      | Rôle                       | Réplication |
| --------- | -------------------------- | ----------- |
| Primary   | Toutes les écritures       | Source      |
| Replica 1 | Lectures (catalogue, avis) | Synchrone   |
| Replica 2 | Failover en cas de panne   | Asynchrone  |

**Pourquoi pas actif-actif ?** Deux nœuds qui écrivent simultanément créent des conflits (ex : deux clients qui réservent le même logement). PostgreSQL ne supporte pas le multi-master nativement.

**PgBouncer** gère le routage et le pooling des connexions : lectures → Replica 1, écritures → Primary.

---

## 5. Haute Disponibilité

### Failover automatique (Patroni)

Supabase intègre Patroni qui détecte les pannes en ~8s et promeut automatiquement un réplica.

- **RTO < 30 secondes** (temps de rétablissement)
- **RPO < 1 minute** (données perdues au maximum)

### Backup

| Type                  | Fréquence  | Rétention |
| --------------------- | ---------- | --------- |
| Snapshot complet      | Quotidien  | 30 jours  |
| Archivage WAL continu | Temps réel | 7 jours   |

### Réponse aux pannes

| Scénario                  | Réponse                                                  |
| ------------------------- | -------------------------------------------------------- |
| Primary crash             | Patroni promeut Replica 2 automatiquement (< 30s)        |
| Replica 1 tombe           | Lectures redirigées vers Primary le temps du redémarrage |
| Corruption de données     | Restauration WAL au point précédant la corruption        |
| Saturation des connexions | PgBouncer limite et file les connexions entrantes        |

---

## 6. Choix CAP par table

| Table           | CAP    | Raison                                   |
| --------------- | ------ | ---------------------------------------- |
| PAYMENT         | **CP** | Incohérence = argent perdu               |
| RESERVATION     | **CP** | Double réservation inacceptable          |
| USER / ROLE     | **CP** | Session invalide = faille de sécurité    |
| LOCATION        | **AP** | Décalage de quelques secondes acceptable |
| CALENDAR        | **AP** | Cache Redis (TTL 30s) suffisant          |
| REVIEW / REPORT | **AP** | Pas de contrainte temps-réel             |

---

## 7. Synthèse

| Décision    | Choix                                  | Justification                                |
| ----------- | -------------------------------------- | -------------------------------------------- |
| Nœuds       | 1 Primary + 2 Replicas                 | RTO < 30s, aucun point de défaillance unique |
| Mode        | Actif-Passif                           | PostgreSQL natif, cohérence forte garantie   |
| Réplication | Synchrone / Asynchrone selon criticité | Compromis CAP par domaine métier             |
| Failover    | Automatique via Patroni                | Rétablissement sans intervention humaine     |
| Backup      | WAL continu + snapshot quotidien       | RPO < 1 min pour les données financières     |
| Pooling     | PgBouncer                              | Tient la charge sans saturer PostgreSQL      |
