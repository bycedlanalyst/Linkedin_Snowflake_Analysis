# 🔍 LinkedIn Job Analysis — with Snowflake & Streamlit

<div align="center">

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![AWS S3](https://img.shields.io/badge/AWS_S3-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

*Analyse de plusieurs milliers d'offres d'emploi LinkedIn via une architecture Medallion sur Snowflake*

</div>

---

## 📌 Description

Ce projet analyse plusieurs milliers d'offres d'emploi LinkedIn en utilisant **Snowflake** comme entrepôt de données et **Streamlit** pour les visualisations interactives.

Il implémente une **architecture Medallion (Bronze / Silver / Gold)** pour garantir la qualité, la traçabilité et la fiabilité des données tout au long du pipeline.

---

## 🏗️ Architecture des Données

```
☁️  S3 (Fichiers bruts)
         │
         ▼
   🟫  BRONZE      ──►  Ingestion brute (VARCHAR / VARIANT) — données telles quelles
         │
         ▼
   🩶  SILVER      ──►  Nettoyage, typage, déduplication
         │
         ▼
   🥇  GOLD        ──►  Agrégations prêtes pour l'analyse
         │
         ▼
   📊  STREAMLIT   ──►  Visualisations interactives
```

---

## 📊 Analyses Réalisées

| # | Analyse | Description |
|---|---------|-------------|
| 1️⃣ | **Top 10 des titres de postes les plus publiés** | Identifie les métiers en tension par industrie |
| 2️⃣ | **Top 10 des postes les mieux rémunérés** | Compare les salaires max moyens par poste et industrie |
| 3️⃣ | **Répartition par taille d'entreprise** | De la TPE (0) à la grande entreprise (7) |
| 4️⃣ | **Répartition par secteur d'activité** | Les 20 secteurs qui recrutent le plus sur LinkedIn |
| 5️⃣ | **Répartition par type d'emploi** | CDI, CDD, stage, temps partiel, etc. |

---

## 🗂️ Structure du Projet

```
📁 linkedin-snowflake-analysis/
│
├── 📄 README.md
│
├── 📁 sql/
│   ├── 01_setup.sql              ── Création BDD, schémas, stage, formats
│   ├── 02_tables_bronze.sql      ── Création des tables Bronze
│   ├── 03_load_bronze.sql        ── Chargement des données dans Bronze
│   ├── 04_tables_silver.sql      ── Création des tables Silver
│   ├── 05_load_silver.sql        ── Transformation Bronze → Silver
│   ├── 06_quality_tests.sql      ── Tests de qualité des données
│   ├── 07_tables_gold.sql        ── Création et alimentation des tables Gold
│   ├── 08_analyses.sql           ── Requêtes SQL des 5 analyses
│   └── 09_automation.sql         ── Snowpipe + tâches planifiées
│
└── 📁 streamlit/
    └── streamlit_app.py          ── Code Streamlit des 5 visualisations
```

---

## 🗄️ Modèle de Données (ERD)

### Bronze / Silver

```
┌─────────────────────┐        ┌──────────────────┐
│    job_postings     │        │    companies     │
├─────────────────────┤        ├──────────────────┤
│ job_id (PK)         │        │ company_id (PK)  │
│ company_name (FK) ──┼────────► name             │
│ title               │        │ description      │
│ description         │        │ company_size     │
│ max_salary          │        │ state / country  │
│ med_salary          │        │ city / zip_code  │
│ min_salary          │        │ url              │
│ pay_period          │        └────────┬─────────┘
│ formatted_work_type │                 │
│ location            │        ┌────────▼─────────┐
│ remote_allowed      │        │company_industries│
│ views / applies     │        ├──────────────────┤
│ listed_time         │        │ company_id (FK)  │
│ expiry              │        │ industry         │
│ sponsored           │        └──────────────────┘
└────────┬────────────┘
         │
    ┌────▼──────┐    ┌──────────────────┐    ┌──────────────────────┐
    │  benefits │    │  job_industries  │    │  employee_counts     │
    ├───────────┤    ├──────────────────┤    ├──────────────────────┤
    │ job_id(FK)│    │ job_id (FK)      │    │ company_id (FK)      │
    │ inferred  │    │ industry_id      │    │ employee_count       │
    │ type      │    └──────────────────┘    │ follower_count       │
    └───────────┘                            │ time_recorded        │
    ┌───────────┐                            └──────────────────────┘
    │ job_skills│
    ├───────────┤
    │ job_id(FK)│
    │ skill_abr │
    └───────────┘
```

### Gold

```
┌──────────────────────────────────┐
│  🥇 Tables Gold (agrégées)       │
├──────────────────────────────────┤
│  top_titres_par_industrie        │
│  top_salaires_par_industrie      │
│  repartition_taille_entreprise   │
│  repartition_secteur             │
│  repartition_type_emploi         │
└──────────────────────────────────┘
```

---

## 📦 Jeu de Données

Les fichiers sources sont disponibles dans le bucket S3 : `s3://snowflake-lab-bucket/`

| Fichier | Format | Description |
|---------|--------|-------------|
| `job_postings.csv` | CSV | Offres d'emploi détaillées |
| `benefits.csv` | CSV | Avantages par offre |
| `employee_counts.csv` | CSV | Nombre d'employés par entreprise |
| `job_skills.csv` | CSV | Compétences requises par offre |
| `companies.json` | JSON | Informations sur les entreprises |
| `company_industries.json` | JSON | Secteurs d'activité par entreprise |
| `company_specialities.json` | JSON | Spécialités par entreprise |
| `job_industries.json` | JSON | Secteurs d'activité par offre |

---

## ✅ Prérequis

- 🔐 Compte Snowflake *(essai gratuit 30 jours disponible)*
- 🐍 Notions de base en **SQL** et **Python**

---

## 🚀 Instructions pour Reproduire le Projet

```bash
# Étape 1 — Créer un compte Snowflake
# https://signup.snowflake.com/

# Étape 2 — Exécuter les scripts SQL dans l'ordre
01_setup.sql → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09

# Étape 3 — Dans Snowflake, créer une Streamlit App
# Sélectionner "Run on warehouse"

# Étape 4 — Coller le contenu de streamlit_app.py et cliquer sur Run
```

---

## ⚙️ Automatisation

| Mécanisme | Détail |
|-----------|--------|
| **Snowpipe** | `AUTO_INGEST = TRUE` — chargement automatique des nouveaux fichiers S3 dans Bronze |
| **Task Bronze → Silver** | Propagation automatique toutes les **60 minutes** |
| **Task Silver → Gold** | Propagation automatique toutes les **70 minutes** |

---

## 🐛 Problèmes Rencontrés & Solutions

| Problème | Solution |
|----------|----------|
| Timestamps en Unix millisecondes | `TO_TIMESTAMP_NTZ(CAST(valeur/1000 AS BIGINT))` |
| `company_name` contient des IDs flottants (ex: `7789.0`) | `SPLIT_PART(company_name, '.', 1)` |
| Doublons dans `job_postings` | Déduplification avec `ROW_NUMBER() + QUALIFY` |
| Fichiers JSON à structure imbriquée | Stockage en `VARIANT` dans Bronze, extraction dans Silver |

---

## 👥 Répartition des Tâches

| Membre | Périmètre |
|--------|-----------|
| **Membre 1** | Setup, Bronze, Silver, Tests qualité |
| **Membre 2** | Gold, Streamlit, Automatisation, README |

---

## 📸 Résultats des Visualisations

### Analyse 1 — Top 10 des titres de postes les plus publiés par industrie
![Analyse 1](Analyse%201%20-%20Top%2010%20des%20titres%20de%20postes%20les%20plus%20publi%C3%A9s%20par%20industrie.png)

### Analyse 2 — Top 10 des postes les mieux rémunérés par industrie
![Analyse 2](Analyse%202%20-%20Top%2010%20des%20postes%20les%20mieux%20r%C3%A9mun%C3%A9r%C3%A9s%20par%20industrie.png)

### Analyse 3 — Répartition des offres par taille d'entreprise
![Analyse 3](Analyse%203%20-%20R%C3%A9partition%20des%20offres%20par%20taille%20d%27entreprise.png)

### Analyse 4 — Répartition des offres par secteur d'activité
![Analyse 4](Analyse%204%20-%20R%C3%A9partition%20des%20offres%20par%20secteur%20d%27activit%C3%A9.png)

### Analyse 5 — Répartition des offres par type d'emploi
![Analyse 5](Analyse%205%20-%20R%C3%A9partition%20des%20offres%20par%20type%20d%27emploi.png)

---

<div align="center">

*Projet réalisé dans le cadre d'un lab Data Engineering avec Snowflake*

</div>
