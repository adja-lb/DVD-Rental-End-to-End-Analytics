# 📊 Dashboard Décisionnel avec Power BI

## 📌 Présentation du Projet
Ce projet consiste en la création d’un écosystème de Business Intelligence de bout en bout basé sur la base de données standardisée **`dvdrental`**. L'objectif était de transformer des données transactionnelles brutes (PostgreSQL) en un outil de pilotage stratégique sous **Power BI**, structuré autour de problématiques concrètes : la rentabilité du catalogue, l'optimisation des flux logistiques et le suivi de la croissance de la base client.

---

## 🛠️ Architecture Technique & Modélisation
Pour garantir la performance des calculs DAX et la scalabilité des rapports, les données ont été modélisées selon une **architecture en étoile (Star Schema)** :
*   **Table de faits :** `fact_rentals` (centralisation des indicateurs transactionnels de location).
*   **Dimensions :** `dim_customers`, `dim_films` (scindée pour isoler la granularité des stocks) et une table de temps unifiée `dim_date` pour le filtrage temporel continu.
*   **Data Prep :** Nettoyage rigoureux des formats `Timestamp` à la source dans Power Query pour éliminer les conflits de granularité horaire et standardiser les filtres temporels.

---

## 📈 Structure du Dashboard & Insights Business

Le rapport est articulé autour de 3 pages stratégiques conçues pour répondre aux besoins de différents départements :

### 1. Executive Summary & Croissance (Vue Direction)
*   **Suivi de l'acquisition :** Implémentation d'une mesure de flux mensuel net (`Nouveaux_Clients_Flux`) utilisant des relations virtuelles en DAX (`Cross-Filtering`) pour isoler et afficher les cohortes de nouveaux clients mois par mois sans biais historique.
*   **Analyse du cycle de vie :** Visualisation combinée du flux d'acquisition mensuel et de la courbe de croissance cumulée globale de la base clients.

### 2. Performance & Optimisation du Catalogue (Vue Ventes)
*   **Analyse Quadrant (Scatter Chart) :** Cartographie du catalogue par volume de location vs. prix de vente pour isoler instantanément le *Dead Stock* et les genres *Sous-évalués*.
*   **Scénarisation d'impact (What-If Parameters) :** Création d'un simulateur de prix dynamique permettant aux équipes commerciales de tester l'impact financier d'une hausse ciblée des prix (ex: +10% sur les catégories à fort volume et bas coût) tout en préservant les "Films Stars" via une segmentation DAX avancée (`AVERAGEX` + `ALL`).

### 3. Gestion des Risques & Logistique (Vue Opérations)
*   **Alerte "Pertes de Stock" :** Modélisation d'une mesure de risque logistique (`Nb_Films_Non_Rendus_Alerte`) calculant de manière dynamique la différence entre la date de location, la durée maximale autorisée par film (`RELATED`) et la date pivot, avec un déclenchement d'alerte automatique à $X + 30$ jours.
*   **Analyse de la Rotation :** Mise en évidence d'une anomalie de rendement (volumes de détention identiques entre les locations courtes de 1 jour et longues de 9 jours), débouchant sur des recommandations d'optimisation tarifaire dégressive.

---

## 🎨 Industrialisation & Design System (Gouvernance BI)
Pour standardiser le reporting, gagner en clarté visuelle et accélérer les futurs développements, un **Design System** complet a été intégré :
*   **Thème JSON personnalisé :** Centralisation de la charte visuelle (polices d'entreprise, suppression des quadrillages encombrants pour alléger la charge cognitive, ombrages légers et coins arrondis à 8px pour un effet d'interface moderne).
*   **Sub-branding sémantique :** Attribution d'un code couleur d'accentuation dédié par équipe (Bleu pour la performance globale, Vert pour le catalogue, Rouge pour les risques logistiques) afin que l'utilisateur identifie instantanément son contexte métier à l'écran.

---

## 🧠 Compétences Clefs Démontrées
*   **Modélisation Décisionnelle :** Schéma en étoile, gestion des dimensions à granularité fine.
*   **DAX Avancé :** Itérateurs temporels (`SUMX`, `AVERAGEX`), manipulation de contextes de filtres complexes (`CALCULATE`, `FILTER`, `ALL`, `VALUES`), relations virtuelles.
*   **Data Storytelling & UX/UI :** Industrialisation de rapports (JSON), conception centrée utilisateur, clarté visuelle (Pixel Perfect).
