# Météo, énergies renouvelables et système électrique français (2020–2025)

**Projet de Python pour la Data Science**

**Équipe : Elyes, Melchior et Romain**

## Présentation

Ce projet étudie les relations entre les conditions météorologiques, la production d'électricité renouvelable et la consommation électrique en France, sur la période **2020–2025**.

**Problématique :** Dans quelle mesure la température, le vent et l'ensoleillement sont-ils associés à la production éolienne et solaire, à la consommation et à la charge résiduelle du système électrique français ?

Nous utilisons deux jeux de données historiques, RTE et Open-Meteo, pour construire une analyse exploratoire reproductible avec Python.

Le projet ne porte pas sur les prix de marché : aucune série de prix de l'électricité n'est utilisée dans les analyses actuelles.

## Sources de données

### RTE — éCO2mix

**Source :** https://odre.opendatasoft.com/

RTE fournit des données nationales de production, de consommation et d'échanges d'électricité.

Les variables utilisées comprennent :

- `consommation` : consommation électrique (MW).
- `eolien` et `solaire` : production éolienne et solaire (MW).
- `nucleaire`, `hydraulique` et `gaz` : autres productions électriques (MW).
- `ech_physiques` : solde des échanges physiques (MW).
- `charge_residuelle_mw` : consommation moins production éolienne et solaire (MW).

Les observations exploitables sont principalement demi-horaires.

### Open-Meteo — données météorologiques historiques

**Source :** https://open-meteo.com/

Nous utilisons des données horaires issues du modèle de réanalyse ERA5, récupérées via Open-Meteo pour trois villes françaises : **Brest, Paris et Marseille**.

Les variables utilisées sont :

- `wind_speed_100m` : vitesse du vent à 100 mètres (m/s).
- `shortwave_radiation` : rayonnement solaire (W/m²).
- `temperature_2m` : température à 2 mètres (°C).

Pour certaines analyses, nous calculons la moyenne des valeurs météorologiques des trois villes. Cette moyenne constitue un indicateur simplifié et ne représente pas exactement les conditions météorologiques de l'ensemble du territoire français.

## Données et préparation

Les résultats suivants correspondent aux sorties obtenues dans nos notebooks sur la période **2020–2025**.

| Étape | Nombre d'observations |
|---|---:|
| Base RTE initiale | 210 408 |
| Base RTE après nettoyage | 105 203 |
| Base météorologique (3 villes) | 157 824 |
| Base horaire fusionnée RTE–météo | 52 601 |

### Nettoyage des données RTE

Le fichier RTE initial contient des observations manquantes, notamment aux quarts d'heure :15 et :45.

Nous supprimons les lignes pour lesquelles la consommation est manquante afin de conserver les observations exploitables à la demi-heure.

### Fusion avec les données météorologiques

Pour croiser les deux sources, nous :

1. Conservons les observations RTE aux heures pleines.
2. Calculons la moyenne horaire des données météorologiques des trois villes.
3. Réalisons une jointure sur les dates communes en UTC.
4. Remplaçons les valeurs négatives de production solaire par zéro dans la table fusionnée.
5. Construisons des indicateurs complémentaires.

La base fusionnée contient **52 601 observations horaires**.

### Indicateurs construits

**Production éolienne et solaire :**

`enr_total = eolien + solaire`

**Part de la consommation couverte par ces deux filières :**

`part_enr = 100 × enr_total / consommation`

**Charge résiduelle :**

`charge_residuelle_mw = consommation - eolien - solaire`

La variable `enr_total` désigne uniquement l'éolien et le solaire, et non l'ensemble des énergies renouvelables.

La charge résiduelle permet d'étudier la consommation qui n'est pas directement couverte par la production éolienne et solaire.

## Analyses réalisées

### 1. Analyse du système électrique — `analyse_rte(1).ipynb`

Ce notebook est consacré à l'exploration des données électriques françaises.

**Préparation des données**

- Conversion des horodatages dans le fuseau horaire `Europe/Paris`.
- Création de variables calendaires : année, mois, jour de la semaine et heure.
- Identification et suppression des lignes sans consommation.

**Dynamique de la consommation**

- Calcul du profil moyen de consommation selon l'heure de la journée.
- Analyse de la consommation moyenne par mois.
- Identification des variations journalières et saisonnières.

**Production renouvelable**

- Comparaison de la production éolienne et solaire selon le mois.
- Étude de leurs profils saisonniers respectifs.

**Charge résiduelle**

- Visualisation de sa distribution.
- Identification des périodes de faible charge résiduelle.
- Extraction des 15 demi-heures présentant les plus faibles valeurs dans l'échantillon.

Ces analyses permettent de mieux comprendre les variations de la demande et de la production électrique au cours du temps.

### 2. Analyse météorologique — `exploration_meteo.ipynb`

Ce notebook étudie les relations entre les conditions météorologiques et les variables électriques.

**Préparation et fusion**

- Lecture des bases RTE et Open-Meteo.
- Nettoyage et harmonisation des horodatages.
- Fusion des observations horaires.
- Construction des variables `enr_total` et `part_enr`.

**Statistiques descriptives**

- Moyennes et écarts-types.
- Valeurs minimales et maximales.
- Quartiles.
- Matrice de corrélation entre les variables météorologiques et électriques.

**Relations entre météo et production renouvelable**

- Production éolienne selon la vitesse du vent.
- Production solaire selon le rayonnement solaire, hors périodes nocturnes.
- Visualisation des moyennes de production par classes météorologiques.

**Température et consommation**

- Étude de la consommation moyenne selon la température.
- Comparaison avec les productions nucléaires et au gaz.
- Analyse descriptive de la thermosensibilité de la demande électrique.

**Conditions météorologiques et renouvelables**

- Classification des observations selon le vent et le rayonnement.
- Création d'une carte thermique représentant la part de la consommation couverte par l'éolien et le solaire selon les conditions météorologiques.

Les graphiques sont actuellement affichés directement dans les notebooks.

## Premiers résultats

L'analyse de la base fusionnée met en évidence plusieurs relations statistiques.

| Variables étudiées | Corrélation de Pearson |
|---|---:|
| Vent et production éolienne | 0,794 |
| Rayonnement solaire et production solaire | 0,878 |
| Température et consommation | −0,589 |
| Température et charge résiduelle | −0,650 |

Ces résultats suggèrent :

- Une association positive importante entre la vitesse du vent et la production éolienne.
- Une association positive importante entre le rayonnement solaire et la production photovoltaïque.
- Une association négative entre la température et la consommation électrique sur la période étudiée.
- Une association négative entre la température et la charge résiduelle.

L'analyse RTE révèle également des variations journalières et saisonnières marquées de la consommation, ainsi que des profils saisonniers différents pour les productions éolienne et solaire.

Ces résultats sont **descriptifs**. Ils ne permettent pas, à eux seuls, d'établir des relations causales.

## Installation et exécution

### Prérequis

- Python 3.
- Les bibliothèques nécessaires, notamment `pandas`, `numpy`, `matplotlib` et `seaborn`.
- Une connexion Internet pour actualiser les données.

### Installation

Créer un environnement virtuel :

```bash
python -m venv .venv
```

Sous Windows, l'activer avec :

```bat
.venv\Scripts\activate
```

Sous macOS ou Linux :

```bash
source .venv/bin/activate
```

Installer ensuite les dépendances :

```bash
python -m pip install -r requirements.txt
```

### Collecte des données

Depuis la racine du projet :

```bash
python collecte_historique_sans_entsoe.py
```

Le script récupère les données RTE et Open-Meteo sur la période étudiée.

Les fichiers principaux sont :

- `data/df_rte.csv`
- `data/df_meteo.csv`

Aucune clé API personnelle n'est nécessaire.

### Exécution des notebooks

Les analyses peuvent être exécutées dans VS Code ou Jupyter.

Les chemins relatifs actuellement utilisés sont différents selon les notebooks :

- `analyse_rte(1).ipynb` utilise `data/df_rte.csv` et suppose un répertoire de travail correspondant à la racine du projet.
- `exploration_meteo.ipynb` utilise `../data/df_rte.csv` et `../data/df_meteo.csv` et suppose un répertoire de travail correspondant au dossier `notebooks/`.

Il faut donc vérifier le répertoire de travail Jupyter avant l'exécution.

## Limites et améliorations possibles

### Représentativité météorologique

Les données proviennent uniquement de trois villes. Leur moyenne ne représente pas précisément les conditions météorologiques sur l'ensemble du territoire français.

Une extension à davantage de points géographiques permettrait d'améliorer cette approximation.

### Harmonisation temporelle

La fusion actuelle utilise les observations RTE aux heures pleines, plutôt que la moyenne des deux demi-heures.

Une harmonisation plus rigoureuse et un contrôle des observations manquantes pourraient améliorer les résultats.

### Fuseaux horaires

L'analyse RTE utilise l'heure française, tandis que le croisement des bases repose sur l'UTC.

Certains libellés et commentaires doivent être harmonisés pour éviter toute ambiguïté.

### Interprétation statistique

Les corrélations peuvent être influencées par les saisons, les heures de la journée, la structure du parc électrique et d'autres facteurs non pris en compte.

De plus, certaines classes de rayonnement créées à partir des quartiles peuvent regrouper des observations de rayonnement nul. Les appellations météorologiques de ces classes doivent donc être interprétées avec prudence.

### Perspectives

Les prochaines étapes envisagées sont :

- Améliorer l'harmonisation temporelle des bases.
- Comparer les relations observées selon les années et les saisons.
- Étudier plus précisément les périodes de faible charge résiduelle.
- Approfondir les analyses statistiques en tenant compte des variations calendaires.

## Reproductibilité

Le projet privilégie une démarche reproductible.

Les données sont issues de sources identifiées et récupérées par un script Python. Les traitements et analyses sont documentés dans les notebooks.

Le code est suivi avec Git et GitHub, et les dépendances sont regroupées dans `requirements.txt`.

Les fichiers de données volumineux peuvent être conservés localement et régénérés à partir du script de collecte.

## Auteurs

**Elyes · Melchior · Romain**

Projet réalisé dans le cadre du cours **Python pour la Data Science**.