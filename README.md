# DataBeton — Hackathon Colas × ESTP × EFREI

DataBeton est un projet d’équipe sur le **pilotage des carrières de granulats** et la **classification du chargement des camions sur photo**. Nous avons préparé des données de carrières avec PySpark, construit un rapport Power BI et entraîné un classifieur d’images avec PyTorch. Le hackathon s’est déroulé du **18 au 20 mai 2026 à l’ESTP Cachan**. Nous étions l’**équipe 6**.

[Sujet du hackathon](docs/Sujet-Hackathon-Colas-2026.pdf) · [Consignes Power BI](docs/Slide-BI-Presentation.pdf) · [Notebook Data Engineering](notebooks/part_DataEng_DataBeton.ipynb) · [Notebook vision](notebooks/part_DataScience_DataBeton_markdown.ipynb) · [Rapport Power BI](bi/PowerBI_DataBeton.pbix) · [Présentation DataBeton](docs/DataBeton.pdf)

## Le sujet

Colas souhaitait réunir, pour chaque carrière, des informations jusque-là réparties entre les livraisons, la production, les réserves, les pannes et les risques climatiques. Un responsable régional devait pouvoir comparer les sites, puis explorer les indicateurs d’une carrière. En parallèle, le challenge vision demandait de prédire le contenu de la benne à partir d’une photo, avec une attention particulière aux camions vides.

| Volet du sujet | Ce qui était demandé | Rendu disponible |
|---|---|---|
| **Data Engineering** | Préparer les données sur Databricks et produire des tables d’analyse pour une fiche carrière. | [Notebook PySpark](notebooks/part_DataEng_DataBeton.ipynb) |
| **Data Analysis / BI** | Créer un rapport Power BI d’au moins trois pages sur l’activité, les installations et les réserves. | [Rapport Power BI à quatre pages](bi/PowerBI_DataBeton.pbix) |
| **Data Science / Vision** | Reconnaître le chargement ou la classe « camion vide » et remettre les prédictions du jeu témoin. | [Notebook PyTorch](notebooks/part_DataScience_DataBeton_markdown.ipynb) et [CSV de prédictions](predictions/predictions_final.csv) |
| **Opportunités métier** | Proposer des usages, des sources complémentaires et des indicateurs, puis présenter l’ensemble au jury. | [Présentation DataBeton — proposition métier](docs/DataBeton.pdf). |

**Ma contribution — Anaëlle Pollart.** J’ai travaillé sur les **images des camions à l’entrée des carrières**, dans le volet Computer Vision. Les rendus et les résultats ci-dessous sont ceux de l’équipe DataBeton.

## 1. Data Engineering — des données de carrière aux tables d’analyse

Le notebook utilise **Python, PySpark et Databricks**. Nous avons préparé sept sources, dont les livraisons, les référentiels sites et produits, les pannes, la production, les réserves et les données climatiques EcoAct. Les transformations harmonisent les types et les codes de valeurs manquantes, contrôlent les doublons et construisent des agrégats selon le besoin métier.

- **Livraisons.** Calcul de volumes, chiffre d’affaires hors taxes et nombres de bons de livraison, de clients et de produits. La sortie du notebook affiche **77 129 802 lignes** dans la table nettoyée.
- **Fiche par carrière et année.** Rapprochement des livraisons, de la production, des réserves et des pannes sur l’identifiant de carrière et l’année. Les montants de livraison sont convertis en euros à l’aide de taux fixes. Cette approximation permet de consolider les montants pour le challenge.
- **Résumé par carrière.** Agrégation en une ligne par site pour comparer l’activité, les volumes, la disponibilité estimée, les réserves et l’exposition climatique. La sortie sauvegardée recense **1 172 carrières distinctes**. Le résumé climatique V2 couvre **253 sites**.

Le code produit également un mix produit, un score multicritère d’activité et un score climatique. Le calcul de disponibilité est une approximation basée sur les durées de panne. La couverture climatique varie selon les sites. [Notebook Data Engineering, cellules 3–37](notebooks/part_DataEng_DataBeton.ipynb)

## 2. Power BI — explorer l’activité et les risques

Le rapport comporte quatre pages.

| Page | Ce qu’on peut y examiner |
|---|---|
| **Fiche carrière** | Carte des sites, chiffre d’affaires, volumes et filtres par activité ou niveau de risque. |
| **Détail par carrière** | Sélection d’un site, évolution annuelle et informations sur le risque climatique. |
| **Santé opérationnelle** | Nombre de pannes par site et par période, avec un indicateur d’exposition au risque incendie. |
| **Durabilité & réserves** | Volumes de réserves, production, pression sur les réserves et familles de risques. |

Le `.pbix` contient le rapport livré. L’actualisation à partir des tables sources dépend de l’environnement Databricks du challenge. [Ouvrir le fichier Power BI](bi/PowerBI_DataBeton.pbix)

## 3. Computer Vision — reconnaître le chargement sur une photo

**Problème.** Le modèle associe une photo à l’une de **sept classes**, soit six codes matériau et la catégorie « camion vide ». La difficulté vient notamment des matériaux visuellement proches, des différences d’éclairage et des bennes dont le contenu est masqué.

**Préparation.** Le CSV de départ comportait **2 101 images** et un seul exemple étiqueté « camion vide ». Après relecture, correction des labels et retrait de 28 images invalides ou trop ambiguës, le notebook conserve **2 073 images**, dont **266** dans la catégorie « camion vide ». Pour le challenge, cette catégorie couvrait aussi certains cas où le contenu n’était pas visible, comme les bennes bâchées. Le jeu est réparti en **1 451 images d’entraînement, 311 de validation et 311 de test interne**.

**Modèle.** Nous avons adapté **EfficientNet-B0**, préentraîné sur ImageNet, avec une couche de sortie à sept classes. Les images sont redimensionnées à 224 × 224 et normalisées. Les variations de luminosité et de contraste et les retournements horizontaux sont appliqués à l’entraînement. La boucle utilise **PyTorch, AdamW et une entropie croisée avec lissage des labels**. Les meilleurs poids sont sélectionnés selon l’accuracy de validation.

**Résultat.** La sortie sauvegardée affiche **92,28 % d’accuracy sur le test interne (287/311)**. Pour le jeu témoin, le code choisit la classe au score maximal et exporte **282 prédictions**. Le CSV de soumission est disponible. Le score officiel et le classement du jury n’y figurent pas.

**Limite.** Certains matériaux étaient surtout photographiés sur un même site. Le classifieur peut apprendre des indices liés à l’arrière-plan, à la caméra ou à l’éclairage. Le test interne est stratifié par classe. Un test sur un site tenu à l’écart de l’entraînement permettrait de mieux mesurer la généralisation. [Démarche technique](METHODOLOGIE.md) · [Notebook de vision](notebooks/part_DataScience_DataBeton_markdown.ipynb) · [Prédictions](predictions/predictions_final.csv)

## 4. Opportunité métier — mieux orienter les matériaux vers le recyclage

La présentation de DataBeton propose d’utiliser une caméra au pont-bascule pour identifier le chargement des camions entrants. La classe prédite alimenterait une fiche numérique. Cette fiche pourrait être rapprochée de données de pesée, de qualité et de filière pour aider à orienter le tri et suivre dans Power BI les volumes et la valorisation des matériaux recyclables.

Le support présente une piste de déploiement progressif et un budget indicatif pour un pilote sur cinq sites. Le classifieur du hackathon reconnaît le chargement sur photo. L’évaluation de sa recyclabilité, le choix de filière, la tarification et le déploiement sur site font partie de la proposition métier. [Présentation DataBeton](docs/DataBeton.pdf)

## Fichiers et exécution

```text
.
├── README.md
├── METHODOLOGIE.md
├── docs/
│   ├── DataBeton.pdf
│   ├── Sujet-Hackathon-Colas-2026.pdf
│   └── Slide-BI-Presentation.pdf
├── notebooks/
│   ├── part_DataEng_DataBeton.ipynb
│   └── part_DataScience_DataBeton_markdown.ipynb
├── bi/
│   └── PowerBI_DataBeton.pbix
├── data/
│   └── labels.csv
└── predictions/
    └── predictions_final.csv
```

- **Consulter.** Ouvrir les notebooks ou les télécharger pour parcourir le code et les sorties enregistrées. Le rapport s’ouvre dans Power BI Desktop.
- **Relancer le volet vision.** Récupérer `hackaton.zip` et `test.zip` si les archives sont proposées en pièces jointes de la Release, puis les extraire. Les chemins Databricks du notebook d’origine sont à adapter à l’environnement utilisé. Les bibliothèques utilisées sont pandas, Pillow, PyTorch, torchvision et scikit-learn.
- **Relancer le volet Data Engineering.** Le notebook dépend des tables et fichiers du lakehouse utilisé pendant le hackathon. Les ZIP d’images concernent le volet vision.
