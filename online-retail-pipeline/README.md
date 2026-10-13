**Online Retail II: Azure Medallion Data Pipeline**

End-to-end data engineering project that ingests the Online Retail II transactional dataset into ADLS Gen2 and processes it through a Bronze → Silver → Gold medallion architecture on Azure Databricks, orchestrated with Azure Data Factory.

**Architecture**

Source (Online Retail II)
        │
        ▼
Azure Data Factory ── retail pipeline (linked services + datasets)
        │
        ▼
ADLS Gen2
  ├── bronze  → raw, as-ingested data
  ├── silver  → cleaned, validated, conformed data
  └── gold    → dimensions + facts for analytics
        │
        ▼
Azure Databricks (Unity Catalog, Lakeflow Declarative Pipelines, Auto Loader)



**Repository Structure**

├── factory/                 # ADF factory configuration
│   └── retail factory config
├── linkedService/           # Retail linked services (ADLS, Databricks, Key Vault)
├── dataset/                 # Retail datasets
├── pipeline/                # Retail pipeline
├── databricks/
│   ├── bronze_transform     # Bronze layer
│   ├── silver_transform     # Silver layer
│   ├── gold_dimensions      # Gold dimension tables
│   └── gold_facts           # Gold fact tables
└── README.md



**Setup**
*Prerequisites*
Azure subscription
ADLS Gen2 storage account with bronze, silver, gold containers
Azure Databricks workspace with Unity Catalog enabled
Azure Data Factory instance
Azure Key Vault holding secrets
Service Principal with the required RBAC roles (Storage Blob Data Contributor, Key Vault Secrets User)
