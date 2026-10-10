# Lakehouse Local Setup

A local, Docker-based playground for experimenting with the three main open table formats — **Delta Lake**, **Apache Hudi**, and **Apache Iceberg** — on top of **Apache Spark** and **MinIO** (S3-compatible storage). It uses a small sample `data.parquet` file (a staff roster: name, country, residence, role, date) to demonstrate core lakehouse features: table creation, time travel, schema evolution, change data capture, and partition management.

## 🏛️ Architecture

```
                ┌─────────────────────────────┐
                │   Jupyter (PySpark 3.5.3)    │
                │  notebooks: delta / hudi /   │
                │  iceberg                     │
                └───────────────┬──────────────┘
                                │ S3A
                ┌───────────────▼──────────────┐
                │           MinIO               │
                │  buckets: raw-data, bronze    │
                └────────────────────────────────┘
                                ▲
                ┌───────────────┴──────────────┐
                │   aws-cli (one-shot seeder)   │
                │  creates buckets & uploads     │
                │  raw_data/*.parquet            │
                └────────────────────────────────┘
```

All services run via Docker Compose. On startup, the `aws-cli` container creates the `raw-data` and `bronze` buckets in MinIO and uploads the local Parquet files from `raw_data/` into `raw-data`. Spark (via Jupyter) then reads from `raw-data` and writes lakehouse tables into `bronze`, using whichever table format the notebook targets (Delta, Hudi, or Iceberg).

## ⚙️ Stack

- **Apache Spark 3.5.3** (PySpark, via `quay.io/jupyter/pyspark-notebook`)
- **MinIO** — S3-compatible object storage
- **Delta Lake** (`delta-spark` 3.2.0)
- **Apache Hudi** (`hudi-spark3.5-bundle` 1.0.2)
- **Apache Iceberg** (`iceberg-spark-runtime-3.5` 1.9.2)
- **Docker Compose**
- **Jupyter Notebook / JupyterLab**

## 📝 Prerequisites

- Git
- Docker (with Docker Compose)
- ~4 GB of free RAM for Spark (some Hudi operations are configured to use up to 4 GB driver/executor memory)

## 📂 Project Structure

```
setup_lakehouse/
├── docker/
│   ├── docker-compose.yaml   # MinIO, aws-cli seeder, Jupyter/PySpark
│   ├── .env                  # MinIO credentials (not versioned)
│   └── notebooks/.env_sample # sample env file
├── notebooks/
│   ├── delta.ipynb           # Delta Lake walkthrough
│   ├── hudi.ipynb            # Apache Hudi walkthrough
│   └── iceberg.ipynb         # Apache Iceberg walkthrough
└── raw_data/                 # source Parquet files (data.parquet, not versioned)
```

# Setup

## Clone the repository

```
git clone https://github.com/wlcamargo/setup_lakehouse
cd setup_lakehouse
```

## Configure environment variables

Copy the sample env file and set your own MinIO credentials:

```
cp docker/notebooks/.env_sample docker/.env
```

Edit `docker/.env` and set `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`.

## Add the source data

Place a `data.parquet` file in `raw_data/`. The notebooks expect it at:

- `raw_data/data.parquet`

The notebooks assume a small roster-style schema — columns `name`, `country`, `residence`, `role`, `date` — but any Parquet file works as long as you adjust the column names used in the notebooks. You can convert a CSV into Parquet with pandas:

```python
import pandas as pd

df = pd.read_csv("raw_data/data.csv", sep=";")
df.to_parquet("raw_data/data.parquet", index=False)
```

## Start the environment

Enter the docker folder and bring the stack up:

```
cd docker
docker compose up -d
```

This starts, in order:

1. **minio** — object storage, exposed on `localhost:9000` (API) and `localhost:9001` (console)
2. **aws** — a one-shot container that creates the `raw-data` and `bronze` buckets and syncs `raw_data/` into `raw-data`
3. **jupyter** — PySpark notebook server, exposed on `localhost:8888` (JupyterLab) and `localhost:4040` (Spark UI)

## Access the services

- **JupyterLab**: [http://localhost:8888](http://localhost:8888) — token is disabled, notebooks live under `work/`
- **MinIO Console**: [http://localhost:9001](http://localhost:9001) — sign in with the credentials from `docker/.env`
- **Spark UI**: [http://localhost:4040](http://localhost:4040) — available while a Spark job/session is running

## Notebooks

Each notebook is self-contained: it creates its own SparkSession, loads `data.parquet` into a `people` database/namespace, and walks through the format's key capabilities.

| Notebook | Format | Covers |
|----------|--------|--------|
| `delta.ipynb` | Delta Lake | Table creation (`CREATE TABLE ... USING delta`), `DESCRIBE DETAIL`/`HISTORY`, time travel (`VERSION AS OF`), `RESTORE TABLE`, Change Data Feed (CDF), schema evolution (`ADD COLUMNS`, `mergeSchema`, column mapping) |
| `hudi.ipynb` | Apache Hudi | Copy-on-Write table creation, Hudi metadata columns (`_hoodie_*`), `SHOW PARTITIONS`, commit history (`show_commits`), time travel (`as.of.instant`), schema evolution on write, partition limitations |
| `iceberg.ipynb` | Apache Iceberg | Table creation via `HadoopCatalog`, snapshot history (`table.history`), data files (`table.files`), time travel and rollback (`rollback_to_snapshot`), schema evolution (`accept-any-schema`), partition evolution and file compaction (`rewrite_data_files`) |

## 📚 References

- [Delta Lake documentation](https://docs.delta.io/)
- [Apache Hudi documentation](https://hudi.apache.org/docs/overview)
- [Apache Iceberg documentation](https://iceberg.apache.org/docs/latest/)
- [MinIO documentation](https://min.io/docs/minio/linux/index.html)

## 🧑🏼‍🚀 Developer

| Developer | LinkedIn | Email | Portfolio |
|-----------|----------|-------|-----------|
| Wallace Camargo | [LinkedIn](https://www.linkedin.com/in/wallace-camargo-35b615171/) | wallacecpdg@gmail.com | [Portfolio](https://wlcamargo.github.io/) |
