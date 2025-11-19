# 📄 Compte rendu – Base de données *Wine Quality (White Wine)*

## 🏷️ 1. Présentation générale

La base de données *Wine Quality* (vin blanc) provient de l’UCI Machine Learning Repository.  
Elle contient des mesures **physico-chimiques** de vins blancs portugais, ainsi qu’un **score de qualité** attribué par des experts.

Elle est destinée à des tâches de **régression** (prédire la qualité) ou de **classification** (qualité bonne ou mauvaise).

---

## 📊 2. Dimensions du dataset

À partir du chargement via `pandas` :

- **Nombre d’échantillons :** 4 898  
- **Nombre de variables explicatives (features) :** 11  
- **Variable cible :** `quality` (entier de 0 à 10)

---

## 🍇 3. Variables disponibles

| Nom de la variable          | Description |
|-----------------------------|-------------|
| fixed acidity               | Acidité fixe (acides non volatils) |
| volatile acidity            | Acidité volatile (acide acétique) |
| citric acid                 | Acide citrique |
| residual sugar              | Sucres résiduels |
| chlorides                   | Chlorures |
| free sulfur dioxide         | SO₂ libre |
| total sulfur dioxide        | SO₂ total |
| density                     | Densité du vin |
| pH                          | Niveau d’acidité pH |
| sulphates                   | Sulfates |
| alcohol                     | Teneur en alcool |
| quality (variable cible)    | Note de qualité (0–10) |

---

## 🏷️ 4. Préparation du problème de classification

La variable `quality` est initialement une note de 0 à 10.  
Le script construit un problème de **classification binaire** :

- **Classe 0 – vin de mauvaise qualité** : `quality ≤ 5`  
- **Classe 1 – vin de bonne qualité** : `quality ≥ 6`

---

## 📈 5. Analyse statistique initiale

Deux types d’analyses ont été réalisées :

### 🔹 5.1. Boîtes à moustaches (boxplots)

Elles permettent de visualiser :

- la distribution de chaque variable,
- les éventuels outliers,
- les différences d’échelle entre les features.

### 🔹 5.2. Matrice de corrélation

Le heatmap permet d’observer :

- des corrélations fortes (ex. entre **density**, **residual sugar**, et **total sulfur dioxide**),
- des corrélations globalement faibles avec la variable cible.

---

## 🔁 6. Split des données

Le dataset est divisé en trois sous-ensembles :

- **Train (≈33%)**
- **Validation (≈33%)**
- **Test (≈33%)**

Le découpage est :

- **stratifié** (conservation des proportions de classes),
- **shuffle** (mélange aléatoire).

---

## 🤖 7. Modèle de classification : k-NN

Un classifieur **k-Nearest Neighbors (k-NN)** a été appliqué.

### 🔹 Étapes :

1. Apprentissage pour différents `k` (1, 3, 5, ..., 35)
2. Calcul de :
   - l’erreur d’entraînement,
   - l’erreur de validation
3. Sélection du meilleur `k` (minimisant l’erreur de validation)
4. Test sur le jeu de test

### Observations :

- **Overfitting** pour les petits `k` (1, 3…)
- **Meilleure généralisation** pour des `k` modérés

---

## 🔧 8. Normalisation

Une normalisation **StandardScaler** a été effectuée :

- centrage des données (moyenne = 0),
- réduction (écart-type = 1).

Importance :

- indispensable pour k-NN car dépend des distances,
- doit être ajustée **uniquement** sur le train, puis appliquée au validation/test.

---

## ✅ 9. Conclusion générale

Ce dataset est utile pour :

- l’analyse exploratoire,
- la classification supervisée,
- l’étude de l’impact de la normalisation,
- la compréhension de l’overfitting.

Le modèle k-NN montre que :

- La performance dépend fortement du choix de `k`
- La normalisation améliore sensiblement les résultats
- La stratification est essentielle pour un split fiable

---


