# 🚀 Production-Ready Data Pipeline: API Ingestion to Lakehouse BI (Databricks)

## 🌟 Executive Summary

This project showcases a complete, end-to-end Extract, Transform, Load (ETL) pipeline implemented on the Databricks Lakehouse Platform. It demonstrates proficiency in core Data Engineering concepts: **Medallion Architecture (Bronze-Silver-Gold)**, **Data Quality (DQ)** implementation, and **Serverless BI Consumption**.

The architecture is structured to simulate a production environment, decoupling the ingestion, cleaning, and aggregation steps into separate, orchestratable units.

### 🛠️ Key Technologies & Concepts

| Category | Component | Focus |
| :--- | :--- | :--- |
| **Platform** | Databricks (Free/Serverless Edition) | Serverless compute management and Lakehouse environment. |
| **Architecture** | Medallion / Lakehouse | Incremental refinement of data quality across three layers. |
| **Persistence** | Delta Lake Tables | ACID compliance, schema enforcement, and versioning. |
| **Languages** | PySpark, Python (`requests`), SQL | Distributed data processing and external API interaction. |
| **BI Layer** | Databricks SQL Warehouse | Direct consumption of Gold layer data for analytical dashboards. |

---

## 💡 Engineering Challenge: Free Edition Constraints

A core challenge was developing a robust pipeline within the limitations of the Databricks Free Edition. This environment imposes restrictions such as unstable Spark sessions and limited persistence options (no reliable global views).

### Solution: Decoupled Architecture

To ensure professional best practices while maintaining functionality:

1.  **Code Decoupling:** The monolithic notebook was split into three distinct Python files (`01_Bronze`, `02_Silver`, `03_Gold`).
2.  **State Management:** Data persistence between stages relies entirely on **Delta Tables** (the Lakehouse layer) rather than memory, simulating a production environment ready for orchestration (e.g., Databricks Workflows or Airflow).

---

## 🌊 Lakehouse Architecture and Data Flow

The data is progressively refined as it moves through the three layers:

### 1. ⚙️ Bronze Layer (`01_Bronze_Ingestion.py`)

* **Source:** OMDb Public RESTful API.
* **Function:** Handles external API calls, implements request pagination logic, and collects raw JSON objects into a Python list.
* **Output:** Delta Table (`omdb_releases_bronze`). Stores the raw, schema-inferred data, including all source errors and audit columns (`ingestion_timestamp`).

### 2. 🧼 Silver Layer (`02_Silver_Transformation.py`)

* **Input:** Reads from the Bronze Delta Table.
* **Function:** Applies crucial **Data Quality (DQ)** and cleaning rules.
    * **DQ:** Conversion of the `Year` column from `StringType` to `IntegerType`.
    * **Filtering:** Removal of records where the year parsing failed (`release_year IS NOT NULL`).
    * **Standardization:** Renaming of source columns (`Title` to `movie_title`) and trimming whitespace.
* **Output:** Delta Table (`omdb_releases_silver`). Clean, standardized data ready for business logic.

### 3. ✨ Gold Layer (`03_Gold_Aggregation.py`)

* **Input:** Reads from the Silver Delta Table.
* **Function:** Applies the final business logic to create the analytical mart.
    * **KPI Calculation:** Aggregates records to count the total number of releases (`total_releases`) per `release_year` and `media_type`.
* **Output:** Delta Table (`omdb_releases_gold`). Optimized for low-latency BI querying via the SQL Warehouse.

---

## 📈 Business Intelligence (BI) & Results

The final Gold layer is consumed directly by a dedicated **Databricks SQL Warehouse** for performant analytical querying.

### 📊 Visualizations

Two complimentary visualizations were created to provide comprehensive analytical insight:

| Visualization | Type | Purpose | Screenshot |
| :--- | :--- | :--- | :--- |
| **Release Trend** | Stacked Bar Chart | Shows the **total volume** of releases over time, highlighting the contribution breakdown (Movie vs. Series) in each year. |  |
| **Media Type Comparison** | Line Chart | Shows the **individual trajectory** of Movies, Series, and Games over time, allowing for easy identification of independent peaks and valleys in each category. |  |

---

## ⚙️ Setup and Execution

1.  **API Key:** Store your OMDb API key in Databricks Secrets (e.g., `dbutils.secrets.get(scope="omdb_scope", key="api_key")`).
2.  **Import:** Import the three files from the `notebooks/` folder into your Databricks Workspace.
3.  **Run Order:** Execute the notebooks sequentially: `01_Bronze` -> `02_Silver` -> `03_Gold`.
4.  **Validate:** Switch to the Databricks SQL Persona and run the query `SELECT * FROM main.default.omdb_releases_gold` to validate the final datamart.
