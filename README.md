# Segmentation RFM Avancée & Stratégie de Persistance SQL

## Description du Projet
Conception et implémentation d'un modèle de segmentation client RFM (Récence, Fréquence, Montant) de bout en bout en SQL. Ce projet intègre une approche de persistance des données pour stocker directement les scores individuels, le score global et les segments marketing qualitatifs dans la table client, optimisant ainsi les ciblages CRM.

---

## Compétences Validées
* **Requêtes analytiques et agrégations :** Utilisation de `MAX`, `COUNT`, `SUM` et de la fonction `DATEDIFF` pour analyser le comportement d'achat.
* **Sous-requêtes corrélées :** Mise en place de liaisons dynamiques (`WHERE p.customer_id = c.customer_id`) pour injecter des métriques calculées dans une table principale.
* **Modélisation de données et typage :** Manipulation des types `INT` et `VARCHAR`, gestion de colonnes persistantes et structuration d'un score agrégé sur 15.
* **Traduction Métier :** Traduction de scores quantitatifs en segments comportementaux alignés avec une stratégie e-commerce (`VIP`, `Fidèles`, `Occasionnels`, `Dormants`).

---

## Script SQL Complet et Structuré

```sql
-- =========================================================================
-- 1. Récence (Recency)
-- =========================================================================
UPDATE customer c
SET c.recency_score = (
    SELECT 
        CASE 
            WHEN DATEDIFF(CURRENT_DATE, MAX(p.payment_date)) <= 90 THEN 5
            WHEN DATEDIFF(CURRENT_DATE, MAX(p.payment_date)) > 90 
                 AND DATEDIFF(CURRENT_DATE, MAX(p.payment_date)) <= 180 THEN 4
            WHEN DATEDIFF(CURRENT_DATE, MAX(p.payment_date)) > 180 
                 AND DATEDIFF(CURRENT_DATE, MAX(p.payment_date)) <= 365 THEN 3
            ELSE 1
        END
    FROM payment p
    WHERE p.customer_id = c.customer_id
);

-- =========================================================================
-- 2. Fréquence (Frequency)
-- =========================================================================
UPDATE customer c
SET c.frequency_score = (
    SELECT 
        CASE 
            WHEN COUNT(p.payment_id) = 1 THEN 1
            WHEN COUNT(p.payment_id) BETWEEN 2 AND 3 THEN 2
            WHEN COUNT(p.payment_id) BETWEEN 4 AND 6 THEN 4
            WHEN COUNT(p.payment_id) >= 7 THEN 5
            ELSE 1
        END
    FROM payment p
    WHERE p.customer_id = c.customer_id
);

-- =========================================================================
-- 3. Montant (Monetary)
-- =========================================================================
UPDATE customer c
SET c.monetary_score = (
    SELECT
    CASE
        WHEN SUM(p.amount) <= 200 THEN 1
        WHEN SUM(p.amount) BETWEEN 200 AND 500 THEN 2
        WHEN SUM(p.amount) BETWEEN 500 AND 900 THEN 3
        WHEN SUM(p.amount) BETWEEN 900 AND 1300 THEN 4
        WHEN SUM(p.amount) > 1300 THEN 5
        ELSE 1
    END
    FROM payment p
    WHERE p.customer_id = c.customer_id
);

-- =========================================================================
-- 4. Score Global
-- =========================================================================
ALTER TABLE customer 
ADD COLUMN global_score INT;

UPDATE customer 
SET global_score = recency_score + frequency_score + monetary_score;

-- =========================================================================
-- 5. Segment Client
-- =========================================================================
ALTER TABLE customer 
ADD COLUMN customer_segment VARCHAR(50);

UPDATE customer
SET customer_segment = CASE
    WHEN global_score >= 13 THEN 'VIP'
    WHEN global_score BETWEEN 10 AND 12 THEN 'Fidèles'
    WHEN global_score BETWEEN 6 AND 9 THEN 'Occasionnels'
    ELSE 'Dormants'
END;
