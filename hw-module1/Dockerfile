FROM python:3.13.11-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

# RUN pip install pandas pyarrow

WORKDIR /code
# add this to skip "uv", "run" in the ENTRYPOINT
ENV PATH="/code/.venv/bin:$PATH"
COPY pyproject.toml .python-version uv.lock ./
RUN uv sync --locked


# COPY from host system to Docker image
COPY ingest_data.py .

# setting ENTRYPOINT for automated container execution
ENTRYPOINT ["python", "ingest_data.py"]