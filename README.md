# BI & Data Mining Project – ComptoirDB

End-to-end decision-support project for the company **ComptoirDB**:
**ETL (Talend) → Star-schema data warehouse → Reporting (Power BI) → Data mining (Weka)**.

> Academic project completed in 2025 by **Monya OUBELLA**.
> No tool installation is required to browse this repository: results are provided as CSV/ARFF files and screenshots.

## Architecture

```mermaid
flowchart LR
    A[(MySQL ComptoirDB<br/>OLTP source)] --> B[Talend ETL<br/>4 jobs]
    B --> C[(Data warehouse<br/>star schema)]
    C --> D[Power BI<br/>dashboards]
    C --> E[Weka<br/>clustering and Apriori]
```

## Data model (star schema)

```mermaid
erDiagram
    DIM_CLIENT  ||--o{ FAIT_VENTES : ID_Client_SK
    DIM_PRODUIT ||--o{ FAIT_VENTES : ID_Produit_SK
    DIM_TEMPS   ||--o{ FAIT_VENTES : ID_Temps_SK
    DIM_CLIENT  { int ID_Client_SK PK string Code_Client_Source string Societe string Ville string Pays }
    DIM_PRODUIT { int ID_Produit_SK PK int Ref_Produit_Source string Nom_Produit string Nom_Categorie }
    DIM_TEMPS   { date ID_Temps_SK PK int Annee int Mois string Nom_Mois int Trimestre int Jour }
    FAIT_VENTES { int ID_Vente PK int ID_Client_SK FK int ID_Produit_SK FK date ID_Temps_SK FK int Quantite decimal Montant_Vente int Delai_Livraison }
```

`Nom_Categorie` is denormalized into `dim_produit` to avoid extra joins (star-schema principle).

## ETL (Talend)

| Job | Role |
|---|---|
| `jDim_Client` | Reads `comptoir_clients`, selects/renames columns with `tMap` |
| `jDim_Produit` | Left join `produits` + `comptoir_categories` to denormalize the category |
| `jDim_Temps` | Extracts order dates, removes duplicates (`tUniqRow`), derives year/month/quarter/day |
| `jFait_Ventes` | Lookups on dimensions to get surrogate keys, computes `Montant = Prix × Quantité` and `Délai = date d'envoi − date de commande` |

## Repository structure

```
├── data/    dim_client.csv, dim_produit.csv, dim_temps.csv, fait_ventes.csv
├── weka/    QUEST_6_A.arff (clustering), QUEST_6_B.arff (association rules)
└── docs/    project report (PDF) and screenshots
```

## Results

- **Revenue by category**: Desserts rank first, Boissons last.
- **Sales over time**: peak around day 8 of July 2023, then a sharp drop.
- **Geography**: sales concentrated in Western Europe, with a presence in Mexico.
- **Logistics**: client "Bon app'" has the longest average delivery time (12 days).
- **Clustering (Weka EM)**: 5 clusters of delivery time, from about 2.4 days (express) to 12 days (problematic delays).
- **Association rules (Apriori)**: products frequently bought together, useful for cross-selling.

## Limitations

The dataset is very small (13 sales facts, 10 transactions), so the data-mining results are illustrative, not statistically significant.
The Talend jobs and the source MySQL database are not included in this repository.

## Tech

Talend Open Studio · MySQL · Power BI · Weka · Star schema / R-OLAP