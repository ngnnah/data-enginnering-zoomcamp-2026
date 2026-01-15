# 🚀 Learning in Public: Data Engineering Zoomcamp - Day 1

**Date:** January 15, 2026  
**Module:** Docker & SQL Fundamentals  
**Environment:** GitHub Codespaces

## 📚 What I Learned Today

Completed Module 1 of the [Data Engineering Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/01-docker-terraform) covering Docker containerization and PostgreSQL integration.

### 🎥 Video Resources
- [Docker & SQL Part 1](https://www.youtube.com/watch?v=QEcps_iskgg&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=11)
- [Docker & SQL Part 2](https://www.youtube.com/watch?v=lP8xXebHmuE&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=12)

## 🔑 Key Concepts Mastered

### 1. Docker Fundamentals
- **Stateless Containers**: Containers don't persist data by default - each run starts fresh
- **Volumes**: Bridge between host machine and container for data persistence
  ```bash
  docker run -it --rm -v $(pwd)/test:/app/test python:3.13.11-slim
  ```
- **Container Management**: ps, rm, network commands for orchestration

### 2. Modern Python Environment with UV
Discovered `uv` (written in Rust) - blazingly fast Python package manager:
```bash
pip install uv
uv init --python 3.13
uv add pandas pyarrow
uv run python pipeline.py
```

### 3. Building Production Docker Images
Created multi-stage Dockerfile with:
- UV package manager integration
- Virtual environment setup
- Automated entry points
- Efficient caching strategies

```dockerfile
FROM python:3.13.11-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/
WORKDIR /code
ENV PATH="/code/.venv/bin:$PATH"
COPY pyproject.toml .python-version uv.lock ./
RUN uv sync --locked
COPY pipeline.py .
ENTRYPOINT ["python", "pipeline.py"]
```

### 4. PostgreSQL in Docker
- Launched Postgres 16 container with persistent volumes
- Connected via `pgcli` CLI tool
- Data persists in Docker-managed volumes even after container removal

```bash
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16
```

### 5. Docker Networking
**Key Insight**: Named containers on the same network can communicate directly!
- Created custom bridge network: `docker network create pg-network`
- Connected PostgreSQL + pgAdmin + data ingestion service
- Containers reference each other by container name (no IP needed)

### 6. Data Ingestion Pipeline
Built a containerized ETL pipeline:
- Downloads NYC Taxi data
- Ingests into PostgreSQL with chunked processing
- Uses `click` for CLI argument parsing
- Dockerized for reproducibility

### 7. Docker Compose Orchestration
**Major Win**: Combined 3 separate `docker run` commands into single `docker-compose.yml`

Services orchestrated:
- **pgdatabase**: PostgreSQL 16 with health checks
- **pgadmin**: Web UI on port 8085
- **taxi_ingest**: Data ingestion with dependency management

```bash
docker-compose up -d
docker-compose run --rm taxi_ingest \
  --pg-user=root \
  --pg-host=pgdatabase \
  --target-table=yellow_taxi_trips_2021_02 \
  --year=2021 --month=2
```

## 💡 Key Takeaways

1. **Containers are stateless by design** - use volumes for persistence
2. **UV is a game-changer** for Python dependency management (much faster than pip)
3. **Docker networks enable seamless container communication** without hardcoded IPs
4. **Docker Compose simplifies multi-container orchestration** - define once, run anywhere
5. **Health checks in compose files** ensure services start in correct order

## 🛠️ Project Structure
```
pipeline/
├── Dockerfile              # Containerized Python pipeline
├── docker-compose.yml      # Multi-service orchestration
├── ingest_data.py         # Data ingestion script with CLI
├── pyproject.toml         # UV dependency management
└── README.md
```

## 📊 Real-World Application
Built a production-ready data engineering pipeline that:
- ✅ Runs consistently across any environment (dev/prod)
- ✅ Handles large datasets with chunked processing
- ✅ Provides web UI for database management
- ✅ Easily configurable via environment variables
- ✅ Reproducible with single command

## 🎯 Next Steps
- Explore Terraform for infrastructure as code
- Dive into data warehousing concepts
- Build more complex ETL pipelines

---

**Learning Philosophy**: Build > Break > Understand > Iterate

#DataEngineering #Docker #Python #LearningInPublic #100DaysOfCode
