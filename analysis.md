


<img width="835" height="773" alt="Capture d&#39;écran 2026-06-28 203045" src="https://github.com/user-attachments/assets/1d0ac2db-0797-4935-8104-4d83d5c57e21" />

```sql
CREATE SCHEMA analytics;
```

```sql
CREATE TABLE analytics.dim_customers AS
SELECT
    c.customer_id,
    c.first_name || ' ' || c.last_name AS full_name,
    c.email,
    a.address,
    ci.city,
    co.country
FROM public.customer c
JOIN public.address a ON c.address_id = a.address_id
JOIN public.city ci ON a.city_id = ci.city_id
JOIN public.country co ON ci.country_id = co.country_id;
```

```sql
-- Sécurité : Supprime la table si tu veux rejouer le script
DROP TABLE IF EXISTS analytics.dim_films;

-- Création de la table dimensionnelle
-- Dans le schéma public, les informations d'un film sont éparpillées :
-- le titre est dans film, la catégorie est dans category (via une table intermédiaire film_category),
-- et la langue est dans language
CREATE TABLE analytics.dim_films AS
SELECT
    f.film_id,
    f.title AS titre_film,
    f.description,
    f.release_year AS annee_sortie,
    f.rental_duration AS duree_location_autorisee,
    f.rental_rate AS tarif_location,
    f.length AS duree_film_minutes,
    f.replacement_cost AS cout_remplacement,
    f.rating AS classification,
    l.name AS langue,
    c.name AS categorie_genre
FROM public.film f
LEFT JOIN public.language l ON f.language_id = l.language_id
LEFT JOIN public.film_category fc ON f.film_id = fc.film_id
LEFT JOIN public.category c ON fc.category_id = c.category_id;

-- Optionnel : Ajouter une clé primaire pour que DataGrip/DBeaver indexe bien la table
ALTER TABLE analytics.dim_films ADD PRIMARY KEY (film_id);
```

```sql
-- Sécurité : Supprime la table si elle existe déjà
DROP TABLE IF EXISTS analytics.dim_customers;

-- Création de la table dimensionnelle des clients
-- Sécurité : Supprime la table si elle existe déjà
DROP TABLE IF EXISTS analytics.dim_customers;

-- Création de la table dimensionnelle des clients (CORRIGÉE)
CREATE TABLE analytics.dim_customers AS
SELECT
    c.customer_id,
    c.first_name AS prenom,
    c.last_name AS nom,
    c.first_name || ' ' || c.last_name AS nom_complet,
    c.email,
    CASE WHEN c.active = 1 OR c.active::text = 'true' THEN 'Actif' ELSE 'Inactif' END AS statut_activite,
    a.address AS adresse,
    a.district AS region_departement,
    a.postal_code AS code_postal,
    a.phone AS telephone,
    ci.city AS ville,
    co.country AS pays
FROM public.customer c
LEFT JOIN public.address a ON c.address_id = a.address_id
LEFT JOIN public.city ci ON a.city_id = ci.city_id
LEFT JOIN public.country co ON ci.country_id = co.country_id;

-- Ajout de la clé primaire
ALTER TABLE analytics.dim_customers ADD PRIMARY KEY (customer_id);
```

```sql
-- Sécurité : Supprime la table si elle existe déjà
DROP TABLE IF EXISTS analytics.fact_rentals;

-- Création de la table de faits centrale
CREATE TABLE analytics.fact_rentals AS
SELECT
    r.rental_id,
    r.customer_id,         -- Clé vers dim_customers
    i.film_id,             -- Clé vers dim_films
    i.store_id,            -- Clé vers ta future dim_stores
    r.rental_date AS date_heure_location,
    r.rental_date::date AS date_key, -- Utile pour lier une dimension temporelle plus tard
    r.return_date AS date_heure_retour,
    -- Calcul de la durée réelle de location en jours
    EXTRACT(DAY FROM (r.return_date - r.rental_date)) AS duree_reelle_jours,
    -- Récupération du montant payé (on s'assure de gérer les doublons potentiels avec un MAX ou un regroupement)
    COALESCE(p.amount, 0) AS montant_paye
FROM public.rental r
LEFT JOIN public.inventory i ON r.inventory_id = i.inventory_id
LEFT JOIN public.payment p ON r.rental_id = p.rental_id;

-- Ajout de la clé primaire
ALTER TABLE analytics.fact_rentals ADD PRIMARY KEY (rental_id);
```

```sql
-- Sécurité : Supprime la table si elle existe déjà
DROP TABLE IF EXISTS analytics.dim_stores;

CREATE TABLE analytics.dim_films AS
SELECT
    f.film_id,
    f.title AS titre_film,
    f.description,
    f.release_year AS annee_sortie,
    f.rental_duration AS duree_location_autorisee,
    f.rental_rate AS tarif_location,
    f.length AS duree_film_minutes,
    f.replacement_cost AS cout_remplacement,
    f.rating AS classification,
    l.name AS langue,
    c.name AS categorie_genre
FROM public.film f
LEFT JOIN public.language l ON f.language_id = l.language_id
LEFT JOIN public.film_category fc ON f.film_id = fc.film_id
LEFT JOIN public.category c ON fc.category_id = c.category_id;

ALTER TABLE analytics.dim_films ADD PRIMARY KEY (film_id);
```

```sql
DROP TABLE IF EXISTS analytics.dim_actors_bridge;

CREATE TABLE analytics.dim_actors_bridge AS
SELECT
    fa.film_id,
    a.actor_id,
    a.first_name || ' ' || a.last_name AS nom_acteur
FROM public.film_actor fa
LEFT JOIN public.actor a ON fa.actor_id = a.actor_id;

-- Clé primaire composite pour garantir l'unicité du couple Film-Acteur
ALTER TABLE analytics.dim_actors_bridge ADD PRIMARY KEY (film_id, actor_id);
```
