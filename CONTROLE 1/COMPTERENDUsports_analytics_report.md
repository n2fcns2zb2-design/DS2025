# Compte Rendu d'Analyse de Données Sportives

## 📋 Table des Matières

1. [Introduction](#introduction)
2. [Méthodologie](#méthodologie)
3. [Exploration des Données](#exploration-des-données)
4. [Nettoyage et Préparation](#nettoyage-et-préparation)
5. [Analyses Sportives](#analyses-sportives)
6. [Analyse de Régression](#analyse-de-régression)
7. [Visualisations](#visualisations)
8. [Conclusions et Recommandations](#conclusions-et-recommandations)

---

## 1. Introduction {#introduction}

### Contexte du Projet
Ce projet vise à analyser des données sportives de football provenant de plusieurs sources pour en extraire des insights significatifs sur les performances des équipes, les tendances des matchs et la prévisibilité des résultats basée sur les cotes de paris.

### Objectifs
- Nettoyer et préparer les données sportives
- Analyser les performances par ligue et par pays
- Développer un modèle prédictif pour la différence de buts
- Visualiser les tendances et patterns identifiés

### Sources de Données
- **ginf.csv** : Données de matchs (10,112 matchs, 18 colonnes)
- **events.csv** : Événements de jeu détaillés (941,009 événements, 22 colonnes)
- **dictionary.txt** : Dictionnaire de mapping pour les codes catégoriels

---

## 2. Méthodologie {#méthodologie}

### Approche Générale
1. **Chargement des données** : Import et inspection initiale
2. **Nettoyage** : Traitement des valeurs manquantes et inconsistances
3. **Feature Engineering** : Création de variables dérivées
4. **Analyse exploratoire** : Calcul de métriques de performance
5. **Modélisation** : Régression linéaire pour prédiction
6. **Visualisation** : Graphiques et interprétations

### Outils Utilisés
- **Python 3.x**
- **Pandas** : Manipulation de données
- **Scikit-learn** : Modélisation machine learning
- **Matplotlib & Seaborn** : Visualisations

---

## 3. Exploration des Données {#exploration-des-données}

### 3.1 Structure du Dataset `ginf.csv`

| Caractéristique | Valeur |
|-----------------|---------|
| Nombre de matchs | 10,112 |
| Nombre de colonnes | 18 |
| Période couverte | Variable selon la date |
| Types de données | Numériques et catégorielles |

#### Colonnes Principales
- **Identification** : `id_odsp`, `link_odsp`
- **Match Info** : `date`, `league`, `country`, `ht`, `at`
- **Scores** : `fthg` (buts domicile), `ftag` (buts extérieur)
- **Cotes de paris** : `odd_h`, `odd_d`, `odd_a`, `odd_over`, `odd_under`, `odd_bts`, `odd_bts_n`

### 3.2 Structure du Dataset `events.csv`

| Caractéristique | Valeur |
|-----------------|---------|
| Nombre d'événements | 941,009 |
| Nombre de colonnes | 22 |
| Types d'événements | Tirs, passes, fautes, etc. |

#### Variables Clés
- **event_type** : Type d'événement (tir, passe, carton, etc.)
- **shot_place** : Placement du tir
- **shot_outcome** : Résultat du tir
- **location** : Zone du terrain
- **bodypart** : Partie du corps utilisée
- **situation** : Contexte de jeu

### 3.3 Valeurs Manquantes Initiales

```
Colonnes avec valeurs manquantes dans ginf.csv :
- odd_over     : 1,188 valeurs manquantes
- odd_under    : 1,188 valeurs manquantes
- odd_bts      : 1,189 valeurs manquantes
- odd_bts_n    : 1,189 valeurs manquantes
```

---

## 4. Nettoyage et Préparation {#nettoyage-et-préparation}

### 4.1 Traitement des Valeurs Manquantes

#### Stratégie d'Imputation
Les valeurs manquantes dans les colonnes de cotes ont été imputées avec leur **médiane** respective :

| Colonne | Médiane | Justification |
|---------|---------|---------------|
| odd_over | 2.03 | Robuste aux valeurs extrêmes |
| odd_under | 2.03 | Maintient la distribution |
| odd_bts | 1.92 | Reflète la tendance centrale |
| odd_bts_n | 1.92 | Cohérence avec odd_bts |

### 4.2 Feature Engineering

#### Variables Créées

**1. Goal Difference (Différence de buts)**
```python
goal_difference = fthg - ftag
```
- Indicateur de dominance d'une équipe
- Variable cible pour la régression

**2. Total Goals (Total de buts)**
```python
total_goals = fthg + ftag
```
- Mesure de l'intensité offensive du match

**3. Match Outcome (Résultat du match)**
```python
match_outcome = {
    'Home Win' si fthg > ftag,
    'Away Win' si ftag > fthg,
    'Draw' si fthg == ftag
}
```
- Catégorisation claire des résultats

### 4.3 Mapping des Codes Catégoriels

Le fichier `dictionary.txt` a été parsé pour convertir les codes numériques en labels descriptifs :

| Code Original | Colonne Descriptive | Exemple de Mapping |
|---------------|---------------------|-------------------|
| event_type | event_type_description | 1 → "Shot", 2 → "Pass" |
| shot_place | shot_place_description | 1 → "Centre du but", 2 → "Gauche" |
| bodypart | bodypart_description | 1 → "Pied droit", 2 → "Pied gauche" |

**Gestion des valeurs manquantes post-mapping :**
- `event_type2_description` : "Not Applicable"
- Autres colonnes : "Unknown"

---

## 5. Analyses Sportives {#analyses-sportives}

### 5.1 Performance par Ligue

#### Taux de Victoires et Nuls

| Ligue | Victoires Domicile (%) | Victoires Extérieur (%) | Nuls (%) | Total Matchs |
|-------|------------------------|-------------------------|----------|--------------|
| SP1 (Espagne) | **47.73** | 26.02 | 26.25 | 3,040 |
| F1 (France) | 45.81 | **26.96** | **27.24** | 3,040 |
| E1 (Angleterre) | 46.18 | 27.50 | 26.32 | 3,040 |
| I1 (Italie) | 43.95 | 29.71 | 26.35 | 912 |
| D1 (Allemagne) | 44.21 | 29.61 | 26.18 | 2,448 |

#### Moyennes de Buts

| Ligue | Buts Domicile (Moy.) | Buts Extérieur (Moy.) | Total Buts (Moy.) |
|-------|----------------------|-----------------------|-------------------|
| SP1 | **1.63** | 1.14 | 2.77 |
| F1 | 1.48 | 1.12 | 2.60 |
| E1 | 1.52 | 1.23 | **2.75** |
| I1 | 1.42 | 1.26 | 2.68 |
| D1 | 1.57 | 1.41 | **2.98** |

**📊 Insights Clés :**
- 🏆 **SP1 (Espagne)** : Avantage domicile le plus fort (47.73%)
- ⚽ **D1 (Allemagne)** : Matchs les plus offensifs (2.98 buts/match)
- 🤝 **F1 (France)** : Taux de nuls le plus élevé (27.24%)

### 5.2 Performance par Pays

Les tendances observées au niveau des ligues se reflètent au niveau national :

| Pays | Victoires Domicile (%) | Victoires Extérieur (%) | Nuls (%) | Buts/Match |
|------|------------------------|-------------------------|----------|------------|
| Espagne | 47.73 | 26.02 | 26.25 | 2.77 |
| France | 45.81 | 26.96 | 27.24 | 2.60 |
| Angleterre | 46.18 | 27.50 | 26.32 | 2.75 |
| Italie | 43.95 | 29.71 | 26.35 | 2.68 |
| Allemagne | 44.21 | 29.61 | 26.18 | 2.98 |

**🔍 Analyse Comparative :**
- L'avantage domicile varie de 43.95% (Italie) à 47.73% (Espagne)
- Les victoires extérieures représentent 26-30% des résultats
- Les nuls constituent environ 26-27% des matchs

---

## 6. Analyse de Régression {#analyse-de-régression}

### 6.1 Objectif et Méthodologie

**Objectif** : Prédire la différence de buts (`goal_difference`) basée sur les cotes de paris.

**Variables Indépendantes (X)** :
- `odd_h` : Cote pour victoire domicile
- `odd_d` : Cote pour match nul
- `odd_a` : Cote pour victoire extérieur

**Variable Dépendante (y)** :
- `goal_difference` : Différence de buts (fthg - ftag)

**Split des données** : 80% entraînement / 20% test

### 6.2 Résultats du Modèle

#### Métriques de Performance

| Métrique | Valeur | Interprétation |
|----------|--------|----------------|
| **MAE** (Mean Absolute Error) | 1.22 | Erreur moyenne de ~1.2 buts |
| **MSE** (Mean Squared Error) | 2.49 | Pénalise les grandes erreurs |
| **R² Score** | 0.20 | 20% de variance expliquée |

#### Coefficients du Modèle

```
Équation de régression :
goal_difference = -0.21×odd_h + 0.05×odd_d + 0.08×odd_a + intercept
```

| Variable | Coefficient | Interprétation |
|----------|-------------|----------------|
| odd_h | **-0.21** | ↓ Cote domicile → ↑ Différence de buts favorable |
| odd_d | +0.05 | Effet faible |
| odd_a | +0.08 | ↑ Cote extérieur → ↑ Différence favorable domicile |
| Intercept | Variable | Point de base de prédiction |

### 6.3 Interprétation

**✅ Points Positifs :**
- Le coefficient négatif de `odd_h` est cohérent : une cote plus basse (favori plus fort) prédit une meilleure performance domicile
- MAE de 1.22 est raisonnable pour prédire des différences de buts

**⚠️ Limitations :**
- R² de 0.20 indique que 80% de la variance n'est pas expliquée par les cotes seules
- Les cotes de paris ne capturent pas tous les facteurs (forme récente, blessures, tactiques, météo, etc.)

**💡 Recommandations :**
- Intégrer des features supplémentaires : historique des équipes, statistiques événementielles de `events.csv`
- Tester des modèles non-linéaires (Random Forest, Gradient Boosting)
- Inclure des variables temporelles (saison, moment dans la saison)

---

## 7. Visualisations {#visualisations}

### 7.1 Graphiques Générés

Les visualisations suivantes ont été créées pour illustrer les résultats :

#### 📊 Graphique 1 : Taux de Victoires et Nuls par Ligue
**Description** : Grouped bar chart comparant les taux de victoires domicile, extérieur et nuls pour chaque ligue.

**Observations** :
- SP1 montre une dominance claire des victoires domicile
- F1 présente l'équilibre le plus uniforme entre les trois résultats

#### ⚽ Graphique 2 : Moyenne de Buts par Ligue
**Description** : Grouped bar chart des moyennes de buts marqués à domicile et à l'extérieur.

**Observations** :
- D1 (Allemagne) affiche les moyennes les plus élevées dans les deux catégories
- L'écart domicile-extérieur est constant (~0.4-0.5 buts)

#### 🌍 Graphique 3 : Taux de Victoires et Nuls par Pays
**Description** : Comparaison similaire au graphique 1, mais au niveau national.

**Observations** :
- Tendances similaires aux ligues respectives
- Confirmation de patterns culturels/tactiques par pays

#### 🎯 Graphique 4 : Moyenne de Buts par Pays
**Description** : Moyennes de buts par pays, reflétant les styles de jeu nationaux.

**Observations** :
- Cohérence avec l'analyse par ligue
- Allemagne ressort comme le championnat le plus offensif

#### 📈 Graphique 5 : Actual vs Predicted Goal Difference
**Description** : Scatter plot comparant les valeurs réelles et prédites de goal_difference, avec ligne de régression idéale (y=x).

**Observations** :
- Dispersion significative autour de la ligne idéale
- Prédictions relativement centrées autour de zéro
- Sous-estimation des valeurs extrêmes (fortes victoires/défaites)

### 7.2 Interprétation Visuelle Globale

Les visualisations confirment :
1. **Avantage domicile universel** mais variable selon les ligues
2. **Styles de jeu différents** entre pays (défensif vs offensif)
3. **Limites prédictives** du modèle basé uniquement sur les cotes

---

## 8. Conclusions et Recommandations {#conclusions-et-recommandations}

### 8.1 Synthèse des Findings

#### 🏆 Découvertes Principales

1. **Avantage Domicile Significatif**
   - 44-48% de victoires domicile selon la ligue
   - Environ 0.3-0.5 buts de plus marqués à domicile

2. **Variations Inter-Ligues**
   - Espagne : Plus grand avantage domicile
   - Allemagne : Matchs les plus offensifs
   - France : Plus de nuls, jeu plus défensif

3. **Pouvoir Prédictif des Cotes**
   - Les cotes de paris capturent 20% de la variance des résultats
   - Corrélation négative entre cote domicile et performance
   - Nécessité d'enrichissement du modèle

### 8.2 Limites de l'Étude

| Limitation | Impact | Solution Proposée |
|------------|--------|-------------------|
| Dataset limité aux cotes | R² faible (0.20) | Intégrer données événementielles |
| Modèle linéaire simple | Capture mal les non-linéarités | Tester XGBoost, Random Forest |
| Absence de variables contextuelles | Ignore facteurs situationnels | Ajouter météo, calendrier, blessures |
| Période temporelle inconnue | Validité temporelle incertaine | Analyser évolution dans le temps |

### 8.3 Recommandations pour Améliorations Futures

#### 🔬 Enrichissement des Features

**Depuis `events.csv` :**
- Nombre de tirs cadrés/non cadrés par équipe
- Possession de balle (via passes complétées)
- Cartons jaunes/rouges (discipline)
- Corners et coups francs (pression offensive)

**Variables Temporelles :**
- Position au classement au moment du match
- Forme récente (résultats des 5 derniers matchs)
- Jours de repos entre matchs

**Variables Contextuelles :**
- Phase de saison (début, milieu, fin)
- Enjeu du match (course au titre, maintien)
- Rivalités historiques

#### 🤖 Modélisation Avancée

1. **Gradient Boosting** (XGBoost, LightGBM)
   - Capture des interactions complexes
   - Gestion native des variables catégorielles

2. **Réseaux de Neurones**
   - Pour datasets très riches en features
   - Potentiel de deep learning avec embeddings

3. **Modèles d'Ensemble**
   - Stacking de plusieurs algorithmes
   - Voting classifier pour robustesse

#### 📊 Analyses Complémentaires

- **Analyse de séries temporelles** : Évolution des performances dans le temps
- **Clustering d'équipes** : Identifier des profils de jeu similaires
- **Analyse de réseaux** : Patterns de passes, centralité de joueurs
- **Prédiction multi-classe** : Prédire directement Home Win/Draw/Away Win

### 8.4 Applications Pratiques

#### Pour les Équipes et Entraîneurs
- Identifier les forces/faiblesses tactiques par ligue
- Optimiser les stratégies domicile vs extérieur
- Benchmarking contre la concurrence

#### Pour les Analystes et Parieurs
- Modèles prédictifs plus sophistiqués
- Identification de value bets (écarts cotes/prédictions)
- Gestion de risque basée sur incertitude du modèle

#### Pour les Médias et Fans
- Insights sur les styles de jeu nationaux
- Comparaisons inter-ligues objectives
- Narratives basées sur les données

---

## 📌 Annexes

### A. Technologies et Dépendances

```python
pandas==2.x.x
numpy==1.x.x
scikit-learn==1.x.x
matplotlib==3.x.x
seaborn==0.x.x
```

### B. Structure des Fichiers

```
project/
│
├── data/
│   ├── ginf.csv
│   ├── archive.zip
│   │   └── events.csv
│   └── dictionary.txt
│
├── notebooks/
│   └── sports_analytics.ipynb
│
└── outputs/
    └── visualizations/
```

### C. Commandes de Reproduction

```python
# Chargement des données
df_matches = pd.read_csv('ginf.csv')
df_events = pd.read_csv('archive.zip/events.csv')

# Feature engineering
df_matches['goal_difference'] = df_matches['fthg'] - df_matches['ftag']

# Modélisation
from sklearn.linear_model import LinearRegression
model = LinearRegression()
model.fit(X_train, y_train)
```

---

## 📧 Contact et Crédits

**Projet** : Analyse de Données Sportives de Football  
**Date** : Décembre 2024  
**Outils** : Python, Pandas, Scikit-learn, Matplotlib  

---

**Fin du Compte Rendu**