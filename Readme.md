# Prix négatifs de l'électricité et énergies renouvelables

## Présentation

Projet Python de data science consacré à l'analyse des prix négatifs sur le marché Day-Ahead français.

**Problématique :** Dans quelle mesure la production renouvelable, la consommation électrique et les échanges européens sont-ils associés à la fréquence et à l'intensité des prix négatifs ?

## Sources de données

- RTE / éCO2mix : production et consommation françaises.
- ENTSO-E : prix Day-Ahead et échanges européens.
- Open-Meteo : données météorologiques historiques.
- Copernicus ERA5 : données de réanalyse météorologique (optionnel).
- Open Power System Data : séries historiques complémentaires (optionnel).

## Installation

Créer un environnement virtuel Python, l'activer, puis installer les dépendances :

`python -m pip install -r requirements.txt`

## Exécution

Depuis la racine du projet :

`python collecte_electricite_apis.py`

Les résultats disponibles sont enregistrés dans le dossier `data/` sous forme de fichiers CSV distincts.

Les données ENTSO-E nécessitent une clé API personnelle, fournie via la variable d'environnement `ENTSOE_API_KEY`.

## Organisation

- `collecte_electricite_apis.py` : téléchargement des données.
- `notebooks/` : exploration, analyses et rapport.
- `figures/` : visualisations.
- `data/` : fichiers générés localement, non distribués automatiquement.

## Reproductibilité

Les données sont téléchargées à partir de leurs sources publiques selon les conditions d'accès applicables. Les clés personnelles ne sont jamais enregistrées sur GitHub.

## Auteurs

Projet réalisé par trois étudiants dans le cadre du cours Python pour la Data Science.