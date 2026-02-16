## Getting Started (Run the Project Locally) 

```git clone https://github.com/Thibaut-Longchamps/bike-demand-forecasting.git```

Toutes les commandes ci-dessous doivent être éxécutées depuis la racine du projet :

```cd /path/to/bike-demand-forecasting```

**1. Create environment**:

Linux :
```python3 -m venv env_bike```

Windows :
```python -m venv env_bike```


**2. Activate environment**:

Linux :
```source env_bike/bin/activate```

Windows :
```./env_bike/Scripts/Activate```



**3. Upgrade PIP**:

```python -m pip install --upgrade pip``` 

**4. Pyproject (setup src package & requirements.txt)**:

```python -m pip install -e .```

## DATA

Avant d'exécuter les notebooks dans l'ordre numéroté, il est nécessaire de placer les données zip (non décompressées) dans leur dossier respectif:
- les données zip de 2024 dans : ```bike-demand-forecasting/data/raw/2024```
- les données zip de 2025 dans : ```bike-demand-forecasting/data/raw/2025```


## Performances Obtenues

Le modèle avec feature engineering surpasse nettement le baseline en CV et en test. En test, il réduit fortement l’erreur (MAE: 152.03 -> 105.95, sMAPE: 0.296-> 0.207) et diminue fortement le biais (+119.6 -> +18.1). L’analyse par segments montre toutefois un point faible sur les jours fériés (MAE 223.7, sMAPE 0.523, biais +198.7), alors que les performances sont meilleures hors vacances, avec une MAE plus faible en heures creuses qu’en heures de pointe.

**Axes d'améliorations**:
- À court terme, la priorité est d’améliorer la performance sur les jours fériés via des features calendaires plus riches (saisons, météo...) et une modélisation spatiale (par station ou par clusters de station proche géographiquement) afin de mieux cerner les dynamiques locales et ainsi anticiper un rééquilibrage du parc des vélos (système d'alerte...). Ce critère est critique dans une logique opérationnelle car une mauvaise anticipation implique des stations en rupture ou en surcharge.


**Temps passé sur le projet** : 4h20

**Utilisation LLM (ChatGPT)**:  
- remplissage y=0 sur les horaires sans location de vélos
- Aide pour la création de la colonne y
- graphique EDA "Daily bike demand distribution by month (2024–2025)" & "Average daily demand by day of week - 2024 vs 2025"
- Brainstorming pour la création des fonctions de feature engineering (découverte de la méthode shift())
- formule sMAPE

**Logiciel et environnement**:
- CPU: AMD Ryzen 5 5500U (6 cores, 12 threads), 2.10 GHz
- RAM: 8 GB
- OS: Ubuntu 22.04.5 LTS (Jammy)

- Platform: WSL2 (Ubuntu on Windows)
