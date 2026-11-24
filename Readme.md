# Météo, énergies renouvelables et système électrique français (2020–2025)

**Projet de Python pour la Data Science**  
**Équipe : Elyes, Melchior et Romain**

## Présentation

Ce projet étudie les relations entre les conditions météorologiques, la production d'électricité renouvelable et la consommation électrique en France, sur la période **2020–2025**.

**Problématique :** Dans quelle mesure la température, le vent et l'ensoleillement sont-ils associés à la production éolienne et solaire, à la consommation et à la charge résiduelle du système électrique français ?

Nous utilisons deux jeux de données historiques, RTE et Open-Meteo, pour construire une analyse exploratoire reproductible avec Python. Le projet **ne porte pas sur les prix de marché** : aucune série de prix de l'électricité n'est utilisée dans les analyses actuelles.

## Sources de données

### RTE — éCO2mix

**Source :** [ODRÉ / RTE](https://odre.opendatasoft.com/)

Données nationales de production, de consommation et d'échanges d'électricité. Les variables exploitées comprennent :

- `consommation` : consommation électrique (MW) ;
- `eolien` et `solaire` : production éolienne et solaire (MW) ;
- `nucleaire`, `hydraulique` et `gaz` : autres productions électriques (MW) ;
- `ech_physiques` : solde des échanges physiques (MW) ;
- `charge_residuelle_mw` : consommation moins production éolienne et solaire (MW).

Les valeurs de consommation et de production exploitables sont principalement **demi-horaires**. Le fichier brut comporte également des lignes à :15 et :45 dont les variables principales sont manquantes.

### Open-Meteo — données historiques

**Source :** [Open-Meteo](https://open-meteo.com/)

Données météorologiques historiques **horaires** issues ici du modèle de réanalyse **ERA5**, récupérées via Open-Meteo pour **Brest, Paris et Marseille** :

- `wind_speed_100m` : vitesse du vent à 100 mètres (m/s) ;
- `shortwave_radiation` : rayonnement solaire (W/m²) ;
- `temperature_2m` : température à 2 mètres (°C).

La moyenne simple des trois villes est utilisée dans le notebook de croisement météo–électricité comme **indicateur météorologique approximatif**. Elle ne doit pas être interprétée comme une véritable moyenne météorologique nationale représentative de la France.

## Données et préparation

Les résultats ci-dessous correspondent aux **sorties enregistrées dans les notebooks fournis**, sur la période 2020–2025 :

| Étape | Nombre d'observations |
|---|---:|
| Fichier RTE brut | 210 408 lignes |
| RTE après suppression des lignes sans consommation | 105 203 demi-heures |
| Fichier météo, toutes villes confondues | 157 824 lignes |
| Table RTE–météo après jointure sur les heures communes | 52 601 heures |

Le notebook de croisement :

1. Supprime les lignes RTE sans consommation.
2. Conserve les mesures RTE aux **heures pleines** (`minute == 0`) ; il ne calcule pas encore une moyenne des deux demi-heures.
3. Calcule, pour chaque heure, la moyenne non pondérée des mesures météo des trois villes.
4. Effectue une jointure interne sur l'horodatage UTC.
5. Ramène les valeurs de production solaire négatives à zéro dans la table fusionnée.
6. Crée les indicateurs suivants :
   - **Production éolienne et solaire :** `enr_total = eolien + solaire` ;
   - **Part de la consommation couverte par ces deux filières :** `part_enr = 100 × enr_total / consommation`.

Le nom `enr_total` désigne ici **uniquement l'éolien et le solaire**, et non la totalité des productions renouvelables (l'hydraulique est notamment exclue).

La **charge résiduelle** est définie par :

$$
\text{Charge résiduelle (MW)} = \text{Consommation} - \text{Éolien} - \text{Solaire}
$$

## Analyses réalisées

### 1. `analyse_rte(1).ipynb` — exploration du système électrique

Ce notebook réalise :

- la lecture de RTE, la conversion des dates vers le fuseau `Europe/Paris` et la création de variables calendaires ;
- le nettoyage des lignes sans consommation ;
- le **profil moyen de consommation par heure** et sa **saisonnalité mensuelle** ;
- la comparaison des productions **éolienne et solaire par mois** ;
- l'étude de la **distribution de la charge résiduelle** ;
- l'identification des **15 demi-heures présentant les plus faibles charges résiduelles** dans l'échantillon.

### 2. `exploration_meteo.ipynb` — croisement météo et électricité

Ce notebook réalise :

- la préparation et la fusion des données RTE et Open-Meteo ;
- des **statistiques descriptives** (moyennes, écarts-types, quartiles, minima, maxima) ;
- une **matrice de corrélation** entre variables météorologiques et électriques ;
- des graphiques de production éolienne selon la vitesse du vent et de production solaire selon le rayonnement (hors nuit) ;
- une analyse descriptive de la **consommation, du nucléaire et du gaz selon la température** ;
- une **carte thermique** de la part de la consommation couverte par l'éolien et le solaire selon des classes de vent et de rayonnement.

Les graphiques sont affichés dans les notebooks. Leur export systématique en fichiers image n'est pas encore mis en place.

## Premiers résultats exploratoires

Sur les **52 601 observations horaires fusionnées**, les corrélations linéaires calculées dans le notebook météo sont notamment :

| Variables | Corrélation de Pearson |
|---|---:|
| Vitesse du vent / production éolienne | **0,794** |
| Rayonnement solaire / production solaire | **0,878** |
| Température / consommation électrique | **−0,589** |
| Température / charge résiduelle | **−0,650** |

Les résultats mettent en évidence des associations entre conditions météorologiques et fonctionnement du système électrique. **Une corrélation ne démontre pas à elle seule un effet causal** : les saisons, les heures de la journée et les évolutions du parc de production peuvent intervenir dans ces relations.

Le notebook RTE fait aussi apparaître une consommation plus élevée en hiver et des profils saisonniers différents pour la production solaire et la production éolienne.

## Installation et exécution

### Prérequis

- Python 3 ;
- les dépendances du fichier `requirements.txt` (notamment `pandas`, `numpy`, `matplotlib` et `seaborn`) ;
- une connexion Internet pour actualiser les données.

Créer un environnement virtuel :

```bash
python -m venv .venv
```

L'activer sous Windows (CMD) :

```bat
.venv\Scripts\activate
```

Ou sous macOS / Linux :

```bash
source .venv/bin/activate
```

Installer ensuite les dépendances :

```bash
python -m pip install -r requirements.txt
```

### Télécharger les données

Depuis la racine du projet :

```bash
python collecte_historique_sans_entsoe.py
```

Les fichiers utilisés par les notebooks sont :

- `data/df_rte.csv` ;
- `data/df_meteo.csv`.

**Aucune clé API n'est requise.** Les données sont conservées séparément avant les analyses.

### Exécuter les notebooks

Ouvrir les notebooks avec VS Code ou Jupyter. **Attention aux chemins relatifs actuellement présents dans le code :**

- `analyse_rte(1).ipynb` lit `data/df_rte.csv` et suppose que le répertoire de travail est la **racine du projet** ;
- `exploration_meteo.ipynb` lit `../data/df_rte.csv` et `../data/df_meteo.csv` et suppose que le répertoire de travail est le dossier **`notebooks/`**.

Vérifier le répertoire de travail de Jupyter avant d'exécuter toutes les cellules.

## Limites et améliorations possibles

- **Représentativité météo :** trois villes ne suffisent pas à résumer précisément l'ensemble du territoire français. Un échantillonnage spatial plus large ou pondéré améliorerait l'analyse.
- **Harmonisation horaire :** la fusion actuelle retient uniquement les observations RTE aux heures pleines. Une moyenne des demi-heures, accompagnée d'un contrôle des périodes manquantes, serait préférable.
- **Fuseaux horaires :** le notebook RTE travaille en heure de Paris, tandis que la fusion météo utilise l'UTC. Certains commentaires ou libellés du notebook RTE doivent être harmonisés avec ce choix.
- **Classes de rayonnement :** les quartiles construits avec un classement des valeurs peuvent séparer arbitrairement des heures ayant un rayonnement nul ; les étiquettes « nuit », « couvert », etc. doivent donc être interprétées avec prudence.
- **Interprétation :** les analyses sont descriptives ; elles ne permettent ni d'établir une causalité ni de déduire directement des prix négatifs ou une saturation du réseau.

Les prochaines pistes de travail sont l'amélioration de la fusion temporelle, les comparaisons entre années et saisons, ainsi qu'une analyse statistique tenant compte des effets calendaires.

## Reproductibilité

Le code et les notebooks sont versionnés avec Git. Les données sont issues de RTE et Open-Meteo et peuvent être téléchargées à nouveau à l'aide du script de collecte. Les CSV volumineux peuvent être conservés localement et exclus du dépôt via `.gitignore`.

## Auteurs

**Elyes · Melchior · Romain**  
Projet réalisé dans le cadre du cours **Python pour la Data Science**.

