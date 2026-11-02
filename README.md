# Test pratique - AI & Automation Engineer
## Surveillance transactionnelle AML/CFT

## 1. Présentation
Ce notebook réalise une analyse exploratoire et un prototype de surveillance transactionnelle sur un dataset de transactions digitales. Il combine contrôles de qualité, règles comportementales AML et détection d’anomalies non supervisée par Isolation Forest.

**Important :** les alertes sont des signaux nécessitant une analyse humaine. Elles ne constituent pas une preuve d’infraction ni une décision automatique de conformité.

## 2. Livrables
- `AML_CFT_Analysis.ipynb` : code source exécutable.
- `dataset_transactions_AML_5Mo.csv` : dataset fourni.
- `outputs/quality_report.csv` : indicateurs de qualité des données.
- `outputs/alerts.csv` : alertes consolidées et dédupliquées issues des règles.
- `outputs/customer_anomalies.csv` : clients atypiques détectés par Isolation Forest.
- `data/processed/transactions_clean.csv` et `transactions_quarantine.csv` : données exploitables et mises en quarantaine.

Les résultats sont produits automatiquement à l’exécution du notebook, pas manuellement.

## 3. Prérequis
- Python 3.10+ recommandé.
- Jupyter Notebook ou JupyterLab.
- Mémoire suffisante pour charger le CSV en RAM.
- Internet uniquement pour installer les bibliothèques ; aucune API IA externe n’est appelée pendant l’analyse.

## 4. Installation
Dans le dossier contenant le notebook et le dataset :

```bash
python -m venv .venv
```

Windows :
```bash
.venv\Scripts\activate
```

Linux/macOS :
```bash
source .venv/bin/activate
```

Installer les dépendances :
```bash
pip install -r requirements.txt
```
ou si vous voulez supprimer le cache
```bash
pip install -r requirements.txt --no-cache-dir
```

## 5. Exécution
```bash
jupyter notebook
```
Ouvrir `AML_CFT_Analysis.ipynb`, puis sélectionner **Kernel → Restart & Run All**.

Exécution non interactive possible :
```bash
jupyter nbconvert --to notebook --execute AML_CFT_Analysis.ipynb --output AML_CFT_Analysis_executed.ipynb
```

## 6. Organisation recommandée
```text
AML_CFT/
├── AML_CFT_Analysis.ipynb
├── dataset_transactions_AML_5Mo.csv
├── README_AML_CFT.md
├── requirements.txt
├── outputs/                  # créé à l'exécution
│   ├── quality_report.csv
│   ├── alerts.csv
│   └── customer_anomalies.csv
└── data/processed/            # créé à l'exécution
    ├── transactions_clean.csv
    └── transactions_quarantine.csv
```
Le notebook recherche le CSV à côté de lui.

## 7. Étapes de traitement
1. Chargement et validation du schéma.
2. Conversion des dates, montants et champs textuels.
3. Profilage des valeurs manquantes, doublons, montants invalides et retards d’ingestion.
4. Quarantaine des lignes non exploitables et export des transactions propres.
5. Analyse descriptive des transactions et statuts.
6. Application des règles R01 (fractionnement), R02 (concentration entrante) et R03 (sortie rapide).
7. Consolidation et déduplication des alertes.
8. Construction de features comportementales au niveau client.
9. Standardisation et entraînement d’Isolation Forest.
10. Export des alertes règles, anomalies ML et comparaison des approches.

## 8. Hypothèses
- `transaction_id` est l’identifiant unique attendu d’une transaction.
- Les montants sont exprimés en MGA.
- Les analyses temporelles utilisent `event_time`; `ingestion_time` sert à mesurer les retards.
- Les transactions `SUCCESS` sont privilégiées pour les analyses comportementales et le ML, conformément au notebook.
- `contamination=0.02` dans Isolation Forest est une hypothèse de proportion d’anomalies, et non une vérité terrain.
- Les seuils R01/R02/R03 sont expérimentaux et doivent être calibrés avec les experts métier et des données réelles.

## 9. Limites
- Dataset de test/synthétique : les résultats ne préjugent pas de la performance en production.
- Historique labellisé absent ou partiel : précision, rappel et taux de faux positifs ne sont pas validés par apprentissage supervisé.
- Isolation Forest détecte des profils statistiques atypiques, pas des fraudes avérées.
- Les règles peuvent générer des faux positifs et ne couvrent que les scénarios explicités.
- Le notebook constitue un prototype batch ; orchestration, monitoring, SLA, gestion de cas et intégration SI restent hors périmètre.
- Les données personnelles et identifiants doivent être protégés selon les politiques de sécurité de l’établissement.

## 10. Outils IA éventuellement employés
- **ChatGPT (OpenAI)** : assistance à la structuration de la démarche, explication/rédaction et revue du code.
- Le calcul des indicateurs et les résultats sont produits par le code Python exécuté localement sur le dataset fourni.
- Aucun service IA externe n’est appelé par le notebook pendant son exécution.
- Isolation Forest est un algorithme ML classique de `scikit-learn`, pas un LLM.

## 11. Tests et reproductibilité
Le notebook contient des assertions de cohérence, notamment : `volume initial = volume exploitable + volume en quarantaine`.
Les tests automatisés complémentaires et le schéma d’architecture sont inclus dans les livrables préparés séparément. Avant remise, exécuter le notebook intégralement et vérifier les exports dans `outputs/`.
`RANDOM_STATE=42` stabilise la graine d’Isolation Forest. Pour reproduire les résultats, conserver le dataset, les versions de bibliothèques et l’ordre d’exécution.
