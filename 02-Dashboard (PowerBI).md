# :chart_with_upwards_trend: Dashboard avec Power BI

Pour rappel, notre projet tente de répondre à 3 questions stratégiques :
1. **[Performance Catalogue](performance-catalogue) :** Quelles catégories de films et quels acteurs génèrent le plus de valeur ?

3. **[Analyse Géographique](analyse-géographique) :** Où se situent nos clients les plus rentables et comment se répartissent les revenus par magasin ?

4. **[Saisonnalité & Comportement](saisonnalité-&-comportement) :** Quels sont les cycles temporels de location (mois, jours de la semaine) et les délais moyens de restitution ?


## Formules DAX
Afin de compléter l'analyse, nous allons implémenter de nouvelles mesures dans la table des faits `fact_rentals`:
- Chiffre d'Affaires : `Chiffre_Affaires = SUM(fact_rentals[montant_paye])`
- Volume des ventes : `Nombre_Locations = COUNT(fact_rentals[rental_id])`
- Panier moyen : `Panier_Moyen = DIVIDE([Chiffre_Affaires], [Nombre_Locations], 0)`

## :vhs: 1. Performance Catalogue

### Q1

### Q2

## :earth_asia: 2. Analyse Géographique

### Q1

### Q2

## ⛅ 3. Saisonnalité & Comportement

### Graphique de tendance (Saisonnalité par mois)

### Filtre temporel dynamique (Slicer)

### Analyse des pics d'activité (Le jour de la semaine)
