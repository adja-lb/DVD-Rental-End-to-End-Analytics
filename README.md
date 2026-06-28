# 🎬 DVD Rental - End-to-End Analytics

## 📌 Présentation du Projet
L'objectif de ce projet est de transformer une base de données transactionnelle brute (OLTP) d'un réseau de magasins de location de DVD (`dvdrental`) en un **modèle de données sémantique décisionnel (OLAP) en étoile**. 

En adoptant une approche **Full-Stack Data Analyst / Analytics Engineer**, ce projet couvre l'intégralité du pipeline data : de l'isolation de l'infrastructure sous **Docker (PostgreSQL)**, à la modélisation dimensionnelle selon la méthodologie **Kimball**, jusqu'à la création d'un tableau de bord décisionnel dans **Power BI**.

---

## 🏗️ Architecture du Pipeline & Flux de Données

Le projet respecte l'isolation des environnements via des schémas SQL distincts pour garantir la robustesse du système :
```
[ Données Brutes (OLTP) ]     ──> Schéma public (Source de vérité / Inchangé)
        │
        │                           (Transformations SQL Complexes)
        ▼ 
[ Modèle en Étoile (OLAP) ]   ──> Schéma analytics (Tables Dimensions & Faits)
        │
        │                           (Connexion Directe Optimisée)
        ▼  
[ Restitution ]               ──> Power BI Desktop (Modèle Sémantique & DAX)
```
---

## 🎯 Problématique Business & Questions Clés
Le management de l'entreprise souhaite optimiser la gestion des stocks de films et identifier les leviers de croissance des revenus. Le projet répond à 3 questions stratégiques :
1. **Performance Catalogue :** Quelles catégories de films et quels acteurs génèrent le plus de valeur ?
2. **Analyse Géographique :** Où se situent nos clients les plus rentables et comment se répartissent les revenus par magasin ?
3. **Saisonnalité & Comportement :** Quels sont les cycles temporels de location (mois, jours de la semaine) et les délais moyens de restitution ?

---

## 🛠️ Stack Technique
* **Infrastructure :** Docker & PostgreSQL
* **IDE / Gestion DB :** DataGrip & DBeaver
* **Modélisation :** SQL (Création de Vues et Tables Dimensionnelles)
* **Visualisation :** Power BI (Modélisation en étoile, Mesures DAX)

---

## 📐 Modélisation Dimensionnelle (Méthodologie Kimball)

Pour casser la complexité des jointures d'origine (Flocon étendu), le schéma `analytics` centralise la donnée autour d'un **Grain unique : Une transaction de location**.

### Le Schéma en Étoile mis en œuvre :
* **Table de Faits :** `fact_rentals` (Mesures de revenus, durées de location, compteurs)
* **Tables Dimensions :**
  * `dim_customers` : Profil complet du client (Identité, email, adresse, ville, pays).
  * `dim_films` : Attributs des films (Titre, description, catégorie, langue, tarif).
  * `dim_stores` : Performance des boutiques et des managers.
  * `dim_date` : Axe temporel (Année, trimestre, mois, jour de la semaine) pour l'analyse des tendances.

---

## 🚀 Structure du Repository & Avancement

- [x] **Étape 1 :** Configuration de l'environnement conteneurisé Postgres sous Docker.
- [x] **Étape 2 :** Restauration de la base source binaire (`dvdrental.tar`) et isolation via le schéma `public`.
- [ ] **Étape 3 :** Écriture des scripts de transformation SQL et création du schéma `analytics`.
- [ ] **Étape 4 :** Connexion optimisée à Power BI et configuration du modèle sémantique.
- [ ] **Étape 5 :** Design du Dashboard (Optimisation de la charge cognitive & UI/UX) et calcul du ROI fictif.

---

## 📈 Impact Business & Livrables Visés (KPIs)
Le dashboard final mettra en évidence :
* Le **Chiffre d'Affaires Global** et le **Panier Moyen** par location.
* Le **Taux de rotation des films** (quelles références dorment sur les étagères).
* Une **matrice de décisions** pour le département des achats (quels genres de films acheter en priorité pour maximiser le ROI).
