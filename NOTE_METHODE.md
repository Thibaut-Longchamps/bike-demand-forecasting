### **1. Définition du problème**

L'entreprise Capital Bikeshare cherche à prévoir la demande sur son parc de vélo afin d'optimiser son organisation (maintenance, staff...)

Cette étude de cas est un problème de régression sur série temporelle. 

**Target**:
la target n'est pas encore définie dans les données brutes. J'ai pris le choix d'agréger les données à une granularité horaire (occurrences) afin que le modèle soit capable de prédire la demande chaque heure de la journée. La force de la demande étant intrinsèquement liée aux variations temporelles (heure de bureau, week-end...)

**Avantages de cette approche**:
Prévoir la demande horaire à H+24 (même heure le lendemain) permet à l’opérateur d’anticiper le rééquilibrage et les ressources terrain sur le planning du lendemain, en exploitant les patterns journaliers/hebdomadaires.

**forecasting**:
Au vu de la contrainte de temps pour réaliser l'étude de cas, les prédictions seront faites sur l'ensemble des stations du réseau, à terme un modèle par station serait beaucoup plus pertinent afin d'optimiser le micro-management de chacune d'elle. 

**Features**:
Seules les colonnes de features temporelles seront utilisées puisque l'agrégation a changé la granularité des données rendant les autres colonnes inutilisables

**apports opérationnels**
Un modèle prévisionnel horaire à 7 jours permet :

- Une meilleure planification des équipes (shift en fonction des charges)
- Une meilleure allocation des ressources disponibles

### **2. Features**

**modèle baseline vs modèle avec feature engineering**

- le modèle baseline est entraîné en utilisant uniquement les features temporelles afin de prédire la variable cible.
- le modèle avec feature engineering est entraîné avec l'aide de nouvelles features qui sont roll 24, roll168, lag 24, lag168

lag_24 : indique la demande observée 24h auparavant
roll_24 : indique la demande glissante observée en moyenne sur les 24 dernières heures 


Data leakage : Cette transformation permet au modèle de mieux cerner les variations temporelles, cependant nécessite également certains ajustements sur le dataset afin d’éviter le data leakage, puisque les variables de type lag et rolling doivent être calculées uniquement à partir des observations passées, sans jamais utiliser d’information provenant du futur (time series split)

Les premières lignes supprimées dans le modèle FE sont également retirées du baseline pour garantir une comparaison équitable.

### **3. Modélisation**

**Pipeline**:
- One hot encoding : sur variable is_holiday
- StandardScaler n’est pas appliqué aux variables temporelles, ni aux variables de type lag et rolling, car le modèle utilisé est un modèle à base d’arbres. Ces types de modèles sont peu sensibles à l’échelle des variables.

**Entraînement**:
Modèle random forest + GridSearchCV (optimisation des hyperparamètres) avec comme score de référence la mae.
Les scores de mape, smape et bias ont été ajoutés pour obtenir une comparaison plus robuste

GridSearchCV découpe automatiquement le jeu dev en folds train/validation (via TimeSeriesSplit), évalue chaque combinaison d’hyperparamètres sur tous les folds, moyenne les scores, puis sélectionne la meilleure configuration et réentraîne ce meilleur modèle sur tout le jeu dev.


### **4. Résultats**

**métriques utilisées**:
- MAE : mesure l’erreur moyenne en valeur absolue en unité réelle (nombre de locations).
- MAPE : mesure l’erreur relative moyenne en pourcentage par rapport aux valeurs réelles (sensible quand les valeurs réelles sont proches de 0).
La MAPE doit être interprétée avec prudence, car la série contient des heures sans demande (y=0) ajoutées lors du remplissage, ce qui peut rendre cette métrique instable ou biaisée.

- sMAPE : version symétrique de la MAPE, plus robuste en présence de faibles valeurs ou de zéros.
- Bias : mesure l’erreur moyenne (positif = sur-prédiction, négatif = sous-prédiction)

**Modèle baseline vs modèle avec feature engineering**:

gridsearch cv :
- Le modèle avec feature engineering est meilleur sur la validation croisée: MAE passe de 190.98 à 155.50, sMAPE de 0.2307 à 0.2000, et le biais se recentre fortement (-39.68 vs -0.39).

métriques sur l'ensemble test :
- Sur le jeu test, l’amélioration est confirmée: MAE baisse de 152.03 à 105.95, MAPE de 0.4490 à 0.2729, sMAPE de 0.2962 à 0.2074, et le biais est nettement réduit (+119.62 à +18.12).

**Analyse des résultats du modèle feature engineering sur différentes temporalités**

- période de vacances:
Le segment holiday est le plus difficile: MAE=223.72, MAPE=0.9273, sMAPE=0.5235, avec un biais très positif (+198.70), ce qui indique une forte sur-prédiction.

- période de pic d'affluence:
En heures de pointe, l’erreur absolue est élevée (MAE=176.05) mais l’erreur relative reste modérée (MAPE=0.2660, sMAPE=0.2006), avec un biais positif limité (+24.49).

- période avec creux d'affluence:
En ce qui concerne les heures creuses la MAE est plus faible (82.57) et le biais reste modéré (+16.00), avec des métriques relatives proches du pic (MAPE=0.2752, sMAPE=0.2097).

### **5. Vision MLOps**:

- Un DAG Airflow est déclenché chaque fin de semaine pour ingérer les données de la semaine écoulée, recalculer les features (lags/rolling), puis générer les prédictions de la semaine suivante.

- Le pipeline sera industrialisé en étapes: ingestion, contrôles qualité des données (logs).
(protoype pandas mais un passage à PySpark représenterait un gain significatif)

- MLflow peut être utilisé pour le versioning de modèle et le suivi des résultats des entraînements hebdomadaires.

- Les performances en production sont monitorées en continu pour surveiller le data drift. Cette dérive peut être gérée grâce à des solutions comme Prometheus (alertes automatiques) et Grafana pour la visualisation.

- Un ré-entraînement périodique (hebdomadaire) est lancé automatiquement, avec comparaison au modèle en production; promotion uniquement si amélioration validée.
