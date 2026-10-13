# Online Retail II: Azure Medallion Data Pipeline

End-to-end data engineering project that ingests the **Online Retail II transactional dataset** into Azure Data Lake Storage Gen2 (ADLS Gen2) and processes it through a **Bronze → Silver → Gold medallion architecture** using Azure Databricks, orchestrated with Azure Data Factory (ADF).

## Architecture

```text
Source: Online Retail II Dataset
              |
              v
     Azure Data Factory
  (Linked Services + Datasets)
              |
              v
      ADLS Gen2 Storage
              |
      +-------+-------+
      |       |       |
      v       v       v
   Bronze   Silver   Gold
    Layer    Layer   Layer
      |       |       |
      v       v       v
  Raw Data  Cleansed  Dimensions
            Data     + Facts
              |
              v
      Azure Databricks
  +---------------------------+
  | Unity Catalog              |
  | Auto Loader                |
  | Lakeflow Declarative       |
  | Pipelines                  |
  +---------------------------+
```

## Repository Structure

```text
Data_Engineering/
├── factory/                 # ADF factory configuration
│   └── retail factory config
├── linkedService/           # ADLS, Databricks, and Key Vault connections
├── dataset/                  # ADF datasets
├── pipeline/                 # ADF pipeline definitions
├── databricks/
│   ├── bronze_transform      # Bronze layer processing
│   ├── silver_transform      # Silver layer processing
│   ├── gold_dimensions      # Gold dimension tables
│   └── gold_facts           # Gold fact tables
└── README.md
```

## Technology Stack

- **Azure Data Factory (ADF):** Pipeline orchestration and data ingestion
- **Azure Data Lake Storage Gen2:** Scalable cloud data storage
- **Azure Databricks:** Distributed data processing with Apache Spark
- **Delta Lake:** Reliable storage and transactional data management
- **Unity Catalog:** Data governance and access control
- **Auto Loader:** Incremental file ingestion
- **Lakeflow Declarative Pipelines:** Declarative data transformation and pipeline management
- **Azure Key Vault:** Secure secret management
- **Git & GitHub:** Version control

## Medallion Architecture

### Bronze Layer
- Ingest raw source data into ADLS Gen2.
- Preserve source data for traceability and reprocessing.
- Support incremental ingestion where configured.

### Silver Layer
- Clean and validate transactional records.
- Handle null values, duplicates, and invalid records.
- Standardize data types and column names.
- Prepare conformed data for downstream analytics.

### Gold Layer
- Build analytical dimension and fact tables.
- Organize curated data for reporting and business intelligence.
- Support customer, product, and sales analysis based on the implemented data model.

## Prerequisites

- An active Azure subscription
- ADLS Gen2 storage account with Bronze, Silver, and Gold storage locations
- Azure Databricks workspace with Unity Catalog enabled
- Azure Data Factory instance
- Azure Key Vault for secret management
- Service principal or managed identity configured with the required permissions

## Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/PatilLokeshReddy/Data_Engineering.git
   cd Data_Engineering
   ```

2. Configure the ADLS Gen2 storage account and the required storage locations.

3. Configure Azure Data Factory linked services and datasets.

4. Set up Azure Databricks connectivity and Unity Catalog permissions.

5. Configure Key Vault references and authentication securely.

6. Deploy or import the ADF pipeline configuration.

7. Run the ingestion pipeline and validate the Bronze, Silver, and Gold outputs.

## Security

- Store credentials and secrets in Azure Key Vault.
- Never commit passwords, access tokens, client secrets, or connection strings to GitHub.
- Grant only the permissions required for each service principal or managed identity.
- Use Azure RBAC and Unity Catalog permissions to control access.

## Project Goals

- Implement an end-to-end Azure data engineering workflow.
- Practice incremental ingestion and data transformation.
- Apply the medallion architecture using Delta Lake.
- Orchestrate data processing using Azure Data Factory.
- Build curated datasets for analytics.

## Author

**Lokesh Reddy Patil**

GitHub: [PatilLokeshReddy](https://github.com/PatilLokeshReddy)
