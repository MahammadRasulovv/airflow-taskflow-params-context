# Airflow TaskFlow API — Params & Context

A hands-on Apache Airflow 3 project demonstrating the TaskFlow API, runtime Params, task context, Jinja templates, macros, Variables, and task dependencies. The local development stack runs with Docker Compose and includes PostgreSQL and MinIO.

## Features

- DAG and Python tasks defined with `@dag` and `@task`
- Country selection with a validated Airflow Param (`AZ`, `TR`, or `GE`; default `AZ`)
- Access to execution context (`ds`) and runtime parameters
- Simple ETL flow: extract sales data, increase sales by 10%, and print the result
- `BashOperator` with Jinja templates for dates and country
- Date calculations using `macros.ds_add`
- Reading the `company_name` Airflow Variable in a Jinja template
- Task dependencies and execution monitoring in the Airflow UI

## DAG workflow

The original six-task workflow is:

```text
extract → transform → load → show_dates → show_macros → show_company_name
```

| Task | Purpose |
| --- | --- |
| `extract` | Read `ds` and `params.country`; return `{"sales": 1000}` |
| `transform` | Increase sales by 10% (result: `1100.0`) |
| `load` | Print the transformed result |
| `show_dates` | Display `ds`, previous date, and country using Jinja |
| `show_macros` | Display date information using the Python task context |
| `show_company_name` | Display the `company_name` Airflow Variable |

> If your current DAG file uses different task names, check the Airflow Graph view for the exact workflow. Previous runs may show older task versions.

## Tech stack

- Apache Airflow 3 (custom Docker image)
- Python
- Docker Compose
- PostgreSQL 16 (Airflow metadata database)
- MinIO (included as a local object-storage service)

## Getting started

### 1. Prerequisites

Install Docker Desktop with Docker Compose and Git.

### 2. Clone the repository

```bash
git clone https://github.com/MahammadRasulovv/airflow-taskflow-params-context.git
cd airflow-taskflow-params-context
```

### 3. Configure environment variables

Copy the example file to `.env`:

**Windows CMD:**

```cmd
copy .env.example .env
```

**macOS / Linux:**

```bash
cp .env.example .env
```

Edit `.env` to provide your own passwords and secrets. Ensure the PostgreSQL credentials in `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN` match `POSTGRES_USER` and `POSTGRES_PASSWORD`.

> Never commit `.env` or real credentials to a public repository.

### 4. Build and initialize Airflow

```bash
docker compose build
docker compose up airflow-init
```

Wait until the initialization container finishes successfully, then start the services:

```bash
docker compose up -d
```

### 5. Open the services

| Service | URL |
| --- | --- |
| Airflow UI | http://localhost:8080 |
| MinIO Console | http://localhost:9001 |
| MinIO API | http://localhost:9000 |

The example Compose initialization command creates a local Airflow administrator account (`admin` / `admin`). **Change these demo credentials before using the environment outside a local lab.**

### 6. Configure the Airflow Variable

In the Airflow UI, create an Airflow Variable:

- **Key:** `company_name`
- **Value:** `DemoCompany` (or any company name you prefer)

This is required for the `show_company_name` task to succeed.

### 7. Run the DAG

1. Open the Airflow UI at http://localhost:8080.
2. Find `country_param_dag`.
3. Trigger the DAG manually. Optionally choose a country (`AZ`, `TR`, or `GE`).
4. Open the Graph view to inspect task dependencies.
5. Select individual tasks and open **Logs** to review their output. Check XCom for task return values.

## Expected results

With the default `country=AZ`:

```text
extract: sales = 1000
transform: sales = 1100.0
load: Final data: {'sales': 1100.0}
show_dates: date, previous date, and country = AZ
show_macros: current and previous dates
show_company_name: Company: DemoCompany
```

Actual dates depend on the DAG run's logical date.

## Useful commands

```bash
# View running services
docker compose ps

# List DAG import errors
docker compose exec airflow-scheduler airflow dags list-import-errors

# Follow scheduler logs
docker compose logs -f airflow-scheduler

# Stop containers
docker compose down
```

To preserve PostgreSQL and MinIO data, use `docker compose down` **without** `-v`.

## Notes

- The DAG file should be placed under `airflow/dags/`, which is mounted to `/opt/airflow/dags` inside the Airflow containers.
- This is a learning project; the sales figures are sample data.
- MinIO is part of the Docker Compose stack, but the example sales DAG does not currently store objects in MinIO.

## Author

[MahammadRasulovv](https://github.com/MahammadRasulovv)
