

<img width="128" height="157" alt="dvd-rental-sample-database-diagram" src="https://github.com/user-attachments/assets/e082c83d-eb02-4efd-88e7-d7dc1cc05bff" />


Création d'un nouveau schéma `analytics` où nous créerons les différentes tables de dimensions et des faits.
```sql
CREATE SCHEMA analytics;
```

Dans le schéma public, les informations géographiques et d'identité de tes clients sont fragmentées à l'extrême (une table pour le client, une pour son adresse, une pour sa ville, et une pour son pays). 

Nous allons rassembler toutes ces données en une seule ligne par client grâce à des jointures successives :
- `public.customer` (L'identité : nom, prénom, email, statut actif)
- `public.address` (L'adresse physique et le code postal)
- `public.city` (Le nom de la ville)
- `public.country` (Le nom du pays)

```sql
-- Sécurité : Supprime la table si tu veux rejouer le script
DROP TABLE IF EXISTS analytics.dim_customers;

-- Création de la table dimensionnelle des clients 
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


Bien qu'il n'y ait que deux `stores` et un `manager` par store, j'ai fait le choix d'établir une table des dimensions pour les magains pour:
- L'évolutivité (Scalability) : Aujourd'hui, la chaîne de DVD n'a que 2 magasins. Mais si demain elle devait ouvrir de nouvelles boutiques, le modèle en étoile restera identique, et il suffira d'ajouter des lignes dans la table `dim_stores` sans toucher aux rapports Power BI. Bon, Netflix est passé par là et ce secteur a totalement disparu depuis de nombreuses années, mais la logique est là ;)

```sql
DROP TABLE IF EXISTS analytics.dim_stores;

-- Création de la table dimensionnelle des magasins
CREATE TABLE analytics.dim_stores AS
SELECT 
    s.store_id,
    st.first_name || ' ' || st.last_name AS nom_manager,
    'Boutique N°' || s.store_id AS nom_boutique,
    a.address AS adresse,
    a.postal_code AS code_postal,
    ci.city AS ville,
    co.country AS pays
FROM public.store s
LEFT JOIN public.staff st ON s.manager_staff_id = st.staff_id
LEFT JOIN public.address a ON s.address_id = a.address_id
LEFT JOIN public.city ci ON a.city_id = ci.city_id
LEFT JOIN public.country co ON ci.country_id = co.country_id;

-- Ajout de la primary key
ALTER TABLE analytics.dim_stores ADD PRIMARY KEY (store_id);
```

On crée une table de dimension temporelle dédiée grâce à la fonction `generate_series`.
Cette table permet notamment de faire des comparaisons d'une année sur l'autre type YoY.
```sql
DROP TABLE IF EXISTS analytics.dim_date;

CREATE TABLE analytics.dim_date AS
SELECT 
    datum::date AS date_key,
    EXTRACT(YEAR FROM datum)::int AS annee,
    EXTRACT(QUARTER FROM datum)::int AS trimestre,
    EXTRACT(MONTH FROM datum)::int AS mois_numero,
    TO_CHAR(datum, 'TMMonth') AS mois_nom, -- Nom du mois en français (selon la conf de ta DB)
    EXTRACT(DAY FROM datum)::int AS jour_numero,
    EXTRACT(ISODOW FROM datum)::int AS jour_semaine_numero,
    TO_CHAR(datum, 'TMDay') AS jour_semaine_nom, -- Nom du jour en français
    CASE WHEN EXTRACT(ISODOW FROM datum) IN (6, 7) THEN 'Week-end' ELSE 'Semaine' END AS type_jour
FROM generate_series('2005-01-01'::date, '2006-12-31'::date, '1 day'::interval) datum;

ALTER TABLE analytics.dim_date ADD PRIMARY KEY (date_key);
```

Pour créer analytics.dim_films, nous allons faire des LEFT JOIN entre ces quatre tables du schéma public :
- `public.film` (La table centrale)
- `public.film_category` (La table de liaison pour les catégories)
- `public.category` (Pour avoir le nom textuel de la catégorie)
- `public.language` (Pour avoir le nom de la langue au lieu d'un ID)

```sql
DROP TABLE IF EXISTS analytics.dim_films;

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

On crée une table intermédiaire `dim_actors_bridge` pour lier les films et les acteurs. Cela permet un niveau d'approfondissement avec l'application d'un filtre dans le dashboard avec le nom d'un acteur et voir tous ses films.
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


Une table de faits doit être construite au grain le plus fin. Notre grain est : Une transaction de location.
Cependant, dans dvdrental, les dates de location sont dans la table rental, mais l'argent (les montants) est dans la table payment.

Nous allons donc lier :
- `public.rental` : Pour avoir l'ID du film loué via l'inventaire, les dates de départ et de retour.
- `public.inventory` : Table intermédiaire obligatoire dans public pour savoir quel film_id correspond à la location.
- `public.payment` : Pour récupérer le montant exact payé pour cette location.

On va en profiter pour calculer une métrique métier directement en SQL : la durée réelle de rétention du DVD par le client (en jours).

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

<img width="850" height="763" alt="Capture d&#39;écran 2026-06-28 213319" src="https://github.com/user-attachments/assets/74dc140b-82d3-449b-8d5d-6011ac35216b" />

